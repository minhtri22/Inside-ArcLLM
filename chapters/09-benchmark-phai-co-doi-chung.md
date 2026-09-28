# Chương 9 — Benchmark phải có đối chứng

> **Câu hỏi của chương:** Nếu ArcLLM chạy được và đã tự tối ưu qua nhiều bước, làm thế nào biết nó thực sự đang đứng ở đâu khi đặt cạnh một runtime trưởng thành?

Ở cuối Chương 8, ta đã có một điều mà lúc bắt đầu cuốn sách chưa hề có.

Một production path.

Nó có thể:

```text
đọc trọng số Q4_K / Q6_K đóng gói
        ↓
giữ decoder resident
        ↓
prefill
        ↓
giữ KV cache trên GPU
        ↓
decode nhiều token
        ↓
tạo logits
        ↓
chọn token tiếp theo
```

Trong P7, một số thay đổi còn tạo ra speedup rất lớn so với chính phiên bản ArcLLM trước đó.

P7-C:

```text
≈ 5,18×
```

P7-E:

```text
≈ 1,85×
```

P7-L:

```text
≈ 1,24×
```

Nếu dừng câu chuyện ở đây, rất dễ có cảm giác:

> “ArcLLM giờ đã nhanh.”

Nhưng câu đó chưa có nghĩa khoa học rõ ràng.

Nhanh hơn **cái gì**?

Trong **workload nào**?

Cùng model hay khác model?

Cùng quantization hay không?

Cùng GPU không?

Một bên chạy lúc máy đang rảnh còn bên kia chạy khi máy nóng?

Một bên sinh 32 token còn bên kia sinh 128 token?

Một bên tính cả thời gian prefill còn bên kia chỉ tính decode?

Nếu những câu hỏi ấy chưa được khóa, từ “nhanh hơn” gần như vô nghĩa.

Đây là lý do benchmark cần **đối chứng — một baseline phù hợp để so sánh trong những điều kiện đủ giống nhau**.

## Benchmark không phải là chạy hai chương trình rồi nhìn con số

Giả sử ta chạy ArcLLM và thấy:

```text
10 token/giây
```

Sau đó tìm trên Internet một người chạy llama.cpp được:

```text
8 token/giây
```

Ta có được phép nói ArcLLM nhanh hơn không?

Không.

Người kia có thể dùng model khác.

GPU khác.

Quant khác.

Prompt khác.

Context khác.

Phiên bản llama.cpp khác.

Thậm chí cách định nghĩa “token/giây” cũng có thể khác.

So sánh như vậy giống như nói:

> “Xe A đi hết đường trong 20 phút. Xe B mất 25 phút. Vậy A nhanh hơn.”

nhưng quên hỏi rằng một chiếc chạy 10 km còn chiếc kia chạy 18 km.

Một benchmark có ý nghĩa phải cố giữ những thứ không phải đối tượng nghiên cứu **giống nhau**.

Trong ArcLLM, từ được dùng là:

**matched benchmark — benchmark ghép cặp, trong đó hai hệ được đặt dưới một tập điều kiện chung đã khóa trước.**

## “Matched” không có nghĩa hai runtime phải giống nhau

Đây là một điểm dễ hiểu nhầm.

ArcLLM và llama.cpp không có cùng kiến trúc bên trong.

Nếu ép chúng có cùng kernel, cùng scheduler và cùng cách tổ chức memory thì ta không còn so hai runtime nữa.

Điều cần match là **bài toán chúng phải giải**.

Có thể hình dung:

```text
             cùng model bytes
             cùng input
             cùng output length
             cùng hardware
             cùng context
             cùng quantization
             cùng quy tắc chọn token
             cùng cách đo
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
     ArcLLM                 llama.cpp
        ↓                       ↓
  cách thực thi A          cách thực thi B
```

Phần ở trên phải được khóa.

Phần bên trong runtime được phép khác.

Chính sự khác biệt đó mới là thứ ta muốn quan sát.

## Trước benchmark này, model cũng đã lớn hơn

Trước khi nói về Q2, cần nhắc lại một bước đã xảy ra trước đó trong hành trình nghiên cứu.

P7 mà Chương 8 vừa kể tập trung vào đường production của model 1.5B.

Nhưng benchmark matched sau đó không còn dùng model đó.

Trước khi Q2 được phép mở, ArcLLM đã phải vượt qua một gate riêng:

> **Q1 — model 7B thật có chạy end-to-end trên máy mục tiêu hay không?**

Chi tiết về những trở ngại kiến trúc của bước 7B sẽ còn xuất hiện ở phần sau của sách, vì chúng tạo ra những bài học quan trọng về representation và abstraction.

Ở đây ta chỉ cần một fact đã được chứng minh trước Q2:

> **Qwen2.5-Coder 7B Q4_K_M đã chạy end-to-end trên ArcLLM với model thật, 28 layer thật, GPU-resident KV và autoregressive decode thật.**

Hai execution độc lập của Q1 cho cùng chuỗi token, cùng logits hash và cùng final-hidden hash.

Chỉ sau khi **feasibility — khả năng chạy thực sự** được thiết lập, benchmark performance mới được mở.

Đây là một nguyên tắc đáng nhớ:

```text
chưa chạy được model thật
        ↓
không dùng microbenchmark
để nói về performance toàn hệ

chạy được model thật
        ↓
mới mở matched benchmark
```

## Đối chứng là llama.cpp

Baseline được chọn là:

**llama.cpp v0.4.1 với Vulkan.**

Nhưng chỉ tên phiên bản thôi vẫn chưa đủ.

Q2 khóa baseline xuống đúng Git release.

Trong quá trình kiểm tra trước phép đo, một chi tiết thú vị được phát hiện.

Một commit ban đầu được ghi là đi cùng `v0.4.1`, nhưng annotated tag — **tag Git chính thức trỏ tới release** — thực tế chỉ tới một commit khác.

Hai commit cách nhau năm commit.

Quan trọng hơn:

> việc này được phát hiện **trước khi có bất kỳ measurement Q2 nào**.

Do đó baseline được sửa về đúng commit của tag `v0.4.1` trước khi chạy benchmark.

Không có outcome performance nào để nhìn rồi mới lựa baseline thuận lợi hơn.

Đây không phải một chi tiết Git vô thưởng vô phạt.

Nó minh họa một nguyên tắc:

> **Đối chứng phải có danh tính chính xác.**

“llama.cpp khoảng phiên bản đó” không đủ tốt.

Cuối cùng baseline được khóa ở đúng release `v0.4.1`, commit:

```text
b29c606e...
```

## Cùng tên model cũng chưa đủ

Cả ArcLLM và llama.cpp phải đọc **đúng cùng một file model theo byte**.

Target là Qwen2.5-Coder 7B Q4_K_M với kích thước:

```text
4.683.074.048 byte
```

Không phải:

> “hai model đều là Qwen2.5-Coder 7B.”

Mà là:

> **cùng SHA-256, cùng số byte.**

Tại sao phải khó tính như vậy?

Hai file cùng tên model có thể khác metadata.

Khác quantization build.

Khác tensor layout.

Thậm chí khác một vài byte.

Nếu ArcLLM chạy file A còn baseline chạy file B, ta đã thêm một biến mới vào experiment.

Q2 loại biến đó.

## Tokenizer cũng bị đưa ra ngoài benchmark

Cả hai hệ nhận trực tiếp cùng **raw token IDs — chính các mã token đã được khóa trước**.

Không để ArcLLM tokenizer một kiểu còn llama.cpp tokenizer kiểu khác.

Không chat template khác nhau.

Không system prompt được thêm âm thầm.

Benchmark bắt đầu từ:

```text
cùng token IDs
```

chứ không phải:

```text
cùng một câu chữ
→ hai tokenizer khác nhau
→ có thể thành hai input khác nhau
```

Việc này làm experiment ít giống trải nghiệm chat hoàn chỉnh hơn.

Nhưng nó giúp trả lời câu hỏi runtime chính xác hơn.

## Những thứ nào được khóa giống nhau?

Q2 giữ cùng:

```text
model bytes

Q4_K_M

GPU / máy

Vulkan

context capacity = 4096

KV cache = F32

8 CPU threads

greedy generation

output = 32 token

không speculative decode
```

Ở llama.cpp:

```text
batch  = 256
ubatch = 256
```

và toàn bộ layer được yêu cầu GPU offload.

Preflight — **kiểm tra trước khi cho phép đo thật** — xác nhận baseline thực sự báo:

```text
29 / 29 layers offloaded
```

Như vậy benchmark không vô tình so:

```text
ArcLLM dùng GPU
vs
llama.cpp đang rơi một phần lớn về CPU
```

## Hai workload, vì một workload không kể được cả câu chuyện

Q2 khóa hai bài thử.

Bài đầu:

**W-S — short/decode-dominant**, tức prompt ngắn để phần sinh token chiếm tỷ trọng lớn hơn.

Prompt:

```text
4 token
```

Output:

```text
32 token
```

Bài thứ hai:

**W-C — context/prefill-sensitive**, tức prompt dài hơn để chi phí xử lý context ban đầu hiện rõ hơn.

Prompt:

```text
256 token
```

Output vẫn là:

```text
32 token
```

Tại sao không chỉ chọn một?

Vì runtime có thể có hai đặc tính rất khác:

```text
xử lý prompt nhanh
nhưng decode chậm
```

hoặc:

```text
prefill chậm
nhưng decode tốt
```

Nếu chỉ đo một workload, ta có thể vô tình chọn đúng vùng thuận lợi cho một hệ.

Hai workload chưa bao phủ mọi thứ trên đời.

Nhưng chúng tạo hai regime — **hai chế độ tải có đặc tính khác nhau** — đã được khóa trước outcome.

## Ba con số khác nhau kể ba câu chuyện khác nhau

Q2 đo ba metric performance chính.

Đầu tiên là:

**TTFT — Time To First Token, thời gian từ lúc bắt đầu xử lý prompt tới khi token đầu tiên sẵn sàng.**

Ví dụ:

```text
bắt đầu prefill: 0 ms

token đầu tiên sẵn sàng:
120 ms
```

thì:

```text
TTFT = 120 ms
```

TTFT quan trọng với cảm giác phản hồi ban đầu.

Một model có thể sinh token sau đó rất nhanh, nhưng nếu mất năm giây mới bắt đầu trả lời, người dùng vẫn cảm thấy nó chậm.

Metric thứ hai:

**decode throughput — tốc độ sinh các token sau khi prefill đã xong.**

Trong Q2 có 32 token output.

Token đầu tiên thuộc TTFT.

Vì vậy decode throughput được tính trên:

```text
token #2 → token #32
```

tức:

```text
31 token
```

Nếu 31 token đó mất:

```text
2,48 giây
```

thì:

```text
decode throughput
= 31 / 2,48
≈ 12,5 token/giây
```

Metric thứ ba:

**E2E latency — end-to-end latency, tổng thời gian từ khi bắt đầu prefill tới khi token cuối cùng của workload sẵn sàng.**

Nếu bắt đầu tại:

```text
0 ms
```

và token thứ 32 xuất hiện tại:

```text
2.600 ms
```

thì:

```text
E2E latency
= 2.600 ms
= 2,6 giây
```

Ba metric trả lời ba câu khác nhau:

```text
TTFT
→ bắt đầu phản hồi nhanh không?

decode tok/s
→ sau khi bắt đầu, sinh tiếp nhanh không?

E2E
→ hoàn thành toàn workload mất bao lâu?
```

Vì thế nói:

> “Runtime X nhanh hơn.”

mà không nói metric nào là thiếu thông tin.

## Một lần chạy không đủ

Mỗi cell — **một tổ hợp system × workload** — có:

```text
1 warmup
+
5 measured attempts
```

Có bốn cell:

```text
ArcLLM × W-S
llama.cpp × W-S

llama.cpp × W-C
ArcLLM × W-C
```

Tổng số measured attempts:

```text
4 × 5
= 20
```

Warmup — **lượt chạy làm nóng trước khi lấy số chính thức** — không được tính vào năm measurement.

Vì sao cần nhiều lần?

Máy tính không phải một chiếc đồng hồ lý tưởng.

Driver có trạng thái.

Cache có trạng thái.

Nhiệt độ thay đổi.

Hệ điều hành có thể làm việc nền.

Một measurement đơn độc có thể là ngoại lệ.

## Vì sao dùng median?

Giả sử năm lần đo cho:

```text
10
11
12
13
100
```

Nếu tính trung bình:

```text
(10 + 11 + 12 + 13 + 100) / 5
= 29,2
```

Con số `29,2` tạo cảm giác workload thường mất gần 30 đơn vị thời gian.

Nhưng bốn trong năm lần thực tế chỉ nằm từ 10 tới 13.

Một lần `100` kéo mean — **trung bình cộng** — lên rất mạnh.

Median — **trung vị** — làm khác.

Sắp xếp:

```text
10
11
12
13
100
```

số nằm giữa là:

```text
12
```

Vậy:

```text
median = 12
```

Q2 vì vậy báo median cho các metric chính.

Nó vẫn giữ minimum và maximum.

Nó không xóa lần chạy 100.

Chỉ là không để một outlier — **giá trị lệch rất xa phần còn lại** — tự mình quyết định con số đại diện.

## MAD cho biết các lần đo phân tán ra sao

Q2 còn dùng **MAD — Median Absolute Deviation, trung vị của độ lệch tuyệt đối so với median**.

Với dãy vừa rồi:

```text
10 11 12 13 100
```

median:

```text
12
```

Khoảng cách tới 12:

```text
|10 - 12| = 2
|11 - 12| = 1
|12 - 12| = 0
|13 - 12| = 1
|100 - 12| = 88
```

Sắp xếp các khoảng cách:

```text
0
1
1
2
88
```

median của chúng là:

```text
MAD = 1
```

Nó cho ta biết phần lớn measurement nằm khá chặt quanh trung tâm dù có một outlier rất lớn.

Q2 cũng không báo **p95 — phân vị thứ 95, tức mức mà khoảng 95% lần đo nằm ở hoặc thấp hơn nó**.

Hình dung nếu có 100 lần đo đã sắp từ nhanh tới chậm:

```text
nhanh nhất
↓
...
lần thứ 95  ← p95
...
lần thứ 100
↑
chậm nhất
```

Nếu `p95 = 250 ms`, ta có thể hiểu gần đúng rằng khoảng 95% các lần đo hoàn thành trong 250 ms hoặc nhanh hơn, còn khoảng 5% chậm hơn mức đó.

Đây là một **tail metric — metric nhìn phần đuôi chậm của phân bố**.

Nhưng với chỉ năm lần đo trong mỗi cell, số mẫu quá ít để một tail metric như p95 có ý nghĩa ổn định.

## Performance chưa phải toàn bộ resource envelope

Q2 không chỉ đo thời gian.

Nó còn quan sát RAM và CPU.

Hai thuật ngữ dễ bị nhầm là:

**working set** và **private bytes**.

Working set có thể hiểu gần đúng là:

> lượng memory của process đang thực sự resident trong physical memory tại thời điểm đó.

Private bytes gần hơn với:

> phần memory đã commit riêng cho process.

Hai con số không đồng nghĩa.

Đặc biệt trên hệ thống UMA — **CPU và GPU chia sẻ một không gian bộ nhớ vật lý lớn** — việc nhìn một con số memory duy nhất rồi kết luận “runtime dùng từng này VRAM” rất dễ sai.

Vì vậy Q2 giữ nhiều metric resource thay vì cố ép chúng thành một số duy nhất.

GPU utilization cũng được thử thu thập.

Nhưng Windows GPU Engine counters cho một số ArcLLM sample báo peak vượt 100%.

Một counter như vậy không còn đủ đáng tin cho claim.

Q2 không “sửa” số đó.

Không đoán lại.

Nó ghi:

> **GPU-utilization peak unreliable.**

và loại metric đó khỏi validity/advantage claim.

Một measurement không đáng tin không trở thành bằng chứng chỉ vì ta muốn có thêm cột trong bảng.

## Hai bên chạy xong cả 20 phép đo

Khi execution hoàn tất:

```text
20 / 20 measured attempts
→ thành công
```

Mỗi attempt tạo đúng:

```text
32 generated tokens
```

Final logits đều finite.

ArcLLM giữ đúng dispatch census đã khóa.

llama.cpp thực sự chạy đúng baseline Vulkan đã pin.

RAM/CPU traces bắt buộc cũng đầy đủ.

Do đó Q2 nhận classification:

> **Q2_MATCHED_CHARACTERIZATION_COMPLETE**

Cần đọc chính xác tên verdict.

Nó không nói:

> ArcLLM thắng.

Cũng không nói:

> llama.cpp thắng.

Nó chỉ nói:

> **Bộ matched characterization đã được thực hiện hợp lệ và đủ dữ liệu để bước khoa học tiếp theo sử dụng.**

## Và đây là bảng mà Q2 nhìn thấy

Median đã đóng băng:

| Workload | System | TTFT | Decode | E2E | Peak working set |
|---|---|---:|---:|---:|---:|
| W-S | ArcLLM | 918,265 ms | 0,3184 tok/s | 98.285,231 ms | ~10,02 GB |
| W-S | llama.cpp | 100,318 ms | 12,7449 tok/s | 2.535,868 ms | ~5,47 GB |
| W-C | llama.cpp | 1.423,482 ms | 13,0354 tok/s | 3.803,029 ms | ~5,48 GB |
| W-C | ArcLLM | 14.706,425 ms | 0,3990 tok/s | 91.043,729 ms | ~10,01 GB |

Chỉ nhìn bảng, mắt người lập tức muốn đưa ra verdict.

Nhưng Q2 contract cấm việc đó.

Tại sao?

Vì Q2 được thiết kế trước với vai trò:

> **characterization — mô tả đầy đủ mặt phẳng performance/resource.**

Không phải:

> **advantage adjudication — phán quyết liệu có một lợi thế thực tế hay không.**

Đó là hai công việc khoa học khác nhau.

## Ratio cũng chỉ là mô tả

Ví dụ W-S có TTFT ratio:

```text
ArcLLM / llama.cpp
≈ 9,154
```

Điều đó chỉ có nghĩa:

> median TTFT của ArcLLM trong cell này bằng khoảng 9,154 lần con số baseline.

Decode throughput ratio:

```text
≈ 0,02499
```

Working-set ratio:

```text
≈ 1,832
```

Private bytes lại khoảng:

```text
≈ 0,9688
```

Ta có nhiều hướng chuyển động khác nhau.

Nếu bây giờ tùy ý chọn:

> “Private bytes thấp hơn khoảng 3%, vậy ArcLLM có advantage.”

thì ta đang cherry-pick — **chọn một metric thuận lợi sau khi đã nhìn outcome**.

Nếu ngược lại nhìn throughput rồi tuyên bố ngay final verdict, ta cũng đang bỏ qua contract đã khóa rằng Q2 chỉ làm characterization.

Bước tiếp theo phải được thiết kế riêng.

## Tại sao không cho Q2 tự chọn “winner”?

Đây là một nguyên tắc rất mạnh.

Nếu cùng một study vừa:

```text
đo tất cả metric
```

rồi sau khi xem bảng mới quyết định:

```text
metric nào quan trọng
threshold bao nhiêu
thế nào mới gọi là advantage
```

ta đã cho outcome tham gia vào việc viết luật chấm outcome.

Q2 cố tình tách hai giai đoạn:

```text
Q2
→ xem toàn bộ mặt phẳng evidence
→ characterization complete

sau đó

Q3
→ freeze claim cụ thể
→ freeze practical threshold
→ fresh evidence
→ PASS / FAIL
```

Đây chính là sự khác nhau giữa:

> **khám phá điều dữ liệu đang nói**

và:

> **xác nhận một claim đã định nghĩa trước.**

## Một kết quả âm tính cũng cần được thiết kế nghiêm túc

Bảng Q2 không cho thấy một candidate advantage rõ ràng trên các primary performance metric hay working-set memory.

Private bytes thấp hơn một chút.

CPU utilization có những khác biệt mô tả.

Nhưng Q2 không được phép biến các tín hiệu nhỏ đó thành claim mới chỉ để cứu một kết quả mong muốn.

Vì vậy câu hỏi của Chương 10 sẽ mạnh hơn nhiều:

> **Nếu ta khóa trước một ranh giới “practical advantage” rồi lấy fresh evidence, ArcLLM có thực sự vượt được ranh giới đó ở bất kỳ regime đã xác định nào không?**

Nếu câu trả lời là không, đó không phải là sự thất bại của phương pháp.

Ngược lại.

Đó chính là lúc phương pháp chứng minh giá trị của nó.

Bởi một research program tốt không tồn tại để bảo vệ kiến trúc mà nó đã xây.

Nó tồn tại để tìm ra kiến trúc đó **thực sự làm được gì — và không làm được gì**.

### Nhớ 3 điều

1. **Matched benchmark không bắt hai runtime phải giống kiến trúc; nó bắt bài toán so sánh phải đủ giống.** Q2 khóa cùng model bytes, raw token IDs, hardware, quant, context, output length và các điều kiện measurement quan trọng.
2. **TTFT, decode throughput và E2E là ba câu hỏi khác nhau.** Một chữ “nhanh” không thay thế được ba metric này; nhiều lần đo và median giúp tránh để một run bất thường định nghĩa toàn bộ kết quả.
3. **Q2 hoàn thành characterization, không chọn winner.** 20/20 attempt thành công tạo ra một mặt phẳng evidence đủ hợp lệ; việc một practical advantage có tồn tại hay không phải được khóa thành câu hỏi riêng và kiểm tra bằng fresh evidence ở Q3.

**Chương 10 — Khi “tự build được” vẫn chưa đủ**

Ở chương tiếp theo, ArcLLM sẽ làm điều khó nhất đối với một project đã đầu tư rất nhiều công sức:

> **định nghĩa trước điều gì sẽ khiến chính kiến trúc của mình không vượt qua được bài kiểm tra — rồi để fresh evidence quyết định.**
