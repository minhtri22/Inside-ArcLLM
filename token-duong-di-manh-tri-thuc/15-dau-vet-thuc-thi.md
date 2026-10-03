# Chương 15 — Dấu vết thực thi: những dấu chân một token để lại

> **Mức đọc: Nghiên cứu**
>
> **Ta đã đi tới tầng vật lý:**
>
> ~~~text
> token hiện tại
>      ↓
> phép tính logic
>      ↓
> dispatch
>      ↓
> bộ nhớ + GPU + đồng bộ
> ~~~
>
> Bây giờ câu hỏi là: **làm sao ghi lại chuỗi hoạt động đó đủ rõ để ta biết công việc nào chạy, lúc nào và thuộc phần nào của mô hình?**

Hãy tưởng tượng muốn điều tra một chuyến tàu.

Nếu chỉ biết:

> Tàu đến ga lúc 10 giờ.

ta chưa biết nhiều.

Nhưng nếu có bản ghi:

~~~text
09:00 rời ga A
09:18 dừng ga B
09:20 rời ga B
09:47 dừng ga C
10:00 đến ga D
~~~

ta có một **dòng thời gian**.

Trong hệ thống tính toán, ý tưởng tương tự được gọi là **dấu vết thực thi (execution trace)**.

## Một trace ghi lại điều gì?

Tùy công cụ và mục đích, trace có thể chứa:

- công việc nào được gửi;
- thời điểm bắt đầu;
- thời điểm kết thúc;
- thứ tự thực thi;
- điểm đồng bộ;
- danh tính logic mà công việc đó phục vụ.

Ta có thể hình dung:

~~~text
thời gian ─────────────────────────────────────→

dispatch A   [██████]
dispatch B          [███]
barrier                 |
dispatch C               [████████]
dispatch D                        [██]
~~~

Đây chưa phải một biểu đồ thật.

Nó chỉ cho thấy trace biến một chuỗi hoạt động vô hình thành thứ ta có thể quan sát theo thời gian.

## Dấu thời gian

Muốn biết một công việc kéo dài bao lâu, ta cần ghi thời điểm.

Một **dấu thời gian (timestamp)** là một giá trị dùng để đánh dấu một thời điểm trong quá trình thực thi.

Đơn giản nhất:

~~~text
bắt đầu = t1
kết thúc = t2

thời lượng = t2 - t1
~~~

Trong GPU, việc lấy timestamp phải tuân theo cách phần cứng và giao diện lập trình cung cấp.

Không nên giả định mọi timestamp có cùng độ chính xác hay cùng ý nghĩa.

## Đồng bộ là gì?

Một công việc đôi khi phải đợi công việc khác hoàn thành trước khi tiếp tục.

Ta gọi các cơ chế kiểm soát thứ tự như vậy là **đồng bộ (synchronization)**.

Ví dụ:

~~~text
A ghi dữ liệu
↓
phải chờ A hoàn thành
↓
B mới được đọc dữ liệu đó
~~~

Một trace có thể ghi những điểm đồng bộ như **hàng rào (barrier)**.

Điều quan trọng là:

> Có một barrier xuất hiện trong trace không có nghĩa toàn bộ khoảng thời gian giữa hai dispatch đều là “chi phí barrier”.

Ta sẽ thấy ngay một ví dụ.

## Một bước sinh token thật

Trong một phép đo trên GPU tích hợp Intel Arc 140V, một bước sinh token có:

~~~text
469 lần giao việc vật lý
469 lần có timestamp
470 bản ghi barrier
451 phép tính logic được đo
~~~

Ngoài ra:

~~~text
phép tính logic chưa đo          = 0
định danh logic không biết       = 0
dispatch không quy chiếu được    = 0
~~~

Điều này rất quan trọng.

Trong phạm vi trace đó, không có dispatch nào bị rơi khỏi bản đồ logic.

Ta có thể vẽ:

~~~text
469 dispatch vật lý
      ↓
469 có thời gian
      ↓
quy chiếu
      ↓
451 phép tính logic
      ↓
0 công việc không rõ danh tính
~~~

## Vì sao 469 lại quy về 451?

Chương 12 đã cho ta câu trả lời.

Một phép tính logic có thể tương ứng nhiều dispatch.

Ví dụ LM head:

~~~text
1 phép tính logic
↓
19 dispatch vật lý
~~~

Vì vậy số lượng physical dispatch có thể lớn hơn số phép tính logic.

## Trace có bao phủ hết thời gian không?

Trong phép đo đó:

~~~text
device span
= 1.240.766.914 ns

tổng thời lượng các dispatch
= 1.239.803.214 ns
~~~

Phần chênh lệch là:

~~~text
963.700 ns
≈ 0,078% device span
~~~

Một cám dỗ rất lớn là nói:

> “0,078% đó chính là chi phí barrier.”

Nhưng bằng chứng không cho phép.

Khoảng chênh có thể chứa:

- khoảng trống giữa dispatch;
- serialization;
- hiệu ứng của điểm lấy timestamp;
- những chi phí khác chưa được phân loại.

Vì vậy tên đúng hơn là:

> **thời gian thiết bị chưa quy chiếu (unattributed device time)**

chứ không phải “barrier cost”.

Đây là một ví dụ rất đẹp về kỷ luật đặt tên.

## Trace không phải lời giải thích

Khi đã có trace, ta có thể trả lời:

> Công việc nào chạy trước?

> Công việc nào chạy sau?

> Mất bao lâu?

> Dispatch này thuộc operation nào?

Nhưng trace chưa tự trả lời:

> Vì sao operation đó chậm?

Ví dụ:

~~~text
operation X
↓
800 ms
~~~

Trace đã **định vị** một vùng đắt.

Nhưng nguyên nhân có thể là:

- đọc bộ nhớ;
- hình học thực thi;
- phụ thuộc dữ liệu;
- độ song song;
- kernel chưa phù hợp;
- điều kiện khác.

Dấu vết cho biết **ở đâu**.

Cơ chế cần thêm bằng chứng để nói **vì sao**.

## Quy chiếu là chiếc cầu quan trọng nhất

Một timeline GPU thuần túy có thể cho ta:

~~~text
dispatch 1
dispatch 2
dispatch 3
...
~~~

Nhưng người nghiên cứu mô hình muốn biết:

~~~text
attention?
FFN-down?
LM head?
lớp nào?
~~~

Do đó trace hữu ích nhất khi có cầu nối:

~~~text
DẤU VẾT VẬT LÝ
dispatch + timestamp
        ↓
QUY CHIẾU
        ↓
DANH TÍNH LOGIC
operation + lớp + vai trò
~~~

Nhờ vậy ta mới có thể cộng thời gian theo các họ phép tính có ý nghĩa đối với mô hình.

## Và khi cộng lại, thứ gì hiện ra?

Khi nhóm các dispatch theo danh tính logic rồi cộng thời gian, ta có thể tìm thấy:

> phần nào đang chiếm nhiều thời gian nhất?

Những vùng như vậy được gọi là **điểm nóng (hotspot)**.

Ở trace này, kết quả không phân bố đều.

Một vài họ phép tính chiếm phần lớn thời gian.

Đó là Chương 16.

### Nhớ 3 điều

1. **Dấu vết thực thi (execution trace) ghi lại công việc vật lý theo thời gian và có thể quy chiếu chúng về phép tính logic.**
2. **Trace 469 dispatch có 469 timestamp, 451 phép tính logic được đo và không có dispatch không quy chiếu trong phạm vi phép đo.**
3. **Trace trả lời rất tốt câu “thời gian nằm ở đâu”, nhưng chưa tự trả lời “vì sao chậm”.** Ngay cả 0,078% thời gian chưa quy chiếu cũng không được tự tiện gọi là chi phí barrier.

**Tiếp theo: [Chương 16 — Thời gian nằm ở đâu?](16-thoi-gian-nam-o-dau.md)**
