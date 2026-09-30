# Chương 12 — Nhiều cách đều có lý, nhưng chỉ thực tế mới trả lời

> **Mức đọc: Nghiên cứu**
>
> **Bạn đang ở bước nào của hành trình nghiên cứu?**
>
> ```text
> Nhiều ý tưởng
>     ↓
> khám phá rộng
>     ↓
> [ chọn một cơ chế đủ rõ ]
>     ↓
> thiết kế phép thử
> ```


> **Câu hỏi của chương:** Khi AI có thể nghĩ ra rất nhiều cách làm một phép tính nhanh hơn, làm thế nào biết ý tưởng nào thực sự đáng đưa vào kiến trúc?

Chương 11 kết thúc ở một trạng thái khá đặc biệt.

Kiến trúc ArcLLM cũ đã đóng.

Nhưng một giả thuyết mới đã đủ cơ sở để được nghiên cứu:

> **Có thể đường giai đoạn sinh token hiện tại đang sử dụng GPU kém hiệu quả vì cách nó chia phép nhân Q4_K/Q6_K của từng token.**

Phần cứng cũng đã được kiểm tra.

Intel Arc 140V thực sự có những khả năng cần thiết để thử những cách phân chia công việc mới.

Nhưng từ đây xuất hiện một vấn đề khác.

Có rất nhiều cách chia công việc.

Ta có thể chia theo hàng đầu ra, chia chiều K, thay kích thước nhóm GPU, gộp nhiều phép tính, dùng nhóm con GPU, thử ma trận phối hợp (cooperative matrix), thay cách giải mã trọng số hoặc kết hợp nhiều thay đổi cùng lúc.

AI có thể tiếp tục sinh thêm phương án gần như vô hạn.

Vì vậy câu hỏi không còn là:

> “Ta còn nghĩ ra được gì nữa?”

Mà phải là:

> **“Trong rất nhiều ý tưởng có vẻ hợp lý, ý tưởng nào đáng tiêu bằng chứng mới để kiểm tra?”**

## Nhắc lại bốn chế độ làm việc E/M/C/T

Ở Chương 8, ta đã gặp bốn chế độ nghiên cứu. Sau bốn chương, nên nhắc lại chúng trước khi đi tiếp.

```text
E — Explore
Khám phá
→ nghĩ rộng, tìm những vùng và ý tưởng đáng xem xét

M — Mechanism
Kiểm tra cơ chế
→ thu hẹp còn một cơ chế đủ rõ để thử

C — Confirm
Xác nhận
→ khóa trước điều kiện PASS/FAIL rồi dùng bằng chứng mới

T — Transfer
Chuyển sang hệ thống thật
→ xem lợi ích có còn tồn tại khi đi ra khỏi phép thử nhỏ
```

Có thể hình dung:

```text
E
nghĩ rộng
↓
M
khóa một cơ chế
↓
C
xác nhận bằng bằng chứng mới
↓
T
kiểm tra trong hệ thống lớn hơn
```

Chương này chủ yếu đi qua **E và M**.

Chương 13 sẽ cho thấy **C** trở nên quan trọng thế nào khi một cơ chế nhanh nhưng không giữ được tính đúng.

Chương 14 sau đó sẽ đi sâu vào **T**: một kết quả tốt ở thành phần có sống sót khi bước vào mô hình thật và toàn bộ hệ thực thi hay không.

Bốn mode không phải bốn loại công việc bắt buộc phải tách thành bốn dự án.

Chúng là bốn cách tư duy giúp ta biết:

> **Ở thời điểm này, ta đang được phép kết luận đến đâu?**

## AI làm cho ý tưởng trở nên rất rẻ

Trước thời kỳ AI hỗ trợ lập trình, nghĩ ra rồi triển khai mười phiên bản chương trình GPU có thể tốn rất nhiều thời gian.

Bây giờ chi phí đó giảm mạnh.

Ta có thể nói:

> “Thử chia K cho 32 lane.”

AI có thể giúp viết shader.

Ta nói:

> “Thử kích thước nhóm khác.”

Một phiên bản khác có thể xuất hiện.

Ta nói:

> “Gộp thêm hai phép tính.”

Lại thêm một phương án.

Điều này rất mạnh.

Nhưng nó tạo ra một nghịch lý:

> **Khi việc tạo phương án trở nên rẻ, khả năng lựa chọn đúng phương án lại trở nên đắt giá hơn.**

Thứ đắt không còn chỉ là **mã nguồn**.

Thứ đắt là:

> **bằng chứng mới — bằng chứng mới chưa bị dùng để lựa chọn chính giả thuyết đang cần kiểm tra.**

Một phép đo mới có thể tiêu thời gian máy, thời gian review, một cơ hội xác nhận độc lập và quan trọng nhất là ranh giới giữa:

```text
dự đoán trước khi thấy kết quả
```

với:

```text
giải thích sau khi đã thấy kết quả
```

Vì vậy không phải mọi ý tưởng AI sinh ra đều xứng đáng được chạy.

## E — Khám phá: được phép nghĩ rộng, nhưng chưa được tin

**chế độ E — Khám phá — Explore, chế độ khám phá** là nơi AI có thể phát huy khả năng mở rộng không gian ý tưởng.

Ta có thể hỏi:

> Có những cách nào để chia phép nhân batch-1 cho GPU?

Các hướng có thể bao gồm:

```text
một lane làm cả hàng

nhiều lane cùng làm một hàng

chia chiều K

chia đầu ra thành các khối xử lý

dùng nhóm con GPU

dùng ma trận phối hợp (cooperative matrix)

gộp các phép chiếu

thay cách giải mã trọng số
```

AI có thể đọc source, thống kê shape, tra khả năng phần cứng, đối chiếu bài báo khoa học và đưa ra nhiều cách phân rã.

Nhưng chế độ E — Khám phá có một ranh giới:

> **Một ý tưởng xuất hiện trong chế độ E — Khám phá mới chỉ là ứng viên. Nó chưa được quyền tiêu bằng chứng xác nhận.**

chế độ E — Khám phá là nơi nghĩ rộng.

Không phải nơi kết luận rộng.

## M — Kiểm tra cơ chế: giữ lại đúng một cơ chế

Bước tiếp theo là **chế độ M — Kiểm tra cơ chế — Mechanism qualification, kiểm tra xem một cơ chế có đủ rõ để đáng thử hay không**.

Thay vì hỏi:

> “Làm giai đoạn sinh token nhanh hơn thế nào?”

ta chọn một thay đổi rất cụ thể.

Đường cũ có thể hình dung:

```text
một đơn vị công việc GPU
↓
phụ trách một hàng đầu ra
↓
tự đi qua toàn bộ chiều K
↓
cộng dần
↓
ghi kết quả
```

Trong GPU, một đơn vị công việc nhỏ như vậy thường được gọi là **invocation — một lần thực thi nhỏ bên trong chương trình GPU**.

Với một phép nhân có:

```text
K = 3584
```

một invocation phải tự đi qua toàn bộ 3.584 vị trí K.

Một cơ chế khác được chọn:

> **Thay vì để một invocation tự làm cả chiều K, cho 32 lane GPU cùng chia phần việc của một hàng đầu ra.**

Đó là **Split-K — chia chiều K của phép nhân cho nhiều đơn vị tính toán cùng xử lý**.

Cách phân chia được khóa:

```text
1 nhóm con GPU
= 32 lane

1 nhóm con GPU
→ phụ trách 1 hàng đầu ra
```

Nếu:

```text
K = 3584
```

và có:

```text
32 lane
```

thì về số lượng phần tử trung bình:

```text
3584 / 32
= 112
```

Mỗi lane tương đương xử lý khoảng 112 vị trí K.

Sau đó các kết quả từng phần phải được cộng lại.

Bước đó gọi là:

> **nhóm con GPU reduction — phép gom và cộng kết quả giữa các lane trong cùng nhóm con GPU.**

Hình ảnh trực giác chuyển từ:

```text
1 người
làm 3584 việc nối tiếp
```

sang:

```text
32 người
chia 3584 việc

≈ 112 việc/người
↓
gom kết quả
```

Điều này **không có nghĩa nhanh hơn 32 lần**.

32 lane vẫn phải đọc dữ liệu, giải mã Q4_K, nhân số, cộng kết quả và phối hợp với nhau.

Nhưng cơ chế đã đủ rõ.

Ta biết chính xác thứ đang thay đổi:

> **work partitioning — cách chia công việc.**

Không đổi mô hình.

Không đổi định dạng Q4_K.

Không gộp chương trình GPU.

Không thử nhiều kích thước nhóm con GPU cùng lúc.

Không thêm bước giải mã toàn bộ trọng số ra một **vùng nhớ khác**.

Đó chính là chế độ M — Kiểm tra cơ chế:

> **Giữ một cơ chế đủ hẹp để nếu kết quả thay đổi, ta còn biết thứ gì đã tạo ra thay đổi đó.**

## Phép thử thành phần cho một tín hiệu rất mạnh

Cơ chế Split-K được kiểm tra trước trên các phép nhân Q4_K thành phần.

Chưa chạy toàn mô hình.

Chưa tuyên bố tốc độ toàn hệ thực thi.

Câu hỏi chỉ là:

> **Cùng shape, cùng dữ liệu và cùng phép tính, cách chia K mới có nhanh hơn chương trình GPU cũ mà vẫn giữ kết quả số học trong ngưỡng đã khóa không?**

Có năm shape Q4_K đại diện cho các phép chiếu thật trong decoder.

Hai process độc lập được chạy.

Ngưỡng tổng hợp đã khóa trước:

```text
mức tăng tốc >= 1,50×
```

và từng shape riêng lẻ phải đạt ít nhất:

```text
>= 1,10×
```

Kết quả:

```text
Process A
≈ 3,137×

Process B
≈ 3,148×
```

Ngay cả shape tăng ít nhất cũng khoảng:

```text
1,66× → 1,67×
```

Tính đúng cũng ĐẠT (PASS).

Đây là một kết quả rất mạnh.

Nhưng câu hợp lệ chỉ là:

> **Cách chia K bằng nhóm con GPU 32 lane làm các phép nhân Q4_K thành phần được thử chạy nhanh hơn đáng kể.**

Không được nhảy thành:

> “ArcLLM nhanh hơn 3×.”

Hai câu đó khác nhau hoàn toàn.

## Con số đẹp rất dễ đánh lừa

`3,14×` là một con số hấp dẫn.

Phản xạ tự nhiên là:

> “Đưa ngay vào hệ thực thi.”

Nhưng nghiên cứu phải hỏi thêm:

> **Phần vừa nhanh hơn chiếm bao nhiêu trong tổng thời gian của hệ thống?**

Giả sử một chương trình mất:

```text
100 giây
```

và phần ta vừa tối ưu chỉ chiếm:

```text
1 giây
```

Ngay cả khi làm phần đó nhanh vô hạn, toàn chương trình vẫn còn gần:

```text
99 giây
```

mức tăng tốc toàn hệ tối đa:

```text
100 / 99
≈ 1,01×
```

Tức chỉ khoảng 1%.

Đây là trực giác của **định luật Amdahl (Amdahl's Law)**.

## Amdahl hỏi phần được tối ưu thực sự lớn tới đâu

Nếu một phần chiếm tỷ lệ:

```text
f
```

trong tổng thời gian, và phần đó được làm nhanh hơn:

```text
s lần
```

thì giới hạn tăng tốc lý tưởng của toàn hệ là:

```text
S = 1 / ((1 - f) + f/s)
```

Ví dụ phần cần tối ưu chiếm:

```text
f = 60%
= 0,6
```

và ta làm nó nhanh hơn:

```text
s = 3×
```

thì:

```text
S
= 1 / ((1 - 0,6) + 0,6/3)

= 1 / (0,4 + 0,2)

= 1 / 0,6

≈ 1,67×
```

thành phần nhanh hơn 3×.

Nhưng toàn hệ lý tưởng chỉ khoảng 1,67×.

Từ đây câu hỏi quan trọng không còn là:

> “chương trình GPU nhanh bao nhiêu?”

Mà là:

> **“Nếu chương trình GPU nhanh hơn, toàn hệ còn bao nhiêu chỗ để hưởng lợi?”**

## Bằng chứng cũ giúp ta biết nơi nào còn dư địa

Q2 cho thấy ở bài đo ngắn W-S, phần thời gian sau token đầu tiên chiếm khoảng:

```text
99,1%
```

tổng E2E độ trễ.

Ở W-C:

```text
83,8%
```

Tức trong hai bài đo này, phần lớn thời gian nằm **sau khi token đầu tiên xuất hiện**.

Nếu tưởng tượng một cách phi thực tế rằng TTFT — thời gian chờ token đầu tiên — có thể được xóa hoàn toàn, giới hạn cải thiện E2E chỉ khoảng:

```text
W-S
≈ 1,009×

W-C
≈ 1,193×
```

Trong khi vùng sau TTFT còn chiếm phần lớn thời gian.

Khoảng còn có khả năng tạo tác động đó gọi là:

> **headroom — khoảng không còn để cải thiện trước khi những phần khác của hệ thống trở thành giới hạn mới.**

Headroom không hứa rằng ta sẽ lấy được phần lợi ích đó.

Nó chỉ giúp trả lời:

> **Nếu thành công, vùng này có đủ lớn để đáng nghiên cứu không?**

## Một kiến trúc tích hợp cho ta bài học khó hơn

Sau đó, các thay đổi giai đoạn sinh token được đưa vào một kiến trúc kế tiếp lớn hơn.

giai đoạn sinh token thực sự tốt hơn.

Trong bốn phép so sánh mới:

```text
≈ 1,22×
→
3,20×
```

E2E độ trễ cũng tốt hơn:

```text
≈ 1,14×
→
2,51×
```

Nếu chỉ nhìn hai dòng này, rất dễ nói:

> “Kiến trúc mới ĐẠT (PASS).”

Nhưng tiêu chuẩn đã khóa còn một điều kiện:

> **TTFT không được xấu đi quá 10%.**

Tỷ lệ TTFT ở bốn cell là:

```text
1,57×
1,34×
1,31×
1,07×
```

Ngưỡng tối đa:

```text
1,10×
```

Ba trong bốn cell vi phạm.

Vì vậy kết luận tổng thể vẫn là:

> **KHÔNG ĐẠT (FAIL).**

Một kiến trúc có thể:

```text
sinh token nhanh hơn
+
E2E tốt hơn
```

mà vẫn:

```text
FAIL
```

bởi nó tạo ra một cái giá mới vượt tiêu chuẩn đã khóa ở nơi khác.

Thực tế không quan tâm sơ đồ kiến trúc đẹp đến đâu.

Nó chỉ trả lời bằng hành vi của cả hệ thống.

## Khám phá và kiểm tra cơ chế bảo vệ dự án khỏi chính khả năng sinh ý tưởng của AI

Nếu không có ranh giới, AI có thể nhìn KHÔNG ĐẠT (FAIL) vừa rồi rồi lập tức đề xuất:

```text
tối ưu TTFT
đổi vùng nhớ
đổi pipeline
gộp thêm chương trình GPU
thử geometry khác
chạy lại bài đo khác
```

Mỗi ý tưởng đều có thể nghe hợp lý.

Và vì triển khai trở nên rẻ, câu:

> “Thử thêm một chút nữa.”

rất dễ lặp mãi.

chế độ E — Khám phá và M ngăn điều đó.

```text
chế độ E — Khám phá
→ được nghĩ rộng

chế độ M — Kiểm tra cơ chế
→ phải khóa một cơ chế rõ ràng

sau khi phép thử đã chạy
→ không được biến cùng một experiment
  thành chuỗi cứu kết quả vô tận
```

AI không thiếu ý tưởng.

Vấn đề ngược lại mới nguy hiểm:

> **AI có thể tạo nhiều giả thuyết hơn số bằng chứng mà ta có thể kiểm tra nghiêm túc.**

Quyền quyết định của con người vì vậy không nhất thiết nằm ở việc tự viết shader.

Nó nằm ở việc quyết định:

> Câu hỏi nào đáng tiêu evidence?

> Ta đang thay một cơ chế hay năm thứ cùng lúc?

> KHÔNG ĐẠT (FAIL) này có được phép đứng yên không?

> Local mức tăng tốc có đủ headroom để đáng đi tiếp không?

## Sau nhiều phép thử, điều nhận được là một bản đồ

Giai đoạn này không tạo ra một “kiến trúc hoàn hảo”.

Nó tạo ra một bản đồ ngày càng rõ:

```text
CPU submit overhead
→ không phải lời giải chính

chỉ giảm số lần giao việc cho GPU
→ không đủ giải thích khoảng cách

chỉ cải thiện TTFT trong hai bài đo này
→ ceiling E2E thấp

Q4 Split-K
→ cơ chế ở cấp thành phần có tín hiệu mạnh

sinh token / phần sau token đầu tiên
→ vùng có headroom toàn hệ lớn
```

Quá trình hội tụ giống:

```text
rất nhiều giả thuyết
↓
một số chết bằng reasoning
↓
một số chết bằng measurement
↓
một số kết quả ĐẠT ở cấp thành phần
↓
một số FAIL khi lên toàn hệ
↓
bản đồ hiểu biết ngày càng rõ
```

Ta bắt đầu biết không chỉ:

> **cái gì có thể nhanh hơn**

mà còn:

> **cái gì đáng nhanh hơn.**

## Đây là cách một dự án với AI bắt đầu trưởng thành

Ở giai đoạn đầu, AI chủ yếu giúp xây.

Đọc GGUF.

Viết shader.

Ghép decoder.

Chạy test.

Từ đây, vấn đề chính không còn là thiếu khả năng triển khai.

Nó là **kỷ luật lựa chọn**.

Workflow trở thành:

```text
con người đặt câu hỏi
↓
AI mở rộng không gian ý tưởng
↓
E — khám phá
↓
evidence cũ loại hướng yếu
↓
M — khóa một cơ chế
↓
AI triển khai phép thử hẹp
↓
C — chỉ được vào sau khi tiêu chuẩn đã khóa
↓
T — chỉ được vào sau khi kết quả nhỏ đủ điều kiện chuyển tiếp
```

Chương này mới đi sâu vào hai bước đầu.

Hai bước tiếp theo sẽ khó hơn.

Bởi một cơ chế có thể rất nhanh.

Nhưng nếu nó không còn tính đúng, hiệu năng thậm chí không được phép lên tiếng.

### Nhớ 3 điều

1. **E/M/C/T là bốn ranh giới của cùng một quá trình nghiên cứu.** Chương 12 tập trung vào E — nghĩ rộng và M — khóa một cơ chế; Chương 13 và 14 lần lượt cho thấy C và T.
2. **AI làm phương án trở nên rẻ, nhưng bằng chứng mới vẫn đắt.** Vì vậy không phải mọi ý tưởng AI sinh ra đều xứng đáng được chạy.
3. **Local mức tăng tốc không phải system value.** Split-K Q4 đạt khoảng `3,14×` ở phép thử thành phần, nhưng Amdahl và một KHÔNG ĐẠT (FAIL) ở cấp hệ thống cho thấy kiến trúc chỉ có giá trị khi toàn hệ thực sự hưởng lợi.
