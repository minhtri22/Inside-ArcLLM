# Chương 7 — Bộ nhớ giúp mô hình không phải tính lại từ đầu (KV cache)

> **Mức đọc: Đi sâu**
>
> **Bạn đang mở phần nào của cỗ máy?**
>
> ```text
> Token cũ ─┐
> Token cũ ─┼→ [ bộ nhớ đệm KV ]
> Token mới ─┘          ↓
>                  token tiếp theo
> ```


> **Câu hỏi của chương:** Sau khi một token đã đi xuyên toàn bộ mô hình, làm thế nào để token tiếp theo sử dụng lại những gì GPU vừa tính thay vì bắt đầu lại từ đầu?

Ở cuối Chương 6, ArcLLM đã đi xuyên toàn bộ sinh tokenr.

Một token ID được biến thành phép nhúng, đi qua 28 lớp giải mã, qua phép chuẩn hóa cuối, tới lớp tạo điểm đầu ra (LM head) và tạo ra điểm dự đoán — **điểm số mà mô hình gán cho các token có thể đứng tiếp theo**.

CPU và GPU thậm chí còn đồng ý về token có điểm cao nhất.

Nhưng P5 vẫn chỉ giống như chụp một bức ảnh.

Mô hình nhận một đầu vào.

Mô hình tính.

Mô hình cho một đầu ra.

Trong thực tế, mô hình ngôn ngữ phải làm điều gì đó động hơn:

```text
đọc các token ban đầu
        ↓
sinh token mới
        ↓
dùng cả lịch sử đó
để sinh token tiếp theo
        ↓
lặp lại
```

P6 là bước ArcLLM bắt đầu làm việc này.

Và để hiểu P6, chúng ta cần làm quen với một khái niệm rất quan trọng:

**bộ nhớ đệm KV — vùng nhớ giữ lại một phần kết quả cơ chế chú ý của các token đã xử lý để có thể tái sử dụng ở bước sau.**

## Nếu không nhớ, mô hình phải làm lại rất nhiều việc

Giả sử mô hình đang xử lý bốn token:

```text
A B C D
```

Sau đó nó sinh thêm token:

```text
E
```

Đến bước tiếp theo, mô hình cần xử lý:

```text
A B C D E
```

Một cách ngây thơ là tính lại toàn bộ A, B, C, D từ đầu.

Rồi khi có F:

```text
A B C D E F
```

lại tính lại mọi thứ một lần nữa.

Ta có thể hình dung:

```text
bước 1:
A B C D
████

bước 2:
A B C D E
█████

bước 3:
A B C D E F
██████
```

Phần bên trái cứ bị tính đi tính lại.

Đó là nơi bộ nhớ đệm KV xuất hiện.

Thay vì quên sạch sau mỗi token, hệ thực thi giữ lại những phần của cơ chế chú ý có thể tái sử dụng.

## K và V là gì?

Ở Chương 4, chúng ta đã gặp cơ chế chú ý — **cơ chế cho phép token hiện tại kết hợp thông tin từ những vị trí khác trong chuỗi**.

Bên trong cơ chế chú ý thường xuất hiện ba nhóm dữ liệu:

```text
Q — Query
K — Key
V — Value
```

Ta chưa cần học công thức cơ chế chú ý.

Có thể dùng một phép ví von rất thô nhưng hữu ích.

**Query — Q** giống như câu hỏi:

> “Ở bước hiện tại tôi đang tìm thông tin gì?”

**Key — K** giống như những nhãn giúp quyết định:

> “Phần thông tin nào trong lịch sử có liên quan tới câu hỏi đó?”

**Value — V** là:

> “Nếu phần ấy liên quan, nội dung nào sẽ được lấy ra để sử dụng?”

Ví dụ hình dung:

```text
token hiện tại
     ↓
Query: "tôi cần gì?"
     ↓
so với các Key đã có
     ↓
chọn mức chú ý
     ↓
kết hợp các Value tương ứng
```

Điều quan trọng với P6 là:

> **K và V của các token cũ không nhất thiết phải được tính lại mỗi lần.**

Ta có thể giữ chúng.

Đó chính là bộ nhớ đệm KV.

## bộ nhớ đệm nghĩa là “giữ thứ đã tính rồi”

Từ **bộ nhớ đệm — bộ nhớ đệm/tái sử dụng** xuất hiện rất nhiều trong máy tính.

Ý tưởng chung rất đơn giản:

> Nếu một kết quả đã được tạo ra, còn cần dùng lại và việc tính lại nó tốn công, hãy giữ nó ở nơi có thể lấy lại nhanh hơn.

bộ nhớ đệm KV áp dụng nguyên tắc này vào cơ chế chú ý.

Thay vì:

```text
token mới
   ↓
tính lại K/V của toàn bộ lịch sử
```

ta muốn:

```text
K/V của token cũ
→ đã nằm trong cache

token mới
→ chỉ bổ sung phần mới
→ attention dùng lại cache cũ
```

P6 là lần đầu ArcLLM kiểm tra con đường này xuyên qua toàn bộ 28 lớp.

## Hai giai đoạn: xử lý đầu vào (giai đoạn xử lý đầu vào) và sinh token (giai đoạn sinh token)

Khi một người gửi cho mô hình một **đoạn đầu vào**, ví dụ:

> “Hôm nay trời…”

Mô hình trước hết phải xử lý những token đã có.

Giai đoạn đó thường được gọi là **giai đoạn xử lý đầu vào — pha xử lý toàn bộ các token đầu vào ban đầu để xây trạng thái cần thiết cho việc sinh tiếp**.

Sau đó mô hình bắt đầu sinh từng token mới.

Mỗi bước như vậy được gọi là **giai đoạn sinh token — pha xử lý token mới nhất dựa trên trạng thái đã tích lũy trước đó**.

Hình dung:

```text
Đầu vào:
A B C D
│ │ │ │
└─┴─┴─┴── xử lý đầu vào
           ↓
        KV cache
           ↓
sinh E
           │
           └──── sinh token
                  ↓
               cập nhật cache
```

P6 khóa một bài thử rất nhỏ và rõ:

```text
4 token đầu vào
→ xử lý đầu vào

sau đó
→ 1 bước sinh token nối tiếp
```

**Autoregressive — tự hồi quy** ở đây chỉ có nghĩa:

> token vừa được chọn trở thành một phần đầu vào cho bước sinh token kế tiếp.

Không cần thêm ý nghĩa nào phức tạp hơn.

## Đầu vào của P6 không phải một câu tiếng Việt

P6 không dùng một câu tự nhiên để làm thí nghiệm.

Nó khóa trực tiếp các token ID:

```text
[1, 17, 42, 256]
```

Đây là một lựa chọn cố ý.

P6 không nghiên cứu bộ tách và mã hóa văn bản.

Không nghiên cứu chất lượng câu trả lời.

Không nghiên cứu mô hình “nói hay” hay “nói dở”.

Câu hỏi duy nhất là:

> **Đường bộ nhớ đệm KV + generation có đúng về mặt số học và trạng thái hay không?**

Dùng token ID cố định giúp loại bỏ những biến không liên quan.

P6 còn khóa:

```text
RoPE base position = 17
max_ctx = 16
```

Nhắc lại:

- **RoPE position — vị trí dùng trong phép mã hóa vị trí**;
- **max_ctx — giới hạn số vị trí ngữ cảnh được cấp cho bài test này**.

Các giá trị này không phải thông số tối ưu cho mọi mô hình.

Chúng chỉ là **điều kiện đã khóa của phép thử P6**.

## Giai đoạn xử lý đầu vào bắt đầu ghi “trí nhớ”

Trong pha giai đoạn xử lý đầu vào, bốn token đầu vào cùng đi qua mô hình.

Ở mỗi một trong 28 lớp, cơ chế chú ý tạo ra K và V tương ứng.

P6 không mang những dữ liệu này về CPU.

Thay vào đó, chúng được ghi vào:

> **persistent Vulkan các vùng nhớ — những vùng nhớ Vulkan được giữ lại để dùng ở bước tiếp theo.**

Ta có thể hình dung mỗi lớp có một cuốn sổ:

```text
Lớp 0
K cache: [...]
V cache: [...]

Lớp 1
K cache: [...]
V cache: [...]

...

Lớp 27
K cache: [...]
V cache: [...]
```

Khi giai đoạn xử lý đầu vào kết thúc, những cuốn sổ này vẫn còn ở GPU.

Không bị xóa.

Không bị đọc ngược về CPU.

Đây là điểm mới của P6.

Ở P5, trọng số đã resident — **cư trú sẵn trong bộ nhớ phục vụ GPU**.

Ở P6, bây giờ cả **trạng thái phát sinh theo chuỗi token** cũng bắt đầu được giữ lại.

## Giai đoạn sinh token đọc lại chính bộ nhớ đệm ấy

Sau giai đoạn xử lý đầu vào, mô hình có điểm dự đoán.

ArcLLM đọc điểm dự đoán về phía CPU để tìm token đứng đầu.

P6 cố tình dùng cách đơn giản và hoàn toàn xác định:

**chọn token có điểm cao nhất argmax — chọn token có logit cao nhất.**

P6 chỉ cần một quy tắc cố định để trả lời câu hỏi tính đúng.

Chuỗi trở thành:

```text
4 token đầu vào
      ↓
xử lý đầu vào trên GPU
      ↓
K/V cache được ghi
      ↓
điểm dự đoán
      ↓
CPU lấy top1
      ↓
token mới
      ↓
sinh token trên GPU
```

Nhưng điểm quan trọng nhất là trong bước giai đoạn sinh token:

> **cơ chế chú ý đọc trực tiếp bộ nhớ đệm K/V mà giai đoạn xử lý đầu vào vừa để lại trên GPU.**

Không có **vòng đi-về trung gian qua CPU** đối với bộ nhớ đệm KV.

CPU không đọc K/V.

CPU không ghi K/V.

bộ nhớ đệm tồn tại và được tiêu thụ trực tiếp trong đường GPU.

## Nhưng CPU vẫn còn tham gia

Điều này không có nghĩa P6 là một hệ thống “GPU tự làm mọi thứ”.

Có một ranh giới điều phối đã được khóa.

CPU vẫn làm ba việc:

```text
đọc điểm dự đoán
↓
tìm token có logit cao nhất
↓
ghi token ID được chọn cho bước sinh token tiếp theo
```

Nói cách khác:

```text
GPU
→ tính model + giữ KV

CPU
→ quyết định top1 theo luật đã khóa

GPU
→ tiếp tục sinh token
```

Đây là **orchestration boundary — ranh giới điều phối giữa CPU và GPU** của P6.

Một điều rất quan trọng cần phân biệt:

CPU đọc **điểm dự đoán**.

CPU không đọc **bộ nhớ đệm KV**.

Hai chuyện không giống nhau.

P6 muốn chứng minh trạng thái cơ chế chú ý có thể ở lại GPU xuyên qua các bước generation, chứ chưa cố loại CPU khỏi mọi phần của hệ thực thi.

## Cách tính tham chiếu trên CPU cũng có bộ nhớ đệm KV riêng

Làm sao biết bộ nhớ đệm GPU đúng?

Như các chương trước, GPU không tự chấm bài.

P6 chạy một **independent CPU reference — cách tính tham chiếu độc lập trên CPU**.

Nhưng lần này CPU reference cũng phải có bộ nhớ đệm KV riêng của nó.

Không thể để:

```text
GPU tạo cache
↓
CPU dùng cùng cache ấy để kiểm tra
```

vì khi đó hai bên không còn độc lập.

Đường kiểm tra đúng hơn là:

```text
GPU path
→ GPU KV cache riêng

CPU reference
→ CPU KV cache riêng

sau đó
→ so kết quả
```

Hai con đường nhận cùng input và cùng mô hình nhưng duy trì trạng thái của riêng mình.

Đến cuối, ta kiểm tra chúng có hội tụ hay không.

## Không chỉ so token cuối

Nếu CPU và GPU vô tình cùng chọn một token nhưng bộ nhớ đệm KV bên trong đã sai, lỗi có thể bộc lộ ở bước sau.

Vì vậy P6 không chỉ hỏi:

> “Hai bên có chọn cùng token không?”

Nó đặt nhiều cổng hơn.

Hai checkpoint điểm dự đoán phải đạt:

```text
max_abs <= 0,10
RMSE    <= 0,01
```

Nhắc lại:

- **max_abs — sai số tuyệt đối lớn nhất**;
- **RMSE — căn trung bình bình phương sai số, phản ánh sai lệch tổng thể của dãy**.

Ngoài ra token chọn token có điểm cao nhất — **token đứng đầu theo logit** — phải giống hệt giữa CPU và GPU.

Sau đó P6 còn kiểm tra trực tiếp phần K và V bộ nhớ đệm đã thật sự được sử dụng.

Gate:

```text
K cache:
max_abs <= 0,02
RMSE    <= 0,005

V cache:
max_abs <= 0,02
RMSE    <= 0,005
```

Và tất cả giá trị phải hữu hạn.

Như vậy P6 kiểm tra cả:

```text
output
+
quyết định token
+
trạng thái được giữ lại bên trong
```

Đây là một gate mạnh hơn nhiều so với chỉ nhìn câu trả lời cuối.

## Token đầu tiên sau giai đoạn xử lý đầu vào

P6 chạy thật.

Sau bốn token đầu vào:

```text
[1, 17, 42, 256]
```

CPU và GPU cùng chọn token:

```text
6228
```

Đây là output token đầu tiên của phép thử.

giai đoạn xử lý đầu vào điểm dự đoán có:

```text
max_abs
= 0,001761436462

RMSE
= 0,0003688782
```

Cả hai nằm trong gate đã khóa.

Token `6228` sau đó được đưa trở lại để thực hiện bước giai đoạn sinh token tiếp theo.

## Và token tiếp theo cũng khớp

giai đoạn sinh token sử dụng chính bộ nhớ đệm KV đã được tạo ở giai đoạn xử lý đầu vào.

Sau bước này, CPU và GPU tiếp tục đồng ý:

```text
token tiếp theo = 17
```

Toàn bộ hai token đầu ra của bài thử vì vậy là:

```text
[6228, 17]
```

Đây là **exact agreement — khớp chính xác token ID**, không phải chỉ gần nhau về điểm số.

giai đoạn sinh token điểm dự đoán cũng ĐẠT (PASS):

```text
max_abs
= 0,001020431519

RMSE
= 0,0002062647367
```

Nhưng vẫn còn câu hỏi quan trọng:

> bộ nhớ đệm mà GPU đang giữ có thực sự giống bộ nhớ đệm tham chiếu hay không?

## Mở “trí nhớ” ra kiểm tra sau cùng

Sau generation, P6 so phần K và V bộ nhớ đệm thực sự đã được dùng.

K bộ nhớ đệm:

```text
max_abs
= 0,01123046875

RMSE
= 0,000235098343
```

V bộ nhớ đệm:

```text
max_abs
= 0,0022777915

RMSE
= 0,000155599446
```

Tất cả đều nằm dưới gate:

```text
max_abs <= 0,02
RMSE    <= 0,005
```

Điều này rất quan trọng.

Ta không chỉ thấy:

```text
CPU token = 17
GPU token = 17
```

rồi tuyên bố mọi thứ đúng.

Ta còn có bằng chứng rằng **trạng thái K/V được giữ và sử dụng bên trong GPU cũng nằm trong giới hạn tính đúng đã định trước**.

## 338 khối số vẫn không rời chỗ

Trong toàn bộ P6, các trọng số mô hình vẫn giữ đúng trạng thái đã chứng minh ở P5:

```text
338 tensors
980.097.536 packed bytes
```

tất cả vẫn resident — **cư trú sẵn trong bốn vùng trọng số**.

P6 không đánh đổi bộ nhớ đệm KV bằng cách phá trạng thái cư trú trong bộ nhớ của trọng số.

Bây giờ có hai loại dữ liệu cần phân biệt:

```text
TRỌNG SỐ MÔ HÌNH
→ những gì mô hình đã học
→ gần như cố định trong suy luận
→ resident từ trước

KV CACHE
→ trạng thái sinh ra từ đầu vào/token hiện tại
→ thay đổi khi generation tiến lên
→ được giữ lại qua các bước sinh token
```

Đây là lần đầu kiến trúc hệ thực thi bắt đầu có “trí nhớ theo phiên chạy”.

## P6 ĐẠT thực sự cho phép nói gì?

Tóm tắt:

```text
đầu vào
→ 4 token IDs: [1, 17, 42, 256]

xử lý đầu vào
→ chạy bốn token
→ ghi K/V vào persistent GPU buffers
  trên cả 28 layer

KV cache
→ không có host read/write
→ không có intermediate host round-trip

CPU orchestration
→ đọc điểm dự đoán
→ chọn top1 bằng greedy argmax
→ ghi token ID cho bước tiếp

sinh token
→ dùng trực tiếp chính GPU KV cache

greedy output tokens
CPU = [6228, 17]
GPU = [6228, 17]

xử lý đầu vào điểm dự đoán
→ PASS

sinh token điểm dự đoán
→ PASS

K cache
→ PASS

V cache
→ PASS
```

P6 vì vậy CLOSED với ĐẠT (PASS).

Nhưng ranh giới vẫn phải giữ.

P6 **chưa chứng minh hệ thực thi nhanh**.

P6 không phép đo so sánh thông lượng.

P6 không chứng minh sequence dài.

P6 không chứng minh mọi chiến lược chọn token.

Và P6 chưa phải đường chạy thực tế.

Điều nó chứng minh là:

> **ArcLLM đã có một đường generation tối thiểu trong đó giai đoạn xử lý đầu vào tạo bộ nhớ đệm KV trên GPU, giai đoạn sinh token tái sử dụng trực tiếp bộ nhớ đệm đó, CPU và GPU cho cùng các token chọn token có điểm cao nhất, và bản thân bộ nhớ đệm K/V cũng vượt qua các cổng tính đúng đã khóa.**

Lần đầu tiên trong hành trình này, mô hình không chỉ tính một lượt.

Nó đã **giữ trạng thái từ quá khứ để tính bước tiếp theo**.

## Từ bằng chứng ban đầu sang đường chạy thực tế hơn

P0 tới P6 đã trả lời một chuỗi câu hỏi ngày càng lớn:

```text
đọc đúng mô hình?
      ↓
đưa dữ liệu lên GPU?
      ↓
từng phép tính đúng?
      ↓
một lớp đúng?
      ↓
28 lớp đúng?
      ↓
giữ được KV cache?
      ↓
sinh bước token tiếp theo đúng?
```

Đến đây một hệ thực thi tối thiểu đã hình thành.

Nhưng nó vẫn còn là một con đường được xây chủ yếu để chứng minh tính đúng.

Câu hỏi tiếp theo thay đổi:

> **Làm thế nào biến những mảnh đã chứng minh này thành một đường chạy Q4_K_M gần với cách mô hình thực tế sẽ được sử dụng hơn?**

Đó là P7.

### Nhớ 3 điều

1. **Giai đoạn xử lý đầu vào tạo K/V từ phần văn bản ban đầu; giai đoạn sinh token tái sử dụng K/V đã có khi xử lý token mới.** Đó là lý do bộ nhớ đệm KV tránh phải tính lại toàn bộ lịch sử ở mỗi bước.
2. **bộ nhớ đệm KV của P6 nằm ở GPU xuyên qua generation.** Không có vòng đi-về trung gian qua CPU đối với K/V.
3. **P6 ĐẠT về tính đúng của quá trình sinh token, chưa phải kết luận về hiệu năng hay một sản phẩm hoàn thiện.** Hai token được chọn `[6228, 17]`, điểm dự đoán và chính bộ nhớ đệm K/V đều vượt qua các ngưỡng đã khóa.

**Chương 8 — đường chạy thực tế không đến từ một chương trình GPU thần kỳ**

Hệ thực thi giờ đã có thể nhớ.

Bước tiếp theo là làm cho con đường đó giống một hệ thực thi sử dụng thực tế hơn.
