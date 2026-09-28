# Chương 6 — Full decoder residency: giữ cả “tòa nhà” trên GPU

> **Câu hỏi của chương:** Một decoder layer đã chạy đúng. Nhưng nếu giữ toàn bộ 28 layer cùng trọng số của model trong đường thực thi GPU, kết quả cuối cùng có còn đúng không?

Ở Chương 5, ArcLLM đã đi được một bước khá xa.

Không còn là một kernel riêng lẻ.

Không còn là một phép RMSNorm hay một phép nhân Q4_K đứng một mình.

Một decoder layer thật của Qwen2 đã chạy từ đầu đến cuối bằng 15 Vulkan dispatch — **15 công việc tính toán GPU** — với các kết quả trung gian tiếp tục nằm ở phía GPU.

P4 PASS.

Nhưng model không chỉ có một layer.

Model đang được ArcLLM sử dụng có **28 decoder layers**.

Nếu coi một layer là một căn phòng, P4 mới chứng minh rằng một căn phòng có thể được xây đúng.

P5 hỏi:

> **Ta có thể dựng cả tòa nhà, giữ toàn bộ những phần cần thiết trong bộ nhớ và cho tín hiệu đi xuyên từ tầng đầu tới tầng cuối mà không phải liên tục quay về CPU giữa chừng hay không?**

Đây là bước chuyển từ **one-layer correctness — tính đúng của một layer** sang **full-decoder correctness — tính đúng của toàn bộ chuỗi decoder**.

## Trước hết: dữ liệu đi vào model bằng cách nào?

Ở các chương trước, chúng ta thường bắt đầu từ một dãy số đã có sẵn.

Nhưng model thật không nhận trực tiếp chữ:

> “Xin chào”

Nó nhận token ID — **mã số của token**.

Giả sử tokenizer biến một token nào đó thành:

```text
token ID = 1234
```

Con số `1234` tự nó chưa mang đủ thông tin để đi qua 28 layer.

Model cần biến token ID ấy thành một dãy số dài hơn.

Công việc đó được gọi là **embedding — biến mã token thành một vector số mà model có thể xử lý**.

Có thể hình dung:

```text
token ID
  1234
    ↓
embedding
    ↓
[0,12, -0,08, 0,44, ...]
```

Dãy số phía dưới mới bắt đầu đi vào các decoder layer.

P5 không dùng một bảng embedding giả.

Nó sử dụng tensor thật:

```text
token_embd.weight
```

được lưu ở dạng Q6_K đóng gói trong GGUF.

Nhắc lại, **Q6_K là một dạng lượng tử hóa — cách lưu trọng số gọn hơn bằng ít bit hơn so với F32**.

GPU vì vậy phải bắt đầu ngay từ dữ liệu model thật.

## Từ một token đi xuyên qua 28 layer

P5 cố tình khóa **sequence length = 1**.

Sequence length — **độ dài chuỗi đang được xử lý** — ở đây chỉ có một token.

Tại sao không dùng một câu dài hơn?

Không phải vì ArcLLM chỉ chạy được một token.

P4 trước đó đã kiểm tra causal GQA attention — **attention có ràng buộc thứ tự token** — với sequence length bằng 4.

P5 muốn cô lập một câu hỏi khác:

> **Nếu bỏ độ phức tạp của chuỗi dài sang một bên, toàn bộ chiều sâu 28 layer có thể chạy đúng trong trạng thái resident hay không?**

Đây là một ví dụ rất hay về cách thiết kế thí nghiệm.

Nếu thay đổi quá nhiều thứ cùng lúc:

```text
chuỗi dài hơn
+
28 layer
+
toàn bộ trọng số
+
LM head
+
bộ nhớ mới
```

rồi kết quả sai, ta sẽ không biết lỗi nằm ở đâu.

P5 vì vậy giữ sequence length ở 1 để tập trung vào **full-depth residency — khả năng giữ và chạy xuyên toàn bộ chiều sâu model**.

## 338 tensor được giữ lại

Ở Chương 2, chúng ta đã đếm:

```text
338 tensor
```

với tổng payload đóng gói:

```text
980.097.536 byte
≈ 934,7 MiB
```
Ở P2, payload này đã được đưa vào bốn vùng lưu trọng số.

Nhưng P5 đi xa hơn.

Không chỉ “có bốn vùng nhớ”.

Runtime giờ phải biết tensor nào thuộc đâu và giữ được toàn bộ tập trọng số cần cho model thật.

P5 giữ:

```text
338 / 338 tensor
```

trong:

```text
4 tensor-aware packed weight arenas
```

Ta tách cụm này:

- **weight arena**: vùng lưu trọng số lớn;
- **packed**: trọng số vẫn giữ dạng đóng gói Q4_K/Q6_K;
- **tensor-aware**: runtime vẫn biết ranh giới và vị trí của từng tensor bên trong các vùng đó.

Có thể hình dung:

```text
Arena 1
┌───────────────────────┐
│ tensor A              │
│ tensor B              │
│ tensor C              │
└───────────────────────┘

Arena 2
┌───────────────────────┐
│ tensor D              │
│ tensor E              │
│ ...                   │
└───────────────────────┘
```

Không phải chỉ ném 934,7 MiB byte vào GPU rồi hy vọng tìm lại được.

Runtime phải biết:

> “Tensor tôi cần cho layer 17 nằm ở đâu?”

Đây là khác biệt giữa **có dữ liệu trong bộ nhớ** và **có một model có thể thực thi**.

## “Resident” lần này có nghĩa mạnh hơn

Ta đã gặp từ **resident — cư trú, tức dữ liệu được giữ sẵn trong vùng nhớ cần dùng**.

Ở P2, ý nghĩa còn khá cơ bản:

> dữ liệu đã được đặt vào các buffer và chưa bị giải phóng.

Đến P5, residency có ý nghĩa thực tế hơn.

Toàn bộ các trọng số cần cho chuỗi:

```text
embedding
   ↓
28 decoder layers
   ↓
final norm
   ↓
LM head
```

được giữ sẵn để đường thực thi có thể đi xuyên model mà không phải mỗi layer lại quay ra nạp trọng số từ đầu.

Ta có thể so hai cách tưởng tượng.

Cách tệ:

```text
Layer 0
→ lấy trọng số
→ tính
→ bỏ

Layer 1
→ lấy trọng số
→ tính
→ bỏ

Layer 2
→ lấy trọng số
→ tính
→ bỏ
```

Còn hướng P5:

```text
toàn bộ trọng số cần thiết
→ đã cư trú

token đi vào
→ layer 0
→ layer 1
→ layer 2
→ ...
→ layer 27
```

Tòa nhà đã có sẵn tất cả các phòng.

Ta chỉ cho tín hiệu đi xuyên qua.

## Sau 28 layer vẫn chưa xong

Khi tín hiệu đi qua layer cuối cùng, model chưa lập tức có token mới.

Còn hai bước quan trọng.

Đầu tiên là **final RMSNorm — phép chuẩn hóa cuối cùng**.

Ta đã gặp RMSNorm ở Chương 4. Nhắc lại ngắn gọn: nó điều chỉnh độ lớn của tín hiệu về một thang phù hợp trước khi bước sang phần tiếp theo.

Sau đó là **LM head — lớp đầu ra biến trạng thái cuối của model thành điểm số cho các token có thể được chọn tiếp theo**.

Có thể hình dung:

```text
hidden state cuối
      ↓
final RMSNorm
      ↓
LM head
      ↓
điểm cho token 0
điểm cho token 1
điểm cho token 2
...
```

Những điểm này được gọi là **logits — điểm số thô mà model gán cho từng token ứng viên**.

Ví dụ đồ chơi:

```text
token A : 1,2
token B : 4,8
token C : 0,7
```

Token B có logit cao nhất.

Trong P5, ArcLLM chỉ cần kiểm tra xem CPU và GPU có đồng ý về **token đứng đầu — top1** hay không. Cách một hệ thống hoàn chỉnh lựa chọn token để sinh văn bản là một lớp khác và không phải câu hỏi của bước này.
## “Tied” embedding và LM head

P5 có một chi tiết thú vị:

Embedding đầu vào và LM head đầu ra dùng chung tensor trọng số:

```text
token_embd.weight
```

Cách này thường được gọi là **tied weights — hai vị trí trong model dùng chung cùng một bộ trọng số**.

Hãy hình dung một cuốn từ điển được dùng ở hai đầu:

```text
đầu vào
token ID
   ↓
cùng bảng trọng số

...

đầu ra
trạng thái model
   ↓
cùng bảng trọng số
   ↓
logits
```

P5 dùng chính tensor Q6_K đóng gói đó cho cả embedding và LM head theo cấu trúc model đã khóa.

Điều này cũng tạo thêm một bài kiểm tra gián tiếp tốt: cùng một khối dữ liệu phải được dùng đúng trong hai vai trò khác nhau của graph.

## LM head quá lớn để xử lý như một cục duy nhất

LM head phải tạo điểm cho rất nhiều token trong vocabulary — **tập các token mà model biết**.

Nếu cố làm tất cả trong một dispatch khổng lồ, runtime có thể đụng phải những giới hạn thực thi không cần thiết.

P5 vì vậy chia các hàng của LM head thành những **chunk — phần nhỏ có kích thước được giới hạn**.

Có thể hình dung:

```text
LM head rất lớn
      ↓
chunk 1
chunk 2
chunk 3
...
      ↓
ghép thành logits cuối
```

Điều quan trọng là việc chia chunk này vẫn diễn ra mà không có **intermediate host round-trip — vòng lặp tính toán trung gian quay ngược về CPU**.

P5 tiếp tục giữ nguyên ranh giới đã được đặt ở P4: dữ liệu trung gian không được kéo về CPU giữa chuỗi chỉ để rồi lại gửi xuống GPU.

## 441 dispatch trong một command buffer

Toàn bộ đường thực thi P5 tạo ra:

```text
441 Vulkan dispatches
```

Nhắc lại:

**dispatch — một lần runtime giao một công việc tính toán cụ thể cho GPU**.

441 là con số rất khác 15 ở P4.

Điều đó hợp lý vì bây giờ ta không chạy một layer nữa mà là toàn bộ:

```text
embedding
+
28 layers
+
final norm
+
LM head
```

Nhưng điều đáng chú ý hơn là:

```text
441 dispatches

→ 1 command buffer
→ 1 submit
→ 1 fence wait
```

Tức toàn bộ chuỗi được ghi vào **một command buffer — một danh sách lệnh GPU**, rồi được gửi xuống queue một lần.

CPU không đứng giữa từng layer để điều phối bằng cách đọc kết quả lên rồi quyết định bước tiếp.

Có thể hình dung:

```text
CPU chuẩn bị toàn bộ kế hoạch
        ↓
submit
        ↓
GPU:
embedding
→ layer 0
→ layer 1
→ ...
→ layer 27
→ norm
→ LM head
        ↓
fence báo xong
        ↓
CPU mới kiểm tra kết quả
```

Đây là điều P5 gọi là **full decoder residency**.

## Hai checkpoint correctness thay vì chỉ một

P5 không chỉ kiểm tra kết quả ở cuối LM head.

Có hai tầng được kiểm tra.

Thứ nhất:

**final normalized hidden — trạng thái cuối sau 28 layer và phép chuẩn hóa cuối**.

Thứ hai:

**logits — các điểm đầu ra sau LM head**.

Việc kiểm tra hai tầng giúp khoanh vùng tốt hơn.

Nếu hidden state cuối đã sai, lỗi có thể nằm trong 28 layer.

Nếu hidden state đúng nhưng logits sai, ta có lý do nhìn gần hơn vào LM head.

P5 khóa trước các gate:

```text
Final normalized hidden:

max_abs <= 0,03
RMSE    <= 0,005
```

và:

```text
Logits:

max_abs <= 0,10
RMSE    <= 0,01
```

Nhắc lại:

- **max_abs — sai số tuyệt đối lớn nhất**: phần tử tệ nhất lệch bao nhiêu;
- **RMSE — căn trung bình bình phương sai số**: cả dãy nhìn chung lệch bao nhiêu.

Ngoài ra, tất cả giá trị phải **finite — hữu hạn**, tức không được xuất hiện những giá trị vô nghĩa như vô cực hoặc `NaN`.

## Kết quả thật nhỏ hơn gate rất nhiều

P5 chạy xong với kết quả final norm:

```text
max_abs
= 0,0002012252808

gate
<= 0,03
```

và:

```text
RMSE
= 0,000007641232208

gate
<= 0,005
```

Đối với logits:

```text
max_abs
= 0,00003051757812

gate
<= 0,10
```

và:

```text
RMSE
= 0,000005368667236

gate
<= 0,01
```

Tất cả đều nằm trong các gate đã khóa.

Nhưng còn một kiểm tra rất trực quan khác.

CPU và GPU cùng chọn:

```text
top1 = 117612
```

Nói bằng tiếng Việt:

> **Token ID có điểm cao nhất ở CPU và GPU là cùng một token ID: 117612.**

Chúng ta chưa cần biết token ID 117612 tương ứng với chuỗi ký tự nào.

P5 không hỏi câu đó.

Điều P5 cần biết là:

> **Hai con đường tính độc lập có đồng ý token nào đứng đầu hay không?**

Và câu trả lời là có.

## Đây đã phải là “model biết nói” chưa?

Chưa.

Đây là điểm rất quan trọng.

Ta đã có:

```text
token ID đầu vào
      ↓
embedding thật
      ↓
28 decoder layers
      ↓
final norm
      ↓
LM head
      ↓
logits
      ↓
top1
```

Trông gần như model hoàn chỉnh rồi.

Nhưng để model **sinh liên tục nhiều token**, còn thiếu một phần lớn.

Giả sử model vừa tạo token mới.

Ở bước sau, token đó phải được thêm vào chuỗi trước đó.

Attention cần nhớ những thông tin đã tính từ các token cũ.

Nếu mỗi token mới lại bắt model tính lại toàn bộ lịch sử từ đầu, chi phí sẽ rất lớn.

Đây là lúc chúng ta cần **KV cache — bộ nhớ lưu lại Key và Value của attention từ các token trước để không phải tính lại mọi thứ từ đầu**.

P5 chưa có KV cache generation path.

Vì vậy P5 **không được phép** tuyên bố:

> “ArcLLM đã có vòng lặp tạo sinh tự hồi quy hoàn chỉnh.”

Muốn model thật sự sinh chuỗi token liên tục, ArcLLM còn cần **KV cache và vòng lặp tạo sinh tự hồi quy (autoregressive generation)**.

Đó là câu hỏi của P6.

## P5 cũng chưa phải benchmark

Có 441 dispatch.

Có một submit.

Có toàn bộ model resident.

Điều này nghe rất dễ dẫn tới câu hỏi:

> “Vậy nhanh chưa?”

P5 không trả lời.

P5 được thiết kế cho **correctness — tính đúng**.

Không phải throughput — **tốc độ xử lý**.

Không phải tokens/s — **số token sinh mỗi giây**.

Không phải TTFT — **thời gian tới token đầu tiên**.

Ta chưa được phép nhìn một kiến trúc chạy đúng rồi tự động gọi nó là nhanh.

Đó là một ranh giới sẽ đặc biệt quan trọng ở những chương sau.

Một runtime có thể:

```text
đúng
nhưng chậm
```

hoặc:

```text
nhanh
nhưng sai
```

ArcLLM chọn thứ tự:

```text
đúng trước
↓
đo sau
↓
tối ưu sau nữa
```

## P5 PASS thực sự có nghĩa gì?

Tóm tắt P5 bằng ngôn ngữ đã quen:

```text
338 / 338 tensor
→ toàn bộ tensor của GGUF được giữ resident

980.097.536 packed bytes
→ khoảng 934,7 MiB dữ liệu trọng số đóng gói

4 tensor-aware weight arenas
→ 4 vùng lưu trọng số, vẫn biết ranh giới từng tensor

token embedding Q6_K
→ biến token ID thành vector đầu vào bằng trọng số thật

28 decoder layers
→ toàn bộ chiều sâu decoder

final RMSNorm
→ chuẩn hóa trạng thái cuối

tied Q6_K LM head
→ dùng chung token_embd.weight để tạo logits đầu ra

441 Vulkan dispatches
→ 441 công việc GPU

1 command buffer
1 submit
1 fence wait
→ cả chuỗi được gửi như một execution liên tục

zero intermediate host reads/writes
→ không có intermediate host round-trip

final norm
max_abs = 0,0002012252808
RMSE    = 0,000007641232208
→ PASS

logits
max_abs = 0,00003051757812
RMSE    = 0,000005368667236
→ PASS

CPU top1 = 117612
GPU top1 = 117612
→ cùng lựa chọn đứng đầu
```

Vì vậy P5 PASS.

Nhưng chỉ nên diễn giải thành:

> **Toàn bộ decoder của model đã chạy trên Vulkan với toàn bộ trọng số cư trú, không có intermediate host round-trip, và cả trạng thái cuối lẫn logits đều vượt qua các cổng correctness đã khóa.**

P5 chưa chứng minh tốc độ.

P5 chưa chứng minh generation nhiều token.

P5 chưa có KV cache path hoàn chỉnh.

Nhưng một ranh giới rất lớn vừa được vượt qua.

Ở P4, ta có một căn phòng.

Ở P5, cả tòa nhà đã đứng.

Bây giờ ta có thể bắt đầu cho nó hoạt động theo thời gian.

## Câu hỏi tiếp theo: model nhớ token trước bằng cách nào?

Nếu ta muốn model sinh:

```text
token 1
↓
token 2
↓
token 3
↓
token 4
```

thì ở token 4, model cần thông tin từ những token trước đó.

Không thể mỗi bước đều phá toàn bộ tòa nhà rồi xây lại từ đầu.

Ta cần một dạng bộ nhớ giữ những phần đã tính có thể tái sử dụng.

Đó là **KV cache**.

Chương tiếp theo sẽ là lần đầu ArcLLM chuyển từ:

> **“một lượt chạy toàn decoder có đúng không?”**

sang:

> **“model có thể giữ trạng thái và tự sinh token tiếp theo, rồi tiếp tục lặp lại quá trình đó không?”**

### Nhớ 3 điều

1. **P5 mở rộng từ một layer lên toàn bộ 28-layer decoder.** 338 tensor và khoảng 934,7 MiB payload đóng gói được giữ resident trong bốn vùng trọng số.
2. **Correctness được kiểm tra ở hai điểm:** final normalized hidden và logits; CPU/GPU cũng đồng ý `top1 = 117612`.
3. **Full decoder PASS chưa phải generation PASS và chưa phải performance PASS.** Muốn model thật sự sinh chuỗi token liên tục, ArcLLM còn cần KV cache và vòng lặp tạo sinh tự hồi quy (autoregressive generation).

**Chương 7 — KV cache và token đầu tiên được sinh liên tục**

Ta đã cho một token đi xuyên cả model.

Bước tiếp theo là làm cho model nhớ những gì vừa xảy ra — để token tiếp theo không phải bắt đầu lại từ đầu.
