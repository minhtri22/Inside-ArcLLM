# Chương 2 — Token cũng không phải là tri thức

> **Mức đọc: Nền tảng**
>
> **Ta đã đi tới đâu?**
>
> ~~~text
> văn bản
>    ↓
> token
>    ↓
> mã token
>    ↓
> ?
> ~~~
>
> Chương này hỏi: **một token có tự mang sẵn ý nghĩa và tri thức của nó hay không?**

Hãy nhìn một token quen thuộc:

~~~text
Paris
~~~

Nếu tôi hỏi:

> Paris là gì?

bạn có thể nghĩ ngay tới thủ đô của Pháp, tháp Eiffel, một thành phố ở châu Âu, sông Seine, lịch sử hay du lịch.

Chỉ một chuỗi ký tự ngắn đã kéo theo rất nhiều liên tưởng.

Vì vậy rất tự nhiên nếu ta hình dung:

> Trong mô hình, token “Paris” chắc cũng mang theo một gói thông tin giống vậy.

Nhưng nếu hiểu theo nghĩa đen, cách hình dung này gây rắc rối.

## Một mã số không thể tự chứa tất cả những điều đó

Ở cuối Chương 1, ta đã thấy token được gắn một mã số:

~~~text
"Paris"
   ↓
mã token
   ↓
8421
~~~

Giả sử 8421 là mã của token đó.

Bản thân con số 8421 không có nghĩa tự nhiên là “thủ đô nước Pháp”.

Ta hoàn toàn có thể đánh số lại:

~~~text
Paris  → 100
London → 101
Hanoi  → 102
~~~

hoặc:

~~~text
Paris  → 9001
London → 73
Hanoi  → 4012
~~~

miễn hệ thống dùng cùng một bảng ánh xạ.

Điều này cho ta một gợi ý:

> **Mã token là danh tính, không phải toàn bộ nội dung tri thức của token.**

## Ví dụ: số áo cầu thủ

Hãy tưởng tượng một cầu thủ mang áo số 10.

Số 10 giúp ta xác định người nào đang được nói tới trong đội hình.

Nhưng số 10 tự nó không chứa chiều cao cầu thủ, khả năng chuyền bóng, lịch sử thi đấu, chiến thuật của đội hay mối quan hệ với đồng đội.

Nó chỉ là một **mã nhận diện**.

Mã token cũng gần với ý tưởng đó.

Nó cho hệ thống biết:

> “Đây là đơn vị nào trong bảng từ vựng của tôi?”

Nhưng muốn bắt đầu tính toán, mô hình cần nhiều hơn một số nguyên.

## Vậy ý nghĩa nằm trong bảng từ vựng?

Ta có thể thử chuyển giả thuyết:

> Có lẽ ý nghĩa nằm trong chính mục từ tương ứng với token.

Nhưng vấn đề vẫn còn.

Một token có thể có nhiều nghĩa tùy câu.

Ví dụ tiếng Anh:

~~~text
bank
~~~

có thể liên quan tới ngân hàng.

Nhưng trong ngữ cảnh khác, nó có thể liên quan tới bờ sông.

Nếu token tự mang một ý nghĩa duy nhất cố định, mô hình sẽ rất khó xử lý sự khác nhau này.

Ta cần một ý tưởng mới:

> **Ý nghĩa mà mô hình sử dụng không chỉ phụ thuộc vào token riêng lẻ. Nó còn phụ thuộc vào ngữ cảnh.**

Nhưng trước khi tới ngữ cảnh, ta cần biết token được biến thành thứ gì để các lớp của mô hình có thể xử lý.

## Từ danh tính tới biểu diễn

Mã token là một số nguyên:

~~~text
8421
~~~

Nhưng các phép tính bên trong mô hình chủ yếu làm việc với những dãy số.

Vì vậy mô hình cần một bước biến:

~~~text
mã token
↓
một dãy số
~~~

Dãy số đó là một **cách biểu diễn (representation)** của token ở thời điểm đó.

Ở bước đầu tiên, ta thường gặp một dạng cụ thể gọi là **phép nhúng (embedding)**.

Ta chưa cần học phép nhúng ở chương này.

Chỉ cần hiểu bước chuyển quan trọng:

~~~text
token
không đi thẳng vào các lớp dưới dạng chữ

token
↓
mã token
↓
biểu diễn bằng số
↓
các lớp bắt đầu biến đổi biểu diễn đó
~~~

Đây là chỗ “đường đi của mảnh tri thức” bắt đầu thay đổi ý nghĩa.

Thứ đi xuyên mô hình không còn là chữ trên màn hình.

Nó là những trạng thái số liên tục được biến đổi.

## Nếu tri thức không nằm nguyên trong token, nó nằm ở đâu?

Đây là một câu hỏi lớn.

Ta chưa thể trả lời chỉ bằng một chương.

Nhưng ta có thể loại vài cách hiểu quá đơn giản.

Tri thức không thể chỉ là:

~~~text
token ID
~~~

bởi token ID chỉ là mã nhận diện.

Tri thức cũng khó có thể chỉ là:

~~~text
một dãy số cố định duy nhất cho token
~~~

bởi cùng token có thể được dùng trong nhiều ngữ cảnh khác nhau.

Và tri thức cũng không chỉ nằm ở:

~~~text
một lớp đơn lẻ
~~~

bởi trạng thái còn tiếp tục được biến đổi qua nhiều lớp.

Một cách hình dung thận trọng hơn là:

~~~text
những gì mô hình đã học
        +
token hiện tại
        +
những token xung quanh
        +
cách các lớp biến đổi trạng thái
        ↓
một biểu diễn phụ thuộc ngữ cảnh
~~~

Ta chưa gọi đây là “tri thức”.

Ta chỉ nói:

> **Ý nghĩa sử dụng được của token dần xuất hiện trong một trạng thái được tạo bởi cả mô hình và ngữ cảnh.**

## “Biểu diễn” là gì?

Đây là một từ sẽ xuất hiện nhiều lần.

**Biểu diễn (representation)** là cách một hệ thống dùng một cấu trúc dữ liệu để mang thông tin về một thứ.

Ví dụ rất đời thường:

Một địa điểm có thể được biểu diễn bằng tên đường, tọa độ, ghim trên bản đồ hoặc mã bưu chính.

Cùng một nơi.

Nhiều cách biểu diễn khác nhau.

Trong mô hình, token cũng được biến thành những cấu trúc số để phép tính có thể làm việc với nó.

Ta chưa cần nói các con số đó “nghĩa là gì” ở từng chiều.

Chỉ cần hiểu:

> **Biểu diễn là dạng mà thông tin được đặt vào để hệ thống có thể tiếp tục xử lý.**

## Một cảnh báo quan trọng

Khi thấy một dãy số đại diện cho token, ta rất dễ muốn gắn nhãn:

~~~text
chiều 1 = quốc gia
chiều 2 = thành phố
chiều 3 = cảm xúc
...
~~~

Không nên vội.

Trong mô hình thật, thông tin có thể được phân bố trên nhiều chiều và nhiều thành phần.

Một khái niệm có thể không nằm gọn trong một con số duy nhất.

Và cùng một chiều có thể tham gia vào nhiều thứ khác nhau.

Đây là lý do ta phải thận trọng với câu:

> “Tôi đã tìm thấy nơi mô hình lưu tri thức X.”

Có thể ta mới chỉ tìm thấy một tín hiệu liên quan.

Tìm thấy tín hiệu liên quan chưa đồng nghĩa tìm thấy nguyên nhân hay nơi lưu duy nhất.

## Vậy token có hoàn toàn không có ý nghĩa?

Cũng không nên đi sang cực đoan kia.

Token là đầu vào thật của mô hình.

Danh tính token quyết định hàng đầu tiên mà mô hình lấy ra ở bước biểu diễn ban đầu.

Những trọng số liên quan đã được học trong quá trình huấn luyện.

Vì vậy token có vai trò rất thật.

Chỉ là:

> **Vai trò đó không nên bị hiểu thành “token tự chứa một viên tri thức hoàn chỉnh”.**

Ta có thể viết:

~~~text
token
= điểm bắt đầu

không phải

token
= toàn bộ câu chuyện
~~~

## Bước tiếp theo

Nếu token chỉ là điểm bắt đầu, thứ đầu tiên mô hình tạo ra từ token là gì?

Ta đã hé lộ câu trả lời:

> một dãy số.

Nhưng dãy số đó đến từ đâu?

Có bao nhiêu số?

Tại sao không dùng luôn mã token?

Và điều gì xảy ra khi token bước vào lớp đầu tiên?

Đó là câu hỏi của Chương 3.

### Nhớ 3 điều

1. **Mã token là mã nhận diện, không phải một gói tri thức.**
2. **Mô hình biến token thành một biểu diễn bằng số để các lớp có thể xử lý.**
3. **Ý nghĩa sử dụng được của token phụ thuộc vào cả những gì mô hình đã học và ngữ cảnh hiện tại; không nên giả định tri thức nằm nguyên trong một token hay một con số duy nhất.**

**Tiếp theo: [Chương 3 — Từ mã token tới một biểu diễn bằng số](03-tu-ma-token-toi-bieu-dien-bang-so.md)**
