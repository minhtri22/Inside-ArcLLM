# Chương 10 — Khi “tự build được” vẫn chưa đủ

> **Câu hỏi của chương:** Sau khi matched benchmark cho thấy một khoảng cách rất lớn, làm thế nào kiểm tra nghiêm túc xem kiến trúc ArcLLM hiện tại có còn một lợi thế thực tế nào đủ lớn và tái lập được hay không?

Chương 9 kết thúc ở một điểm hơi khó chịu.

ArcLLM đã chạy được model 7B thật.

Nó đã có:

```text
28 layer thật
↓
GPU-resident KV
↓
autoregressive generation
↓
production kernels
↓
matched benchmark với llama.cpp
```

Nhưng bảng Q2 không đẹp.

Ở hai workload đã khóa, llama.cpp có TTFT thấp hơn rất nhiều, decode throughput cao hơn rất nhiều và working set cũng thấp hơn.

Một phản xạ rất tự nhiên lúc này là:

> “Tối ưu thêm đi.”

Có thể fuse thêm kernel.

Đổi tile.

Đổi scheduler.

Tìm workload khác.

Thử context khác.

Hoặc nhìn vào một metric nhỏ đang có lợi rồi nói:

> “Ít nhất ArcLLM vẫn thắng ở điểm này.”

Nhưng nếu làm vậy, ta không còn kiểm tra kiến trúc nữa.

Ta đang **bảo vệ kiến trúc**.

Q3 được mở để ngăn chính điều đó.

## Câu hỏi không còn là “có chỗ nào thắng không?”

Q2 đã hoàn thành vai trò của nó:

> **matched characterization — mô tả ArcLLM và baseline trên cùng một mặt phẳng đo.**

Q3 có vai trò khác.

Nó đặt một giả thuyết có thể bị bác bỏ.

Giả thuyết được gọi là:

**H-NPA — bounded no-practical-advantage hypothesis.**

Đọc bằng tiếng Việt:

> **Trong hai workload W-S và W-C đã khóa, kiến trúc ArcLLM hiện tại chưa chứng minh được một lợi thế đủ có ý nghĩa thực tế và tái lập qua hai phiên chạy mới.**

Từ **bounded — có phạm vi giới hạn** rất quan trọng.

Q3 không nói:

> “ArcLLM không bao giờ có thể có lợi thế.”

Nó chỉ hỏi:

> **Với chính kiến trúc hiện tại, chính hardware này, chính model này và hai regime đã khóa, có advantage thực tế nào vượt qua contract hay không?**

Đó là một claim nhỏ hơn.

Nhưng kiểm chứng được.

## Muốn bác bỏ giả thuyết âm tính thì phải làm gì?

H-NPA không thể bị bác bỏ chỉ vì một metric nào đó đẹp lên ở một lần chạy.

Q3 đặt luật:

> Phải có **cùng một workload + cùng một primary benefit dimension** vượt qua practical-advantage gate trong **cả hai fresh session**.

Tách câu này ra.

**Primary benefit dimension — chiều lợi ích chính** là một trong bốn metric được phép tạo advantage:

```text
TTFT
decode throughput
E2E latency
peak working set
```

Nếu W-S thắng về TTFT ở Session A nhưng sang Session B lại chỉ thắng về memory, điều đó chưa đủ.

Nếu W-S thắng TTFT ở Session A nhưng không lặp lại ở Session B, cũng chưa đủ.

Claim phải tái lập đúng nơi nó tuyên bố tồn tại.

## “Practical advantage” phải được định nghĩa trước

Không phải mọi thay đổi 1% đều nên được gọi là advantage có ý nghĩa thực tế.

Q3 vì vậy khóa threshold trước execution.

Đối với TTFT:

```text
Arc / baseline <= 0,90
```

nghĩa là ArcLLM phải có TTFT thấp hơn ít nhất 10%.

Ví dụ:

```text
baseline TTFT = 100 ms
```

Muốn PASS benefit gate:

```text
ArcLLM TTFT <= 90 ms
```

vì:

```text
90 / 100 = 0,90
```

Với decode throughput, hướng tốt lại ngược lại:

```text
Arc / baseline >= 1,10
```

Ví dụ:

```text
baseline = 10 token/s
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
Arc / baseline <= 0,90
```

Còn peak working set có threshold mạnh hơn:

```text
Arc / baseline <= 0,85
```

tức thấp hơn ít nhất 15%.

Ví dụ baseline dùng:

```text
5 GB
```

thì ArcLLM phải xuống tối đa khoảng:

```text
5 × 0,85
= 4,25 GB
```

mới vượt benefit gate về working set.

Những threshold này là **contract của Q3**.

Chúng không được tuyên bố là ngưỡng phổ quát cho mọi runtime hay mọi ứng dụng.

Điều quan trọng là chúng được khóa **trước khi nhìn outcome Q3**.

## Thắng một metric nhưng phá ba metric khác thì sao?

Đây là nơi Q3 thêm một lớp bảo vệ rất quan trọng:

**blocking-harm guard — hàng rào ngăn một lợi ích nhỏ được gọi là advantage khi nó phải trả giá quá lớn ở những metric chính khác.**

Giả sử một runtime giảm memory 20%.

Nghe rất tốt.

Nhưng đồng thời:

```text
TTFT chậm gấp 2
decode chỉ còn một nửa
E2E chậm gấp 3
```

Có nên gọi nó là “practical advantage” chỉ vì memory tốt hơn?

Q3 nói: không.

Ngoài việc phải thắng ít nhất một primary dimension, **tất cả** các metric chính khác phải không xấu hơn baseline quá 10%.

Guard được khóa:

```text
TTFT:
Arc / baseline <= 1,10

decode throughput:
Arc / baseline >= 0,90

E2E:
Arc / baseline <= 1,10

working set:
Arc / baseline <= 1,10
```

Ví dụ một candidate có:

```text
working set
= 0,80× baseline
```

→ lợi hơn 20%, vượt benefit threshold 15%.

Nhưng nếu:

```text
TTFT
= 1,50× baseline
```

thì blocking-harm guard FAIL.

Không được gọi là practical advantage.

Đây là một nguyên tắc rất hữu ích:

> **Không được dùng một điểm sáng nhỏ để che một cái giá lớn ở phần còn lại của hệ thống.**

## Private bytes thấp hơn không đủ

Trong Q2 có một con số nhìn qua khá hấp dẫn.

Private bytes của ArcLLM thấp hơn baseline khoảng 3%.

Nếu đang cố “tìm điểm thắng”, đây là chỗ rất dễ bám vào.

Q3 khóa từ trước:

```text
private bytes
CPU utilization
GPU counters
```

chỉ là **supporting metrics — metric hỗ trợ**.

Chúng vẫn được ghi.

Nhưng không được tự mình tạo verdict advantage.

Lý do khoa học ở đây không phải vì chúng vô giá trị.

Mà vì contract đã xác định trước bốn primary dimension được dùng để adjudicate:

```text
TTFT
decode throughput
E2E
working set
```

Sau outcome, không được đổi luật và đưa một supporting metric lên thành primary chỉ vì nó thuận lợi.

## Không được sửa ArcLLM trước Q3

Một điểm còn mạnh hơn nữa:

> **Kiến trúc ArcLLM bị đóng băng.**

Trước Q3 adjudication, bị cấm:

```text
kernel tuning
scheduler tuning
architecture change
workload search
đổi baseline
đổi threshold
```

Tức Q3 không hỏi:

> “Nếu ta tiếp tục tối ưu, ArcLLM có thể thắng không?”

Nó hỏi:

> **“Kiến trúc mà Q2 vừa đo có thật sự chứa một advantage tái lập hay không?”**

Đây là một confirmatory study — **nghiên cứu xác nhận**.

Không còn tuning theo outcome.

## Vì sao cần fresh evidence?

Q2 đã cho ta bảng dữ liệu.

Tại sao không dùng luôn bảng đó để adjudicate Q3?

Bởi threshold Q3 được thiết kế sau khi Q2 đã cho thấy bề mặt performance.

Nếu lại dùng chính Q2 để xác nhận claim, ta sẽ vừa dùng evidence để hình thành câu hỏi, vừa dùng cùng evidence đó để tự trả lời.

Q3 vì vậy yêu cầu:

**fresh reproduction — bằng chứng mới được tạo sau khi hypothesis và threshold đã khóa.**

Có hai session độc lập:

```text
Session A
Session B
```

Mỗi session chạy:

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
= 20 attempts/session
```

Hai session:

```text
20 × 2
= 40 fresh measured attempts
```

Ngoài ra mỗi cell vẫn có warmup trước measurement.

## Hai session còn đảo thứ tự chạy

Session A:

```text
ArcLLM W-S
↓
llama.cpp W-S
↓
llama.cpp W-C
↓
ArcLLM W-C
```

Session B đảo lại:

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

Nếu ArcLLM luôn chạy trước và llama.cpp luôn chạy sau, thứ tự có thể bị trộn với performance.

Đảo thứ tự ở session thứ hai không loại được mọi loại nhiễu.

Nhưng nó giúp tránh một bias quá hiển nhiên.

## Environment cũng phải giữ matched

Hai fresh session vẫn khóa:

```text
cùng model
cùng baseline
cùng GPU driver
cùng hardware
cùng power scheme
AC power
cùng workloads
```

Session A và Session B là hai process/run độc lập.

Không phải cùng một process chạy hai vòng rồi gọi là reproduction.

Q3 muốn kiểm tra:

> kết quả có sống sót qua một lần khởi động execution mới hay không?

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
finite logits
timing hợp lệ
resource traces
```

Vì vậy nếu outcome âm tính, không thể nói:

> “Có lẽ experiment chưa chạy được nên chưa biết.”

Evidence đủ để adjudicate.

Đây là lý do verdict sau cùng **không phải UNRESOLVED**.

## Session A nói gì?

Ở W-S, tỷ lệ ArcLLM so với baseline là:

```text
TTFT
= 12,422×

decode
= 0,02473×

E2E
= 39,209×

working set
= 1,836×
```

Nhắc lại cách đọc.

Với latency:

```text
> 1
```

nghĩa là ArcLLM chậm hơn.

Với throughput:

```text
< 1
```

nghĩa là ArcLLM sinh token chậm hơn.

Với working set:

```text
> 1
```

nghĩa là ArcLLM dùng working-set memory cao hơn.

Không primary benefit nào gần threshold.

W-C Session A:

```text
TTFT
= 9,198×

decode
= 0,02172×

E2E
= 31,355×

working set
= 1,829×
```

Cũng không có benefit candidate.

Blocking-harm guard FAIL.

## Session B có đảo kết luận không?

Session B là fresh reproduction.

W-S:

```text
TTFT
= 14,801×

decode
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

decode
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

ở cả hai workload.

Không có cùng workload + cùng primary dimension nào có thể reproduce advantage, bởi thậm chí **không có primary benefit nào PASS ngay trong một session**.

## Private bytes và CPU vẫn có signal

Một chi tiết đáng chú ý vẫn tái lập.

Private bytes của ArcLLM thấp hơn baseline khoảng 3%.

CPU utilization được ghi nhận thấp hơn đáng kể.

Q3 không giấu các kết quả đó.

Nhưng contract nói rõ:

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
Session A
+
Session B
```

Không điều kiện nào như vậy xuất hiện.

Vì thế:

> **H-NPA không bị falsify — không bị bằng chứng bác bỏ.**

Q3 đi tới verdict đã khóa từ trước:

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

> **Với model, hardware, workloads, baseline và architecture đã khóa, evidence không hỗ trợ một practical regime advantage.**

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
decoder layer
↓
full decoder
↓
KV cache
↓
generation
↓
production path
↓
7B execution
↓
matched benchmark
```

Rất nhiều thứ PASS.

Thật dễ để tất cả PASS trước đó tạo ra một loại attachment:

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

## Stop rule mới là phần khó nhất

Q3 đã khóa trước rằng nếu verdict là:

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

Không search workload khác.

Không hạ threshold.

Không đổi baseline.

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

> **vá tiếp cùng kiến trúc để cố đảo verdict Q3.**

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
bottleneck nào
↓
mechanism nào
↓
vì sao mechanism đó có thể thay đổi bottleneck
↓
điều gì sẽ giết hypothesis
```

### Nhớ 3 điều

1. **Q3 khóa practical advantage trước fresh evidence.** Performance cần ít nhất 10% benefit, working set cần ít nhất 15%, và không được trả giá quá 10% ở các primary dimension khác.
2. **40/40 fresh attempts hoàn tất nhưng không có primary benefit nào PASS.** Hai session độc lập đều không bác bỏ H-NPA; private bytes và CPU utilization chỉ là supporting observations.
3. **Verdict là `FEASIBLE_NO_DEMONSTRATED_ADVANTAGE`, không phải “ArcLLM không chạy được”.** Feasibility đã được chứng minh; thứ không được chứng minh là practical regime advantage của kiến trúc hiện tại. Vì vậy current architecture line phải đóng thay vì tuning vô hạn.

**Chương 11 — Từ thất bại sang một câu hỏi đúng hơn**

Q3 không cho ArcLLM một chiến thắng performance.

Nhưng nó cho thứ có giá trị hơn cho bước tiếp theo:

> **một ranh giới rõ ràng để biết kiến trúc cũ phải dừng ở đâu — và một kiến trúc kế tiếp chỉ được sinh ra khi có một mechanism mới đủ mạnh để biện minh cho nó.**
