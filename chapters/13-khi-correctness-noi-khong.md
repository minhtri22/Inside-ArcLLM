# Chương 13 — Khi correctness nói “không”

> **Mức đọc: Nghiên cứu**
>
> **Bản đồ xuyên suốt — đang mở: correctness gate / Confirm**
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
> ▶ **Đang mở ở chương này:** correctness gate / Confirm.


> **Câu hỏi của chương:** Nếu một cơ chế đã chứng minh rằng nó có thể chạy nhanh hơn, nhưng khi áp dụng sang một loại trọng số khác nó không còn giữ được kết quả đúng, ta nên sửa tiếp hay phải dừng?

Chương 12 kết thúc với một kết quả rất hấp dẫn.

Một cơ chế Split-K dùng 32 lane trong cùng subgroup đã làm các phép nhân Q4_K thành phần nhanh hơn khoảng:

```text
3,14×
```

ở mức tổng hợp.

Các shape riêng lẻ đều vượt ngưỡng đã khóa.

Tính đúng cũng PASS.

Đây chính xác là loại kết quả rất dễ tạo ra một suy luận:

> “Cơ chế này tốt. Hãy áp dụng nó cho các phép nhân lượng tử hóa khác.”

Trong model thật, không phải mọi trọng số đều có cùng định dạng.

Ngoài Q4_K còn có Q6_K.

Câu hỏi tiếp theo vì vậy rất tự nhiên:

> **Cùng một cách chia K cho 32 lane, giữ nguyên cơ chế đã thắng ở Q4_K, có tiếp tục hoạt động với Q6_K hay không?**

Đây là lúc **Mode C — Confirm, chế độ xác nhận** trở thành nhân vật chính.

## Mode C không hỏi “ta có thể làm nó chạy không?”

Mode C hỏi một câu khó hơn:

> **Một cơ chế đã được định nghĩa trước có vượt qua contract đã khóa trước hay không?**

Điểm khác biệt rất lớn nằm ở hai chữ:

> **đã khóa.**

Trước khi nhìn outcome Q6, luật đã được quyết định.

Cơ chế Split-K không được phép đổi.

Geometry — **cách bố trí công việc trên GPU** — không được phép đổi.

Subgroup vẫn:

```text
32 lane
```

Workgroup vẫn:

```text
128 invocation
```

Một subgroup vẫn phụ trách:

```text
1 output row
```

K vẫn được chia theo bước:

```text
32
```

Cách cộng kết quả từng phần vẫn dùng cùng loại subgroup reduction.

Không được thấy kết quả xấu rồi đổi ngay sang subgroup 16.

Không được thử geometry khác.

Không được gộp kernel thêm.

Không được chuyển sang cooperative matrix.

Nếu làm những việc đó, ta không còn trả lời câu hỏi ban đầu nữa.

Ta đã tạo ra một hypothesis mới.

## Q4_K và Q6_K giống nhau ở đâu — và khác nhau ở đâu?

Cả Q4_K và Q6_K đều là những cách lưu trọng số đã được lượng tử hóa và đóng gói.

**Quantization — lượng tử hóa** có thể hiểu là:

> thay vì lưu mọi trọng số bằng một số thực lớn như F32, ta biểu diễn chúng bằng ít bit hơn cùng một số thông tin scale cần thiết để tái tạo giá trị gần đúng khi tính toán.

Q4_K dùng ít bit hơn cho giá trị lượng tử hóa.

Q6_K dùng nhiều bit hơn và có cách đóng gói metadata khác.

Điểm quan trọng của experiment không phải học thuộc cấu trúc byte.

Điểm quan trọng là:

> **Một cơ chế chia công việc tốt cho Q4_K không mặc nhiên đúng cho Q6_K.**

Cách đọc block khác.

Cách giải mã giá trị khác.

Thứ tự cộng những giá trị gần đúng cũng có thể tạo sai số khác.

Vì vậy Q6 không được coi là:

> “Q4 nhưng nhiều bit hơn nên chắc chắn dễ hơn.”

Nó phải tự đi qua correctness gate.

## Correctness phải đứng trước performance

Thứ tự phép thử đã được khóa:

```text
1. baseline so với CPU reference

2. candidate so với CPU reference

3. candidate so với baseline

4. chỉ khi tất cả correctness PASS
   mới được đo performance
```

`CPU reference — kết quả tham chiếu trên CPU` đóng vai trò như một đường tính độc lập để kiểm tra GPU.

Phần quan trọng nhất:

> **Performance nằm sau correctness.**

Nếu candidate sai, không có:

```text
candidate chạy nhanh bao nhiêu?
```

Câu hỏi đó chưa được phép tồn tại.

## Hai ngưỡng tính đúng được khóa từ trước

Correctness contract dùng hai đại lượng quen thuộc.

Thứ nhất:

> **max_abs — sai lệch tuyệt đối lớn nhất giữa hai kết quả.**

Ngưỡng:

```text
max_abs <= 0,02
```

Thứ hai:

> **RMSE — căn trung bình bình phương sai số**, dùng để nhìn sai lệch tổng thể thay vì chỉ điểm tệ nhất.

Ngưỡng:

```text
RMSE <= 0,005
```

Ta thử một ví dụ đơn giản.

CPU reference:

```text
[1,000
 2,000
 3,000]
```

Candidate:

```text
[1,003
 1,996
 3,002]
```

Sai lệch tuyệt đối:

```text
0,003
0,004
0,002
```

Vậy:

```text
max_abs
= 0,004
```

nhỏ hơn:

```text
0,02
```

RMSE:

```text
sqrt(
  (0,003² + 0,004² + 0,002²) / 3
)

≈ 0,0031
```

cũng nhỏ hơn:

```text
0,005
```

Ví dụ này PASS cả hai gate.

Ngược lại, chỉ cần một giá trị lệch:

```text
0,021
```

thì:

```text
max_abs > 0,02
```

và correctness FAIL ngay cả khi những giá trị khác rất gần.

Các ngưỡng này không phải “độ đúng tuyệt đối của mọi model”.

Chúng là contract đã được đóng băng cho phép thử này.

## Q6 compile được

Candidate Q6 được triển khai.

Shader compile thành công.

Native build cũng thành công.

Đây là một điểm đáng nhấn mạnh.

```text
compile PASS
```

không có nghĩa:

```text
science PASS
```

Một chương trình có thể:

```text
viết đúng cú pháp
↓
compiler chấp nhận
↓
build thành executable
↓
GPU chạy được
```

mà vẫn:

```text
tính sai
```

Infrastructure và correctness là hai lớp khác nhau.

Q6 vượt qua lớp đầu.

Nhưng rồi tới correctness.

## Baseline PASS — candidate FAIL

Harness — **chương trình điều khiển phép thử** — kiểm tra theo thứ tự:

```text
baseline_cpu
↓
candidate_cpu
↓
candidate_baseline
```

Kết quả được giữ lại là:

```text
status = ERROR

correctness gate failed:
candidate_cpu
```

Điều này cho ta biết một việc rất quan trọng.

Để đi tới:

```text
candidate_cpu
```

thì bước trước:

```text
baseline_cpu
```

đã phải PASS.

Nghĩa là đường baseline Q6 vẫn phù hợp với CPU reference trong case đang kiểm tra.

Sau đó chính candidate Q6 vi phạm correctness contract.

Verdict:

```text
Q6_STAGE_FAIL_CORRECTNESS
```

Nói bằng tiếng Việt:

> **Cùng cơ chế Split-K đã thắng ở Q4_K, khi giữ nguyên và chuyển sang Q6_K, không còn giữ được tính đúng trong ngưỡng đã khóa.**

## Nhưng ta không biết sai bao nhiêu

Có một chi tiết rất đáng học từ đây.

Harness dùng kiểu:

> **fail-fast — gặp lỗi đầu tiên thì dừng ngay.**

Vì vậy khi `candidate_cpu` FAIL, chương trình dừng.

Nó không tiếp tục đo performance.

Và artifact giữ lại không chứa chính xác:

```text
max_abs thực tế = bao nhiêu?
RMSE thực tế = bao nhiêu?
```

Ta chỉ biết:

> ít nhất một điều kiện correctness đã bị vi phạm.

Điều kiện có thể là:

```text
kết quả không finite
```

hoặc:

```text
max_abs > 0,02
```

hoặc:

```text
RMSE > 0,005
```

hoặc nhiều điều cùng lúc.

Magnitude — **độ lớn chính xác của lỗi** — không được giữ lại.

Một phản xạ rất tự nhiên là:

> “Chạy lại để xem chính xác sai bao nhiêu.”

Nhưng đó là lúc Mode C phải làm việc.

## Không được rerun chỉ vì ta tò mò

Phép thử đã có một contract.

Correctness FAIL là stop condition.

Evidence đầu tiên hợp lệ.

Không có lỗi hạ tầng.

Không có source drift.

Không có lỗi packaging làm invalid experiment.

Vì vậy outcome phải được chấp nhận.

Trong chính study này, bị cấm:

```text
rerun Q6

nới threshold

đổi geometry

thử subgroup khác

đổi reduction

tuning để cứu performance

đưa vào model thật
```

Điều này nghe có vẻ cứng nhắc.

Tại sao không sửa cho nó đúng?

Bởi câu hỏi nghiên cứu không phải:

> “Ta có thể bằng mọi cách làm ra một kernel Q6 nhanh và đúng không?”

Câu hỏi đã khóa là:

> **“Giữ nguyên cơ chế Split-K và hình học thực thi đã thắng ở Q4, rồi dùng một Q6_K candidate với bộ đọc packed-Q6 tương ứng, correctness có còn giữ được hay không?”**

Evidence đã trả lời:

> **Không, trong contract hiện tại.**

Nếu ta đổi cơ chế sau khi thấy outcome, ta đang hỏi câu khác.

Câu khác có thể đáng nghiên cứu.

Nhưng nó phải được mở như một study mới.

## Đây là một FAIL khoa học, không phải lỗi implementation chưa sửa xong

Ta cần phân biệt ba trường hợp.

Trường hợp thứ nhất:

```text
compiler lỗi
```

Có thể chỉ là implementation defect.

Trường hợp thứ hai:

```text
file sai
driver lỗi
runner hỏng
provenance không khớp
```

Có thể experiment chưa hợp lệ.

Nhưng ở Q6:

```text
shader compile
→ PASS

native build
→ PASS

baseline correctness
→ PASS

candidate correctness
→ FAIL
```

và independent review không tìm thấy lỗi hạ tầng hay provenance đủ để vô hiệu hóa phép thử.

Vì vậy đây không phải:

> “chưa chạy được.”

Nó là:

> **một kết quả khoa học âm tính hợp lệ.**

## Performance của Q6 bằng bao nhiêu?

Câu trả lời là:

> **Không biết.**

Không phải:

> “chậm.”

Không phải:

> “nhanh.”

Mà là:

> **không có phép đo performance hợp lệ.**

Số cặp timing Q6:

```text
0
```

Target model:

```text
không được load
```

Performance measurement:

```text
không được chạy
```

Đây là một boundary rất quan trọng.

Từ:

```text
correctness FAIL
```

không được suy ra:

```text
performance FAIL
```

Hai claim hoàn toàn khác nhau.

Q6 có thể về lý thuyết rất nhanh nhưng sai.

Hoặc chậm.

Ta không biết.

Và study này không được phép đi tìm câu trả lời performance nữa.

## Q4 PASS không có nghĩa là Q6 cũng vậy

Đây là bài học lớn hơn Q6.

Q4 đã cho kết quả rất đẹp:

```text
~3,14× aggregate component speedup
```

Correctness PASS.

Nhưng điều đó chỉ chứng minh:

> **cơ chế Split-K này có hiệu lực trên họ shape Q4_K đã kiểm tra.**

Nó không chứng minh:

```text
mọi quantization
mọi shape
mọi model
mọi GPU
```

sẽ cùng PASS.

Q6 đã bác bỏ một claim rộng hơn:

> **cùng cơ chế Split-K và hình học thực thi có thể chuyển từ Q4_K sang Q6_K, với bộ đọc packed-Q6 tương ứng, mà vẫn giữ correctness contract.**

Nói cách khác:

```text
Q4 PASS
≠
universal PASS
```

Một result mạnh vẫn có biên giới.

## Đây chính là Mode C

Ở Mode E, AI được quyền nghĩ rất nhiều.

Ở Mode M, ta giữ lại một mechanism.

Mode C thay đổi thái độ hoàn toàn.

Trước execution:

```text
khóa hypothesis
khóa implementation
khóa correctness gate
khóa performance gate
khóa stop rule
```

Sau execution:

```text
đọc evidence
↓
PASS hoặc FAIL
↓
không sửa luật theo outcome
```

AI lúc này không có nhiệm vụ:

> “Tìm cách cứu hypothesis.”

Nó có nhiệm vụ:

> **Giúp bảo toàn contract, kiểm tra evidence và chỉ ra kết luận nhỏ nhất mà dữ liệu thực sự hỗ trợ.**

Đây là sự khác biệt rất lớn.

Nếu AI luôn được yêu cầu:

> “Tiếp tục cho tới khi PASS.”

thì mọi FAIL chỉ trở thành một lỗi tạm thời cần sửa.

Khi đó dự án gần như mất khả năng học được rằng:

> **một giả thuyết ban đầu thực sự sai hoặc không tổng quát như ta nghĩ.**

## Con người phải có khả năng nói “dừng”

Q6 là một ví dụ rất nhỏ nhưng quan trọng về quyền quyết định.

AI hoàn toàn có thể đề xuất:

> thử reduction khác;

> tăng tolerance;

> thay cách accumulate;

> dùng subgroup 16;

> đổi local size;

> đo performance trước xem có đáng sửa correctness không.

Nhưng chính vì những phương án đó dễ sinh ra, con người càng phải giữ ranh giới:

> **Không. Câu hỏi hiện tại đã được trả lời.**

Muốn nghiên cứu:

> “Tại sao Q6 sai?”

được.

Nhưng đó là câu hỏi mới.

Có thể mở một study mới về numerical conditioning — **độ nhạy số học khi thứ tự và cách cộng thay đổi**.

Có thể so các topology reduction khác nhau.

Có thể hỏi layout Q6 tạo ra boundary ở đâu.

Nhưng không được quay lại và viết lại lịch sử rằng SA1-Q6 chưa FAIL.

FAIL đó phải được giữ nguyên.

## Vì sao FAIL này có giá trị?

Trước Q6, một niềm tin hợp lý có thể là:

```text
Split-K subgroup32 tốt cho Q4
↓
có lẽ cùng mechanism cũng tốt cho Q6
```

Sau Q6:

```text
Q4
→ efficacy + correctness supported

Q6
→ unchanged mechanism violates correctness
```

Bản đồ hiểu biết đã tốt hơn.

Ta biết rằng quantization format không chỉ là một chi tiết lưu trữ.

Nó có thể tạo ra boundary thực sự cho cách tổ chức phép tính.

Đó là tri thức có giá trị.

Nếu Q6 chỉ được “sửa cho tới khi chạy được”, boundary này có thể biến mất khỏi lịch sử.

## Một FAIL đúng có thể quý hơn một PASS dễ dãi

Giả sử sau khi thấy lỗi, ta đổi threshold:

```text
max_abs
0,02
→
0,05
```

và candidate PASS.

Ta có một PASS.

Nhưng PASS đó trả lời câu hỏi nào?

Không còn là câu hỏi đã đăng ký ban đầu.

Hoặc giả sử đổi geometry ba lần cho tới khi một phiên bản đúng.

Ta có thể có một kernel mới.

Nhưng ta đã mất thông tin:

> **cơ chế nguyên bản không chuyển được sang Q6.**

Trong nghiên cứu, mục tiêu không phải tối đa số PASS.

Mục tiêu là:

> **tối đa lượng tri thức đáng tin mà mỗi PASS và FAIL mang lại.**

## Cánh cửa sang Mode T vẫn đóng

Chương 14 sẽ nói về **Mode T — Transfer**, tức đưa một mechanism đã xác nhận sang hệ thống thật.

Nhưng Q6 không được đi tới đó.

Chuỗi của Q6 dừng tại:

```text
E
có ý tưởng mở rộng sang Q6

↓

M
giữ nguyên mechanism Q4

↓

C
correctness FAIL

↓

STOP
```

Không có:

```text
T
```

cho Q6 trong study này.

Đây cũng là lý do bốn mode không phải dây chuyền mà mọi hypothesis đều bắt buộc phải đi hết.

Một hypothesis có thể chết ở E.

Có thể chết ở M.

Có thể chết ở C.

Chỉ những thứ sống sót mới được quyền đi tiếp.

### Nhớ 3 điều

1. **Mode C khóa luật trước rồi để evidence phán xét.** Q6 giữ nguyên cơ chế Split-K đã PASS ở Q4 và phải vượt correctness trước khi performance được phép đo.
2. **Q6 FAIL về correctness, không FAIL về performance.** Performance không được chạy, số measurement pair bằng 0 và model thật không được load; vì vậy không được nói Q6 nhanh hay chậm.
3. **Một PASS không tự động tổng quát sang miền khác.** Q4 chứng minh cơ chế có giá trị trong phạm vi Q4_K đã thử. Q6 cho thấy cùng cơ chế giữ nguyên không vượt được correctness contract ở một định dạng lượng tử hóa khác.

**Chương 14 — Từ một cơ chế tốt tới hệ thống thật**

Q6 dừng trước khi được chuyển tiếp.

Nhưng Q4 thì đã sống sót.

Câu hỏi kế tiếp vì vậy không còn là:

> “Component này có nhanh không?”

Mà là:

> **“Khi mang chính cơ chế đã PASS vào model thật, với trọng số thật, activation thật và toàn bộ decode graph, lợi ích đó còn tồn tại không?”**
