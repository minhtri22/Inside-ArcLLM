# Chương 10 — Từ trạng thái cuối tới token tiếp theo

> **Mức đọc: Đi sâu**
>
> **Ta đã đi hết phần bên trong mô hình tới đây:**
>
> ~~~text
> văn bản
>   ↓
> token
>   ↓
> biểu diễn
>   ↓
> nhiều lớp
>   ↓
> trạng thái cuối
>   ↓
> [ ? ]
>   ↓
> token tiếp theo
> ~~~
>
> Chương này mở chiếc hộp cuối cùng: **làm sao một trạng thái số được biến thành lựa chọn token tiếp theo?**

Hãy quay lại ví dụ quen thuộc:

~~~text
Hà Nội là thủ đô của
~~~

Sau khi chuỗi này đi qua các lớp Transformer, mô hình có một trạng thái cuối cho vị trí cần dự đoán tiếp.

Nhưng trạng thái đó không phải chữ:

~~~text
Việt
~~~

Nó vẫn là một dãy số.

Cần thêm một bước.

## Mỗi token ứng viên nhận một điểm

Mô hình có một tập các token mà nó biết.

Ta có thể tưởng tượng:

~~~text
Việt
Nam
Pháp
Hà
là
.
...
~~~

Từ trạng thái cuối, mô hình tính một điểm cho từng token có thể đứng tiếp.

Các điểm thô này được gọi là **điểm dự đoán (logits)**.

Ví dụ minh họa:

~~~text
Việt   12,4
Nam     7,1
Pháp    2,0
Tokyo   0,3
...     ...
~~~

Các số trên chỉ là ví dụ.

Điều quan trọng là:

> **logit cao hơn nghĩa là mô hình đang ưu tiên token đó hơn trong bước hiện tại.**

## Lớp tạo điểm đầu ra

Phép biến trạng thái cuối thành các logits thường được thực hiện bởi **lớp tạo điểm đầu ra (language-model head, LM head)**.

Ta có thể hình dung:

~~~text
trạng thái cuối
      ↓
lớp tạo điểm đầu ra
      ↓
một điểm cho từng token trong từ vựng
~~~

Nếu từ vựng có hàng chục nghìn token, đầu ra cũng có hàng chục nghìn điểm.

Đây là một lý do lớp đầu ra có thể trở thành một phép tính lớn.

Nửa sau cuốn sách sẽ quay lại câu chuyện này khi ta nhìn xuống phần cứng.

## Logit chưa phải xác suất

Logit là điểm thô.

Nếu muốn biểu diễn chúng thành một phân bố xác suất, ta có thể áp dụng **softmax**.

Ví dụ tưởng tượng:

~~~text
logits
Việt   12,4
Nam     7,1
Pháp    2,0

     ↓ softmax

xác suất tương đối
Việt   0,98
Nam    0,02
Pháp   rất nhỏ
~~~

Một lần nữa, số chỉ để minh họa.

Điểm cần nhớ:

~~~text
logit
≠
xác suất

softmax(logits)
→
phân bố xác suất
~~~

## Chọn token có điểm cao nhất

Cách đơn giản nhất là chọn token có điểm cao nhất.

Đây là **chọn tham lam (greedy decoding)**.

Ví dụ:

~~~text
Việt  12,4  ← cao nhất
Nam    7,1
Pháp   2,0

→ chọn [Việt]
~~~

Sau đó chuỗi trở thành:

~~~text
Hà Nội là thủ đô của Việt
~~~

Mô hình lại chạy thêm một bước.

Ở bước sau:

~~~text
Nam   ← điểm cao nhất
...
~~~

và ta có:

~~~text
Hà Nội là thủ đô của Việt Nam
~~~

## Nhưng chatbot không phải lúc nào cũng chọn điểm cao nhất

Nếu lúc nào cũng chọn token đứng đầu, đầu ra sẽ rất quyết định và dễ lặp lại.

Nhiều hệ thống dùng **lấy mẫu (sampling)**.

Thay vì luôn chọn token cao nhất, hệ thống lấy token từ phân bố xác suất theo một số quy tắc.

Có thể hình dung:

~~~text
Việt   0,70
Pháp   0,10
Nam    0,08
...
~~~

Token có xác suất cao có cơ hội lớn hơn được chọn, nhưng không nhất thiết luôn thắng.

Các tham số như nhiệt độ (temperature), top-k hay top-p điều chỉnh cách lấy mẫu.

Cuốn sách này không cần đi sâu vào chúng.

Điều quan trọng là:

> **mô hình tạo ra phân bố; chiến lược giải mã quyết định cách chọn token từ phân bố đó.**

## Toàn bộ vòng lặp sinh văn bản

Bây giờ ta có thể vẽ toàn bộ quá trình từ chuỗi hiện tại tới chuỗi dài hơn:

~~~text
chuỗi hiện tại
      ↓
chia thành token
      ↓
biểu diễn bằng số
      ↓
nhiều lớp Transformer
      ↓
trạng thái cuối
      ↓
LM head
      ↓
logits
      ↓
chọn / lấy mẫu
      ↓
1 token mới
      ↓
nối vào chuỗi
      ↓
lặp lại
~~~

Đây là vòng lặp làm cho một mô hình sinh văn bản có thể tạo cả đoạn dài.

Mỗi vòng thêm một token.

Nhiều vòng liên tiếp tạo thành câu, đoạn và văn bản.

## Vậy token mới có phải “kết quả của một token cũ” không?

Không nên nghĩ theo quan hệ một-một:

~~~text
token A
↓
token B
~~~

Token mới được dự đoán từ **trạng thái của toàn chuỗi ngữ cảnh hiện tại**, sau nhiều lớp biến đổi.

Vì vậy hình đúng hơn là:

~~~text
[token 1] [token 2] [token 3] ... [token n]
          ↓
     toàn chuỗi được xử lý
          ↓
     trạng thái cuối
          ↓
     token n+1
~~~

Token mới là kết quả của cả quá trình, không phải một token cũ tự biến thành token mới.

## “Mảnh tri thức” ở cuối đường đã xuất hiện chưa?

Ta đã đi khá xa.

Từ:

~~~text
token
~~~

tới:

~~~text
biểu diễn
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
logits
↓
token mới
~~~

Nhưng vẫn chưa có một điểm nào cho phép ta nói:

> “Đây chính là viên tri thức.”

Thứ ta thấy là:

> **một quá trình biến đổi trạng thái dẫn tới một phân bố dự đoán.**

Đây là kết luận quan trọng của nửa đầu cuốn sách.

Tri thức mà mô hình thể hiện có vẻ không phải một vật thể chạy nguyên vẹn qua dây chuyền.

Nó xuất hiện trong **quan hệ giữa trọng số đã học, ngữ cảnh và chuỗi biến đổi**.

## Nhưng những hộp ta vừa vẽ có thật sự chạy như vậy trên GPU không?

Đây là câu hỏi mở sang nửa sau.

Cho tới giờ, ta nói:

~~~text
attention
FFN
LM head
~~~

như những hộp logic.

Nhưng phần cứng không nhìn thấy sơ đồ sách.

GPU nhận những công việc cụ thể.

Một phép tính logic có thể bị chia thành nhiều công việc vật lý.

Nhiều phép tính logic cũng có thể được gộp.

Bộ nhớ, đồng bộ và cách lập lịch đều tham gia.

Nói cách khác:

> **Đường đi logic của token chưa phải đường đi vật lý của token.**

Đó là chiếc cửa tiếp theo.

### Nhớ 3 điều

1. **Trạng thái cuối được biến thành điểm dự đoán (logits) cho các token có thể đứng tiếp.**
2. **Chiến lược giải mã có thể chọn token cao nhất hoặc lấy mẫu từ phân bố; token mới được nối vào chuỗi rồi toàn quá trình lặp lại.**
3. **Token mới không phải do một token cũ tự biến thành nó; nó được dự đoán từ trạng thái của toàn ngữ cảnh sau nhiều lớp biến đổi.**

**Tiếp theo: [Chương 11 — Một phép tính logic không phải một chương trình GPU](11-phep-tinh-logic-khong-phai-kernel.md)**
