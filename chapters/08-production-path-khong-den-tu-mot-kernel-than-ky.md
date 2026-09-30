# Chương 8 — Production path không đến từ một kernel thần kỳ

> **Mức đọc: Đi sâu**
>
> **Bản đồ xuyên suốt — đang mở: benchmark / tối ưu**
>
> ```text
> văn bản → token → tensor
>                     ↓
>          model / parameters
>                     ↓
>                  runtime
>                     ↓
>            CPU / GPU / bộ nhớ
>                     ↓
>      RMSNorm / attention / FFN
>                     ↓
>              decoder layer
>                     ↓
>             nhiều decoder layer
>                     ↓
>          KV cache / sinh token
>                     ↓
>        benchmark / tối ưu
>                     ↓
> representation / lifecycle / kiến trúc runtime
> ```
>
> ▶ **Đang mở ở chương này:** benchmark / tối ưu.


> **Câu hỏi của chương:** Khi runtime đã tính đúng, làm thế nào biến nó thành một đường chạy thực tế hơn mà không tối ưu theo cảm tính?

Ở cuối Chương 7, ArcLLM đã làm được một việc rất quan trọng.

Model không chỉ chạy một lượt rồi dừng.

Nó đã có:

```text
prefill
   ↓
KV cache trên GPU
   ↓
decode
   ↓
token mới
   ↓
tiếp tục dùng trạng thái cũ
```

CPU và GPU cho cùng hai token greedy:

```text
[6228, 17]
```

Logits đúng trong gate.

K/V cache đúng trong gate.

Không có `intermediate host round-trip` đối với KV cache.

P6 PASS.

Nếu chỉ nhìn vào correctness — **tính đúng** — đây đã là một runtime tối thiểu khá hoàn chỉnh.

Nhưng nếu thử dùng con đường đó cho một lượng công việc lớn hơn, một câu hỏi mới xuất hiện ngay:

> **Nó có chạy đủ thực tế không?**

Đó là P7.

Và P7 sẽ dạy chúng ta một bài học quan trọng:

> **Performance hiếm khi được giải quyết bằng cách đoán ra một “kernel thần kỳ”.**

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

## Q4_K_M không có nghĩa mọi tensor đều là Q4_K

P7 được gọi là bước xây **Q4_K_M production path — đường thực thi Q4_K_M gần với cách runtime thực tế sẽ sử dụng model hơn**.

Ta cần làm rõ tên này trước.

Trong model mà ArcLLM đang dùng, Chương 2 đã cho thấy có:

```text
F32
Q4_K
Q6_K
```

trong cùng GGUF.

Vì vậy chữ `Q4_K_M` trong tên model **không có nghĩa 338 tensor đều là Q4_K**.

Đường P7 vẫn phải xử lý trực tiếp cả trọng số Q4_K và Q6_K đóng gói.

Điều quan trọng đối với câu chuyện của chúng ta là:

> **Runtime không bung toàn bộ model về F32 trước khi tính.**

Các weight — **trọng số** — tiếp tục ở dạng packed — **đóng gói** — như những chương trước đã xây dựng.

## Từ bài thử nhỏ sang 512 token

P6 cố ý rất nhỏ.

Prompt chỉ có bốn token.

Context được giới hạn để kiểm tra correctness của KV cache.

P7-A làm điều ngược lại: bắt đầu đưa workload về gần một đường sử dụng thực tế hơn.

Hai shorthand xuất hiện:

```text
pp512
tg128
```

Trong phạm vi P7:

- **pp512 — prompt processing 512 token**, tức pha prefill xử lý 512 token đầu vào;
- **tg128 — token generation 128 token**, tức vòng decode sinh tiếp 128 token.

Có thể hình dung:

```text
512 token đầu vào
        ↓
      prefill
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

## Một số thứ buộc phải thay đổi khi scale

Một cơ chế chạy được ở 4 hoặc 16 vị trí chưa chắc chạy được ở 512.

P7-A vì vậy phải thay một số phần đã đủ cho proof nhưng chưa đủ cho workload lớn hơn.

Attention prefill chuyển sang **online softmax — cách tính softmax theo luồng để không phụ thuộc vào một mảng cố định chỉ chứa được số lượng token nhỏ**.

Các phép nhân trọng số Q4_K/Q6_K cho batch 512 được tổ chức thành **2-D packed GEMM — phép nhân ma trận GPU chia công việc theo hai chiều trong khi vẫn đọc trọng số đóng gói**.

Ngoài ra, những đối tượng Vulkan tốn công chuẩn bị như pipeline và descriptor set được chuẩn bị rồi giữ lại để tái sử dụng qua các bước decode.

Nhắc ngắn:

- **pipeline**: cấu hình đã chuẩn bị để GPU biết shader nào và cách nào sẽ được thực thi;
- **descriptor set**: tập thông tin giúp shader biết buffer nào chứa dữ liệu nó cần.

P7 không muốn mỗi token mới lại dựng lại toàn bộ những thứ này từ đầu.

338 tensor vẫn resident.

KV cache vẫn resident.

Correctness vẫn phải giữ.

Nhưng workload lớn hơn rất nhiều.

P7-A chạy.

Và kết quả đầu tiên khá sốc.

## Đúng — nhưng cực chậm

P7-A vẫn vượt qua các regression gate — **các cổng kiểm tra để chắc rằng những kernel vừa thay đổi không làm sai kết quả**.

Tức correctness không bị phá.

Nhưng performance của `pp512` là khoảng:

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

Những con số R8-VK ở đây **không phải PASS threshold — không phải ngưỡng mà P7-A buộc phải vượt qua**.

Vì vậy ta không được viết:

> “P7-A FAIL vì chậm hơn reference.”

Scientific verdict của P7-A là:

> **Measurement PASS — phép đo đã chạy hợp lệ, correctness vẫn giữ, và evidence cho thấy production path hiện tại còn một khoảng cách performance rất lớn.**

Đây là một khác biệt quan trọng.

PASS của phép đo không có nghĩa performance tốt.

Nó chỉ có nghĩa:

> **Ta đã đo được một vấn đề thật.**

## Vậy chậm ở CPU hay GPU?

Đến đây rất dễ đoán:

> “Có lẽ CPU gọi Vulkan quá nhiều.”

Hoặc:

> “Có lẽ overhead của runtime quá lớn.”

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

Điều này không nói chính xác kernel nào chậm.

Nhưng nó loại được một giả thuyết lớn:

> **Host orchestration — phần điều phối phía CPU — không phải bottleneck chính đầu tiên cần đánh.**

Evidence chỉ về **device-side execution — phần tính toán phía GPU**.

Từ đây P7-B mở profiling — **đo thời gian bên trong từng nhóm công việc GPU**.

## Không tối ưu cả khu rừng — tìm cây lớn nhất trước

P7-B dùng **Vulkan timestamp queries — dấu thời gian do GPU ghi lại quanh các dispatch**.

Ý tưởng rất đơn giản:

```text
dispatch A
→ mất bao lâu?

dispatch B
→ mất bao lâu?

dispatch C
→ mất bao lâu?
```

Sau đó các dispatch có quan hệ về cấu trúc hoặc cùng cơ chế được gom thành một **họ tác vụ tính toán (compute/kernel family)**.

Ví dụ:

```text
attention

Q/K/V projections

FFN gate/up

FFN down

LM head

...
```

Từ **family — họ** ở đây không có nghĩa gom tùy ý nhiều phép tính không liên quan. Nó chỉ một tập tác vụ có quan hệ vì cùng vai trò, cùng primitive hoặc cùng cơ chế thực thi, nên một thay đổi kiến trúc có thể tác động lên cả họ.

Lần đầu tiên ArcLLM không chỉ biết:

> “GPU chậm.”

Nó bắt đầu biết:

> **“Họ tác vụ nào trên GPU đang ăn phần thời gian lớn nhất?”**

Evidence chỉ vào các phép GEMM đóng gói trong FFN.

**FFN — Feed-Forward Network —** là nhánh biến đổi tín hiệu mà ta đã gặp từ Chương 4.

P7-C vì vậy chỉ đụng vào **một họ tác vụ có quan hệ chặt với nhau — các phép GEMM đóng gói trong FFN — thay vì sửa nhiều phần không liên quan cùng lúc**.

Không sửa attention cùng lúc.

Không sửa LM head.

Không sửa decode.

## Một thay đổi đầu tiên tạo khác biệt lớn

P7-C thử **tiling — chia phép nhân ma trận thành các khối nhỏ để GPU có thể tái sử dụng dữ liệu hiệu quả hơn** cho FFN Q4_K và Q6_K.

Các regression số học vẫn PASS.

Logits vẫn đúng.

Top1 vẫn đúng.

Và trong A/B cùng run, thời gian `pp512` thay đổi mạnh:

```text
baseline
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

Speedup:

```text
≈ 5,18×
```

Gate đã khóa cho option này là:

```text
>= 1,10×
```

Nên P7-C PASS.

Đây là một cải thiện lớn.

Nhưng ArcLLM không lập tức kết luận:

> “Tiling là đáp án cuối.”

Nó quay lại đo.

## Bottleneck di chuyển

Sau khi FFN được cải thiện, P7-D profile lại graph.

Lần này phần lớn thời gian chuyển sang các **attention projections — những phép nhân tạo và biến đổi Q/K/V/O cho attention**.

Chỉ riêng nhóm này chiếm khoảng 59% chain time trong profile đó.

P7-E vì vậy tái sử dụng primitive tiled GEMM đã được chứng minh, nhưng chỉ áp dụng nó vào Q/K/V/O projections.

A/B:

```text
40,44 token/giây
→
74,77 token/giây
```

Speedup:

```text
≈ 1,85×
```

Gate:

```text
>= 1,10×
```

PASS.

Rồi lại đo.

Sau đó P7-G thử tăng mức tái sử dụng theo token với **token tile16 — xử lý một khối 16 token trong cấu trúc kernel đó**.

Kết quả:

```text
≈ 1,225×
```

PASS.

Ta bắt đầu thấy một pattern:

```text
đo
↓
đánh đúng bottleneck
↓
PASS
↓
đo lại

không phải

đo một lần
↓
tối ưu mọi thứ
```

Bởi mỗi lần một bottleneck được giảm, bottleneck tiếp theo có thể đổi.

## Đây là lúc lớp quản trị nghiên cứu phải xuất hiện

Ở Chương 5, `lineage.md` mới chỉ xuất hiện như một cuốn sổ lịch sử.

Đến P7, chỉ ghi lịch sử thôi chưa đủ.

Một AI có thể nghĩ ra hàng chục ý tưởng tối ưu rất nhanh:

```text
tăng tile?

fuse kernel?

đổi layout?

vec4?

K64?

K128?

thêm subgroup — một nhóm nhỏ các lane GPU có thể phối hợp thực thi?

...
```

Nếu cứ implementation → run → sửa → run cho tới khi số đẹp lên, ta rất dễ biến nghiên cứu thành tuning không có điểm dừng.

Đây là đúng thời điểm cuốn sách cần giới thiệu bốn **research modes — chế độ quản trị nghiên cứu** mà chúng ta sẽ dùng về sau.

Lưu ý quan trọng:

> **Các tên E/M/C/T không được gắn ngược vào lịch sử P7 như thể P7 khi đó đã được thực thi chính thức dưới các nhãn này.**

Ta đang dùng chúng như một cách đọc và quản trị những nghiên cứu tương tự từ đây về sau.

### Mode E — Explore

**Mode E — khám phá nhanh** dùng khi câu hỏi còn là:

> “Vấn đề nằm ở đâu?”

> “Mechanism nào đáng thử?”

> “Signal này có đáng theo tiếp không?”

Ở Mode E có thể dùng evidence đã tiêu — **spent evidence**, profile cũ, decomposition, ablation nhỏ hoặc phép tính nhanh.

Mục đích không phải tạo claim cuối.

Mục đích là **loại nhanh những hướng không đáng tiêu thêm evidence mới**.

Các timestamp profile P7-B, P7-D, P7-H, P7-M là hình ảnh rất dễ hiểu cho tinh thần này:

```text
đừng đoán bottleneck
→ nhìn evidence trước
```

### Mode M — Mechanism qualification

Sau khi Mode E chỉ ra một candidate, **Mode M — kiểm tra mechanism có đủ cơ sở để đáng chạy confirmatory hay không**.

Chỉ chọn **một mechanism**.

Ví dụ:

> “Nếu tăng token tile từ 16 lên 32, việc tái sử dụng dữ liệu có đủ leverage để tạo speedup có ý nghĩa không?”

Hoặc:

> “Nếu gate và up dùng chung **activation — dữ liệu trung gian đang chảy qua model —** theo cùng một **tile — khối dữ liệu nhỏ được xử lý cùng nhau —** trong một fused kernel, có giảm đủ công việc không?”

Không mở năm ý tưởng cùng lúc.

Câu hỏi phải đủ hẹp để kết quả có thể giết hoặc giữ chính mechanism đó.

### Mode C — Confirm

Nếu mechanism sống sót, mới đi sang **Mode C — phép xác nhận PASS/FAIL đã khóa trước**.

Ví dụ P7 thường dùng:

```text
A = baseline hiện tại
B = candidate mới

3 trial A/B xen kẽ
correctness phải giữ
speedup gate >= 1,10×
```

Quan trọng nhất:

> **Không hạ gate sau khi nhìn outcome.**

FAIL là FAIL.

### Mode T — Transfer / carry-through

Chỉ khi Mode C PASS mới có lý do đưa mechanism sang đường lớn hơn.

Đó là **Mode T — mang bằng chứng đã qua xác nhận vào hệ thống tiếp theo và kiểm tra nó còn giữ được giá trị hay không**.

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

> **Fresh evidence là research capital — bằng chứng mới là vốn nghiên cứu.**

Không nên đốt một fresh run chỉ để hỏi một câu mà evidence cũ đã đủ sức giết.

## Và rồi các giả thuyết bắt đầu chết

Sau P7-G, profile lại cho thấy FFN vẫn là vùng lớn.

Một ý tưởng tự nhiên là:

> Nếu tile16 tốt, tile32 có tốt hơn không?

P7-I thử đúng câu đó.

Correctness PASS.

Nhưng performance:

```text
speedup
≈ 1,0034×
```

Gate:

```text
>= 1,10×
```

FAIL.

Không thử tile64 chỉ vì tile32 chưa thắng.

P7-I được giữ như một negative result — **kết quả âm tính**.

Tiếp theo P7-J thử một loader vec4 để đọc/dequant nhiều weight liên tiếp hiệu quả hơn.

Correctness vẫn PASS.

Performance:

```text
≈ 0,9822×
```

Không những không đạt 1,10×, nó còn chậm hơn baseline.

FAIL.

P7-K thử tăng output-row tile từ 8 lên 16.

Kết quả:

```text
≈ 1,0684×
```

Đây là một ví dụ rất hay.

1,0684× nghĩa là candidate **có nhanh hơn**.

Nhưng gate đã khóa là:

```text
1,10×
```

Vậy verdict vẫn là:

> **FAIL.**

Trong nghiên cứu, “có cải thiện” và “PASS contract” là hai câu khác nhau.

Nếu một công việc mất 100 giây, speedup 1,068× tương ứng còn khoảng:

```text
100 / 1,068
≈ 93,6 giây
```

Có cải thiện.

Nhưng chưa đủ mức mà experiment đã định nghĩa là có ý nghĩa để promote.

Không được thấy 1,068 rồi sửa gate từ 1,10 xuống 1,05.

Nếu làm vậy, gate chỉ còn là cách hợp thức hóa outcome.

## P7-L: fusion thực sự vượt gate

Sau những negative đó, P7-L thử một mechanism khác.

Trong FFN có hai phép nhân gần nhau:

```text
gate
up
```

Baseline dùng:

```text
56 dispatch riêng
```

cho 28 layer.

P7-L thử **fusion — gộp hai công việc liên quan vào một kernel** để gate và up có thể dùng chung một phần dữ liệu đầu vào và lịch dispatch.

Kết quả:

```text
56 gate/up dispatch
→ 28 fused dispatch
```

Correctness PASS.

Full logits và top1 giữ nguyên trong các trial.

Speedup A/B:

```text
≈ 1,2429×
```

Gate:

```text
>= 1,10×
```

PASS.

P7-L được freeze — **đóng băng làm production candidate**.

Nhưng ngay cả lúc đó, P7 vẫn chưa đóng.

Bước tiếp theo vẫn là:

> **profile lại winner.**

## Winner cũng phải bị soi lại

P7-M đo chính graph P7-L đã thắng.

Prefill lúc này có:

```text
441 dispatch
```

Một cached decode step có:

```text
469 dispatch
```

Trong prefill, tỷ trọng thời gian GPU xấp xỉ:

```text
fused gate/up       44,40%
FFN down             30,42%
attention              9,34%
attn Q/K/V             7,59%
attn output             5,86%
```

Phần barrier/unattributed — **thời gian không quy được rõ vào các kernel chính hoặc dùng cho đồng bộ** — chỉ khoảng:

```text
0,024%
```

Một thông điệp rất rõ xuất hiện:

> **Không còn cơ sở để đổ lỗi chính cho orchestration hay barrier.**

Phần lớn chi phí vẫn nằm trong những phép tính thật.

## Một PASS không có nghĩa mọi fusion tiếp theo đều tốt

P7-L fusion thành công.

Một phản xạ dễ mắc là:

> “Fusion tốt. Fuse thêm.”

P7-N thử gộp tiếp SwiGLU vào gate+up.

Correctness PASS.

Performance cũng tăng:

```text
≈ 1,0516×
```

Nhưng gate vẫn là:

```text
>= 1,10×
```

FAIL.

Không rescue.

Không nói:

> “5% cũng khá mà.”

P7-O sau đó thử một mechanism độc lập ở FFN-down: tăng K tile từ 32 lên 64.

Correctness PASS sau khi implementation defect được sửa.

Performance:

```text
≈ 0,8837×
```

Chậm hơn rõ rệt.

FAIL.

Và lần này quyết định không phải:

> “Thử K128 xem sao.”

Mà là:

> **Dừng P7.**

## Tại sao dừng khi vẫn còn ý tưởng?

Bởi một research program không được đánh giá bằng số ý tưởng còn có thể nghĩ ra.

Sau P7-L:

- gate/up đã có winner;
- FFN-down đã được challenge thêm và K64 thất bại;
- fusion rộng hơn với SwiGLU không vượt gate;
- nhiều biến thể tiling/dequant đã thất bại;
- profile cho thấy các family còn lại nhỏ hơn những bottleneck ban đầu;
- muốn tiếp tục có nguy cơ phải thay nhiều family cùng lúc, làm mất khả năng biết chính xác thứ gì tạo ra gain.

P7 vì vậy đóng với:

> **P7-L production path frozen.**

Đây là điểm hội tụ.

Không phải vì code không thể tối ưu thêm.

Mà vì evidence hiện tại không còn biện minh cho việc tiếp tục kéo dài P7.

## Production path cuối P7 gồm những gì?

Đường được freeze giữ:

```text
Q4_K / Q6_K packed weights
→ đọc trực tiếp, không bung toàn model

whole decoder resident
→ trọng số toàn decoder giữ sẵn

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

prefill attention
→ online attention

decode
→ validated cached-decode graph
```

Đây là ý nghĩa của từ **production path** trong P7.

Nó không có nghĩa:

> “ArcLLM đã là sản phẩm production hoàn thiện.”

Nó có nghĩa hẹp hơn:

> **Trong phạm vi model và architecture hiện tại, đã có một composition được chọn từ evidence, correctness đã giữ, các candidate quan trọng đã được thử, winner đã được freeze và P7 có thể đóng.**

## Và vẫn chưa được phép nói ArcLLM nhanh

Đây là giới hạn quan trọng nhất của Phần I.

P7 đã tạo ra những speedup nội bộ rất lớn.

P7-C từng đạt hơn 5× so với baseline của chính experiment đó.

P7-E đạt khoảng 1,85×.

P7-L đạt khoảng 1,24×.

Nhưng ta **không được nhân chúng lại rồi nói ArcLLM nhanh hơn runtime khác**.

Ta cũng không được lấy một throughput đẹp ở một run rồi so với một con số llama.cpp được chạy ở điều kiện khác.

P7 ghi nhận rằng absolute throughput — **tốc độ tuyệt đối** — thay đổi đáng kể giữa các run trên target machine.

Vì vậy các quyết định tối ưu dựa chủ yếu vào:

> **same-run interleaved A/B — chạy baseline và candidate xen kẽ trong cùng phiên đo.**

Lý do trực giác rất đơn giản.

Nếu hôm thứ Hai máy nóng, có process nền và trạng thái driver khác hôm thứ Ba, so:

```text
A hôm thứ Hai
với
B hôm thứ Ba
```

rất dễ trộn performance của kernel với trạng thái của cả máy.

So A/B xen kẽ trong cùng run giúp giảm phần nhiễu đó.

Chương 9 sẽ đi sâu hơn vào cách benchmark và các thống kê như median.

## Phần I kết thúc ở một ranh giới rất quan trọng

Ta đã đi từ:

```text
một file GGUF
↓
tensor store
↓
Vulkan device
↓
primitive
↓
một layer
↓
28 layer
↓
KV cache
↓
autoregressive generation
↓
profile
↓
nhiều PASS và FAIL
↓
P7-L production path
```

Đó là một thành tựu kỹ thuật có thật.

Nhưng science chưa cho phép kết luận:

> “ArcLLM cạnh tranh được với runtime trưởng thành.”

Để trả lời điều đó, ta cần một thứ mà đến giờ cuốn sách cố tình chưa làm:

> **một benchmark có đối chứng phù hợp.**

Không phải reference để nhìn cho biết.

Không phải tốc độ của một run cũ.

Không phải cảm giác “nhanh hơn nhiều”.

Mà là:

```text
cùng model
cùng quant
cùng hardware
cùng workload
cùng cách đo
        ↓
ArcLLM
vs
baseline phù hợp
```

Phần II bắt đầu từ đó.

### Nhớ 3 điều

1. **Performance optimization phải bắt đầu bằng measurement, không bằng danh sách ý tưởng.** P7 liên tục profile → chọn một bottleneck → thử một mechanism → đo lại.
2. **Một candidate có nhanh hơn vẫn có thể FAIL.** P7-K tăng khoảng 6,8% nhưng không vượt gate `1,10×`; P7-N tăng khoảng 5,2% nhưng vẫn FAIL. Gate không được sửa sau outcome.
3. **P7-L là production-path winner, không phải bằng chứng ArcLLM thắng runtime khác.** Muốn đưa ra claim đó, cuốn sách phải chuyển sang benchmark matched ở Chương 9.

**Chương 9 — Benchmark phải có đối chứng**

Từ đây, câu hỏi không còn là:

> “ArcLLM có thể tự cải thiện chính nó bao nhiêu?”

Mà là:

> **“Khi đặt cạnh một baseline trưởng thành dưới cùng điều kiện, ArcLLM thực sự đang đứng ở đâu?”**
