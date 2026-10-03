# Chương 8 — Đường cộng tắt: vì sao thông tin cũ không biến mất hoàn toàn?

> **Mức đọc: Đi sâu**
>
> **Ta đang mở chiếc hộp nào?**
>
> ~~~text
> trạng thái cũ ──────────┐
>                        │
>       phép biến đổi     │
>            ↓           │
>       phần thay đổi ───┘
>            ↓
>      trạng thái mới
> ~~~
>
> Chương này giải thích **đường cộng tắt (residual connection)** — một cấu trúc giúp mỗi lớp thêm thay đổi mới mà không buộc phải thay thế toàn bộ tín hiệu cũ.

Hãy tưởng tượng bạn đang sửa một tài liệu.

Có hai cách.

Cách thứ nhất:

> Xóa toàn bộ trang cũ rồi viết lại từ đầu.

Cách thứ hai:

> Giữ bản cũ, tạo phần chỉnh sửa, rồi áp phần chỉnh sửa đó lên nội dung đang có.

Transformer gần với trực giác thứ hai hơn.

## Một phép cộng rất quan trọng

Ta ký hiệu trạng thái đi vào một khối là:

~~~text
x
~~~

Khối biến đổi tạo ra:

~~~text
F(x)
~~~

Đường cộng tắt tạo:

~~~text
x + F(x)
~~~

Không cần học công thức sâu hơn.

Ý quan trọng là:

~~~text
trạng thái mới
=
trạng thái cũ
+
phần thay đổi
~~~

Đó là lý do từ “residual” thường được dịch gần với ý **phần dư / phần thay đổi còn lại**.

Trong sách, ta dùng cách gọi dễ hiểu hơn:

> **đường cộng tắt (residual connection)**

## Vì sao gọi là “đường tắt”?

Hãy nhìn sơ đồ:

~~~text
             ┌───────────────┐
             │               ↓
trạng thái ──┼→ phép biến đổi ─→ kết quả
             │               │
             └───────────────┘
                     cộng
                      ↓
               trạng thái mới
~~~

Một phần tín hiệu đi qua phép biến đổi.

Một đường khác đi vòng qua phép biến đổi.

Hai đường gặp lại ở phép cộng.

Vì vậy ta có thể nghĩ nó như một “đường tắt”.

## Điều này giúp gì cho câu chuyện của token?

Nhớ rằng ta đang đi theo trạng thái của một token qua nhiều lớp.

Nếu mỗi lớp chỉ thay toàn bộ trạng thái:

~~~text
x0
↓
x1
↓
x2
↓
x3
↓
...
~~~

ta dễ hình dung như mỗi bước viết đè hoàn toàn lên bước trước.

Nhưng với residual:

~~~text
x0
↓
x0 + Δ1
↓
x0 + Δ1 + Δ2
↓
x0 + Δ1 + Δ2 + Δ3
↓
...
~~~

Ký hiệu Δ ở đây chỉ là hình ảnh trực giác cho “phần thay đổi”, không phải công thức chính xác cho mọi lớp.

Nó giúp thấy rằng trạng thái có thể được **cập nhật dần**.

## Attention cũng đi qua residual

Trong một lớp kiểu phổ biến:

~~~text
trạng thái ban đầu
      │
      ├──────────────┐
      ↓              │
  attention          │
      ↓              │
  phần thay đổi      │
      └────── cộng ──┘
             ↓
      trạng thái giữa
~~~

Sau đó FFN lại có một residual khác:

~~~text
trạng thái giữa
      │
      ├──────────────┐
      ↓              │
     FFN             │
      ↓              │
  phần thay đổi      │
      └────── cộng ──┘
             ↓
      trạng thái ra
~~~

Do đó một lớp không đơn giản là:

~~~text
attention → FFN
~~~

mà gần hơn với:

~~~text
cập nhật bằng attention
        +
giữ đường cũ

rồi

cập nhật bằng FFN
        +
giữ đường cũ
~~~

## Residual có nghĩa thông tin cũ luôn được bảo toàn nguyên vẹn không?

Không.

Đây là chỗ cần cẩn thận.

Tín hiệu cũ có một đường đi trực tiếp qua phép cộng.

Nhưng sau nhiều lớp, trạng thái tổng thể vẫn có thể thay đổi rất mạnh.

Các phép cộng tiếp theo, chuẩn hóa và biến đổi tiếp tục tác động.

Vì vậy không nên nói:

> “Residual đảm bảo mọi thông tin cũ không bao giờ mất.”

Câu đúng hơn là:

> **Residual tạo một đường trực tiếp để trạng thái trước tham gia vào trạng thái sau, thay vì buộc mọi thông tin phải đi xuyên qua toàn bộ phép biến đổi.**

## Một ví dụ đời thường khác

Hãy tưởng tượng một bản đồ được chỉnh sửa qua nhiều phiên bản.

Phiên bản 1 có đường phố.

Phiên bản 2 thêm trạm xe buýt.

Phiên bản 3 thêm tuyến tàu.

Phiên bản 4 thêm tình trạng giao thông.

Nếu mỗi bước giữ lại nền cũ rồi thêm lớp thông tin mới, ta có một quá trình tích lũy.

Đường cộng tắt không phải một lớp bản đồ theo nghĩa đen, nhưng trực giác “giữ nền + thêm thay đổi” khá gần với cách ta nên nghĩ về dòng trạng thái.

## Dòng trạng thái tích lũy

Khi residual lặp lại qua nhiều lớp, người ta thường nói tới một **dòng residual (residual stream)**.

Ở mức nhập môn, hãy hiểu:

> **Đó là dòng trạng thái chính được truyền xuyên qua các lớp và liên tục nhận thêm các phần cập nhật.**

Ta có thể vẽ:

~~~text
dòng trạng thái
      ↓
  + attention
      ↓
dòng trạng thái
      ↓
  + FFN
      ↓
dòng trạng thái
      ↓
  sang lớp tiếp
~~~

Đây là một cách nhìn rất quan trọng cho các chương sau.

Vì khi ta hỏi:

> “Một token đã biến dạng thế nào sau lớp 5?”

thứ ta thật sự muốn so sánh không phải chữ token.

Ta muốn nhìn **trạng thái số trên dòng residual** ở các thời điểm khác nhau.

## Câu hỏi mở: vì sao không ghi phần đã học lại vào tệp mô hình để lần sau đi tắt?

Đây là một câu hỏi rất tự nhiên khi ta vừa thấy:

~~~text
x
↓
x + F(x)
↓
x + F(x) + G(...)
↓
...
~~~

Ta có thể nghĩ:

> Nếu mô hình đã từng gặp một ngữ cảnh và đã tạo ra một phần thay đổi hữu ích, tại sao không ghi luôn phần đó xuống tệp mô hình?

Ví dụ trực giác:

~~~text
ban đầu:
x

đã gặp a, b:
x + a + b

sau này gặp thêm c:
x + a + b + c
~~~

Nếu lần sau gặp lại đúng ngữ cảnh, liệu mô hình có thể lấy sẵn:

~~~text
x + a + b + c
~~~

rồi bỏ qua một số lớp?

Ý tưởng này **không vô lý**.

Nhưng có một điểm rất quan trọng cần tách ra.

### Tệp mô hình lưu gì?

Tệp mô hình chủ yếu lưu những thứ đã được học tương đối ổn định như:

- trọng số;
- bảng nhúng;
- cấu trúc cần thiết để dựng mạng;
- một số thông tin cấu hình.

Còn:

~~~text
x
F(x)
x + F(x)
~~~

trong ví dụ của chương này là **trạng thái đang được tạo ra trong lúc chạy**.

Nó phụ thuộc vào:

- token hiện tại;
- vị trí;
- toàn bộ ngữ cảnh đang có;
- trạng thái từ lớp trước;
- trọng số của chính lớp đang chạy.

Vì vậy:

~~~text
trọng số trong tệp mô hình
≠
trạng thái tạm thời của một lần chạy
~~~

### Vì sao không thể đơn giản viết thành x + a + b + c?

Bởi các phần thay đổi không hoàn toàn độc lập.

Lớp sau không chỉ nhận lại x ban đầu.

Nó nhận **trạng thái đã bị lớp trước thay đổi**.

Ví dụ đơn giản hơn:

~~~text
x1 = x0 + F1(x0)

x2 = x1 + F2(x1)

x3 = x2 + F3(x2)
~~~

Chú ý:

~~~text
F2 nhận x1
không phải x0

F3 nhận x2
không phải x0
~~~

Do đó ta không thể mặc định viết:

~~~text
x3 = x0 + a + b + c
~~~

với a, b, c là ba mảnh cố định có thể lấy ra độc lập rồi cộng lại ở bất kỳ ngữ cảnh nào.

Phần thay đổi ở lớp sau phụ thuộc vào kết quả của lớp trước.

Nói cách khác:

> **Quá trình có tính phụ thuộc đường đi.**

### Cùng một token nhưng context khác thì phần thay đổi cũng khác

Giả sử token là:

~~~text
đá
~~~

Trong:

~~~text
hòn đá
~~~

và:

~~~text
đá bóng
~~~

trạng thái đi vào các lớp đã khác nhau.

Vì vậy F(x) cũng có thể khác.

Nếu ta ghi mọi trạng thái theo mọi context vào tệp mô hình, số trường hợp có thể tăng cực lớn.

Ta sẽ dần biến tệp mô hình thành một kho chứa vô số trạng thái từng gặp.

### Nhưng có cách nào gần với ý tưởng này không?

Có.

Trong hệ thống hiện đại đã có những kỹ thuật mang tinh thần tương tự, nhưng chúng không đơn giản là “ghi x + F(x) vào weights”.

Ví dụ:

~~~text
cùng prefix đã xử lý
↓
giữ lại trạng thái trung gian cần thiết
↓
không tính lại toàn bộ prefix
~~~

Đó là trực giác của **bộ nhớ đệm khóa–giá trị (KV cache)** và một số dạng **bộ nhớ đệm phần đầu vào (prefix cache / prompt cache)**.

Ngoài ra còn có những hướng khác như:

- bộ nhớ ngoài (external memory);
- trọng số thay đổi nhanh (fast weights);
- học trong lúc chạy (online learning);
- thích nghi ở thời điểm suy luận (test-time adaptation);
- kiến trúc cho phép bỏ qua một số lớp khi đã đủ tự tin.

Nhưng mỗi hướng đều cần một cơ chế riêng để trả lời:

~~~text
lưu cái gì?
↓
khi nào được dùng lại?
↓
ngữ cảnh nào được coi là đủ giống?
↓
dùng lại có còn đúng không?
↓
có làm mô hình thay đổi ngoài ý muốn không?
~~~

### Một ranh giới rất quan trọng

Nếu chỉ **cache trạng thái của một context đã xử lý**, ta đang tránh tính lại một phần công việc.

Nếu **ghi ngược điều mới học vào trọng số của model**, ta đang thay đổi chính mô hình.

Hai việc này khác nhau.

~~~text
CACHE
→ nhớ kết quả của một lần xử lý

UPDATE MODEL
→ thay đổi thứ mô hình đã học
~~~

## Đây có phải nơi “mảnh tri thức” chạy qua?

Có thể nói đây là một nơi rất quan trọng để theo dõi sự thay đổi.

Nhưng vẫn không nên kết luận:

> “Dòng residual chính là tri thức.”

Nó là một cấu trúc số mà nhiều thành phần của mô hình liên tục đọc, biến đổi và cập nhật.

Tri thức mà mô hình thể hiện còn phụ thuộc vào trọng số, ngữ cảnh, kiến trúc và cách đầu ra được tạo.

Ta đang theo dấu **trạng thái**, không tìm một viên tri thức vật lý.

## Nhiều residual nối tiếp nhau sẽ tạo ra gì?

Ta đã có:

~~~text
lớp 1
↓
cập nhật trạng thái

lớp 2
↓
cập nhật tiếp

lớp 3
↓
cập nhật tiếp
~~~

Nhưng tại sao cần nhiều lớp như vậy?

Một lớp có thể làm ít hay nhiều đến đâu?

Có phải lớp 1 học từ, lớp 2 học câu, lớp 3 học tri thức?

Câu trả lời không đơn giản như vậy.

Đó là nội dung của Chương 9.

### Nhớ 3 điều

1. **Đường cộng tắt (residual connection) cho phép trạng thái cũ đi trực tiếp tới phép cộng với phần thay đổi mới.**
2. **Residual không đảm bảo mọi thông tin được giữ nguyên mãi mãi; nó chỉ tạo một đường trực tiếp để trạng thái trước tham gia vào trạng thái sau.**
3. **Dòng residual (residual stream) là một cách hữu ích để nghĩ về trạng thái số đang được cập nhật xuyên qua nhiều lớp.**

**Tiếp theo: [Chương 9 — Một lớp chưa biết cả câu — vậy nhiều lớp làm được gì?](09-nhieu-lop-lam-duoc-gi.md)**
