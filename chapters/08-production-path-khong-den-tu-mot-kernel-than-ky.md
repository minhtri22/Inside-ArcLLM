# Chương 8 — Từ “chạy được” tới một đường chạy thực tế

> **Mức đọc: Đi sâu**
>
> **Bạn đang mở phần nào của cỗ máy?**
>
> ```text
> Từng phép tính đúng
>         ↓
> Nhiều lớp chạy đúng
>         ↓
> Sinh nhiều token
>         ↓
> [ đo → tìm nút thắt → tối ưu ]
> ```


> **Câu hỏi của chương:** Khi hệ thực thi đã tính đúng, làm thế nào biến nó thành một đường chạy thực tế hơn mà không tối ưu theo cảm tính?

Ở cuối Chương 7, ArcLLM đã làm được một việc rất quan trọng.

Mô hình không chỉ chạy một lượt rồi dừng.

Nó đã có:

```text
xử lý đầu vào
   ↓
KV cache trên GPU
   ↓
sinh token
   ↓
token mới
   ↓
tiếp tục dùng trạng thái cũ
```

CPU và GPU cho cùng hai token chọn token có điểm cao nhất:

```text
[6228, 17]
```

điểm dự đoán đúng trong gate.

bộ nhớ đệm K/V đúng trong gate.

Không có `intermediate host round-trip` đối với bộ nhớ đệm KV.

P6 ĐẠT (PASS).

Nếu chỉ nhìn vào tính đúng — **tính đúng** — đây đã là một hệ thực thi tối thiểu khá hoàn chỉnh.

Nhưng nếu thử dùng con đường đó cho một lượng công việc lớn hơn, một câu hỏi mới xuất hiện ngay:

> **Nó có chạy đủ thực tế không?**

Đó là P7.

Và P7 sẽ dạy chúng ta một bài học quan trọng:

> **hiệu năng hiếm khi được giải quyết bằng cách đoán ra một “chương trình GPU thần kỳ”.**

Con đường thực tế hơn thường là:

```text
đo
↓
xác định nơi tốn thời gian
↓
chọn đúng một giả thuyết
↓
thử
↓
PASS hoặc FAIL
↓
đo lại
```

Rồi lặp.

## Q4_K_M không có nghĩa mọi khối số đều là Q4_K

P7 được gọi là bước xây **đường chạy thực tế Q4_K_M (production path)** — đường thực thi gần với cách hệ thống thật sẽ sử dụng mô hình hơn.

Ta cần làm rõ tên này trước.

Trong mô hình mà ArcLLM đang dùng, Chương 2 đã cho thấy có:

```text
F32
Q4_K
Q6_K
```

trong cùng GGUF.

Vì vậy chữ `Q4_K_M` trong tên mô hình **không có nghĩa 338 khối số đều là Q4_K**.

Đường P7 vẫn phải xử lý trực tiếp cả trọng số Q4_K và Q6_K đóng gói.

Điều quan trọng đối với câu chuyện của chúng ta là:

> **hệ thực thi không bung toàn bộ mô hình về F32 trước khi tính.**

Các trọng số — **trọng số** — tiếp tục ở dạng packed — **đóng gói** — như những chương trước đã xây dựng.

## Từ bài thử nhỏ sang chuỗi 512 token

P6 cố ý rất nhỏ.

Đầu vào chỉ có bốn token.

ngữ cảnh được giới hạn để kiểm tra tính đúng của bộ nhớ đệm KV.

P7-A làm điều ngược lại: bắt đầu đưa bài đo về gần một đường sử dụng thực tế hơn.

Hai shorthand xuất hiện:

```text
pp512
tg128
```

Trong phạm vi P7:

- **pp512 — xử lý 512 token đầu vào**, tức bài đo phần văn bản ban đầu dài hơn;
- **tg128 — token generation 128 token**, tức vòng giai đoạn sinh token sinh tiếp 128 token.

Có thể hình dung:

```text
512 token đầu vào
        ↓
      xử lý đầu vào
        ↓
   KV cache lớn dần
        ↓
token 513
token 514
...
token 640
```

Tổng ngữ cảnh đi tới 640 vị trí.

Đây đã khác rất xa bài thử bốn token của P6.

## Một số thứ buộc phải thay đổi khi mở rộng quy mô

Một cơ chế chạy được ở 4 hoặc 16 vị trí chưa chắc chạy được ở 512.

P7-A vì vậy phải thay một số phần đã đủ cho **bằng chứng ban đầu** nhưng chưa đủ cho bài đo lớn hơn.

cơ chế chú ý giai đoạn xử lý đầu vào chuyển sang **cách tính softmax theo luồng (online softmax)** — không phụ thuộc vào một mảng cố định chỉ chứa được số lượng token nhỏ.

Các phép nhân trọng số Q4_K/Q6_K cho batch 512 được tổ chức thành **2-D packed GEMM — phép nhân ma trận GPU chia công việc theo hai chiều trong khi vẫn đọc trọng số đóng gói**.

Ngoài ra, những đối tượng Vulkan tốn công chuẩn bị như pipeline và descriptor set được chuẩn bị rồi giữ lại để tái sử dụng qua các bước giai đoạn sinh token.

Nhắc ngắn:

- **pipeline**: cấu hình đã chuẩn bị để GPU biết shader nào và cách nào sẽ được thực thi;
- **descriptor set**: tập thông tin giúp shader biết vùng nhớ nào chứa dữ liệu nó cần.

P7 không muốn mỗi token mới lại dựng lại toàn bộ những thứ này từ đầu.

338 khối số vẫn resident.

bộ nhớ đệm KV vẫn resident.

tính đúng vẫn phải giữ.

Nhưng bài đo lớn hơn rất nhiều.

P7-A chạy.

Và kết quả đầu tiên khá sốc.

## Đúng — nhưng cực chậm

P7-A vẫn vượt qua các regression gate — **các cổng kiểm tra để chắc rằng những chương trình GPU vừa thay đổi không làm sai kết quả**.

Tức tính đúng không bị phá.

Nhưng hiệu năng của `pp512` là khoảng:

```text
7,10 token/giây
```

Trong khi một reference — **mốc tham khảo bên ngoài đã được khóa trước** — từ R8-VK là khoảng:

```text
677,87 token/giây
```

Với `tg128`:

```text
ArcLLM P7-A
≈ 3,38 token/giây

R8-VK reference
≈ 40,35 token/giây
```

Cần đọc thật cẩn thận.

Những con số R8-VK ở đây **không phải ngưỡng ĐẠT (PASS threshold)** — không phải ngưỡng mà P7-A buộc phải vượt qua.

Vì vậy ta không được viết:

> “P7-A KHÔNG ĐẠT (FAIL) vì chậm hơn reference.”

Scientific kết luận của P7-A là:

> **Phép đo ĐẠT (measurement PASS)** — phép đo chạy hợp lệ, tính đúng vẫn giữ, và bằng chứng cho thấy đường chạy thực tế hiện tại còn một khoảng cách hiệu năng rất lớn.

Đây là một khác biệt quan trọng.

ĐẠT (PASS) của phép đo không có nghĩa hiệu năng tốt.

Nó chỉ có nghĩa:

> **Ta đã đo được một vấn đề thật.**

## Vậy chậm ở CPU hay GPU?

Đến đây rất dễ đoán:

> “Có lẽ CPU gọi Vulkan quá nhiều.”

Hoặc:

> “Có lẽ overhead của hệ thực thi quá lớn.”

Nhưng P7 không tối ưu từ suy đoán đó.

Nó đo.

Ở `pp512`, thời gian submit/wait — **khoảng thời gian gửi công việc rồi chờ GPU hoàn thành** — gần bằng toàn bộ wall time — **thời gian thực tế từ đầu tới cuối**:

```text
wall
≈ 72.123,8 ms

submit/wait
≈ 72.122,8 ms
```

Chênh lệch rất nhỏ.

Với `tg128`, phần lớn thời gian cũng nằm trong vùng GPU đang thực thi.

Điều này không nói chính xác chương trình GPU nào chậm.

Nhưng nó loại được một giả thuyết lớn:

> **Điều phối phía CPU (host orchestration)** không phải nút thắt chính đầu tiên cần xử lý.

Evidence chỉ về **thực thi phía thiết bị (device-side execution)** — phần tính toán phía GPU.

Từ đây P7-B mở đo đạc hiệu năng — **đo thời gian bên trong từng nhóm công việc GPU**.

## Không tối ưu cả khu rừng — tìm cây lớn nhất trước

P7-B dùng **truy vấn dấu thời gian Vulkan (Vulkan timestamp queries)** — dấu thời gian do GPU ghi lại quanh các lần giao việc cho GPU.

Ý tưởng rất đơn giản:

```text
lần giao việc A
→ mất bao lâu?

lần giao việc B
→ mất bao lâu?

lần giao việc C
→ mất bao lâu?
```

Sau đó các lần giao việc cho GPU có quan hệ về cấu trúc hoặc cùng cơ chế được gom thành một **họ tác vụ tính toán (compute/kernel family)**.

Ví dụ:

```text
attention

Q/K/V projections

FFN gate/up

FFN down

LM head

...
```

Từ **họ tác vụ (family)** ở đây không có nghĩa gom tùy ý nhiều phép tính không liên quan. Nó chỉ một tập tác vụ có quan hệ vì cùng vai trò, cùng phép tính nền tảng hoặc cùng cơ chế thực thi, nên một thay đổi kiến trúc có thể tác động lên cả họ.

Lần đầu tiên ArcLLM không chỉ biết:

> “GPU chậm.”

Nó bắt đầu biết:

> **“Họ tác vụ nào trên GPU đang ăn phần thời gian lớn nhất?”**

Evidence chỉ vào các phép GEMM đóng gói trong FFN.

**FFN — Feed-Forward Network —** là nhánh biến đổi tín hiệu mà ta đã gặp từ Chương 4.

P7-C vì vậy chỉ đụng vào **một họ tác vụ có quan hệ chặt với nhau — các phép GEMM đóng gói trong FFN — thay vì sửa nhiều phần không liên quan cùng lúc**.

Không sửa cơ chế chú ý cùng lúc.

Không sửa lớp tạo điểm đầu ra (LM head).

Không sửa giai đoạn sinh token.

## Một thay đổi đầu tiên tạo khác biệt lớn

P7-C thử **chia khối (tiling)** — chia phép nhân ma trận thành các khối nhỏ để GPU tái sử dụng dữ liệu hiệu quả hơn cho FFN Q4_K và Q6_K.

Các regression số học vẫn ĐẠT (PASS).

điểm dự đoán vẫn đúng.

Top1 vẫn đúng.

Và trong A/B cùng run, thời gian `pp512` thay đổi mạnh:

```text
mốc đối chứng
≈ 61,8 giây

optimized
≈ 11,9 giây
```

Tương ứng khoảng:

```text
8,28 token/giây
→
42,92 token/giây
```

mức tăng tốc:

```text
≈ 5,18×
```

Gate đã khóa cho option này là:

```text
>= 1,10×
```

Nên P7-C ĐẠT (PASS).

Đây là một cải thiện lớn.

Nhưng ArcLLM không lập tức kết luận:

> “Tiling là đáp án cuối.”

Nó quay lại đo.

## nút thắt di chuyển

Sau khi FFN được cải thiện, P7-D profile lại graph.

Lần này phần lớn thời gian chuyển sang các **các phép chiếu của cơ chế chú ý (attention projections)** — những phép nhân tạo và biến đổi Q/K/V/O.

Chỉ riêng nhóm này chiếm khoảng 59% chain time trong profile đó.

P7-E vì vậy tái sử dụng phép tính nền tảng tiled GEMM đã được chứng minh, nhưng chỉ áp dụng nó vào Q/K/V/O projections.

A/B:

```text
40,44 token/giây
→
74,77 token/giây
```

mức tăng tốc:

```text
≈ 1,85×
```

Gate:

```text
>= 1,10×
```

ĐẠT (PASS).

Rồi lại đo.

Sau đó P7-G thử tăng mức tái sử dụng theo token với **khối 16 token (token tile16)** — xử lý 16 token trong cùng cấu trúc chương trình GPU.

Kết quả:

```text
≈ 1,225×
```

ĐẠT (PASS).

Ta bắt đầu thấy một pattern:

```text
đo
↓
đánh đúng nút thắt
↓
PASS
↓
đo lại

không phải

đo một lần
↓
tối ưu mọi thứ
```

Bởi mỗi lần một nút thắt được giảm, nút thắt tiếp theo có thể đổi.

## Đây là lúc lớp quản trị nghiên cứu phải xuất hiện

Ở Chương 5, `lineage.md` mới chỉ xuất hiện như một cuốn sổ lịch sử.

Đến P7, chỉ ghi lịch sử thôi chưa đủ.

Một AI có thể nghĩ ra hàng chục ý tưởng tối ưu rất nhanh:

```text
tăng khối xử lý?

gộp chương trình GPU?

đổi layout?

vec4?

K64?

K128?

thêm nhóm con GPU để nhiều làn tính toán phối hợp?

...
```

Nếu cứ implementation → run → sửa → run cho tới khi số đẹp lên, ta rất dễ biến nghiên cứu thành tuning không có điểm dừng.

Đây là đúng thời điểm cuốn sách cần giới thiệu bốn **các chế độ nghiên cứu (research modes)** mà chúng ta sẽ dùng về sau.

Lưu ý quan trọng:

> **Các tên E/M/C/T không được gắn ngược vào lịch sử P7 như thể P7 khi đó đã được thực thi chính thức dưới các nhãn này.**

Ta đang dùng chúng như một cách đọc và quản trị những nghiên cứu tương tự từ đây về sau.

### E — Khám phá (Explore)

**E — Khám phá (Explore)** dùng khi câu hỏi còn là:

> “Vấn đề nằm ở đâu?”

> “Mechanism nào đáng thử?”

> “Signal này có đáng theo tiếp không?”

Ở Mode E có thể dùng evidence đã tiêu — **spent evidence**, profile cũ, decomposition, ablation nhỏ hoặc phép tính nhanh.

Mục đích không phải tạo **kết luận cuối cùng**.

Mục đích là **loại nhanh những hướng không đáng tiêu thêm evidence mới**.

Các timestamp profile P7-B, P7-D, P7-H, P7-M là hình ảnh rất dễ hiểu cho tinh thần này:

```text
đừng đoán nút thắt
→ nhìn evidence trước
```

### M — Kiểm tra cơ chế (Mechanism qualification)

Sau khi bước E — Khám phá chỉ ra một **phương án thử**, bước M hỏi: **cơ chế này có đủ cơ sở để đáng mở phép xác nhận hay không?**

Chỉ chọn **một mechanism**.

Ví dụ:

> “Nếu tăng token khối xử lý từ 16 lên 32, việc tái sử dụng dữ liệu có đủ leverage để tạo mức tăng tốc có ý nghĩa không?”

Hoặc:

> “Nếu gate và up dùng chung **dữ liệu trung gian — dữ liệu trung gian đang chảy qua mô hình —** theo cùng một **khối xử lý — khối dữ liệu nhỏ được xử lý cùng nhau —** trong một fused chương trình GPU, có giảm đủ công việc không?”

Không mở năm ý tưởng cùng lúc.

Câu hỏi phải đủ hẹp để kết quả có thể giết hoặc giữ chính mechanism đó.

### C — Xác nhận (Confirm)

Nếu mechanism sống sót, mới đi sang **C — Xác nhận (Confirm)** — phép xác nhận ĐẠT/KHÔNG ĐẠT đã khóa trước.

Ví dụ P7 thường dùng:

```text
A = mốc đối chứng hiện tại
B = phương án thử mới

3 trial A/B xen kẽ
tính đúng phải giữ
ngưỡng tăng tốc >= 1,10×
```

Quan trọng nhất:

> **Không hạ gate sau khi nhìn outcome.**

KHÔNG ĐẠT (FAIL) là KHÔNG ĐẠT (FAIL).

### T — Kiểm tra khi đưa lên toàn hệ (Transfer / carry-through)

Chỉ khi Mode C ĐẠT (PASS) mới có lý do đưa mechanism sang đường lớn hơn.

Đó là **T — Kiểm tra khi đưa lên toàn hệ (Transfer / carry-through)** — mang bằng chứng đã qua xác nhận vào hệ thống lớn hơn để xem giá trị còn giữ được hay không.

Có thể hình dung:

```text
E
tìm nơi đáng đào
 ↓
M
giữ đúng một mechanism
 ↓
C
fresh confirmation
 ↓
PASS?
 ├─ không → quay lại E
 └─ có
      ↓
      T
      carry-through
```

Một nguyên tắc đặc biệt quan trọng là:

> **Bằng chứng mới (fresh evidence) là vốn nghiên cứu (research capital).**

Không nên đốt một fresh run chỉ để hỏi một câu mà evidence cũ đã đủ sức giết.

## Và rồi các giả thuyết bắt đầu chết

Sau P7-G, profile lại cho thấy FFN vẫn là vùng lớn.

Một ý tưởng tự nhiên là:

> Nếu tile16 tốt, tile32 có tốt hơn không?

P7-I thử đúng câu đó.

tính đúng ĐẠT (PASS).

Nhưng hiệu năng:

```text
mức tăng tốc
≈ 1,0034×
```

Gate:

```text
>= 1,10×
```

KHÔNG ĐẠT (FAIL).

Không thử tile64 chỉ vì tile32 chưa thắng.

P7-I được giữ như một negative result — **kết quả âm tính**.

Tiếp theo P7-J thử một loader vec4 để đọc/dequant nhiều trọng số liên tiếp hiệu quả hơn.

tính đúng vẫn ĐẠT (PASS).

hiệu năng:

```text
≈ 0,9822×
```

Không những không đạt 1,10×, nó còn chậm hơn mốc đối chứng.

KHÔNG ĐẠT (FAIL).

P7-K thử tăng output-row khối xử lý từ 8 lên 16.

Kết quả:

```text
≈ 1,0684×
```

Đây là một ví dụ rất hay.

1,0684× nghĩa là phương án thử **có nhanh hơn**.

Nhưng gate đã khóa là:

```text
1,10×
```

Vậy kết luận vẫn là:

> **KHÔNG ĐẠT (FAIL).**

Trong nghiên cứu, “có cải thiện” và “**vượt tiêu chuẩn đã khóa**” là hai câu khác nhau.

Nếu một công việc mất 100 giây, mức tăng tốc 1,068× tương ứng còn khoảng:

```text
100 / 1,068
≈ 93,6 giây
```

Có cải thiện.

Nhưng chưa đủ mức mà experiment đã định nghĩa là có ý nghĩa để promote.

Không được thấy 1,068 rồi sửa gate từ 1,10 xuống 1,05.

Nếu làm vậy, gate chỉ còn là cách hợp thức hóa outcome.

## P7-L: gộp phép tính thực sự vượt ngưỡng đã khóa

Sau những negative đó, P7-L thử một mechanism khác.

Trong FFN có hai phép nhân gần nhau:

```text
gate
up
```

mốc đối chứng dùng:

```text
56 lần giao việc riêng
```

cho 28 lớp.

P7-L thử **gộp phép tính (fusion)** — gộp hai công việc liên quan vào một chương trình GPU để gate và up có thể dùng chung một phần dữ liệu đầu vào và lịch lần giao việc cho GPU.

Kết quả:

```text
56 lần giao việc gate/up
→ 28 lần giao việc đã gộp
```

tính đúng ĐẠT (PASS).

Full điểm dự đoán và top1 giữ nguyên trong các trial.

mức tăng tốc A/B:

```text
≈ 1,2429×
```

Gate:

```text
>= 1,10×
```

ĐẠT (PASS).

P7-L được **đóng băng làm phương án cho đường chạy thực tế**.

Nhưng ngay cả lúc đó, P7 vẫn chưa đóng.

Bước tiếp theo vẫn là:

> **đo lại phương án đang tốt nhất.**

## Phương án đang tốt nhất cũng phải bị soi lại

P7-M đo chính graph P7-L đã thắng.

giai đoạn xử lý đầu vào lúc này có:

```text
441 lần giao việc
```

Một cached giai đoạn sinh token step có:

```text
469 lần giao việc
```

Trong giai đoạn xử lý đầu vào, tỷ trọng thời gian GPU xấp xỉ:

```text
fused gate/up       44,40%
FFN down             30,42%
attention              9,34%
attn Q/K/V             7,59%
attn output             5,86%
```

Phần điểm đồng bộ/unattributed — **thời gian không quy được rõ vào các chương trình GPU chính hoặc dùng cho đồng bộ** — chỉ khoảng:

```text
0,024%
```

Một thông điệp rất rõ xuất hiện:

> **Không còn cơ sở để đổ lỗi chính cho orchestration hay điểm đồng bộ.**

Phần lớn chi phí vẫn nằm trong những phép tính thật.

## Một kết quả ĐẠT không có nghĩa mọi cách gộp phép tính tiếp theo đều tốt

P7-L gộp phép tính thành công.

Một phản xạ dễ mắc là:

> “gộp phép tính tốt. Fuse thêm.”

P7-N thử gộp tiếp SwiGLU vào gate+up.

tính đúng ĐẠT (PASS).

hiệu năng cũng tăng:

```text
≈ 1,0516×
```

Nhưng gate vẫn là:

```text
>= 1,10×
```

KHÔNG ĐẠT (FAIL).

Không rescue.

Không nói:

> “5% cũng khá mà.”

P7-O sau đó thử một mechanism độc lập ở FFN-down: tăng K khối xử lý từ 32 lên 64.

tính đúng ĐẠT (PASS) sau khi implementation defect được sửa.

hiệu năng:

```text
≈ 0,8837×
```

Chậm hơn rõ rệt.

KHÔNG ĐẠT (FAIL).

Và lần này quyết định không phải:

> “Thử K128 xem sao.”

Mà là:

> **Dừng P7.**

## Tại sao dừng khi vẫn còn ý tưởng?

Bởi một research program không được đánh giá bằng số ý tưởng còn có thể nghĩ ra.

Sau P7-L:

- hai nhánh gate/up đã có phương án đang tốt nhất;
- FFN-down đã được challenge thêm và K64 thất bại;
- gộp phép tính rộng hơn với SwiGLU không vượt gate;
- nhiều biến thể tiling/dequant đã thất bại;
- profile cho thấy các family còn lại nhỏ hơn những nút thắt ban đầu;
- muốn tiếp tục có nguy cơ phải thay nhiều family cùng lúc, làm mất khả năng biết chính xác thứ gì tạo ra gain.

P7 vì vậy đóng với:

> **P7-L đường chạy thực tế frozen.**

Đây là điểm hội tụ.

Không phải vì **mã** không thể tối ưu thêm.

Mà vì evidence hiện tại không còn biện minh cho việc tiếp tục kéo dài P7.

## Đường chạy thực tế cuối P7 gồm những gì?

Đường được freeze giữ:

```text
Q4_K / Q6_K packed weights
→ đọc trực tiếp, không bung toàn mô hình

whole sinh tokenr resident
→ trọng số toàn sinh tokenr giữ sẵn

GPU-resident KV cache
→ trạng thái attention giữ trên GPU

attention Q/K/V/O
→ tiled projection path từ P7-E

FFN gate + up
→ fused path từ P7-L

FFN down
→ row8 × token16 × K32 path từ P7-G

SwiGLU
→ vẫn là bước riêng

cơ chế chú ý ở giai đoạn xử lý đầu vào
→ online attention

sinh token
→ đồ thị sinh token dùng bộ nhớ đệm đã xác nhận
```

Đây là ý nghĩa của từ **đường chạy thực tế** trong P7.

Nó không có nghĩa:

> “ArcLLM đã là một sản phẩm hoàn thiện để sử dụng thật.”

Nó có nghĩa hẹp hơn:

> **Trong phạm vi mô hình và kiến trúc hiện tại, ta đã chọn được một cách ghép từ bằng chứng, giữ được tính đúng, thử các phương án quan trọng, đóng băng phương án đang tốt nhất và có thể kết thúc P7.**

## Và vẫn chưa được phép nói ArcLLM nhanh

Đây là giới hạn quan trọng nhất của Phần I.

P7 đã tạo ra những mức tăng tốc nội bộ rất lớn.

P7-C từng đạt hơn 5× so với mốc đối chứng của chính experiment đó.

P7-E đạt khoảng 1,85×.

P7-L đạt khoảng 1,24×.

Nhưng ta **không được nhân chúng lại rồi nói ArcLLM nhanh hơn hệ thực thi khác**.

Ta cũng không được lấy một thông lượng đẹp ở một run rồi so với một con số llama.cpp được chạy ở điều kiện khác.

P7 ghi nhận rằng absolute thông lượng — **tốc độ tuyệt đối** — thay đổi đáng kể giữa các run trên target machine.

Vì vậy các quyết định tối ưu dựa chủ yếu vào:

> **Đo A/B xen kẽ trong cùng lượt chạy (same-run interleaved A/B)** — mốc đối chứng và phương án thử được chạy luân phiên trong cùng phiên đo.

Lý do trực giác rất đơn giản.

Nếu hôm thứ Hai máy nóng, có process nền và trạng thái trình điều khiển khác hôm thứ Ba, so:

```text
A hôm thứ Hai
với
B hôm thứ Ba
```

rất dễ trộn hiệu năng của chương trình GPU với trạng thái của cả máy.

So A/B xen kẽ trong cùng run giúp giảm phần nhiễu đó.

Chương 9 sẽ đi sâu hơn vào cách phép đo so sánh và các thống kê như trung vị.

## Phần I kết thúc ở một ranh giới rất quan trọng

Ta đã đi từ:

```text
một tệp GGUF
↓
tensor store
↓
thiết bị Vulkan
↓
phép tính nền tảng
↓
một lớp
↓
28 lớp
↓
KV cache
↓
autoregressive generation
↓
profile
↓
nhiều PASS và FAIL
↓
đường chạy thực tế P7-L
```

Đó là một thành tựu kỹ thuật có thật.

Nhưng science chưa cho phép kết luận:

> “ArcLLM cạnh tranh được với hệ thực thi trưởng thành.”

Để trả lời điều đó, ta cần một thứ mà đến giờ cuốn sách cố tình chưa làm:

> **một phép đo so sánh có đối chứng phù hợp.**

Không phải reference để nhìn cho biết.

Không phải tốc độ của một run cũ.

Không phải cảm giác “nhanh hơn nhiều”.

Mà là:

```text
cùng mô hình
cùng quant
cùng hardware
cùng bài đo
cùng cách đo
        ↓
ArcLLM
vs
mốc đối chứng phù hợp
```

Phần II bắt đầu từ đó.

### Nhớ 3 điều

1. **hiệu năng optimization phải bắt đầu bằng measurement, không bằng danh sách ý tưởng.** P7 liên tục profile → chọn một nút thắt → thử một mechanism → đo lại.
2. **Một phương án thử có nhanh hơn vẫn có thể KHÔNG ĐẠT.** P7-K tăng khoảng 6,8% nhưng không vượt ngưỡng `1,10×`; P7-N tăng khoảng 5,2% nhưng vẫn KHÔNG ĐẠT. Ngưỡng không được sửa sau khi đã thấy kết quả.
3. **P7-L là phương án tốt nhất cho đường chạy nội bộ lúc đó, không phải bằng chứng ArcLLM thắng hệ thực thi khác.** Muốn đưa ra kết luận như vậy, cuốn sách phải chuyển sang phép đo đối chứng cùng điều kiện ở Chương 9.

**Chương 9 — Muốn biết nhanh hay chậm, phải có một mốc để so**

Từ đây, câu hỏi không còn là:

> “ArcLLM có thể tự cải thiện chính nó bao nhiêu?”

Mà là:

> **“Khi đặt cạnh một mốc đối chứng trưởng thành dưới cùng điều kiện, ArcLLM thực sự đang đứng ở đâu?”**
