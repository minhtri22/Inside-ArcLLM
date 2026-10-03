# Chương 7 — Nhánh biến đổi tín hiệu: điều gì xảy ra sau cơ chế chú ý?

> **Mức đọc: Đi sâu**
>
> **Ta đang mở chiếc hộp nào?**
>
> ~~~text
> thông tin từ ngữ cảnh
>         ↓
> trạng thái hiện tại
>         ↓
> [ nhánh biến đổi tín hiệu ]
>         ↓
> trạng thái đã biến đổi
> ~~~
>
> Chương này giải thích trực giác của **mạng truyền thẳng (Feed-Forward Network, FFN)** — phần thường xuất hiện sau cơ chế chú ý trong mỗi lớp Transformer.

Ở Chương 6, ta thấy attention giúp một vị trí lấy thêm thông tin từ những vị trí khác.

Nhưng sau khi lấy được thông tin, trạng thái vẫn chưa “xong”.

Nó còn phải được biến đổi.

Đây là vai trò chính của FFN.

## Một phép so sánh đơn giản

Hãy tưởng tượng bạn đang đọc một hồ sơ.

Attention giống bước:

> “Tôi cần lấy thêm thông tin nào từ những trang khác?”

Sau khi lấy được thông tin đó, bạn vẫn còn phải:

> “Ghép nó với trạng thái hiện tại và xử lý tiếp.”

FFN gần với bước thứ hai hơn.

Nó không chủ yếu đi lấy dữ liệu từ token khác.

Nó biến đổi trạng thái của **từng vị trí** bằng cùng một bộ trọng số đã học.

## “Truyền thẳng” nghĩa là gì?

Tên tiếng Anh là **Feed-Forward Network, FFN**.

“Truyền thẳng” ở đây chỉ cách tín hiệu đi qua một chuỗi phép biến đổi theo một hướng.

Ở mức rất đơn giản:

~~~text
trạng thái vào
     ↓
mở rộng
     ↓
biến đổi
     ↓
thu về
     ↓
trạng thái ra
~~~

Ta chưa cần công thức.

Chỉ cần thấy rằng FFN không phải một phép cộng đơn giản.

Nó tạo ra một phép biến đổi học được.

## Vì sao lại mở rộng chiều?

Giả sử trạng thái hiện tại là một vectơ có kích thước:

~~~text
d
~~~

FFN thường đưa nó sang một không gian lớn hơn:

~~~text
d
↓
một kích thước lớn hơn
↓
d
~~~

Có thể hình dung như mở một bản ghi ngắn thành một không gian làm việc rộng hơn để thực hiện biến đổi, rồi nén kết quả về kích thước mà phần còn lại của mô hình đang dùng.

Ví dụ minh họa:

~~~text
[ trạng thái 4 chiều ]
        ↓
[ không gian 12 chiều ]
        ↓
[ trạng thái 4 chiều ]
~~~

Các con số chỉ để minh họa.

Mô hình thật có kích thước lớn hơn nhiều.

## Tại sao không chỉ dùng phép biến đổi tuyến tính?

Nếu ta chỉ liên tục nhân với ma trận rồi cộng mà không có phần phi tuyến, khả năng biểu diễn của chuỗi phép biến đổi sẽ bị hạn chế.

Vì vậy FFN có một bước **phi tuyến (nonlinearity)**.

Có thể hình dung:

~~~text
trạng thái
   ↓
biến đổi tuyến tính
   ↓
phi tuyến
   ↓
biến đổi tiếp
   ↓
trạng thái mới
~~~

Phi tuyến cho phép mô hình tạo ra những kiểu biến đổi phong phú hơn.

Một số mô hình hiện đại còn dùng các cơ chế **gating** — có thể hiểu là những nhánh giúp điều tiết phần tín hiệu nào được giữ hoặc nhấn mạnh.

Tên cụ thể như SwiGLU có thể xuất hiện trong tài liệu kỹ thuật, nhưng chưa cần học ở đây.

## Attention và FFN khác nhau ở đâu?

Ta có thể dùng một sơ đồ ngắn:

~~~text
ATTENTION
"nên lấy thêm thông tin từ đâu?"
        ↓
trộn thông tin giữa các vị trí

FFN
"sau khi có thông tin đó,
 trạng thái tại vị trí này
 nên được biến đổi thế nào?"
        ↓
biến đổi tại từng vị trí
~~~

Đây là trực giác, không phải ranh giới tuyệt đối về “ý nghĩa” của hai khối.

Cả hai đều dùng trọng số đã học.

Cả hai đều tham gia vào trạng thái cuối.

## Một ví dụ với từ “đá”

Quay lại hai câu:

~~~text
Tôi nhặt một hòn đá.

Tôi thích đá bóng.
~~~

Attention có thể giúp vị trí “đá” nhận những tín hiệu khác nhau từ ngữ cảnh xung quanh.

Sau đó FFN nhận hai trạng thái đã khác nhau đó.

Vì đầu vào khác, kết quả biến đổi cũng có thể khác.

Ta có thể hình dung:

~~~text
"đá" trong câu A
        ↓
ngữ cảnh A
        ↓
trạng thái A
        ↓
FFN
        ↓
trạng thái A'

"đá" trong câu B
        ↓
ngữ cảnh B
        ↓
trạng thái B
        ↓
FFN
        ↓
trạng thái B'
~~~

FFN không cần “biết” từ đá theo cách con người.

Nó chỉ thực hiện phép biến đổi số đã học trên trạng thái hiện tại.

Nhưng chính chuỗi biến đổi đó góp phần làm hai ngữ cảnh đi theo hai đường khác nhau.

## FFN có phải nơi chứa tri thức không?

Đây là một câu hỏi rất dễ bị trả lời quá mạnh.

Một số nghiên cứu cho thấy các trọng số trong FFN có thể liên quan tới nhiều mẫu thông tin và hành vi mà mô hình đã học.

Nhưng câu:

> “Tri thức nằm trong FFN.”

là quá đơn giản.

Vì mô hình còn có:

- bảng nhúng;
- attention;
- nhiều lớp;
- đường cộng tắt;
- trạng thái theo ngữ cảnh;
- lớp đầu ra;
- rất nhiều trọng số tương tác với nhau.

Do đó cuốn sách sẽ giữ câu nói thận trọng hơn:

> **FFN là một phần quan trọng của quá trình biến đổi trạng thái đã học, nhưng không nên được coi như một chiếc tủ chứa tri thức độc lập.**

## Một lớp có hai kiểu chuyển động

Tới đây, ta có thể nhìn một lớp theo hai hướng:

~~~text
1. chuyển động NGANG giữa các vị trí
   attention

2. chuyển động SÂU trong trạng thái của từng vị trí
   FFN
~~~

Đây là cách hình dung hữu ích.

Nhưng đừng hiểu chữ “ngang” và “sâu” như thuật ngữ toán học chính thức.

Nó chỉ giúp phân biệt:

- attention chủ yếu trộn thông tin giữa vị trí;
- FFN chủ yếu biến đổi trạng thái tại mỗi vị trí.

## Nếu cứ biến đổi mãi, trạng thái cũ có mất hết không?

Đây là câu hỏi quan trọng.

Nếu mỗi khối chỉ lấy đầu vào rồi thay hoàn toàn bằng đầu ra mới, việc giữ thông tin cũ qua hàng chục lớp sẽ khó hơn.

Transformer dùng một cấu trúc rất quan trọng:

> **đường cộng tắt (residual connection)**

cho phép tín hiệu cũ đi vòng qua phép biến đổi rồi được cộng trở lại.

Ta sẽ mở chiếc hộp đó ở chương sau.

### Nhớ 3 điều

1. **Mạng truyền thẳng (FFN) biến đổi trạng thái tại từng vị trí sau khi thông tin ngữ cảnh đã được trộn.**
2. **FFN thường mở rộng trạng thái sang một không gian lớn hơn, áp dụng biến đổi phi tuyến rồi đưa về kích thước cần dùng.**
3. **FFN không nên được gọi đơn giản là “nơi chứa tri thức”.** Nó là một phần trong chuỗi biến đổi phân tán trên nhiều thành phần của mô hình.

**Tiếp theo: [Chương 8 — Đường cộng tắt: vì sao thông tin cũ không biến mất hoàn toàn?](08-duong-cong-tat.md)**
