# Chương 20 — Từ phép đo tới điều ta thật sự biết

> **Mức đọc: Nâng cao**
>
> Ta bắt đầu bằng một câu hỏi nghe rất đơn giản:
>
> **Một token mang “mảnh tri thức” đi qua mô hình như thế nào?**
>
> Sau 19 chương, câu hỏi đó đã đổi hình dạng.

Ở đầu sách, cách hình dung tự nhiên là:

~~~text
token
=
một chiếc hộp nhỏ
chứa một phần ý nghĩa
~~~

Rồi chiếc hộp đó đi xuyên qua mô hình và cuối cùng tạo ra câu trả lời.

Bây giờ ta biết bức tranh thật phức tạp hơn.

## Đường thứ nhất: bên trong mô hình

Ta đã đi qua:

~~~text
văn bản
↓
token
↓
mã token
↓
biểu diễn ban đầu
↓
ngữ cảnh
↓
attention
↓
FFN
↓
residual
↓
nhiều lớp
↓
quỹ đạo biểu diễn
↓
trạng thái cuối
↓
logits
↓
token tiếp theo
~~~

Không có bước nào cho ta một chiếc hộp ghi:

~~~text
TRI THỨC
~~~

Thay vào đó, ta thấy một **quá trình biến đổi**.

## Token không phải tri thức

Token là một đơn vị đầu vào.

Mã token là danh tính.

Biểu diễn ban đầu là một trạng thái số đã học.

Ngữ cảnh làm trạng thái thay đổi.

Nhiều lớp tiếp tục cập nhật nó.

Trọng số của mô hình tham gia vào tất cả những phép biến đổi đó.

Vì vậy:

> **Không có bằng chứng nào trong hành trình này cho phép ta coi một token đơn lẻ là một viên tri thức hoàn chỉnh.**

## Vectơ cũng không phải toàn bộ tri thức

Ta cũng không nên đổi cực đoan:

~~~text
token không chứa tri thức
↓
vậy một vectơ nào đó chắc chứa tri thức
~~~

Một trạng thái ở một lớp chỉ là một lát cắt của quá trình.

Thông tin có thể:

- phân tán trên nhiều chiều;
- phụ thuộc ngữ cảnh;
- được tạo bởi nhiều lớp;
- cần trọng số ở lớp sau mới trở thành hành vi;
- chỉ bộc lộ khi đầu vào phù hợp.

Do đó câu:

> “Tri thức nằm ở vectơ X.”

thường cần nhiều bằng chứng hơn rất nhiều so với việc tìm một vectơ tương quan với một khái niệm.

## Một cách nói thận trọng hơn

Ta có thể hình dung tri thức mà mô hình thể hiện được **phân bố** trên nhiều thành phần:

~~~text
trọng số đã học
      +
cấu trúc mô hình
      +
ngữ cảnh hiện tại
      +
các trạng thái trung gian
      +
chuỗi biến đổi qua nhiều lớp
      ↓
hành vi dự đoán
~~~

Sơ đồ này không phải định nghĩa triết học về tri thức.

Nó chỉ là một cách nói kỹ thuật thận trọng hơn:

> **Những gì mô hình có thể biểu hiện không nhất thiết nằm gọn ở một token, một neuron, một chiều hay một lớp duy nhất.**

## Đường thứ hai: bên ngoài mô hình

Từ Chương 11, ta đổi góc nhìn.

Ta thấy:

~~~text
phép tính logic
↓
kernel
↓
dispatch
↓
bộ nhớ
↓
thực thi vật lý
↓
dấu vết
↓
dấu thời gian
↓
bộ đếm
↓
bằng chứng
~~~

Ở đây xuất hiện một loại “tri thức” khác.

Không phải tri thức của mô hình.

Mà là:

> **tri thức của con người về mô hình.**

## Tri thức của người quan sát không đến thẳng từ con số

Ta có thể vẽ:

~~~text
HIỆN TƯỢNG
    ↓
KÊNH ĐO
    ↓
SỐ LIỆU
    ↓
KIỂM TRA ĐỘ ĐẦY ĐỦ
    ↓
BẰNG CHỨNG
    ↓
DIỄN GIẢI
    ↓
KẾT LUẬN
~~~

Nếu kênh đo có vấn đề, con số có thể không đủ.

Nếu bằng chứng chỉ định vị hotspot, ta không được viết nguyên nhân.

Nếu counter trả 0 nhưng độ đầy đủ của phép đo không đạt, ta không được gọi đại lượng vật lý bằng 0.

Nếu khuôn mẫu rất đẹp nhưng chưa có phép thử phân biệt phù hợp, ta không được viết kết luận nhân quả.

## Hai hành trình gặp nhau

Đây là điểm cuốn sách muốn đi tới.

Bên trong mô hình:

~~~text
token
↓
trạng thái
↓
biến đổi
↓
hành vi
~~~

Phía người quan sát:

~~~text
hiện tượng
↓
phép đo
↓
bằng chứng
↓
tri thức
~~~

Một bên là **quá trình mô hình tạo và biến đổi trạng thái**.

Một bên là **quá trình con người tạo ra hiểu biết đáng tin về quá trình đó**.

Tên “Đường đi của mảnh tri thức” vì vậy có hai nghĩa.

## Nghĩa thứ nhất: thông tin được biến đổi

Một token không mang nguyên một mảnh tri thức bất biến.

Nhưng nó tham gia vào một chuỗi biến đổi nơi thông tin từ:

- danh tính token;
- vị trí;
- ngữ cảnh;
- trọng số đã học;

được kết hợp và cập nhật.

Ta theo dấu sự biến đổi đó.

## Nghĩa thứ hai: bằng chứng trở thành hiểu biết

Một phép đo cũng không phải tri thức hoàn chỉnh.

Nó đi qua:

~~~text
quan sát
↓
kiểm tra
↓
đối chứng
↓
giới hạn
↓
diễn giải
~~~

Chỉ khi giữ được nguồn gốc và ranh giới bằng chứng, ta mới có một kết luận đáng tin hơn.

## “Không biết” cũng là một trạng thái tri thức

Đây có lẽ là bài học khó nhất.

Ta thường thích hai đáp án:

~~~text
đúng
sai
~~~

Nhưng khoa học thường có thêm:

~~~text
CHƯA ĐỦ BẰNG CHỨNG
CHƯA GIẢI QUYẾT
NẰM NGOÀI PHẠM VI ĐÃ ĐO
~~~

Những trạng thái này không phải thất bại ngôn ngữ.

Chúng ngăn ta biến một lỗ hổng hiểu biết thành một câu chuyện bịa ra cho tròn.

## Quay lại ví dụ đầu tiên

Ta bắt đầu với:

~~~text
Hà Nội là thủ đô của ...
~~~

Rồi mô hình sinh:

~~~text
Việt
~~~

sau đó:

~~~text
Nam
~~~

Lúc đầu, ta có thể tưởng:

> token “Hà Nội” mang theo mẩu tri thức “thủ đô Việt Nam”.

Bây giờ ta có cách hỏi tốt hơn:

> Những trọng số đã học, ngữ cảnh hiện tại và quỹ đạo biểu diễn đã tương tác thế nào để trạng thái cuối ưu tiên token “Việt”, rồi “Nam”?

Đây là câu hỏi khó hơn.

Nhưng cũng chính xác hơn.

## Và nếu muốn đi sâu hơn nữa?

Ta có thể tiếp tục hỏi:

- biểu diễn thay đổi bao nhiêu qua từng lớp?
- khác biệt nhỏ nào lặp lại?
- phần nào của trạng thái có khả năng dự đoán một thuộc tính?
- nếu can thiệp vào trạng thái, hành vi có đổi như dự đoán không?
- kết quả có giữ qua nhiều đầu vào và mô hình khác nhau không?

Những câu hỏi đó mở ra những con đường nghiên cứu mới.

Nhưng cuốn sách này dừng trước khi biến chúng thành kết luận.

Nó chỉ trao cho người đọc một kỷ luật:

> **Theo dấu trước. Đo trước. Phân biệt bằng chứng với diễn giải. Chỉ nói tới nơi dữ liệu cho phép.**

## Bản đồ cuối cùng

Nếu phải nén cả cuốn sách thành một hình:

~~~text
                 BÊN TRONG MÔ HÌNH

văn bản
  ↓
token
  ↓
biểu diễn
  ↓
ngữ cảnh + trọng số
  ↓
biến đổi qua nhiều lớp
  ↓
token tiếp theo


                 PHÍA NGƯỜI QUAN SÁT

hiện tượng
  ↓
dấu vết
  ↓
phép đo
  ↓
kiểm tra độ đầy đủ
  ↓
bằng chứng
  ↓
diễn giải
  ↓
điều ta thật sự biết
~~~

Hai đường không giống nhau.

Nhưng chúng gặp nhau ở một nguyên tắc:

> **Đừng nhầm biểu diễn với bản chất. Đừng nhầm phép đo với thực tại.**

### Nhớ 3 điều

1. **Token không phải một viên tri thức.** Điều mô hình biểu hiện được tạo ra từ sự tương tác giữa trọng số đã học, ngữ cảnh và chuỗi biến đổi trạng thái.
2. **Một phép đo cũng không phải tri thức hoàn chỉnh.** Nó chỉ trở thành bằng chứng hữu ích khi nguồn gốc, độ đầy đủ và giới hạn diễn giải được giữ rõ.
3. **CHƯA GIẢI QUYẾT cũng là một kết quả có giá trị.** Biết chính xác mình chưa biết gì tốt hơn một lời giải thích đẹp nhưng vượt quá bằng chứng.

---

Ta bắt đầu bằng cách đi theo một token để tìm “mảnh tri thức”.

Cuối cùng, điều ta tìm thấy không phải một vật thể nằm yên trong token.

Ta tìm thấy **một chuỗi biến đổi**.

Và song song với nó là một chuỗi khác:

~~~text
quan sát
↓
đo
↓
kiểm tra
↓
bằng chứng
↓
hiểu biết
~~~

Có lẽ đó mới là đường đi quan trọng nhất của “mảnh tri thức” trong cuốn sách này:

> **không chỉ là cách thông tin thay đổi bên trong mô hình, mà còn là cách con người học để biết tới đâu mình thật sự hiểu sự thay đổi ấy.**
