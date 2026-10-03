# Chương 17 — Bộ đếm phần cứng: con số đo được có nghĩa gì?

> **Mức đọc: Nghiên cứu**
>
> **Ta đã biết nơi thời gian tập trung.**
>
> Bây giờ ta muốn hỏi:
>
> ~~~text
> hotspot
>    ↓
> phần cứng đang làm gì?
>    ↓
> [ bộ đếm phần cứng ]
> ~~~

Hãy tưởng tượng bảng đồng hồ trên ô tô.

Nó có thể cho bạn:

- tốc độ;
- vòng tua;
- nhiệt độ;
- mức nhiên liệu.

Những con số đó giúp hiểu chiếc xe đang hoạt động thế nào.

GPU cũng có những phép đo nội bộ tương tự.

Ta thường gọi chúng là **bộ đếm phần cứng (hardware counters)**.

## Bộ đếm có thể đo gì?

Tùy phần cứng và trình điều khiển, bộ đếm có thể liên quan tới:

- số công việc;
- lượng truy cập bộ nhớ;
- tỷ lệ cache;
- mức sử dụng đơn vị tính toán;
- trạng thái chờ;
- tần số;
- số chu kỳ;
- nhiều đại lượng khác.

Nghe rất hấp dẫn.

Ta có hotspot.

Ta đọc counter.

Ta tìm nguyên nhân.

Nhưng thực tế không đơn giản như vậy.

## Con số đi qua một kênh đo

Khi bạn nhìn thấy:

~~~text
counter X = 42
~~~

số 42 không từ “thực tại vật lý” nhảy thẳng vào trang sách.

Nó đi qua một chuỗi:

~~~text
hiện tượng vật lý
      ↓
bộ đếm trong phần cứng
      ↓
khả năng phần cứng cung cấp
      ↓
trình điều khiển
      ↓
giao diện truy vấn
      ↓
công cụ thu thập
      ↓
con số ta nhìn thấy
~~~

Chuỗi này là **kênh đo (measurement channel)**.

Nếu một mắt xích có giới hạn, con số cuối có thể không đủ để trả lời câu hỏi ta muốn hỏi.

## 268 bộ đếm không có nghĩa có 268 câu trả lời

Trong hệ thống dùng làm ví dụ, trình điều khiển công bố:

~~~text
268 bộ đếm
~~~

Thoạt nhìn, đây là rất nhiều dữ liệu.

Nhưng để lấy toàn bộ inventory cần:

~~~text
12 lượt đo
~~~

Không phải mọi counter đều có thể thu cùng lúc.

Và nhiều counter có thể gây chi phí khi đo.

Do đó nguyên tắc tốt là:

> **Không đo mọi thứ chỉ vì có thể đo.**

Hãy chọn các counter liên quan tới giả thuyết đang kiểm tra.

## Đo có thể làm thay đổi hệ đang đo

Đây là một vấn đề phổ biến trong quan sát hệ thống.

Thêm instrumentation có thể:

- tăng thời gian;
- thay cách lập lịch;
- thay tải lên GPU;
- thêm lượt chạy;
- thay trạng thái cache.

Vì vậy **lần chạy có instrument** không nên tự động thay thế phép đo thời gian bình thường.

Có thể hình dung:

~~~text
chạy bình thường
→ đo hiệu năng

chạy có counter
→ chẩn đoán cơ chế
~~~

Hai loại bằng chứng có mục đích khác nhau.

## Khi counter trả về 0

Đây là phần quan trọng nhất của chương.

Trong phép đo thật, một số counter quan trọng không dùng được một cách đáng tin cậy.

Ví dụ:

~~~text
GpuTime = 0
~~~

ở cả ba nhóm counter được dùng.

Một cách đọc vội là:

> “GPU time bằng 0.”

Điều đó vô lý vì GPU rõ ràng đã chạy công việc.

Cách đọc đúng hơn là:

> **Kênh đo cho counter này không đủ để diễn giải giá trị 0 như đại lượng vật lý bằng 0.**

Đây là khác biệt cực kỳ quan trọng:

~~~text
counter = 0
≠
physical quantity = 0
~~~

## Cùng một chỉ số còn có thể bất nhất

Có những counter cho kết quả khác nhau tùy nhóm đo.

Ví dụ trong dữ liệu thật:

~~~text
XVE_STALL

nhóm execution/occupancy → 0
nhóm stall-cause         → khác 0
~~~

Một counter khác:

~~~text
GPU_MEMORY_REQUEST_QUEUE_FULL

nhóm memory/cache → có mẫu khác 0
nhóm stall-cause  → 0
~~~

Nếu các phép đo được dùng để xác nhận một cơ chế, những bất nhất như vậy là vấn đề nghiêm trọng.

## Độ đầy đủ của bằng chứng

Ta cần một khái niệm mới:

> **độ đầy đủ của phép đo (measurement adequacy)**

Câu hỏi không phải chỉ là:

> “Có số không?”

Mà là:

> “Số này có đủ đáng tin và đủ liên quan để phân biệt những cơ chế ta đang so sánh không?”

Một counter có thể:

- tồn tại;
- đọc được;
- trả số;

nhưng vẫn **không đủ** cho câu hỏi khoa học đang kiểm tra.

## FAIL của phép đo không phải FAIL của phần cứng

Trong nghiên cứu này, việc thu thập dữ liệu đã chạy đủ theo kế hoạch.

Nhưng phần **độ đầy đủ của counter** không đạt yêu cầu để xác nhận cơ chế.

Kết luận chính thức là:

~~~text
KHÔNG ĐẠT / CHƯA GIẢI QUYẾT
~~~

Điều đó **không** có nghĩa:

> GPU không có cơ chế nào.

Nó có nghĩa:

> **Bằng chứng hiện có chưa đủ để chọn một cơ chế làm kết luận nhân quả.**

Đây là một dạng tri thức rất quan trọng.

Ta đã biết ranh giới của phép đo.

## Vậy các khuôn mẫu khác có bỏ đi không?

Không.

Dữ liệu vẫn có những khác biệt định hướng thú vị về:

- đọc DRAM;
- cache;
- stall;
- mức sử dụng ALU.

Những khuôn mẫu đó có thể giúp tạo giả thuyết.

Nhưng chúng không được phép vượt qua verdict chính thức.

Ta có:

~~~text
khuôn mẫu
↓
gợi ý giả thuyết
↓
KHÔNG tự động
↓
causal conclusion
~~~

Chương 19 sẽ quay lại đúng câu chuyện này.

## Và nếu hai số chỉ khác nhau một chút?

Counter không phải trường hợp duy nhất ta phải cẩn thận.

Có lúc hai trạng thái gần như giống nhau.

Có lúc phép đo chỉ chênh rất nhỏ.

Bản năng của ta thường là:

> “Nhỏ thế thì bỏ qua.”

Nhưng trước khi bỏ qua, cần hỏi:

> Nhỏ so với độ phân giải nào?

> Có lặp lại không?

> Có cấu trúc không?

Đó là Chương 18.

### Nhớ 3 điều

1. **Bộ đếm phần cứng (hardware counter) đi qua một kênh đo; con số cuối không phải sự thật vật lý trực tiếp không qua trung gian.**
2. **Trong hệ thống được đo có 268 counter và cần 12 lượt để lấy toàn bộ inventory, nhưng một số counter quan trọng vẫn không đủ đáng tin cho câu hỏi nhân quả.**
3. **Counter bằng 0 không tự động nghĩa đại lượng vật lý bằng 0.** Khi kênh đo không đủ, kết luận đúng có thể là **CHƯA GIẢI QUYẾT**, không phải bịa một nguyên nhân.

**Tiếp theo: [Chương 18 — Khi một khác biệt quá nhỏ vẫn đáng để hỏi](18-khac-biet-nho-van-dang-hoi.md)**
