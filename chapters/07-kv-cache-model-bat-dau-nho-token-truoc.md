# Chương 7 — KV cache: model bắt đầu nhớ token trước

> **Câu hỏi của chương:** Sau khi một token đã đi xuyên toàn bộ model, làm thế nào để token tiếp theo sử dụng lại những gì GPU vừa tính thay vì bắt đầu lại từ đầu?

Ở cuối Chương 6, ArcLLM đã đi xuyên toàn bộ decoder.

Một token ID được biến thành embedding, đi qua 28 decoder layer, qua phép chuẩn hóa cuối, tới LM head và tạo ra logits — **điểm số mà model gán cho các token có thể đứng tiếp theo**.

CPU và GPU thậm chí còn đồng ý về token có điểm cao nhất.

Nhưng P5 vẫn chỉ giống như chụp một bức ảnh.

Model nhận một đầu vào.

Model tính.

Model cho một đầu ra.

Trong thực tế, model ngôn ngữ phải làm điều gì đó động hơn:

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

**KV cache — vùng nhớ giữ lại một phần kết quả attention của các token đã xử lý để có thể tái sử dụng ở bước sau.**

## Nếu không nhớ, model phải làm lại rất nhiều việc

Giả sử model đang xử lý bốn token:

```text
A B C D
```

Sau đó nó sinh thêm token:

```text
E
```

Đến bước tiếp theo, model cần xử lý:

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

Đó là nơi KV cache xuất hiện.

Thay vì quên sạch sau mỗi token, runtime giữ lại những phần của attention có thể tái sử dụng.

## K và V là gì?

Ở Chương 4, chúng ta đã gặp attention — **cơ chế cho phép token hiện tại kết hợp thông tin từ những vị trí khác trong chuỗi**.

Bên trong attention thường xuất hiện ba nhóm dữ liệu:

```text
Q — Query
K — Key
V — Value
```

Ta chưa cần học công thức attention.

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

Đó chính là KV cache.

## Cache nghĩa là “giữ thứ đã tính rồi”

Từ **cache — bộ nhớ đệm/tái sử dụng** xuất hiện rất nhiều trong máy tính.

Ý tưởng chung rất đơn giản:

> Nếu một kết quả đã được tạo ra, còn cần dùng lại và việc tính lại nó tốn công, hãy giữ nó ở nơi có thể lấy lại nhanh hơn.

KV cache áp dụng nguyên tắc này vào attention.

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

P6 là lần đầu ArcLLM kiểm tra con đường này xuyên qua toàn bộ 28 layer.

## Hai pha: prefill và decode

Khi một người gửi cho model một prompt, ví dụ:

> “Hôm nay trời…”

model trước hết phải xử lý những token đã có.

Giai đoạn đó thường được gọi là **prefill — pha xử lý toàn bộ các token đầu vào ban đầu để xây trạng thái cần thiết cho việc sinh tiếp**.

Sau đó model bắt đầu sinh từng token mới.

Mỗi bước như vậy được gọi là **decode — pha xử lý token mới nhất dựa trên trạng thái đã tích lũy trước đó**.

Hình dung:

```text
Prompt:
A B C D
│ │ │ │
└─┴─┴─┴── prefill
           ↓
        KV cache
           ↓
sinh E
           │
           └──── decode
                  ↓
               cập nhật cache
```

P6 khóa một bài thử rất nhỏ và rõ:

```text
4 token đầu vào
→ prefill

sau đó
→ 1 bước autoregressive decode
```

**Autoregressive — tự hồi quy** ở đây chỉ có nghĩa:

> token vừa được chọn trở thành một phần đầu vào cho bước sinh token kế tiếp.

Không cần thêm ý nghĩa nào phức tạp hơn.

## Prompt của P6 không phải câu tiếng Việt

P6 không dùng một câu tự nhiên để làm thí nghiệm.

Nó khóa trực tiếp các token ID:

```text
[1, 17, 42, 256]
```

Đây là một lựa chọn cố ý.

P6 không nghiên cứu tokenizer.

Không nghiên cứu chất lượng câu trả lời.

Không nghiên cứu model “nói hay” hay “nói dở”.

Câu hỏi duy nhất là:

> **Đường KV cache + generation có đúng về mặt số học và trạng thái hay không?**

Dùng token ID cố định giúp loại bỏ những biến không liên quan.

P6 còn khóa:

```text
RoPE base position = 17
max_ctx = 16
```

Nhắc lại:

- **RoPE position — vị trí dùng trong phép mã hóa vị trí**;
- **max_ctx — giới hạn số vị trí ngữ cảnh được cấp cho bài test này**.

Các giá trị này không phải thông số tối ưu cho mọi model.

Chúng chỉ là contract của phép thử P6.

## Prefill bắt đầu ghi “trí nhớ”

Trong pha prefill, bốn token đầu vào cùng đi qua model.

Ở mỗi một trong 28 layer, attention tạo ra K và V tương ứng.

P6 không mang những dữ liệu này về CPU.

Thay vào đó, chúng được ghi vào:

> **persistent Vulkan buffers — những vùng nhớ Vulkan được giữ lại để dùng ở bước tiếp theo.**

Ta có thể hình dung mỗi layer có một cuốn sổ:

```text
Layer 0
K cache: [...]
V cache: [...]

Layer 1
K cache: [...]
V cache: [...]

...

Layer 27
K cache: [...]
V cache: [...]
```

Khi prefill kết thúc, những cuốn sổ này vẫn còn ở GPU.

Không bị xóa.

Không bị đọc ngược về CPU.

Đây là điểm mới của P6.

Ở P5, trọng số đã resident — **cư trú sẵn trong bộ nhớ phục vụ GPU**.

Ở P6, bây giờ cả **trạng thái phát sinh theo chuỗi token** cũng bắt đầu được giữ lại.

## Decode đọc lại chính cache ấy

Sau prefill, model có logits.

ArcLLM đọc logits về phía CPU để tìm token đứng đầu.

P6 cố tình dùng cách đơn giản và hoàn toàn xác định:

**greedy argmax — chọn token có logit cao nhất.**

P6 chỉ cần một quy tắc cố định để trả lời câu hỏi correctness.

Chuỗi trở thành:

```text
4 token prompt
      ↓
prefill trên GPU
      ↓
K/V cache được ghi
      ↓
logits
      ↓
CPU lấy top1
      ↓
token mới
      ↓
decode trên GPU
```

Nhưng điểm quan trọng nhất là trong bước decode:

> **attention đọc trực tiếp K/V cache mà prefill vừa để lại trên GPU.**

Không có **intermediate host round-trip** đối với KV cache.

CPU không đọc K/V.

CPU không ghi K/V.

Cache tồn tại và được tiêu thụ trực tiếp trong đường GPU.

## Nhưng CPU vẫn còn tham gia

Điều này không có nghĩa P6 là một hệ thống “GPU tự làm mọi thứ”.

Có một ranh giới điều phối đã được khóa.

CPU vẫn làm ba việc:

```text
đọc logits
↓
tìm token có logit cao nhất
↓
ghi token ID được chọn cho bước decode tiếp theo
```

Nói cách khác:

```text
GPU
→ tính model + giữ KV

CPU
→ quyết định top1 theo luật đã khóa

GPU
→ tiếp tục decode
```

Đây là **orchestration boundary — ranh giới điều phối giữa CPU và GPU** của P6.

Một điều rất quan trọng cần phân biệt:

CPU đọc **logits**.

CPU không đọc **KV cache**.

Hai chuyện không giống nhau.

P6 muốn chứng minh trạng thái attention có thể ở lại GPU xuyên qua các bước generation, chứ chưa cố loại CPU khỏi mọi phần của runtime.

## CPU reference cũng có KV cache riêng

Làm sao biết cache GPU đúng?

Như các chương trước, GPU không tự chấm bài.

P6 chạy một **independent CPU reference — cách tính tham chiếu độc lập trên CPU**.

Nhưng lần này CPU reference cũng phải có KV cache riêng của nó.

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

Hai con đường nhận cùng input và cùng model nhưng duy trì trạng thái của riêng mình.

Đến cuối, ta kiểm tra chúng có hội tụ hay không.

## Không chỉ so token cuối

Nếu CPU và GPU vô tình cùng chọn một token nhưng KV cache bên trong đã sai, lỗi có thể bộc lộ ở bước sau.

Vì vậy P6 không chỉ hỏi:

> “Hai bên có chọn cùng token không?”

Nó đặt nhiều cổng hơn.

Hai checkpoint logits phải đạt:

```text
max_abs <= 0,10
RMSE    <= 0,01
```

Nhắc lại:

- **max_abs — sai số tuyệt đối lớn nhất**;
- **RMSE — căn trung bình bình phương sai số, phản ánh sai lệch tổng thể của dãy**.

Ngoài ra token greedy — **token đứng đầu theo logit** — phải giống hệt giữa CPU và GPU.

Sau đó P6 còn kiểm tra trực tiếp phần K và V cache đã thật sự được sử dụng.

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

## Token đầu tiên sau prefill

P6 chạy thật.

Sau bốn token prompt:

```text
[1, 17, 42, 256]
```

CPU và GPU cùng chọn token:

```text
6228
```

Đây là output token đầu tiên của phép thử.

Prefill logits có:

```text
max_abs
= 0,001761436462

RMSE
= 0,0003688782
```

Cả hai nằm trong gate đã khóa.

Token `6228` sau đó được đưa trở lại để thực hiện bước decode tiếp theo.

## Và token tiếp theo cũng khớp

Decode sử dụng chính KV cache đã được tạo ở prefill.

Sau bước này, CPU và GPU tiếp tục đồng ý:

```text
token tiếp theo = 17
```

Toàn bộ hai token đầu ra của bài thử vì vậy là:

```text
[6228, 17]
```

Đây là **exact agreement — khớp chính xác token ID**, không phải chỉ gần nhau về điểm số.

Decode logits cũng PASS:

```text
max_abs
= 0,001020431519

RMSE
= 0,0002062647367
```

Nhưng vẫn còn câu hỏi quan trọng:

> Cache mà GPU đang giữ có thực sự giống cache tham chiếu hay không?

## Mở “trí nhớ” ra kiểm tra sau cùng

Sau generation, P6 so phần K và V cache thực sự đã được dùng.

K cache:

```text
max_abs
= 0,01123046875

RMSE
= 0,000235098343
```

V cache:

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

Ta còn có bằng chứng rằng **trạng thái K/V được giữ và sử dụng bên trong GPU cũng nằm trong giới hạn correctness đã định trước**.

## 338 tensor vẫn không rời chỗ

Trong toàn bộ P6, các trọng số model vẫn giữ đúng trạng thái đã chứng minh ở P5:

```text
338 tensors
980.097.536 packed bytes
```

tất cả vẫn resident — **cư trú sẵn trong bốn vùng trọng số**.

P6 không đánh đổi KV cache bằng cách phá residency của weights.

Bây giờ có hai loại dữ liệu cần phân biệt:

```text
MODEL WEIGHTS
→ những gì model đã học
→ gần như cố định trong inference
→ resident từ trước

KV CACHE
→ trạng thái sinh ra từ prompt/token hiện tại
→ thay đổi khi generation tiến lên
→ persistent qua các bước decode
```

Đây là lần đầu kiến trúc runtime bắt đầu có “trí nhớ theo phiên chạy”.

## P6 PASS thực sự cho phép nói gì?

Tóm tắt:

```text
prompt
→ 4 token IDs: [1, 17, 42, 256]

prefill
→ chạy bốn token
→ ghi K/V vào persistent GPU buffers
  trên cả 28 layer

KV cache
→ không có host read/write
→ không có intermediate host round-trip

CPU orchestration
→ đọc logits
→ chọn top1 bằng greedy argmax
→ ghi token ID cho bước tiếp

decode
→ dùng trực tiếp chính GPU KV cache

greedy output tokens
CPU = [6228, 17]
GPU = [6228, 17]

prefill logits
→ PASS

decode logits
→ PASS

K cache
→ PASS

V cache
→ PASS
```

P6 vì vậy CLOSED với PASS.

Nhưng ranh giới vẫn phải giữ.

P6 **chưa chứng minh runtime nhanh**.

P6 không benchmark throughput.

P6 không chứng minh sequence dài.

P6 không chứng minh mọi chiến lược chọn token.

Và P6 chưa phải production path.

Điều nó chứng minh là:

> **ArcLLM đã có một đường generation tối thiểu trong đó prefill tạo KV cache trên GPU, decode tái sử dụng trực tiếp cache đó, CPU và GPU cho cùng các token greedy, và bản thân K/V cache cũng vượt qua các cổng correctness đã khóa.**

Lần đầu tiên trong hành trình này, model không chỉ tính một lượt.

Nó đã **giữ trạng thái từ quá khứ để tính bước tiếp theo**.

## Từ proof sang đường chạy thực tế hơn

P0 tới P6 đã trả lời một chuỗi câu hỏi ngày càng lớn:

```text
đọc đúng model?
      ↓
đưa dữ liệu lên GPU?
      ↓
từng phép tính đúng?
      ↓
một layer đúng?
      ↓
28 layer đúng?
      ↓
giữ được KV cache?
      ↓
sinh bước token tiếp theo đúng?
```

Đến đây một runtime tối thiểu đã hình thành.

Nhưng nó vẫn còn là một con đường được xây chủ yếu để chứng minh correctness.

Câu hỏi tiếp theo thay đổi:

> **Làm thế nào biến những mảnh đã chứng minh này thành một đường chạy Q4_K_M gần với cách model thực tế sẽ được sử dụng hơn?**

Đó là P7.

### Nhớ 3 điều

1. **Prefill — xử lý prompt ban đầu — tạo K/V; decode — xử lý token mới — tái sử dụng K/V đã có.** Đó là lý do KV cache tránh phải tính lại toàn bộ lịch sử ở mỗi bước.
2. **KV cache của P6 nằm ở GPU xuyên qua generation.** Không có intermediate host round-trip đối với K/V.
3. **P6 PASS là generation-correctness PASS, chưa phải performance hay production PASS.** Hai token greedy `[6228, 17]`, logits và chính K/V cache đều vượt qua các gate đã khóa.

**Chương 8 — Production path không đến từ một kernel thần kỳ**

Runtime giờ đã có thể nhớ.

Bước tiếp theo là làm cho con đường đó giống một runtime sử dụng thực tế hơn.
