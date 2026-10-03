# Chương 4 — Cùng một token, ngữ cảnh khác, biểu diễn khác

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
> biểu diễn ban đầu
>    ↓
> [ ngữ cảnh bắt đầu làm trạng thái thay đổi ]
> ~~~
>
> Chương này chỉ trả lời một câu hỏi: **vì sao cùng một token nhưng đứng trong hai câu khác nhau lại có thể dẫn tới trạng thái khác nhau bên trong mô hình?**

Hãy nhìn một từ rất quen:

~~~text
đá
~~~

Bây giờ đặt nó vào hai câu:

~~~text
Tôi nhặt một hòn đá.

Tôi thích đá bóng.
~~~

Chữ “đá” xuất hiện ở cả hai câu.

Nhưng ta hiểu hai lần xuất hiện đó theo hai cách khác nhau.

Trong câu đầu, “đá” là một vật.

Trong câu sau, “đá” là một hành động.

Nếu mô hình chỉ nhìn riêng token “đá” và giữ nguyên một ý nghĩa cố định từ đầu tới cuối, hai câu trên sẽ là một vấn đề lớn.

Nhưng mô hình không xử lý token hoàn toàn cô lập như vậy.

## Bắt đầu từ cùng một danh tính

Để dễ hình dung, giả sử trong cả hai câu từ “đá” được bộ tách giữ thành một token riêng.

Ta có thể tạm vẽ:

~~~text
Câu A:
[Tôi] [nhặt] [một] [hòn] [đá]

Câu B:
[Tôi] [thích] [đá] [bóng]
~~~

Token “đá” có cùng danh tính.

Nếu cùng một bảng nhúng được dùng, phần biểu diễn lấy trực tiếp từ danh tính token đó bắt đầu từ cùng một hàng số đã học.

Nhưng hai token “đá” không đứng trong cùng hoàn cảnh.

Một bên có:

~~~text
nhặt
một
hòn
~~~

Một bên có:

~~~text
thích
bóng
~~~

Những token xung quanh tạo ra hai **ngữ cảnh** khác nhau.

**Ngữ cảnh (context)** là phần thông tin xung quanh mà mô hình có thể sử dụng khi xử lý một vị trí.

Ở đây, ta chưa cần một định nghĩa phức tạp hơn.

## Danh tính giữ nguyên, trạng thái có thể thay đổi

Đây là điểm rất quan trọng.

Ta có thể hình dung:

~~~text
token "đá"
    ↓
cùng mã token
    ↓
cùng điểm bắt đầu từ bảng nhúng
    ↓
nhưng nằm trong hai chuỗi khác nhau

Câu A                         Câu B
[Tôi nhặt một hòn đá]        [Tôi thích đá bóng]
          ↓                           ↓
   qua các lớp (layers)        qua các lớp (layers)
          ↓                           ↓
 trạng thái A                  trạng thái B
~~~

Token vẫn là token “đá”.

Nhưng **trạng thái số** mà mô hình đang giữ cho vị trí đó có thể dần khác nhau.

Đây là chỗ ta cần thêm một thuật ngữ mới.

## Trạng thái ẩn là gì?

Khi dữ liệu đi qua mô hình, mỗi vị trí được gắn với một dãy số đang được cập nhật.

Dãy số trung gian đó thường được gọi là **trạng thái ẩn (hidden state)**.

Từ “ẩn” không có nghĩa nó bí mật.

Nó chỉ có nghĩa đây là trạng thái nằm **bên trong** mô hình, không phải văn bản đầu vào hay token đầu ra mà người dùng trực tiếp nhìn thấy.

Ta có thể hình dung:

~~~text
token
↓
biểu diễn ban đầu
↓
trạng thái ẩn sau lớp 1
↓
trạng thái ẩn sau lớp 2
↓
trạng thái ẩn sau lớp 3
↓
...
~~~

Mỗi bước vẫn là những con số.

Nhưng những con số đó không nhất thiết giữ nguyên.

## Ngữ cảnh đi vào trạng thái bằng cách nào?

Ta chưa học cơ chế cụ thể.

Chương 6 mới mở hộp **cơ chế chú ý (attention)**.

Ở đây chỉ cần biết một điều:

> **Các lớp xử lý Transformer (layers) cho phép trạng thái ở một vị trí được cập nhật bằng thông tin liên quan tới những vị trí khác trong chuỗi.**

Với câu:

~~~text
Tôi nhặt một hòn đá.
~~~

trạng thái ở vị trí “đá” có thể được cập nhật trong một môi trường chứa các token “nhặt”, “một”, “hòn”.

Với câu:

~~~text
Tôi thích đá bóng.
~~~

trạng thái ở vị trí “đá” nằm trong một môi trường khác.

Do đó, cùng một danh tính token có thể đi tới hai trạng thái khác nhau.

## Một cách hình dung đời thường

Hãy tưởng tượng bạn nhận được một tờ giấy chỉ ghi:

~~~text
Mai
~~~

Bạn chưa biết “Mai” là ai.

Sau đó tờ giấy được bổ sung:

~~~text
Mai đang ở bệnh viện.
~~~

Bạn có thêm thông tin.

Rồi:

~~~text
Mai mặc áo blouse trắng.
~~~

Rồi:

~~~text
Mai đang khám cho bệnh nhân.
~~~

Tên “Mai” không đổi.

Nhưng trạng thái hiểu của bạn về người đang được nói tới đã thay đổi.

Mô hình không hiểu theo đúng cách con người hiểu, và trạng thái số trong Transformer không phải một “hồ sơ bằng lời”.

Nhưng phép so sánh này giúp ta giữ đúng trực giác:

> **Danh tính đầu vào có thể giữ nguyên trong khi trạng thái dùng để xử lý nó được cập nhật theo ngữ cảnh.**

## Biểu diễn theo ngữ cảnh

Khi biểu diễn của một vị trí đã chịu ảnh hưởng của ngữ cảnh xung quanh, ta có thể gọi nó là **biểu diễn theo ngữ cảnh (contextual representation)**.

So với biểu diễn ban đầu:

~~~text
mã token
↓
bảng nhúng
↓
biểu diễn ban đầu
~~~

ta có một đường dài hơn:

~~~text
mã token
↓
biểu diễn ban đầu
↓
+ vị trí trong chuỗi
↓
+ thông tin từ những token liên quan
↓
+ nhiều lần biến đổi qua các lớp
↓
biểu diễn theo ngữ cảnh
~~~

Dấu “+” trong hình không có nghĩa hệ thống chỉ làm một phép cộng đơn giản.

Nó chỉ nói rằng nhiều nguồn thông tin cùng tham gia vào trạng thái cuối cùng.

## Vị trí cũng quan trọng

Hãy thử đổi thứ tự:

~~~text
chó cắn người
~~~

và:

~~~text
người cắn chó
~~~

Nếu chỉ biết ba token xuất hiện mà không biết thứ tự, hai chuỗi có vẻ giống nhau.

Nhưng nghĩa của chúng khác hẳn.

Vì vậy mô hình còn cần biết token nằm ở **vị trí nào** trong chuỗi.

Các Transformer khác nhau có thể đưa thông tin vị trí vào phép tính theo những cách khác nhau.

Ta chưa cần mở cơ chế đó ở đây.

Chỉ cần nhớ:

> **Danh tính token chưa đủ. Ngữ cảnh và vị trí cũng tham gia vào trạng thái mà mô hình tạo ra.**

## Điều này có nghĩa mô hình đã “hiểu” giống con người chưa?

Chưa thể kết luận như vậy.

Ta có thể quan sát rằng trạng thái số thay đổi theo ngữ cảnh.

Ta có thể kiểm tra rằng mô hình dự đoán khác nhau khi ngữ cảnh khác nhau.

Nhưng từ đó nhảy thẳng tới câu:

> “Mô hình hiểu từ này giống hệt con người.”

là đi quá bằng chứng.

Ta nên giữ câu nói hẹp hơn:

> **Mô hình tạo ra các trạng thái phụ thuộc ngữ cảnh, và những trạng thái đó được dùng cho các phép tính tiếp theo.**

Đây là điều đủ để đi tiếp.

## “Mảnh tri thức” bây giờ đang ở đâu?

Ta đã loại thêm một cách hiểu quá đơn giản.

Nó không nằm nguyên trong:

~~~text
token
~~~

Nó cũng không nằm nguyên trong:

~~~text
mã token
~~~

Và giờ ta thấy:

~~~text
biểu diễn ban đầu
~~~

cũng chưa phải câu chuyện hoàn chỉnh.

Trạng thái còn tiếp tục thay đổi khi token đi qua nhiều lớp.

Có thể hình dung:

~~~text
token
↓
điểm bắt đầu
↓
ngữ cảnh tác động
↓
trạng thái thay đổi
↓
ngữ cảnh tiếp tục được tích hợp
↓
trạng thái lại thay đổi
↓
...
~~~

Nếu muốn hiểu “mảnh tri thức” đang đi đâu, ta buộc phải nhìn vào chính chuỗi biến đổi này.

## Nhưng các lớp thực sự làm gì?

Ta đã nhiều lần nói:

> “đi qua các lớp”.

Nhưng một lớp là gì?

Bên trong nó có những phần nào?

Phần nào giúp một vị trí lấy thông tin từ vị trí khác?

Phần nào tiếp tục biến đổi trạng thái sau đó?

Và vì sao trạng thái cũ không bị xóa sạch mỗi lần đi qua một lớp?

Đó là đoạn đường tiếp theo.

### Nhớ 3 điều

1. **Cùng một token có thể dẫn tới những trạng thái khác nhau khi ngữ cảnh khác nhau.**
2. **Trạng thái ẩn (hidden state) là dãy số trung gian mà mô hình đang giữ và cập nhật cho một vị trí.**
3. **Biểu diễn theo ngữ cảnh (contextual representation) không chỉ phụ thuộc vào danh tính token; vị trí, những token xung quanh và các lớp xử lý đều tham gia.**

**Tiếp theo: [Chương 5 — Dòng tín hiệu đi xuyên Transformer](05-dong-tin-hieu-di-xuyen-transformer.md)**
