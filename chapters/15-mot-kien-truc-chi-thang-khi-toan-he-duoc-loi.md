# Chương 15 — Chỉ có ích khi toàn hệ thực sự được lợi

> **Mức đọc: Nghiên cứu**
>
> **Bạn đang ở bước nào của hành trình nghiên cứu?**
>
> ```text
> Cải thiện bên trong ArcLLM
>         ↓
> [ đo lại với đối chứng bên ngoài ]
>         ↓
> giá trị ở cấp toàn hệ
> ```


> **Câu hỏi của chương:** Sau khi một cơ chế đã giúp ArcLLM nhanh hơn khoảng hai lần ở cấp toàn hệ nội bộ, làm thế nào biết cải thiện đó thực sự đã đưa hệ thực thi tới gần một hệ thống trưởng thành hơn hay chưa?

Chương 14 kết thúc bằng một kết quả rất đáng kể.

Cơ chế Q4 Split-K không chỉ nhanh trong một phép thử nhỏ.

Nó đã sống sót qua:

```text
bài thử cố định
↓
trọng số và dữ liệu trung gian thật
↓
ngữ nghĩa của toàn mô hình
↓
giai đoạn sinh token
↓
toàn bộ lượt chạy
```

Độ trễ ở giai đoạn sinh token cải thiện theo trung bình hình học khoảng:

```text
2,20×
```

Toàn bộ lượt chạy cải thiện khoảng:

```text
2,00×
```

TTFT vẫn vượt qua hàng rào không-thụt-lùi.

Nếu chỉ nhìn ArcLLM trước và sau thay đổi, đây là một thành công rõ ràng.

Nhưng có một câu hỏi chưa được trả lời:

> **Nhanh hơn chính mình rất nhiều có đồng nghĩa đã trở thành một hệ thực thi nhanh hay chưa?**

Không nhất thiết.

Một người chạy 100 mét trong 40 giây rồi cải thiện xuống 20 giây đã nhanh gấp đôi chính mình.

Nhưng điều đó chưa nói người ấy đang đứng ở đâu so với những người khác.

Hệ thực thi cũng vậy.

## Sau một kết quả ĐẠT lớn, phép đo cũ đã hết hạn

Ở Chương 9 và 10, ArcLLM từng được so với llama.cpp.

Nhưng đó là kiến trúc cũ.

Sau I002, ArcLLM đã thay đổi.

56 phép gate/up Q4_K trong mỗi token giai đoạn sinh token giờ chạy bằng một mechanism khác.

Vì vậy không được lấy:

```text
ArcLLM mới
```

rồi so trực tiếp với:

```text
số llama.cpp cũ
```

và gọi đó là phép đo đối chứng cùng điều kiện.

Ta cần một **phép đối chứng mới cùng điều kiện**.

Đây là một nguyên tắc dễ bỏ qua:

> **Khi phương án thử thay đổi đáng kể, bằng chứng đối chứng phải được làm mới.**

Một phép đo so sánh cũ có thể cho bối cảnh lịch sử.

Nó không tự động trở thành bằng chứng xác nhận cho kiến trúc mới.

## Đối chứng mới phải khóa lại từ đầu

Nghiên cứu tiếp theo giữ nguyên phương án thử I002 đã đóng.

Không tối ưu thêm.

Đối chứng vẫn là phiên bản llama.cpp đã pin từ trước.

Hai bên tiếp tục dùng:

```text
cùng mô hình GGUF
cùng byte của mô hình
cùng phần cứng
cùng W-S và W-C
cùng 32 token đầu ra
cùng mã token đầu vào thô
cùng greedy generation
```

Nhưng lần này có một vấn đề thực tế.

Máy nghiên cứu là một máy phát triển dùng chung.

Tải nền có thể thay đổi theo thời gian.

Nếu chạy toàn bộ ArcLLM trước rồi vài phút sau mới chạy llama.cpp, ta dễ trộn:

```text
khác biệt hệ thực thi
+
khác biệt trạng thái máy
```

Vì vậy phép thử dùng **đo theo cặp (paired measurement)** — hai phép đo được đặt gần nhau về thời gian.

Một cặp có dạng:

```text
ArcLLM
↓
llama.cpp
```

hoặc ngược lại:

```text
llama.cpp
↓
ArcLLM
```

Thứ tự được đảo xen kẽ.

Mỗi cell có:

```text
5 cặp
```

Có bốn cell.

Vậy tổng cộng:

```text
4 × 5
= 20 cặp
```

Mỗi cặp có hai lần suy luận:

```text
20 × 2
= 40 lần đo
```

Cả 20/20 cặp đều hợp lệ.

## Đây vẫn là phép mô tả đặc tính, không phải cuộc thi lấy cúp

Tên chính xác của kết quả là:

```text
I003_MATCHED_EXTERNAL_CHARACTERIZATION_COMPLETE
```

`Characterization — mô tả đặc tính bằng phép đo`.

Nghiên cứu này không đặt một “điểm tổng hợp” rồi tuyên bố ai thắng.

Nó hỏi:

> **Sau I002, khoảng cách thực tế giữa ArcLLM mới và đối chứng trưởng thành còn bao nhiêu?**

Đó là câu hỏi cần trả lời trước khi chọn bất kỳ chương trình GPU tiếp theo nào.

## Kết quả khá rõ

Tỷ lệ dưới đây đều là:

```text
phương án ArcLLM / llama.cpp
```

Với độ trễ, lớn hơn `1` nghĩa là ArcLLM mất nhiều thời gian hơn.

| Cell | TTFT | giai đoạn sinh token độ trễ | giai đoạn sinh token thông lượng | E2E |
|---|---:|---:|---:|---:|
| A/W-S | 7,50× | 8,83× | 0,113× | 8,73× |
| A/W-C | 8,76× | 9,21× | 0,109× | 9,16× |
| B/W-C | 9,70× | 13,04× | 0,077× | 11,51× |
| B/W-S | 6,16× | 10,95× | 0,091× | 10,75× |

Khi lấy trung bình hình học của bốn cell:

```text
độ trễ sinh token
≈ 10,38×
```

và:

```text
độ trễ toàn lượt
≈ 9,97×
```

giai đoạn sinh token thông lượng tỷ lệ:

```text
≈ 0,096
```

Con số `0,096` có thể hơi khó cảm nhận.

Giả sử đối chứng sinh được:

```text
10 token/giây
```

một tỷ lệ `0,096` tương đương khoảng:

```text
10 × 0,096
= 0,96 token/giây
```

Tức trong phép so sánh mới này, ArcLLM vẫn chỉ đạt gần một phần mười giai đoạn sinh token thông lượng của đối chứng.

Nói ngắn gọn:

> **I002 tạo ra một bước tiến nội bộ rất lớn, nhưng khoảng cách bên ngoài vẫn còn lớn.**

Hai điều này hoàn toàn có thể cùng đúng.

## Không được lấy khoảng cách cũ trừ khoảng cách mới

Một phản xạ rất hấp dẫn là nhìn lại Q2.

ArcLLM cũ từng có khoảng cách giai đoạn sinh token rất lớn.

Bây giờ khoảng cách chỉ còn khoảng 10×.

Rồi nói:

> “Vậy I002 đã đóng được chính xác bao nhiêu phần trăm khoảng cách.”

Nhưng phép tính đó không hợp lệ.

Q2 và phép đo mới không có cùng thiết kế thực thi.

Chúng cũng xảy ra ở những trạng thái môi trường khác nhau.

Vì vậy nghiên cứu khóa rõ:

> **Không được trộn đại số số liệu Q2 lịch sử với số liệu mới để tạo một causal estimate.**

Ta có thể nói:

> Q2 cho thấy kiến trúc cũ có khoảng cách rất lớn.

Và:

> phép đối chứng mới cho thấy kiến trúc sau I002 vẫn còn khoảng cách rất lớn.

Nhưng không được lấy hai bảng khác thời điểm rồi suy ra chính xác:

```text
I002 đã loại bỏ X% khoảng cách với llama.cpp
```

nếu experiment không được thiết kế để đo đúng quantity đó.

Đây là một ví dụ khác về **kỷ luật khi đưa ra kết luận**.

## Một chỉ số tốt cũng không cứu toàn bộ bức tranh

bộ nhớ cho một bài học tương tự.

Peak working set của ArcLLM mới so với llama.cpp vẫn khoảng:

```text
1,83×
```

Tức working set cao hơn khoảng 83%.

Nhưng private bytes lại khoảng:

```text
0,97×
```

tức thấp hơn đối chứng vài phần trăm.

Hai con số đi hai hướng khác nhau.

Nếu ta muốn bảo vệ ArcLLM, rất dễ chọn:

> “Private bytes thấp hơn.”

Nếu muốn chỉ trích ArcLLM, lại có thể chọn:

> “Working set cao hơn.”

Cả hai câu riêng lẻ đều bỏ mất bức tranh.

Điều đúng hơn là:

> **Các chỉ số bộ nhớ đang kể những câu chuyện khác nhau; chưa có bằng chứng để ép chúng thành một kết luận bộ nhớ duy nhất.**

Đây chính là lý do một hệ thực thi không nên được đánh giá bằng chỉ số thuận lợi nhất của nó.

## Vậy cải thiện 2× ở Chương 14 có vô nghĩa không?

Không.

Đây là chỗ cần phân biệt hai câu hỏi.

Chương 14 hỏi:

> **Mechanism này có tạo giá trị thật bên trong ArcLLM không?**

Câu trả lời là:

> **Có.**

giai đoạn sinh token và E2E đều cải thiện mạnh, semantics được giữ, TTFT guard ĐẠT (PASS).

Chương 15 hỏi:

> **Sau cải thiện đó, ArcLLM đã ở đâu so với một hệ thực thi trưởng thành?**

Câu trả lời từ phép đo mới là:

> **Khoảng cách vẫn còn lớn.**

Không có mâu thuẫn.

Ta có thể viết:

```text
mechanism có giá trị thật
        ↓
nhưng
        ↓
một mechanism chưa đủ
để giải quyết toàn bộ khoảng cách tới mức trưởng thành
```

Đây chính là ý nghĩa của tiêu đề chương:

> **Một kiến trúc chỉ có giá trị ở cấp toàn hệ khi những lợi ích cục bộ thực sự thay đổi hành vi của toàn hệ — và toàn hệ vẫn phải được đặt trước một đối chứng phù hợp.**

## Thành công có thể làm bằng chứng cũ trở nên lỗi thời

Sau I002, gate/up đã thay đổi rất mạnh.

Trước đó, một profile cho thấy gate/up chiếm gần:

```text
59,67%
```

chi phí GPU giai đoạn sinh token.

Nhưng sau khi phần đó được tăng tốc nhiều lần, tỷ lệ này không còn được phép coi là profile hiện tại.

Hãy lấy một ví dụ đơn giản.

Ban đầu hệ thống mất:

```text
100 ms
```

trong đó phần A mất:

```text
60 ms
```

và phần còn lại:

```text
40 ms
```

Nếu A nhanh hơn 3×:

```text
60 / 3
= 20 ms
```

thì tổng mới:

```text
20 + 40
= 60 ms
```

Lúc này A không còn chiếm 60%.

Nó chỉ còn:

```text
20 / 60
≈ 33%
```

nút thắt — **nơi chiếm chi phí lớn nhất** — có thể đã chuyển sang phần khác.

Đây là một nguyên tắc rất quan trọng:

> **Mỗi tối ưu lớn đều có thể làm bản đồ nút thắt cũ hết hạn.**

Vì vậy sau I003, quyết định khoa học không phải:

> “Gate/up xong rồi, tối ưu LM head tiếp.”

Cũng không phải:

> “Theo profile cũ thì FFN-down đứng thứ ba, làm nó đi.”

Bằng chứng lịch sử không được dùng như một danh sách việc cần làm.

## Bước đúng tiếp theo đôi khi chỉ là đo lại

I003 khóa trước rằng nếu khoảng cách bên ngoài vẫn lớn, bước tiếp theo phải là:

> **profile lại chính kiến trúc sau I002.**

Không chọn chương trình GPU mới trước.

Không lấy ranking cũ.

Không để AI nhìn danh sách chương trình GPU rồi chọn phần tiếp theo theo cảm giác.

Lý do rất đơn giản:

```text
ta vừa thay đổi hệ thống
↓
phân bố chi phí có thể đã đổi
↓
bằng chứng cũ về nút thắt có thể đã lỗi thời
↓
phải đo lại
```

**lỗi thời (stale)** — đã cũ đến mức không còn đủ an toàn để dùng như trạng thái hiện tại.

Một success vì vậy không chỉ tạo ra hiệu năng.

Nó còn có thể **phá hiệu lực của measurement cũ**.

Đó là một điều rất dễ quên.

## Đây là lúc vòng E/M/C/T bắt đầu lại

Ở Chương 12, E/M/C/T trông giống một đường thẳng:

```text
E
↓
M
↓
C
↓
T
```

Đến đây ta thấy hình ảnh đầy đủ hơn.

Nó là một vòng:

```text
E
khám phá
↓
M
khóa cơ chế
↓
C
xác nhận
↓
T
chuyển vào hệ thật
↓
hệ thống thay đổi
↓
ĐO LẠI
↓
E mới
```

Mode T thành công không cho phép ta cứ thế chọn optimization tiếp theo.

Nó tạo ra một **trạng thái hệ thống mới**.

Và trạng thái mới cần evidence mới.

## Đây cũng là cách làm việc với AI khác với “cứ để tác nhân AI làm tiếp”

Hãy tưởng tượng ta giao mục tiêu:

> “Làm ArcLLM nhanh nhất có thể.”

Một AI có khả năng **viết mã** tốt có thể chạy một vòng rất hấp dẫn:

```text
profile
↓
chọn nút thắt
↓
sửa
↓
thấy nhanh hơn
↓
chọn nút thắt tiếp theo
↓
sửa tiếp
```

Nếu không có ranh giới, nó có thể làm như vậy rất lâu.

Nhưng một dự án nghiên cứu không chỉ cần tốc độ triển khai.

Nó cần biết:

> **Khi nào evidence cũ hết hạn?**

> **Khi nào một ĐẠT (PASS) chỉ là local ĐẠT (PASS)?**

> **Khi nào cần phép đo so sánh lại với thế giới bên ngoài?**

> **Khi nào không được chọn bước tiếp theo từ lịch sử?**

> **Khi nào phải dừng một nhánh dù AI còn nghĩ ra được rất nhiều cách sửa?**

Đó là phần con người không nên giao đi một cách vô điều kiện.

## Vai trò của con người không phải cạnh tranh viết mã với AI

Qua bốn chương của Phần III, ta có thể nhìn vai trò hai phía rõ hơn.

AI rất phù hợp để đọc hàng nghìn dòng source, sinh phương án thử, dựng harness, chạy QA, tính tỷ lệ, kiểm tra artifact, tìm mismatch và tổng hợp evidence.

Nhưng quyền quyết định khoa học nằm ở những ranh giới khác:

```text
câu hỏi nào đáng hỏi?

kết luận nào đang được kiểm tra?

điều gì phải khóa trước outcome?

FAIL có được chấp nhận không?

PASS này có phạm vi tới đâu?

evidence nào đã trở nên cũ?

bước tiếp theo cần thêm implementation
hay cần thêm measurement?
```

Đây là lý do làm việc với AI không đồng nghĩa:

> “để AI tự tối ưu.”

Một cách chính xác hơn là:

> **dùng AI để làm tăng năng lực thực thi, trong khi con người giữ quyền định nghĩa câu hỏi và quyền thay đổi niềm tin dựa trên bằng chứng.**

## Nhìn lại toàn bộ Phần III

Ta bắt đầu Phần III với rất nhiều khả năng.

Chương 12 cho phép AI và con người mở rộng không gian ý tưởng, nhưng dùng Mode E và M để không biến mọi ý tưởng thành một experiment.

Chương 13 cho thấy Mode C có thể giết một hướng ngay trước hiệu năng: Q4 ĐẠT (PASS) không có nghĩa Q6 cũng vậy.

Chương 14 cho thấy Mode T: một cơ chế chỉ thực sự có giá trị khi hiệu ứng sống sót từ phép thử nhỏ tới mô hình thật và E2E.

Chương 15 đưa câu chuyện ra ngoài ArcLLM.

Sau một improvement nội bộ khoảng 2×, phép đối chứng mới vẫn cho thấy khoảng cách bên ngoài xấp xỉ một bậc độ lớn ở giai đoạn sinh token và E2E.

Ta vì vậy đi từ:

```text
“có ý tưởng hay”
```

tới:

```text
“thành phần nhanh”
```

rồi:

```text
“hệ thực thi nội bộ nhanh hơn”
```

và cuối cùng:

```text
“nhưng toàn hệ vẫn còn một khoảng cách lớn”
```

Mỗi bước đều cần một loại evidence khác.

## Một kết quả tốt không phải tín hiệu để tăng tốc độ ra quyết định

Có lẽ đây là bài học quan trọng nhất của Phần III.

Khi experiment KHÔNG ĐẠT (FAIL), ta thường biết phải thận trọng.

Nhưng khi experiment ĐẠT (PASS) rất đẹp, ta dễ mất cảnh giác hơn.

Một ĐẠT (PASS) `5×`.

Một khả năng giữ lợi ích khi đi lên toàn hệ `2×`.

Một E2E cải thiện rõ rệt.

Tất cả đều tạo cảm giác:

> “Đúng hướng rồi, tiếp tục thật nhanh.”

Nhưng chính lúc đó ta càng cần hỏi:

> **Hệ thống bây giờ đã khác trước. Ta còn biết nút thắt hiện tại ở đâu không?**

Nếu câu trả lời là:

> “Chưa.”

thì bước tiếp theo không phải optimization.

Nó là measurement.

## Phần III kết thúc ở đây

Bốn mode E/M/C/T đã đi hết một vòng hoàn chỉnh:

```text
E — Explore
mở không gian ý tưởng

↓

M — Mechanism
khóa một cơ chế có thể bị kiểm tra

↓

C — Confirm
để bằng chứng mới quyết định PASS/FAIL

↓

T — Transfer
xem hiệu ứng có sống sót trong hệ thống thật

↓

MEASURE AGAIN
vì hệ thống vừa thay đổi
```

Và câu chuyện quản trị với AI cũng hội tụ về một nguyên tắc:

> **AI có thể làm cho việc tạo, triển khai và kiểm tra phương án nhanh hơn rất nhiều. Nhưng tốc độ nghiên cứu không được đo bằng số experiment chạy được. Nó được đo bằng tốc độ ta loại bỏ điều sai mà vẫn giữ được khả năng biết vì sao.**

ArcLLM đã cải thiện đáng kể.

Nhưng phép đối chứng mới cho thấy nó vẫn còn một khoảng cách lớn.

Vì vậy cuốn sách chuyển sang một loại câu hỏi khác.

Không còn chỉ:

> “chương trình GPU nào nên nhanh hơn?”

Mà bắt đầu hỏi:

> **hệ thực thi cần được tổ chức thành những ranh giới nào để ta có thể thay đổi một phần, đo một phần và vẫn biết chính xác điều gì đã tạo ra kết quả?**

Đó là nơi Phần IV bắt đầu.

### Nhớ 3 điều

1. **Nhanh hơn chính mình không đồng nghĩa đã gần đối chứng.** Sau khả năng giữ lợi ích khi đi lên toàn hệ khoảng `2,20×` giai đoạn sinh token và `2,00×` E2E, phép đối chứng mới vẫn cho thấy ArcLLM có giai đoạn sinh token độ trễ khoảng `10,38×` và E2E độ trễ khoảng `9,97×` đối chứng trong các cell đã đo.
2. **Một tối ưu lớn làm evidence cũ về nút thắt có thể hết hạn.** Sau khi gate/up thay đổi mạnh, không được dùng ranking lịch sử để tự động chọn chương trình GPU tiếp theo; phải profile lại hệ thống mới.
3. **E/M/C/T là một vòng, không phải đường chạy tới ĐẠT rồi kết thúc.** Khi bước kiểm tra toàn hệ làm hệ thống thay đổi, **phép đo mới** lại mở một vòng **Khám phá** mới. Con người giữ quyền quyết định câu hỏi, phạm vi kết luận và lúc dừng; AI giúp mở rộng năng lực thực thi và kiểm tra.

> **Phần III kết thúc tại đây.**
>
> Ta đã học cách nghĩ ra nhiều phương án mà không chạy tất cả, chấp nhận KHÔNG ĐẠT (FAIL) mà không cứu kết quả, chuyển một ĐẠT (PASS) nhỏ vào hệ thống thật, và cuối cùng đặt chính thành công đó trở lại trước một phép đối chứng mới.
