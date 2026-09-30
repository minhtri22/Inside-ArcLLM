# Chương 10 — Tự xây được vẫn chưa có nghĩa là tốt hơn

> **Mức đọc: Nghiên cứu**
>
> **Bạn đang ở bước nào của hành trình nghiên cứu?**
>
> ```text
> Kết quả đối chứng
>         ↓
> ArcLLM còn cách xa
>         ↓
> [ kiểm tra lại bằng bằng chứng mới ]
>         ↓
> chấp nhận kết quả dù không có lợi
> ```


> **Câu hỏi của chương:** Sau khi phép đo đối chứng cùng điều kiện cho thấy một khoảng cách rất lớn, làm thế nào kiểm tra nghiêm túc xem kiến trúc ArcLLM hiện tại có còn một lợi thế thực tế nào đủ lớn và tái lập được hay không?

Chương 9 kết thúc ở một điểm hơi khó chịu.

ArcLLM đã chạy được mô hình 7B thật.

Nó đã có:

```text
28 lớp xử lý thật
↓
bộ nhớ đệm KV giữ trên GPU
↓
sinh token nối tiếp
↓
các chương trình GPU của đường chạy thật
↓
phép đo đối chứng cùng điều kiện với llama.cpp
```

Nhưng bảng Q2 không đẹp.

Ở hai bài đo đã khóa, llama.cpp có TTFT thấp hơn rất nhiều, giai đoạn sinh token thông lượng cao hơn rất nhiều và working set cũng thấp hơn.

Một phản xạ rất tự nhiên lúc này là:

> “Tối ưu thêm đi.”

Có thể gộp thêm các chương trình GPU.

Đổi khối xử lý.

Đổi cách lập lịch.

Tìm bài đo khác.

Thử ngữ cảnh khác.

Hoặc nhìn vào một chỉ số nhỏ đang có lợi rồi nói:

> “Ít nhất ArcLLM vẫn thắng ở điểm này.”

Nhưng nếu làm vậy, ta không còn kiểm tra kiến trúc nữa.

Ta đang **bảo vệ kiến trúc**.

Q3 được mở để ngăn chính điều đó.

## Câu hỏi không còn là “có chỗ nào thắng không?”

Q2 đã hoàn thành vai trò của nó:

> **matched mô tả đặc tính — mô tả ArcLLM và mốc đối chứng trên cùng một mặt phẳng đo.**

Q3 có vai trò khác.

Nó đặt một giả thuyết có thể bị bác bỏ.

Giả thuyết được gọi là:

**H-NPA — giả thuyết “chưa chứng minh được lợi thế thực tế trong phạm vi đã khóa”.**

Đọc bằng tiếng Việt:

> **Trong hai bài đo W-S và W-C đã khóa, kiến trúc ArcLLM hiện tại chưa chứng minh được một lợi thế đủ có ý nghĩa thực tế và tái lập qua hai phiên chạy mới.**

Cụm **“trong phạm vi đã khóa”** rất quan trọng.

Q3 không nói:

> “ArcLLM không bao giờ có thể có lợi thế.”

Nó chỉ hỏi:

> **Với chính kiến trúc hiện tại, chính hardware này, chính mô hình này và hai regime đã khóa, có advantage thực tế nào vượt qua tiêu chuẩn đã khóa hay không?**

Đó là một **kết luận hẹp hơn**.

Nhưng kiểm chứng được.

## Muốn bác bỏ giả thuyết âm tính thì phải làm gì?

H-NPA không thể bị bác bỏ chỉ vì một chỉ số nào đó đẹp lên ở một lần chạy.

Q3 đặt luật:

> Phải có **cùng một bài đo + cùng một primary benefit dimension** vượt qua practical-advantage gate trong **cả hai fresh phiên đo**.

Tách câu này ra.

**Primary benefit dimension — chiều lợi ích chính** là một trong bốn chỉ số được phép tạo advantage:

```text
TTFT
thông lượng sinh token
độ trễ toàn lượt
peak working set
```

Nếu W-S thắng về TTFT ở phiên đo A nhưng sang phiên đo B lại chỉ thắng về bộ nhớ, điều đó chưa đủ.

Nếu W-S thắng TTFT ở phiên đo A nhưng không lặp lại ở phiên đo B, cũng chưa đủ.

Một kết luận phải tái lập đúng nơi nó tuyên bố tồn tại.

## “Lợi thế thực tế” phải được định nghĩa trước

Không phải mọi thay đổi 1% đều nên được gọi là advantage có ý nghĩa thực tế.

Q3 vì vậy khóa threshold trước thực thi.

Đối với TTFT:

```text
Arc / đối chứng <= 0,90
```

nghĩa là ArcLLM phải có TTFT thấp hơn ít nhất 10%.

Ví dụ:

```text
TTFT đối chứng = 100 ms
```

Muốn ĐẠT (PASS) benefit gate:

```text
ArcLLM TTFT <= 90 ms
```

vì:

```text
90 / 100 = 0,90
```

Với giai đoạn sinh token thông lượng, hướng tốt lại ngược lại:

```text
Arc / đối chứng >= 1,10
```

Ví dụ:

```text
đối chứng = 10 token/s
```

ArcLLM phải đạt ít nhất:

```text
11 token/s
```

vì:

```text
11 / 10 = 1,10
```

E2E cũng giống TTFT:

```text
Arc / đối chứng <= 0,90
```

Còn peak working set có threshold mạnh hơn:

```text
Arc / đối chứng <= 0,85
```

tức thấp hơn ít nhất 15%.

Ví dụ mốc đối chứng dùng:

```text
5 GB
```

thì ArcLLM phải xuống tối đa khoảng:

```text
5 × 0,85
= 4,25 GB
```

mới vượt benefit gate về working set.

Những threshold này là **tiêu chuẩn đã khóa của Q3**.

Chúng không được tuyên bố là ngưỡng phổ quát cho mọi hệ thực thi hay mọi ứng dụng.

Điều quan trọng là chúng được khóa **trước khi nhìn outcome Q3**.

## Tốt ở một chỉ số nhưng làm hỏng ba chỉ số khác thì sao?

Đây là nơi Q3 thêm một lớp bảo vệ rất quan trọng:

**blocking-harm guard — hàng rào ngăn một lợi ích nhỏ được gọi là advantage khi nó phải trả giá quá lớn ở những chỉ số chính khác.**

Giả sử một hệ thực thi giảm bộ nhớ 20%.

Nghe rất tốt.

Nhưng đồng thời:

```text
TTFT chậm gấp 2
sinh token chỉ còn một nửa
E2E chậm gấp 3
```

Có nên gọi nó là “practical advantage” chỉ vì bộ nhớ tốt hơn?

Q3 nói: không.

Ngoài việc phải thắng ít nhất một primary dimension, **tất cả** các chỉ số chính khác phải không xấu hơn mốc đối chứng quá 10%.

Guard được khóa:

```text
TTFT:
Arc / đối chứng <= 1,10

thông lượng sinh token:
Arc / đối chứng >= 0,90

E2E:
Arc / đối chứng <= 1,10

working set:
Arc / đối chứng <= 1,10
```

Ví dụ một phương án thử có:

```text
working set
= 0,80× đối chứng
```

→ lợi hơn 20%, vượt benefit threshold 15%.

Nhưng nếu:

```text
TTFT
= 1,50× đối chứng
```

thì blocking-harm guard KHÔNG ĐẠT (FAIL).

Không được gọi là practical advantage.

Đây là một nguyên tắc rất hữu ích:

> **Không được dùng một điểm sáng nhỏ để che một cái giá lớn ở phần còn lại của hệ thống.**

## Dùng ít bộ nhớ riêng hơn vẫn chưa đủ

Trong Q2 có một con số nhìn qua khá hấp dẫn.

Private bytes của ArcLLM thấp hơn mốc đối chứng khoảng 3%.

Nếu đang cố “tìm điểm thắng”, đây là chỗ rất dễ bám vào.

Q3 khóa từ trước:

```text
private bytes
CPU utilization
GPU counters
```

chỉ là **supporting các chỉ số — chỉ số hỗ trợ**.

Chúng vẫn được ghi.

Nhưng không được tự mình tạo kết luận advantage.

Lý do khoa học ở đây không phải vì chúng vô giá trị.

Mà vì tiêu chuẩn đã khóa đã xác định trước bốn primary dimension được dùng để adjudicate:

```text
TTFT
thông lượng sinh token
E2E
working set
```

Sau outcome, không được đổi luật và đưa một supporting chỉ số lên thành primary chỉ vì nó thuận lợi.

## Không được sửa ArcLLM trước Q3

Một điểm còn mạnh hơn nữa:

> **Kiến trúc ArcLLM bị đóng băng.**

Trước Q3 adjudication, bị cấm:

```text
tinh chỉnh chương trình GPU
tinh chỉnh cách lập lịch
architecture change
tìm bài đo
đổi mốc đối chứng
đổi threshold
```

Tức Q3 không hỏi:

> “Nếu ta tiếp tục tối ưu, ArcLLM có thể thắng không?”

Nó hỏi:

> **“Kiến trúc mà Q2 vừa đo có thật sự chứa một advantage tái lập hay không?”**

Đây là một confirmatory study — **nghiên cứu xác nhận**.

Không còn tuning theo outcome.

## Vì sao cần bằng chứng mới?

Q2 đã cho ta bảng dữ liệu.

Tại sao không dùng luôn bảng đó để adjudicate Q3?

Bởi threshold Q3 được thiết kế sau khi Q2 đã cho thấy bề mặt hiệu năng.

Nếu lại dùng chính Q2 để xác nhận kết luận, ta sẽ vừa dùng **bằng chứng** để hình thành câu hỏi, vừa dùng chính bằng chứng đó để tự trả lời.

Q3 vì vậy yêu cầu:

**fresh reproduction — bằng chứng mới được tạo sau khi hypothesis và threshold đã khóa.**

Có hai phiên đo độc lập:

```text
Phiên đo A
Phiên đo B
```

Mỗi phiên đo chạy:

```text
2 systems
×
2 workloads
×
5 measured attempts
```

Tức:

```text
2 × 2 × 5
= 20 lượt thử / phiên
```

Hai phiên đo:

```text
20 × 2
= 40 fresh measured attempts
```

Ngoài ra mỗi cell vẫn có warmup trước measurement.

## Hai phiên đo còn đảo thứ tự chạy

Phiên đo A:

```text
ArcLLM W-S
↓
llama.cpp W-S
↓
llama.cpp W-C
↓
ArcLLM W-C
```

Phiên đo B đảo lại:

```text
llama.cpp W-S
↓
ArcLLM W-S
↓
ArcLLM W-C
↓
llama.cpp W-C
```

Đây gọi là **counterbalancing — đổi thứ tự giữa các phiên để giảm nguy cơ thứ tự chạy tự tạo ra lợi thế hệ thống**.

Hình dung nếu GPU nóng dần theo thời gian.

Nếu ArcLLM luôn chạy trước và llama.cpp luôn chạy sau, thứ tự có thể bị trộn với hiệu năng.

Đảo thứ tự ở phiên đo thứ hai không loại được mọi loại nhiễu.

Nhưng nó giúp tránh một bias quá hiển nhiên.

## Môi trường cũng phải giữ cùng điều kiện

Hai fresh phiên đo vẫn khóa:

```text
cùng mô hình
cùng mốc đối chứng
cùng trình điều khiển GPU
cùng hardware
cùng power scheme
AC power
cùng workloads
```

Phiên đo A và phiên đo B là hai process/run độc lập.

Không phải cùng một process chạy hai vòng rồi gọi là reproduction.

Q3 muốn kiểm tra:

> kết quả có sống sót qua một lần khởi động thực thi mới hay không?

## Và cả 40 lần đều chạy thành công

Q3 hoàn tất:

```text
40 / 40 measured attempts
→ SUCCESS
```

Không crash.

Không measurement-invalid.

Không thiếu cell.

Mỗi attempt có:

```text
32 generated tokens
điểm dự đoán hữu hạn
timing hợp lệ
resource traces
```

Vì vậy nếu outcome âm tính, không thể nói:

> “Có lẽ experiment chưa chạy được nên chưa biết.”

Evidence đủ để adjudicate.

Đây là lý do kết luận sau cùng **không phải UNRESOLVED**.

## phiên đo A nói gì?

Ở W-S, tỷ lệ ArcLLM so với mốc đối chứng là:

```text
TTFT
= 12,422×

sinh token
= 0,02473×

E2E
= 39,209×

working set
= 1,836×
```

Nhắc lại cách đọc.

Với độ trễ:

```text
> 1
```

nghĩa là ArcLLM chậm hơn.

Với thông lượng:

```text
< 1
```

nghĩa là ArcLLM sinh token chậm hơn.

Với working set:

```text
> 1
```

nghĩa là ArcLLM dùng working-set bộ nhớ cao hơn.

Không primary benefit nào gần threshold.

W-C phiên đo A:

```text
TTFT
= 9,198×

sinh token
= 0,02172×

E2E
= 31,355×

working set
= 1,829×
```

Cũng không có benefit phương án thử.

Blocking-harm guard KHÔNG ĐẠT (FAIL).

## phiên đo B có đảo kết luận không?

Phiên đo B là fresh reproduction.

W-S:

```text
TTFT
= 14,801×

sinh token
= 0,01761×

E2E
= 55,009×

working set
= 1,836×
```

W-C:

```text
TTFT
= 9,605×

sinh token
= 0,02800×

E2E
= 26,451×

working set
= 1,829×
```

Một lần nữa:

```text
không primary benefit PASS
```

và:

```text
blocking-harm guard FAIL
```

ở cả hai bài đo.

Không có cùng bài đo + cùng primary dimension nào có thể reproduce advantage, bởi thậm chí **không có primary benefit nào ĐẠT (PASS) ngay trong một phiên đo**.

## Bộ nhớ riêng và CPU vẫn có tín hiệu

Một chi tiết đáng chú ý vẫn tái lập.

Private bytes của ArcLLM thấp hơn mốc đối chứng khoảng 3%.

CPU utilization được ghi nhận thấp hơn đáng kể.

Q3 không giấu các kết quả đó.

Nhưng tiêu chuẩn đã khóa nói rõ:

> chúng không được tự mình tạo regime advantage.

Đây là sự khác biệt giữa:

```text
observation
```

và:

```text
verdict
```

Một observation vẫn có thể đáng nhớ.

Nó thậm chí có thể gợi ý một câu hỏi nghiên cứu trong tương lai.

Nhưng nó không được thay đổi luật adjudication hiện tại.

## H-NPA không bị bác bỏ

Nhắc lại:

H-NPA nói rằng kiến trúc hiện tại **không chứng minh được practical advantage tái lập** trong hai regime đã khóa.

Muốn bác bỏ nó, ta cần:

```text
cùng workload
+
cùng primary benefit
+
PASS threshold
+
PASS blocking-harm guard
+
Phiên đo A
+
Phiên đo B
```

Không điều kiện nào như vậy xuất hiện.

Vì thế:

> **H-NPA không bị falsify — không bị bằng chứng bác bỏ.**

Q3 đi tới kết luận đã khóa từ trước:

> **FEASIBLE_NO_DEMONSTRATED_ADVANTAGE**

Ta nên dịch rất cẩn thận:

> **Đã chứng minh khả năng chạy, nhưng chưa chứng minh được lợi thế thực tế trong các regime được kiểm tra.**

Không phải:

> “ArcLLM vô dụng.”

Không phải:

> “Kiến trúc này không thể bao giờ tốt hơn.”

Không phải:

> “llama.cpp luôn thắng trong mọi tình huống.”

Chỉ là:

> **Với mô hình, hardware, các bài đo, mốc đối chứng và architecture đã khóa, evidence không hỗ trợ một practical regime advantage.**

Đó là phạm vi hợp lệ của kết luận.

## Đây là một kết quả âm tính có giá trị

Nhìn lại từ Chương 1.

Ta đã tự xây:

```text
GGUF reader
↓
tensor store
↓
Vulkan core
↓
kernels
↓
sinh tokenr layer
↓
full sinh tokenr
↓
KV cache
↓
generation
↓
đường chạy thực tế
↓
thực thi mô hình 7B
↓
phép đo đối chứng cùng điều kiện
```

Rất nhiều thứ ĐẠT (PASS).

Thật dễ để tất cả ĐẠT (PASS) trước đó tạo ra một loại attachment:

> “Đã đi xa thế này rồi, nhất định kiến trúc phải có lợi thế.”

Science không cho phép suy luận đó.

**Feasibility — chạy được** không đồng nghĩa với:

**advantage — tốt hơn ở một regime có ý nghĩa.**

Một chiếc máy có thể được xây thành công.

Nó có thể chạy đúng.

Nó có thể rất thú vị về mặt kỹ thuật.

Và matched evidence vẫn có thể nói:

> hiện tại chưa có lợi thế thực tế được chứng minh.

Hai điều đó không mâu thuẫn.

## Quy tắc dừng mới là phần khó nhất

Q3 đã khóa trước rằng nếu kết luận là:

```text
FEASIBLE_NO_DEMONSTRATED_ADVANTAGE
```

thì:

> **đóng current ArcLLM architecture line.**

Không có:

```text
Q3-A
Q3-B
Q3-C
```

để tiếp tục thử cho tới khi thắng.

Không search bài đo khác.

Không hạ threshold.

Không đổi mốc đối chứng.

Không dùng một project khác để “cứu” outcome.

Đây là điểm hội tụ thực sự.

Có thể hình dung:

```text
Q1
chạy được?
↓
YES

Q2
đo matched được?
↓
YES

Q3
advantage có reproduce?
↓
NO

→ CLOSE CURRENT ARCHITECTURE LINE
```

Dừng ở đây không phải thất bại của nghiên cứu.

Không dừng mới là nguy hiểm.

## Nhưng câu chuyện ArcLLM chưa kết thúc

Đóng **current architecture line** không có nghĩa cấm đặt câu hỏi mới mãi mãi.

Nó chỉ cấm:

> **vá tiếp cùng kiến trúc để cố đảo kết luận Q3.**

Một architecture kế tiếp chỉ có thể được mở nếu có một **mechanism mới đủ độc lập và có lý do causal cụ thể**.

Đó là ranh giới dẫn tới Chương 11.

Thay vì hỏi:

> “Tối ưu tiếp chỗ nào?”

ta sẽ hỏi một câu mạnh hơn:

> **“Q2/Q3 đã cho ta biết cấu trúc hiện tại thua ở đâu; có một mechanism hoàn toàn khác nào đáng để mở một successor architecture study hay không?”**

Đây là sự chuyển đổi từ:

```text
optimization
```

sang:

```text
architecture intervention
```

Và lần này, một ý tưởng mới sẽ không được phép bước vào chỉ vì nó nghe hay.

Nó phải giải thích được:

```text
nút thắt nào
↓
mechanism nào
↓
vì sao cơ chế đó có thể thay đổi nút thắt
↓
điều gì sẽ giết hypothesis
```

### Nhớ 3 điều

1. **Q3 khóa practical advantage trước bằng chứng mới.** hiệu năng cần ít nhất 10% benefit, working set cần ít nhất 15%, và không được trả giá quá 10% ở các primary dimension khác.
2. **40/40 fresh attempts hoàn tất nhưng không có primary benefit nào ĐẠT (PASS).** Hai phiên đo độc lập đều không bác bỏ H-NPA; private bytes và CPU utilization chỉ là supporting observations.
3. **kết luận là `FEASIBLE_NO_DEMONSTRATED_ADVANTAGE`, không phải “ArcLLM không chạy được”.** Feasibility đã được chứng minh; thứ không được chứng minh là practical regime advantage của kiến trúc hiện tại. Vì vậy current architecture line phải đóng thay vì tuning vô hạn.

**Chương 11 — Từ thất bại sang một câu hỏi đúng hơn**

Q3 không cho ArcLLM một chiến thắng hiệu năng.

Nhưng nó cho thứ có giá trị hơn cho bước tiếp theo:

> **một ranh giới rõ ràng để biết kiến trúc cũ phải dừng ở đâu — và một kiến trúc kế tiếp chỉ được sinh ra khi có một mechanism mới đủ mạnh để biện minh cho nó.**
