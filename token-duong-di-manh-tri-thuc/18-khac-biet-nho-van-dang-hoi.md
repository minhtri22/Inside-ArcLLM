# Chương 18 — Khi một khác biệt quá nhỏ vẫn đáng để hỏi

> **Mức đọc: Nghiên cứu**
>
> **Ta đang đổi câu hỏi:**
>
> ~~~text
> hai giá trị gần như giống nhau
>          ↓
>      "coi như bằng?"
>          ↓
> hay phần chênh lệch đó
>      có điều gì đáng hỏi?
> ~~~
>
> Chương này không dạy rằng mọi sai khác nhỏ đều quan trọng. Nó dạy một kỷ luật đơn giản hơn: **đừng gọi một sai khác là “vô nghĩa” trước khi biết nó nhỏ so với cái gì.**

Hãy nhìn hai số:

~~~text
A = 1,000000
B = 1,000003
~~~

Ta có thể nói:

~~~text
A ≈ B
~~~

Điều đó có thể hoàn toàn hợp lý.

Nhưng ta cũng có thể tính:

~~~text
B - A = 0,000003
~~~

Ba phần triệu.

Rất nhỏ.

Câu hỏi là:

> **Nhỏ đến mức có thể bỏ qua, hay nhỏ nhưng có cấu trúc?**

Hai câu này không giống nhau.

## Sai khác là gì?

Nếu có hai giá trị:

~~~text
A
B
~~~

ta có thể tính **sai khác (difference)**:

~~~text
Δ = B - A
~~~

Ký hiệu Δ chỉ phần chênh lệch.

Nếu hai giá trị rất gần nhau, Δ sẽ nhỏ.

Trong một số ngữ cảnh, phần còn lại sau khi trừ như vậy có thể được gọi là **phần dư (residual)**.

Ở đây cần tránh nhầm với **đường cộng tắt (residual connection)** ở Chương 8.

Hai chữ residual giống nhau trong tiếng Anh, nhưng đang nói về hai chuyện khác:

~~~text
residual connection
→ cấu trúc trong mạng

residual / difference
→ phần chênh còn lại giữa hai đại lượng
~~~

## Nhỏ so với cái gì?

Giả sử một chiếc cân chỉ hiển thị tới 1 gram.

Bạn đặt hai vật lên cân:

~~~text
100 g
100 g
~~~

Có thể chúng thật sự nặng bằng nhau.

Nhưng cũng có thể:

~~~text
100,2 g
100,4 g
~~~

và chiếc cân chỉ không đủ độ phân giải để cho bạn thấy.

**Độ phân giải (resolution)** là mức chi tiết nhỏ nhất mà một phép đo có thể phân biệt đáng tin.

Vì vậy:

> **Không quan sát thấy khác biệt không tự động nghĩa không có khác biệt.**

Ngược lại:

> **Quan sát thấy một khác biệt cực nhỏ cũng chưa chắc khác biệt đó là tín hiệu thật.**

## Nhiễu cũng tạo ra sai khác

Mọi phép đo thực tế đều có thể chịu ảnh hưởng của:

- nhiễu;
- làm tròn;
- sai số số học;
- thay đổi nhiệt độ;
- trạng thái hệ thống;
- cách lấy mẫu;
- giới hạn của dụng cụ.

Ta gọi vùng dao động nền mà phép đo thường có là **mức nhiễu (noise floor)**.

Hãy tưởng tượng:

~~~text
tín hiệu cần tìm: 0,000003

nhiễu tự nhiên:
+0,000010
-0,000008
+0,000006
...
~~~

Khi nhiễu lớn hơn tín hiệu, một sai khác đơn lẻ rất khó diễn giải.

## Lặp lại là câu hỏi đầu tiên

Giả sử lần 1:

~~~text
B - A = +0,000003
~~~

Lần 2:

~~~text
B - A = -0,000004
~~~

Lần 3:

~~~text
B - A = +0,000001
~~~

Lần 4:

~~~text
B - A = -0,000006
~~~

Kết quả đổi dấu thất thường.

Điều này trông giống nhiễu hơn.

Nhưng nếu:

~~~text
lần 1  +0,000003
lần 2  +0,000003
lần 3  +0,000004
lần 4  +0,000003
~~~

ta có lý do để hỏi tiếp.

Khả năng một hiện tượng xuất hiện lại khi lặp phép đo được gọi là **khả năng lặp lại (repeatability)**.

Lặp lại chưa chứng minh nguyên nhân.

Nhưng nó giúp phân biệt một dao động ngẫu nhiên với một ứng viên tín hiệu đáng điều tra.

## Cấu trúc quan trọng hơn chỉ riêng độ lớn

Giả sử ta theo dõi sai khác qua nhiều lớp:

~~~text
lớp 1   +0,000001
lớp 2   +0,000002
lớp 3   +0,000004
lớp 4   +0,000008
lớp 5   +0,000016
~~~

Một khuôn mẫu như vậy khác với:

~~~text
lớp 1   -0,000007
lớp 2   +0,000003
lớp 3   -0,000001
lớp 4   +0,000009
lớp 5   -0,000004
~~~

Trong ví dụ đầu, phần chênh có một cấu trúc tăng dần.

Trong ví dụ sau, nó dao động hỗn loạn hơn.

Ta có thể hỏi:

> Sai khác có phụ thuộc lớp không?

> Có phụ thuộc vị trí token không?

> Có giữ cùng hướng qua nhiều lần chạy không?

> Khi đầu vào thay đổi nhẹ, sai khác có phản ứng theo một quy luật nào không?

Đây là cách một sai khác rất nhỏ có thể trở thành **một câu hỏi khoa học**, chứ chưa phải kết luận.

## Một sơ đồ để nghĩ về “biến dạng”

Ở Chương 9, ta có quỹ đạo biểu diễn:

~~~text
x0 → x1 → x2 → x3 → ... → xN
~~~

Giả sử ta có hai điều kiện rất gần nhau:

~~~text
A: x0 → x1 → x2 → x3 → ... → xN
B: y0 → y1 → y2 → y3 → ... → yN
~~~

Ta có thể nhìn phần chênh:

~~~text
Δ0 = y0 - x0
Δ1 = y1 - x1
Δ2 = y2 - x2
...
ΔN = yN - xN
~~~

Câu hỏi mới không chỉ là:

> Δ có nhỏ không?

Mà còn:

> **Δ thay đổi thế nào trên đường đi?**

Có thể:

~~~text
rất nhỏ
↓
biến mất
~~~

hoặc:

~~~text
rất nhỏ
↓
giữ ổn định
~~~

hoặc:

~~~text
rất nhỏ
↓
tăng dần
~~~

hoặc:

~~~text
rất nhỏ
↓
đổi hướng
↓
lan sang nhiều chiều
~~~

Mỗi dạng mở ra một câu hỏi khác.

## Nhưng “có cấu trúc” vẫn chưa phải “có ý nghĩa”

Đây là hàng rào quan trọng nhất.

Giả sử một sai khác:

- lặp lại;
- tăng theo lớp;
- phụ thuộc đầu vào.

Ta vẫn chưa được phép nói:

> “Đây chính là tri thức.”

Hay:

> “Đây là cơ chế gây ra hành vi X.”

Ta mới có thể nói:

> **Sai khác này có cấu trúc đủ để đáng kiểm tra tiếp.**

Có cấu trúc là một bước mạnh hơn nhiễu ngẫu nhiên.

Nhưng nó vẫn còn cách kết luận nhân quả một đoạn dài.

## Một trình tự an toàn

Khi gặp sai khác nhỏ:

~~~text
có sai khác
    ↓
độ lớn bao nhiêu?
    ↓
so với độ phân giải?
    ↓
so với mức nhiễu?
    ↓
có lặp lại?
    ↓
có cấu trúc?
    ↓
có phản ứng khi điều kiện thay đổi?
    ↓
mới đặt giả thuyết mạnh hơn
~~~

Điều nguy hiểm nhất là nhảy:

~~~text
Δ ≠ 0
↓
"đã tìm thấy tín hiệu"
~~~

hoặc chiều ngược lại:

~~~text
Δ rất nhỏ
↓
"chắc chắn vô nghĩa"
~~~

Cả hai đều quá sớm.

## Một con số nhỏ có thể là gì?

Khi thấy phần chênh cực nhỏ, ít nhất có nhiều khả năng:

- sai số số học;
- nhiễu phép đo;
- giới hạn độ phân giải;
- khác biệt trạng thái thật nhưng không quan trọng;
- khác biệt thật và có cấu trúc;
- dấu hiệu của một cơ chế chưa hiểu.

Không thể chọn một câu chỉ vì nó thú vị nhất.

Phải để bằng chứng loại dần các khả năng.

## Từ sai khác tới nguyên nhân

Chương này dừng ở đây.

Ta đã có thể hỏi:

> “Sai khác có thật và có cấu trúc không?”

Nhưng câu tiếp theo khó hơn:

> “Nếu có cấu trúc, cái gì gây ra nó?”

Ở đó, một khuôn mẫu đẹp không còn đủ.

Ta phải bước vào câu chuyện của **tương quan và nhân quả**.

### Nhớ 3 điều

1. **Một sai khác rất nhỏ không tự động vô nghĩa; trước hết phải so nó với độ phân giải, mức nhiễu và khả năng lặp lại của phép đo.**
2. **Cấu trúc theo lớp, vị trí, thời gian hoặc điều kiện có thể biến một sai khác nhỏ thành câu hỏi đáng nghiên cứu.**
3. **Có cấu trúc chưa có nghĩa đã biết nguyên nhân.** Sai khác là điểm bắt đầu của một giả thuyết, không phải kết luận cuối.

**Tiếp theo: [Chương 19 — Một khuôn mẫu đẹp vẫn chưa phải nguyên nhân](19-khuon-mau-dep-chua-phai-nguyen-nhan.md)**
