# Chương 14 — Từ một cơ chế tốt tới hệ thống thật

> **Mức đọc: Nghiên cứu**
>
> **Bản đồ xuyên suốt — đang mở: Transfer / carry-through**
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
> ▶ **Đang mở ở chương này:** Transfer / carry-through.


> **Câu hỏi của chương:** Một cơ chế đã PASS trong phép thử thành phần có còn tạo ra lợi ích khi nó phải làm việc với model thật, dữ liệu thật và toàn bộ đường sinh token hay không?

Chương 13 cho ta hai con đường khác nhau.

Q6 dừng ở Mode C.

Cùng một cơ chế Split-K và hình học thực thi đã hoạt động tốt với Q4_K nhưng không giữ được tính đúng khi áp dụng sang Q6_K bằng candidate có bộ đọc packed-Q6 tương ứng.

Vì vậy:

```text
Q6
→ correctness FAIL
→ STOP
```

Nhưng Q4 thì khác.

Nó đã vượt qua phép thử thành phần.

Cơ chế đã rõ.

Tính đúng đã giữ.

Speedup cũng đủ lớn.

Q4 vì thế được quyền bước sang mode cuối cùng trong chuỗi E/M/C/T:

> **Mode T — Transfer, chuyển một cơ chế đã được xác nhận sang bối cảnh thực tế lớn hơn để xem bằng chứng có còn đứng vững hay không.**

Trong nghiên cứu ArcLLM, ta còn dùng từ:

> **carry-through — lợi ích có thực sự truyền xuyên qua các tầng của hệ thống hay không.**

Đây không phải là một cách nói hoa mỹ.

Nó mô tả một vấn đề rất thật:

```text
kernel nhanh hơn
↓
phép tính thật có nhanh hơn?
↓
decode có nhanh hơn?
↓
toàn bộ lượt sinh token có nhanh hơn?
```

Một mũi tên có thể đứt ở bất kỳ đâu.

## Phép thử nhỏ và model thật không phải cùng một thế giới

Ở phép thử Q4 trước đó, ta dùng các **fixture — bộ dữ liệu kiểm thử cố định**.

Chúng rất hữu ích.

Ta biết chính xác shape.

Biết dữ liệu nào được đưa vào.

Biết phép tính nào được chạy.

Có thể so kernel cũ và kernel mới trong một môi trường rất sạch.

Nhờ vậy ta có thể trả lời khá chắc chắn:

> **Cách chia K mới có làm phép tính Q4_K này nhanh hơn không?**

Nhưng model thật phức tạp hơn.

Trọng số là trọng số thật của model 7B.

`Activation — dữ liệu trung gian do model tạo ra trong lúc chạy` không còn là fixture nhân tạo.

Nó phụ thuộc vào:

- prompt;
- layer hiện tại;
- token hiện tại;
- trạng thái trước đó của model.

Một kernel có thể rất đẹp trên fixture nhưng khi gặp dữ liệu thật lại:

```text
sai số lớn hơn
```

hoặc:

```text
không nhanh như dự đoán
```

hoặc thậm chí:

```text
nhanh ở chính phép tính đó
nhưng phần còn lại của runtime nuốt mất lợi ích
```

Vì vậy Mode T không hỏi lại câu hỏi của Mode C.

Nó hỏi một câu mới:

> **Hiệu ứng đã xác nhận có sống sót khi bối cảnh trở nên thật hơn không?**

## Trước khi chuyển, phải biết chính xác chuyển vào đâu

Một local speedup không đáng được tích hợp chỉ vì nó lớn.

Ta vẫn cần biết phần đó có thực sự quan trọng trong model thật hay không.

Một phép đo riêng trên exact model 7B cho thấy họ phép tính:

```text
FFN gate + up
```

chiếm trung vị khoảng:

```text
59,67%
```

thời gian của chuỗi công việc GPU trong decode.

Nói đơn giản:

> **Trong phần công việc GPU đã đo của một token decode, gần 60% thời gian nằm ở gate và up.**

Đây là một khác biệt rất lớn so với việc chọn kernel chỉ vì nó “có vẻ đáng tối ưu”.

Ta đã có hai mảnh bằng chứng độc lập:

```text
mảnh 1
gate/up chiếm ~59,67% chi phí GPU decode

mảnh 2
cơ chế Split-K đã tăng tốc đúng shape gate/up Q4_K
khoảng 5,33× → 5,50× trong phép thử thành phần tương ứng
```

Hai mảnh ghép vào nhau.

Bây giờ mới xuất hiện một ứng viên transfer đủ mạnh:

> **Mang đúng cơ chế Split-K đã PASS vào đúng 56 phép gate/up Q4_K trong decode của model thật.**

Không phải tất cả kernel.

Không phải toàn bộ FFN.

Không phải Q6.

Chỉ:

```text
28 layer
×
2 phép gate + up
=
56 vị trí / token
```

## Amdahl cho ta một dự đoán trước khi chạy

Ở Chương 12, ta đã gặp định luật Amdahl.

Nếu một phần chiếm tỷ lệ:

```text
f = 0,5967
```

và phần đó có thể nhanh hơn khoảng:

```text
s = 5,33×
```

thì giới hạn tăng tốc đơn giản của vùng đang xét là:

```text
S = 1 / ((1 - f) + f/s)
```

Thế số:

```text
S
= 1 / ((1 - 0,5967) + 0,5967/5,33)

= 1 / (0,4033 + 0,1119)

≈ 1,94×
```

Tức nếu hiệu ứng thành phần chuyển sang model thật một cách thuận lợi, ta có lý do kỳ vọng một chuyển động lớn cỡ gần 2× trong chuỗi decode liên quan.

Nhưng cần đọc câu này rất cẩn thận:

> **1,94× là một dự đoán theo mô hình Amdahl, không phải kết quả experiment.**

Nó giúp ta quyết định:

> “Câu hỏi này có đáng chạy không?”

Nó không được dùng thay cho measurement.

## Mode T cũng phải khóa phạm vi

Đây là chỗ rất dễ phá hỏng một transfer study.

Ta có thể nói:

> “Tiện thể đã sửa gate/up thì tối ưu thêm FFN-down.”

Hoặc:

> “Hay fuse gate và up luôn.”

Hoặc:

> “Thử local size khác để chắc chắn có bản tốt nhất.”

Nếu làm vậy, dù runtime nhanh hơn, ta sẽ không còn biết:

> **Hiệu ứng Q4 Split-K có thực sự transfer không?**

Vì vậy thay đổi được khóa rất hẹp:

```text
decode only

Q4_K only

28 layer

gate + up only

56 node / token
```

Những phần sau giữ nguyên:

```text
prefill
Q/K/V
attention
attention output
FFN-down
SwiGLU
RMSNorm
LM head
model
quantization
KV cache
generation semantics
```

Cơ chế cũng giữ nguyên:

```text
subgroup = 32 lane

4 subgroup / workgroup

1 subgroup / output row

K chia qua 32 lane
```

Mode T không phải:

> “Tích hợp rồi tối ưu tiếp.”

Nó là:

> **“Mang đúng thứ đã PASS sang môi trường thật mà không để các thay đổi khác che mất câu trả lời.”**

## Bước đầu tiên: trọng số thật và activation thật

Transfer đầu tiên chưa chạy toàn bộ benchmark.

Nó lấy chính model 7B và kiểm tra cơ chế trên:

- trọng số Q4_K thật;
- activation thật do baseline tạo ra.

Ba layer được lấy mẫu:

```text
0
13
27
```

Ba vị trí decode:

```text
0
15
30
```

Hai operator:

```text
gate
up
```

và hai workload:

```text
W-S
W-C
```

Mỗi tổ hợp session/workload có:

```text
3 layer
×
3 vị trí
×
2 operator
=
18 phép so sánh
```

Có bốn cell:

```text
A/W-S
A/W-C
B/W-C
B/W-S
```

nên tổng cộng:

```text
18 × 4
=
72 phép so sánh
```

Tất cả 72 phải vượt correctness gate cũ:

```text
max_abs <= 0,02
RMSE    <= 0,005
```

## Kết quả thật còn sát hơn contract rất nhiều

Cả:

```text
72 / 72
```

phép so sánh đều PASS.

Sai lệch lớn nhất quan sát được:

```text
max_abs
≈ 0,00003815
```

trong khi giới hạn là:

```text
0,02
```

Ta có thể tính khoảng cách:

```text
0,02 / 0,00003815
≈ 524
```

Tức worst case vẫn nhỏ hơn ngưỡng khoảng **524 lần**.

RMSE tệ nhất:

```text
≈ 0,00000363
```

so với giới hạn:

```text
0,005
```

Khoảng cách:

```text
0,005 / 0,00000363
≈ 1 378
```

xấp xỉ **1 378 lần** — tức khoảng một nghìn ba trăm bảy mươi tám lần.

Nói dễ hiểu:

> **Cơ chế không chỉ vừa đủ vượt correctness gate. Nó còn có khoảng cách khá lớn so với giới hạn đã khóa.**

Đây là bằng chứng đầu tiên rằng hiệu ứng từ fixture đã chuyển được sang trọng số và activation thật.

## Còn tốc độ thành phần thì sao?

Ngưỡng đã khóa yêu cầu trung vị speedup ở mỗi cell phải ít nhất:

```text
1,50×
```

Kết quả:

| Cell | Trung vị speedup thành phần |
|---|---:|
| A/W-S | 3,69× |
| A/W-C | 7,79× |
| B/W-C | 6,30× |
| B/W-S | 9,00× |

Không chỉ bốn trung vị PASS.

Toàn bộ:

```text
72 / 72
```

phép đo riêng lẻ đều vượt:

```text
1,50×
```

Phép yếu nhất vẫn đạt khoảng:

```text
2,92×
```

Điều này rất quan trọng.

Ta không còn chỉ biết:

> “Một fixture Q4 có thể chạy nhanh.”

Ta đã biết:

> **Chính cơ chế đó vẫn giữ tính đúng và vẫn tạo speedup lớn khi gặp trọng số và activation thật của model.**

Nhưng Mode T vẫn chưa kết thúc.

## Một component thật vẫn chưa phải toàn bộ model

Ta có thể tưởng tượng chuỗi:

```text
fixture
↓
real weights + real activation
```

đã PASS.

Nhưng còn một bước rất quan trọng:

> Nếu thay 56 node đó trong một lượt inference thật, token model sinh ra có còn giống baseline không?

Bởi một sai lệch số rất nhỏ có thể truyền qua nhiều layer.

Một giá trị logit có thể thay đổi.

Top-1 có thể đổi.

Một token đổi có thể làm toàn bộ chuỗi token sau đó rẽ sang đường khác.

Vì vậy trước khi đo performance toàn hệ, model phải vượt **semantic guard — hàng rào kiểm tra rằng ý nghĩa đầu ra vẫn được giữ**.

Trong phép thử này, baseline và candidate đều sinh:

```text
32 token
```

cho mỗi cặp.

Bốn cặp warmup dùng để kiểm tra semantics đều cho chuỗi token giống nhau.

Sau đó trong 20 cặp measurement:

```text
20 / 20
```

cũng giữ:

```text
candidate token IDs
=
baseline token IDs
```

Logits hữu hạn.

Dispatch census hợp lệ.

Không dùng CPU để thay thế model math.

Tới đây mới có thể nói:

> **Cơ chế không chỉ chạy đúng ở phép toán cục bộ; nó còn giữ được hành vi sinh token của model trong toàn bộ các cặp đã kiểm tra.**

## Bây giờ mới được hỏi: decode có nhanh hơn không?

Đây là tầng tiếp theo của carry-through.

Nếu gate/up nhanh hơn nhưng decode không nhúc nhích, ta sẽ có:

```text
component PASS
↓
system carry-through FAIL
```

Ngưỡng decode đã khóa khá rõ.

Mỗi cell phải có:

```text
candidate latency / baseline latency
<= 0,90
```

tức candidate phải giảm ít nhất 10% decode latency.

Và speedup trung bình hình học trên bốn cell phải ít nhất:

```text
1,25×
```

Kết quả thực tế:

| Cell | Decode speedup |
|---|---:|
| A/W-S | 2,33× |
| A/W-C | 2,04× |
| B/W-C | 2,06× |
| B/W-S | 2,39× |

Trung bình hình học:

```text
≈ 2,20×
```

Không chỉ median.

Cả:

```text
20 / 20
```

cặp đo riêng lẻ đều có decode tốt hơn.

Cặp yếu nhất vẫn khoảng:

```text
1,49×
```

Vậy mũi tên tiếp theo cũng đứng vững:

```text
component speedup
↓
decode speedup
```

## Amdahl dự đoán gần 2× — thực tế khoảng 2,20×

Nhắc lại dự đoán đơn giản ban đầu.

Gate/up chiếm gần:

```text
59,67%
```

và hiệu ứng thành phần cho ta lý do kỳ vọng movement ở cấp decode quanh vùng gần 2×.

Khi dùng số đo thành phần thật ở từng cell, dự đoán Amdahl nằm khoảng:

```text
1,89× → 2,10×
```

Decode thực tế:

```text
2,04× → 2,39×
```

Hai cell thậm chí vượt mô hình đơn giản.

Điều này không có nghĩa Amdahl “sai”.

Amdahl ở đây chỉ là một mô hình thô dựa trên một phân vùng chi phí lịch sử.

Hệ thống thật còn có:

- tương tác giữa các kernel;
- thay đổi thời gian chờ;
- hiệu ứng cache;
- các chi phí không được mô hình hóa hoàn toàn.

Điều quan trọng hơn là:

> **Tín hiệu component không biến mất khi bước vào decode thật.**

Nó carry-through rất rõ.

## Nhưng ta đã từng bị TTFT chặn một lần

Chương 12 đã kể một bài học khó.

Một successor trước đó làm decode và E2E tốt hơn, nhưng TTFT xấu đi quá mức đã khóa.

Vì vậy lần này TTFT phải được giữ như một hàng rào độc lập.

Ngưỡng:

```text
candidate TTFT / baseline TTFT
<= 1,10
```

trong cả bốn cell.

Kết quả median:

```text
A/W-S  0,884
A/W-C  1,068
B/W-C  1,007
B/W-S  1,034
```

Cả bốn đều dưới:

```text
1,10
```

PASS.

Có một số cặp riêng lẻ vượt 1,10.

Cặp tệ nhất khoảng:

```text
1,181
```

Nhưng contract đã khóa từ trước là:

> **đánh giá trên median của từng cell**, không phải bắt mọi cặp riêng lẻ đều dưới 1,10.

Vì vậy không được đổi luật sau khi thấy một sample xấu.

Đây cũng là Mode C đang tiếp tục bảo vệ Mode T.

## Cuối cùng: người dùng nhìn thấy toàn bộ lượt chạy

Decode nhanh hơn là tốt.

Nhưng người dùng không trải nghiệm một con số decode cô lập.

Họ trải nghiệm toàn bộ lượt chạy.

Vì vậy E2E — **thời gian từ đầu đến cuối** — vẫn phải đi đúng hướng.

Kết quả:

| Cell | E2E speedup |
|---|---:|
| A/W-S | 2,31× |
| A/W-C | 1,70× |
| B/W-C | 1,74× |
| B/W-S | 2,35× |

Trung bình hình học của bốn median:

```text
≈ 2,00×
```

Và:

```text
20 / 20
```

cặp riêng lẻ đều cải thiện E2E.

Chuỗi bằng chứng giờ đã dài hơn rất nhiều:

```text
Q4 component fixture
PASS
↓
trọng số thật + activation thật
PASS
↓
chuỗi token toàn model
PASS
↓
decode
~2,20×
↓
TTFT guard
PASS
↓
E2E
~2,00×
```

Đây mới là ý nghĩa của **carry-through**.

## Mode T không phải “đưa vào production”

Từ `Transfer` rất dễ bị hiểu thành:

> “Đã PASS rồi thì triển khai sản phẩm.”

Không phải.

Mode T chỉ nói:

> **Một effect đã xác nhận trong môi trường nhỏ có còn tồn tại khi ta tăng mức độ thực tế của hệ thống hay không?**

Trong chương này, mức độ thực tế tăng từng bước:

```text
fixture nhân tạo

↓

trọng số model thật
activation thật

↓

toàn bộ token semantics

↓

decode toàn model

↓

E2E
```

Mỗi tầng có quyền giết hypothesis.

Không có tầng nào được mặc định PASS chỉ vì tầng trước PASS.

## Vai trò của AI ở Mode T thay đổi một lần nữa

Ở Mode E, AI giúp mở rộng ý tưởng.

Ở Mode M, AI giúp biến một ý tưởng thành mechanism cụ thể.

Ở Mode C, AI giúp triển khai và kiểm tra dưới contract đã khóa.

Đến Mode T, một nguy cơ mới xuất hiện:

> **AI rất dễ “giúp quá mức”.**

Ví dụ thấy component đã PASS, AI có thể đề xuất:

- tối ưu thêm vài kernel khác trước khi tích hợp;
- thay luôn FFN-down;
- thêm fusion;
- chỉnh scheduler;
- làm sạch một vài bottleneck “tiện thể”.

Những thay đổi đó có thể làm runtime nhanh hơn.

Nhưng chúng phá câu hỏi transfer.

Nếu candidate thắng, ta không còn biết phần nào đã carry-through.

Vì vậy vai trò quản trị của con người trong Mode T là giữ nguyên câu hỏi:

> **Mang đúng mechanism đã PASS sang đúng bối cảnh cần kiểm tra. Không thêm cứu trợ.**

AI thực hiện phần nặng:

```text
kiểm tra 56 node mục tiêu
xác minh binding
chạy correctness
so token
thu timing
kiểm tra đủ 20 cặp
tổng hợp gate
```

Con người giữ quyền:

```text
cái gì được thay
cái gì không được thay
gate nào có quyền chặn
kết luận nào đủ bằng chứng
```

Đây là một dạng cộng tác khác với việc chỉ bảo AI:

> “Làm nó nhanh nhất có thể.”

## Vậy Q4 đã thắng chưa?

Câu trả lời cần rất chính xác.

Ta có thể nói:

> **Cơ chế Q4 gate/up subgroup32 Split-K đã chứng minh được carry-through trong ArcLLM trên exact model 7B và các workload đã kiểm tra.**

Ta có thể nói:

```text
decode geomean
≈ 2,20×

E2E geomean
≈ 2,00×
```

so với reference ArcLLM tương ứng.

Nhưng ta **chưa được nói**:

> “ArcLLM bây giờ nhanh hơn llama.cpp.”

Đối chứng llama.cpp ở Q2/Q3 thuộc kiến trúc ArcLLM cũ.

Candidate bây giờ đã thay đổi.

Muốn trả lời câu hỏi bên ngoài:

> **“Khoảng cách với runtime trưởng thành đã đóng được bao nhiêu?”**

ta cần một phép so sánh mới, cùng điều kiện, với candidate mới.

Mode T đã chứng minh:

```text
local mechanism
→ real system value
```

Nó chưa chứng minh:

```text
real system value
→ external advantage
```

Đó là ranh giới dẫn sang chương kế tiếp.

### Nhớ 3 điều

1. **Mode T kiểm tra sự sống sót của bằng chứng.** Một component PASS phải lần lượt sống sót qua trọng số thật, activation thật, semantics của model, decode và E2E trước khi được gọi là carry-through.
2. **Transfer phải giữ phạm vi hẹp.** Trong phép thử này chỉ 56 gate/up Q4_K node của decode được thay. Nếu đồng thời sửa nhiều phần khác, ta sẽ mất khả năng biết cơ chế nào tạo ra kết quả.
3. **I002 tạo ra một cải thiện ArcLLM nội bộ có ý nghĩa: khoảng `2,20×` decode và `2,00×` E2E, đồng thời giữ TTFT guard và token semantics.** Nhưng đây vẫn chưa phải bằng chứng ArcLLM thắng llama.cpp; đối chứng bên ngoài phải được đo lại với kiến trúc mới.

**Chương 15 — Một kiến trúc chỉ thắng khi toàn hệ được lợi**

Ta đã đi hết một vòng:

```text
E
ý tưởng

↓
M
cơ chế

↓
C
xác nhận

↓
T
carry-through
```

Q4 đã sống sót qua cả bốn.

Nhưng một câu hỏi cuối của Phần III vẫn còn:

> **Một cải thiện nội bộ rất lớn có thực sự thay đổi vị trí của cả runtime trước thế giới bên ngoài — và khi bằng chứng trả lời, con người phải quyết định dừng hay tiếp tục như thế nào?**
