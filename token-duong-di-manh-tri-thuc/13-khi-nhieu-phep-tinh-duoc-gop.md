# Chương 13 — Khi nhiều phép tính được gộp thành một công việc

> **Mức đọc: Đi sâu**
>
> **Chương trước:**
>
> ~~~text
> 1 phép tính logic
> ↓
> nhiều công việc vật lý
> ~~~
>
> **Chương này hỏi chiều ngược lại:**
>
> ~~~text
> nhiều phép tính logic
> ↓
> 1 công việc vật lý?
> ~~~

Câu trả lời là: **có thể**.

Một hệ thực thi không bắt buộc giữ nguyên ranh giới của sơ đồ mô hình khi đưa công việc xuống GPU.

Đôi khi nhiều phép tính được gộp lại.

## Ví dụ đời thường: rửa và cắt rau

Trên danh sách công việc:

~~~text
1. rửa rau
2. để ráo
3. cắt rau
~~~

Đó là ba bước logic.

Nhưng trong một quy trình khác, một máy có thể vừa rửa vừa chuyển rau qua bộ phận cắt trong cùng dây chuyền.

Ta vẫn có thể nói về hai công việc:

~~~text
rửa
cắt
~~~

nhưng ranh giới vật lý giữa chúng không còn giống hai máy độc lập.

GPU cũng có ý tưởng tương tự.

## Gộp phép tính là gì?

**Gộp phép tính (fusion)** là việc kết hợp nhiều phép tính logic vào một đường thực thi vật lý chung.

Ví dụ tưởng tượng:

~~~text
phép tính A
phép tính B
      │
      └──── gộp ────→ một kernel
                         ↓
                    một dispatch
~~~

Lợi ích tiềm năng có thể là:

- giảm số lần giao việc;
- tránh ghi dữ liệu trung gian ra bộ nhớ rồi đọc lại;
- giữ dữ liệu gần đơn vị tính toán hơn;
- giảm một số chi phí đồng bộ.

Nhưng “có thể” không có nghĩa “luôn luôn”.

## Vì sao fusion có thể giúp?

Giả sử hai phép tính nối tiếp:

~~~text
A
↓
ghi kết quả ra bộ nhớ
↓
B đọc lại
↓
B
~~~

Nếu gộp được:

~~~text
A
↓
kết quả trung gian vẫn ở gần phép tính
↓
B
~~~

ta có thể giảm một phần di chuyển dữ liệu.

Đây là trực giác.

Hiệu quả thật còn phụ thuộc:

- kích thước công việc;
- số thanh ghi cần dùng;
- bộ nhớ dùng chung;
- độ song song;
- giới hạn kernel;
- phần cứng.

Fusion cũng có thể làm kernel phức tạp hơn và giảm hiệu quả ở chỗ khác.

## Một dispatch có thể phục vụ nhiều danh tính logic

Nếu fusion xảy ra, hệ quan sát gặp một bài toán mới.

Ta có:

~~~text
dispatch 42
↓
thực hiện
A + B
~~~

Vậy thời gian của dispatch 42 thuộc về:

~~~text
A?
B?
hay cả hai?
~~~

Không thể cứ nhân đôi toàn bộ thời gian cho cả A và B, vì như vậy tổng thời gian logic sẽ lớn hơn thời gian vật lý thật.

Cũng không thể tùy tiện chia 50/50 nếu không có cơ sở.

Đây là lý do **quy chiếu thời gian** trở nên khó hơn khi nhiều phép tính chia sẻ một công việc vật lý.

## Trace thực nghiệm dùng sau này có thấy fusion không?

Trong trace 469 dispatch mà ta sẽ dùng ở Chương 15–16, số **phép tính logic chia sẻ cùng một dispatch** được ghi nhận là:

~~~text
0
~~~

Điều đó có nghĩa:

> **Trace cụ thể đó không cung cấp ví dụ trực tiếp cho trường hợp nhiều phép tính logic chia sẻ một dispatch.**

Đây là một ranh giới quan trọng.

Ta dạy fusion vì nó là một khả năng thực thi có thật và cần được mô hình quan sát tính tới.

Nhưng ta không lấy trace đó rồi nói:

> “Trace đã chứng minh fusion.”

Nó không chứng minh điều đó.

## Vậy tại sao vẫn cần chương này?

Bởi nếu ta xây một cách suy nghĩ chỉ đúng khi:

~~~text
1 phép tính = 1 dispatch
~~~

thì nó sẽ vỡ ngay khi hệ thực thi thay đổi.

Một mô hình quan sát tốt phải chấp nhận cả ba khả năng:

~~~text
1 logic → 1 vật lý

1 logic → nhiều vật lý

nhiều logic → 1 vật lý
~~~

Thậm chí hệ thống phức tạp còn có thể có quan hệ nhiều-nhiều.

## Danh tính logic phải tồn tại độc lập với cách chạy

Giả sử hôm nay FFN-gate và FFN-up chạy riêng:

~~~text
gate → dispatch A
up   → dispatch B
~~~

Ngày mai hệ thực thi gộp:

~~~text
gate + up
     ↓
dispatch C
~~~

Nếu danh tính khoa học của phép đo phụ thuộc hoàn toàn vào “dispatch số mấy”, ta đã mất khả năng so sánh hai phiên bản.

Ta cần giữ:

~~~text
gate
up
~~~

như hai công việc logic ổn định hơn, rồi quan sát cách chúng ánh xạ xuống vật lý ở từng phiên bản.

Đây là giá trị của việc tách:

> **cái mô hình cần làm**

khỏi:

> **phần cứng đang làm nó bằng cách nào**

## Fusion và “mảnh tri thức”

Ở nửa đầu cuốn sách, ta đi theo trạng thái token.

Sang đây, có một bài học mới:

> **Đường đi vật lý không bảo toàn nguyên ranh giới khái niệm mà ta vẽ trên sơ đồ.**

Một trạng thái logic có thể được:

- chia nhỏ;
- gộp công việc;
- di chuyển trong bộ nhớ;
- xử lý theo nhiều hình học khác nhau.

Vì vậy nếu ta muốn “đi theo token” ở phần cứng, ta phải cẩn thận với câu:

> “Token đang ở kernel X.”

Thực tế có thể là:

> một phần công việc phục vụ trạng thái của token đang được thực hiện trong một kernel, cùng với công việc logic khác.

## Tiếp theo: bộ nhớ

Cho tới đây ta chủ yếu nói:

~~~text
phép tính
↓
kernel
↓
dispatch
~~~

Nhưng phép tính cần dữ liệu.

Trọng số phải được đọc.

Trạng thái trung gian phải nằm đâu đó.

Kết quả phải được ghi.

GPU có thể tính rất nhanh nhưng vẫn phải chờ dữ liệu.

Vì vậy đường đi của token không chỉ là đường của **tính toán (compute)**.

Nó còn là đường của **bộ nhớ (memory)**.

### Nhớ 3 điều

1. **Gộp phép tính (fusion) có thể đưa nhiều phép tính logic vào cùng một công việc vật lý.**
2. **Fusion có thể giảm một số chi phí như dispatch hoặc di chuyển dữ liệu trung gian, nhưng không tự động làm hệ thống nhanh hơn.**
3. **Trace 469/451 được dùng trong sách không trực tiếp chứng minh fusion vì trace đó có 0 phép tính logic dùng chung một dispatch; chương này dạy khả năng tổng quát, không gán sai bằng chứng cho trace.**

**Tiếp theo: [Chương 14 — Bộ nhớ cũng nằm trên đường đi của token](14-bo-nho-tren-duong-di-token.md)**
