# Chương 11 — Một phép tính logic không phải một chương trình GPU

> **Mức đọc: Đi sâu**
>
> **Ta đang đổi tầng quan sát:**
>
> ~~~text
> sơ đồ mô hình
>      ↓
> phép tính logic
>      ↓
> [ hệ thực thi biến nó thành công việc thật ]
>      ↓
> GPU
> ~~~
>
> Nửa đầu cuốn sách nói về token trong **mô hình**. Từ chương này, ta nhìn xuống một tầng thấp hơn: **phần cứng thật sự phải làm gì để những phép tính đó xảy ra?**

Ở Chương 10, ta nói:

~~~text
trạng thái cuối
↓
lớp tạo điểm đầu ra
↓
logits
~~~

Trên sơ đồ, “lớp tạo điểm đầu ra” trông giống một chiếc hộp.

Nhưng GPU không nhận một tờ giấy ghi:

> “Hãy chạy LM head.”

Phần cứng nhận những công việc cụ thể hơn.

Đây là chỗ ta phải tách hai thế giới.

## Thế giới thứ nhất: phép tính logic

Khi mô tả mô hình, ta dùng những tên như:

- cơ chế chú ý;
- phép chiếu Q/K/V;
- FFN-up;
- FFN-down;
- lớp tạo điểm đầu ra.

Đó là các **phép tính logic (semantic operations)**.

Từ “semantic” ở đây không có nghĩa “ngữ nghĩa ngôn ngữ”.

Nó chỉ nói rằng:

> **đây là danh tính công việc theo ý nghĩa của đồ thị mô hình.**

Ví dụ:

~~~text
FFN-down ở lớp 12
~~~

là một công việc logic cụ thể.

Ta biết nó đang đóng vai trò gì trong mô hình.

## Thế giới thứ hai: công việc vật lý

GPU cần một chương trình tính toán cụ thể.

Một chương trình nhỏ chạy trên GPU thường được gọi là **chương trình GPU (kernel)**.

Khi hệ thực thi gửi một lần công việc GPU đi chạy, ta gọi đó là một **lần giao việc (dispatch)**.

Ta có thể tạm hình dung:

~~~text
phép tính logic
      ↓
chọn / chuẩn bị chương trình GPU
      ↓
gửi một hoặc nhiều lần giao việc
      ↓
GPU thực thi
~~~

Điều quan trọng là:

> **Một phép tính logic không bắt buộc tương ứng đúng một lần giao việc.**

## Ví dụ đời thường: “chuyển nhà”

Trên danh sách công việc, bạn có một mục:

~~~text
Chuyển tủ sách sang nhà mới
~~~

Đó là **một công việc theo ý nghĩa**.

Nhưng thực tế có thể cần:

~~~text
tháo kệ
↓
chuyển thùng 1
↓
chuyển thùng 2
↓
chuyển khung
↓
lắp lại
~~~

Một dòng trong kế hoạch có thể trở thành nhiều việc vật lý.

Ngược lại, một chuyến xe cũng có thể chở đồ của nhiều mục công việc cùng lúc.

Sơ đồ logic và cách thi công thật không nhất thiết một-một.

## Một phép đo thật cho thấy điều đó

Trong một lần theo dấu **một bước sinh token thật** trên GPU tích hợp Intel Arc 140V, hệ thống ghi được:

~~~text
469 lần giao việc vật lý
451 phép tính logic được đo
~~~

Tất cả 469 lần giao việc đều có dấu thời gian.

Không có lần giao việc nào bị bỏ lại mà không quy chiếu được về công việc logic đã biết trong phạm vi phép đo đó.

Nhưng:

~~~text
469 ≠ 451
~~~

Chỉ riêng sự khác nhau này đã đủ để bác bỏ cách hình dung:

> “Mỗi hộp logic luôn tương ứng đúng một công việc GPU.”

Tuy nhiên ta cũng phải giữ ranh giới:

> **469 và 451 là kết quả của một mô hình, một hệ thực thi, một phần cứng và một lần đo cụ thể.**

Đó không phải con số chung cho mọi LLM.

## Tại sao cần giữ hai danh tính riêng?

Giả sử ta chỉ lưu thông tin vật lý:

~~~text
dispatch 127 mất 3 ms
~~~

Ta biết có một công việc chậm.

Nhưng chưa biết nó thuộc phần nào của mô hình.

Ngược lại, nếu chỉ lưu:

~~~text
FFN-down lớp 12
~~~

ta biết ý nghĩa logic.

Nhưng chưa biết phần cứng đã chia và chạy nó thế nào.

Vì vậy khi muốn quan sát sâu, ta cần một cầu nối:

~~~text
danh tính logic
      ↕
quy chiếu
      ↕
công việc vật lý
~~~

**Quy chiếu (attribution)** là việc gắn một quan sát vật lý trở lại công việc logic mà nó phục vụ.

## Một sơ đồ có hai tầng

Ta có thể vẽ cùng một bước sinh token theo hai cách:

~~~text
TẦNG LOGIC

Attention
   ↓
FFN-up
   ↓
FFN-gate
   ↓
FFN-down
   ↓
LM head
~~~

và:

~~~text
TẦNG VẬT LÝ

dispatch 1
dispatch 2
dispatch 3
...
dispatch 469
~~~

Giữa hai tầng là một bản ánh xạ.

Không có ánh xạ đó, ta chỉ có hai danh sách rời nhau.

## “Chương trình GPU” và “lần giao việc” có giống nhau không?

Không hoàn toàn.

**Chương trình GPU (kernel)** là phần mã tính toán.

**Lần giao việc (dispatch)** là một lần cụ thể mà chương trình đó được gửi đi với một cấu hình công việc cụ thể.

Cùng một kernel có thể được dispatch nhiều lần.

Ví dụ:

~~~text
kernel A
  ↓
dispatch cho khối dữ liệu 1

kernel A
  ↓
dispatch cho khối dữ liệu 2

kernel A
  ↓
dispatch cho khối dữ liệu 3
~~~

Vì vậy khi đếm “bao nhiêu lần GPU nhận việc”, ta đang đếm dispatch, không nhất thiết đếm số kernel khác nhau.

## Tại sao điều này quan trọng khi đi theo token?

Ở nửa đầu sách, ta nói:

> token đi qua lớp 1, lớp 2, lớp 3...

Đó là cách nhìn logic.

Nhưng ở phần cứng, cùng đoạn đường có thể trông như:

~~~text
token hiện tại
↓
hàng chục phép tính logic
↓
hàng trăm lần giao việc vật lý
↓
đọc / ghi bộ nhớ
↓
đồng bộ
↓
tiếp tục
~~~

Nếu muốn hỏi:

> “Token mất thời gian ở đâu?”

ta không thể chỉ nhìn sơ đồ Transformer.

Ta phải biết các phép tính logic đã trở thành công việc vật lý nào.

## Và một phép tính có thể bị chia nhỏ

Trong phép đo vừa nhắc, một phép tính logic đặc biệt ở cuối mô hình không chạy bằng một dispatch duy nhất.

Nó được thực thi bằng **19 lần giao việc vật lý**.

Đây là ví dụ rất rõ cho quan hệ:

~~~text
1 phép tính logic
↓
nhiều công việc vật lý
~~~

Tại sao hệ thực thi lại chia như vậy?

Đó là câu hỏi của Chương 12.

### Nhớ 3 điều

1. **Phép tính logic (semantic operation) mô tả công việc theo ý nghĩa của mô hình; chương trình GPU (kernel) và lần giao việc (dispatch) mô tả cách công việc thật sự được đưa xuống phần cứng.**
2. **Hai tầng không bắt buộc ánh xạ một-một.** Trong một bước sinh token được đo, 451 phép tính logic tương ứng với 469 lần giao việc vật lý.
3. **Muốn biết thời gian của phần cứng thuộc về phần nào trong mô hình, ta cần quy chiếu (attribution) giữa danh tính logic và công việc vật lý.**

**Tiếp theo: [Chương 12 — Một phép tính có thể trở thành nhiều lần giao việc cho phần cứng](12-mot-phep-tinh-nhieu-lan-giao-viec.md)**
