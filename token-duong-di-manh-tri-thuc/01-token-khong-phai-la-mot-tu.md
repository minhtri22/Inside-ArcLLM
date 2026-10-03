# Chương 1 — Token không phải là một từ

> **Mức đọc: Nền tảng**
>
> **Ta đang đứng ở đâu?**
>
> ~~~text
> văn bản
>    ↓
> [ chia thành những mảnh nhỏ ]
>    ↓
> token
> ~~~
>
> Chương này chỉ trả lời một câu hỏi: **token là gì và vì sao “một token = một từ” chỉ là cách hiểu tạm thời?**

Hãy bắt đầu bằng một câu rất bình thường:

~~~text
Tôi thích cà phê.
~~~

Nếu được yêu cầu chia câu này thành những mảnh nhỏ, có lẽ ta sẽ làm:

~~~text
Tôi | thích | cà | phê | .
~~~

Đây là một cách rất tốt để bắt đầu hiểu token.

Ở mức đầu tiên, ta có thể nói:

> **Token là một mảnh văn bản nhỏ mà mô hình xử lý.**

Cách hiểu này chưa hoàn toàn chính xác.

Nhưng nó đủ để ta đi bước đầu tiên mà không bị ngợp.

## Vì sao không nói luôn “token là một từ”?

Bởi vì mô hình không nhất thiết chia văn bản theo cách con người chia từ.

Hãy lấy một ví dụ khác:

~~~text
ChatGPT
~~~

Một bộ tách có thể giữ nguyên thành một mảnh.

Một bộ khác có thể chia thành:

~~~text
Chat | GPT
~~~

Một bộ khác nữa có thể chia khác.

Điều tương tự xảy ra với từ dài, từ hiếm, tên riêng, số, dấu câu, khoảng trắng, tiếng Việt, tiếng Anh và mã nguồn.

Vì vậy câu “một token là một từ” rất dễ nhớ nhưng quá mạnh.

Cách nói tốt hơn là:

> **Một token thường tương ứng với một mảnh văn bản. Mảnh đó có thể là một từ, một phần của từ, dấu câu hoặc một chuỗi ký tự khác.**

## Ai quyết định cách chia?

Không phải người dùng.

Không phải phần cứng đang chạy mô hình.

Cũng không phải mỗi lớp Transformer tự nghĩ ra cách chia riêng.

Có một thành phần đứng trước mô hình làm việc đó.

Ta gọi nó là **bộ tách và mã hóa văn bản (tokenizer)**.

Có thể hình dung:

~~~text
"Tôi thích cà phê."
        ↓
bộ tách và mã hóa
        ↓
[token A] [token B] [token C] ...
~~~

Tên tiếng Anh tokenizer rất phổ biến, nên ta giữ lại trong ngoặc khi giới thiệu.

Sau này nếu bạn đọc tài liệu thấy chữ tokenizer, bạn đã biết nó đang nói tới phần nào của quá trình.

## Token không nhất thiết nhìn giống phần chữ ta tưởng tượng

Ta thích nghĩ rằng:

~~~text
token
=
một đoạn chữ có nghĩa
~~~

Nhưng hệ thống không bắt buộc phải chiều theo trực giác đó.

Một token có thể chứa dấu cách ở đầu.

Một từ có thể bị chia làm nhiều token.

Hai từ ngắn có thể được biểu diễn theo cách không giống cách ta tự chia bằng mắt.

Điều này không phải lỗi.

Mục tiêu của bộ tách không phải tạo ra những mảnh đẹp nhất cho con người.

Mục tiêu là tạo ra một bộ đơn vị mà mô hình có thể xử lý hiệu quả và nhất quán.

## Một ví dụ gần gũi: bộ chữ ghép

Hãy tưởng tượng ta chỉ có một hộp những mảnh ghép chữ.

Trong hộp không có mọi từ tiếng Việt.

Thay vào đó có những mảnh như:

~~~text
"cà"
" ph"
"ê"
" rất"
" ngon"
"."
~~~

Muốn tạo câu “cà phê rất ngon.”, ta ghép những mảnh có sẵn lại.

Bộ token hoạt động gần với trực giác đó hơn là một cuốn từ điển “mỗi từ đúng một mục”.

Hình dung này vẫn chưa phải toàn bộ cơ chế của các bộ tách hiện đại.

Nhưng nó giúp trả lời một câu hỏi quan trọng:

> Vì sao một từ đôi khi lại tốn nhiều token?

Bởi hệ thống có thể phải dùng nhiều mảnh nhỏ hơn để ghép thành từ đó.

## Token có ý nghĩa không?

Có.

Và không.

Một token như “Paris” rõ ràng gợi cho con người rất nhiều ý nghĩa.

Nhưng token trong hệ thống trước hết chỉ là **một đơn vị được bộ mã hóa nhận diện**.

Mô hình chưa trực tiếp cầm cả “Paris, nước Pháp, tháp Eiffel, lịch sử, địa lý...” trong một chiếc hộp token.

Ta cần phân biệt:

~~~text
mảnh văn bản
≠
toàn bộ ý nghĩa mà con người liên tưởng tới mảnh đó
~~~

Đây là bước đầu tiên để hiểu tên cuốn sách.

Nếu token không tự chứa nguyên một mảnh tri thức, vậy tri thức xuất hiện bằng cách nào?

Chưa vội.

Trước hết còn một bước quan trọng hơn.

## Token sẽ được đổi thành một con số

Máy tính không gửi chữ “Paris” đi xuyên qua hàng chục lớp theo đúng dạng ký tự mà ta đang nhìn.

Mỗi token được gắn với một mã số.

Ta gọi đó là **mã token (token ID)**.

Ví dụ minh họa:

~~~text
"Paris" → 8421
~~~

Con số 8421 ở đây chỉ là ví dụ.

Điều cần nhớ là:

~~~text
văn bản
↓
token
↓
mã token
~~~

Nhưng mã token cũng chưa phải thứ các lớp của mô hình thực sự dùng để làm phần lớn phép tính.

Ở chương sau nữa, ta sẽ thấy nó còn phải được đổi thành một biểu diễn bằng số phong phú hơn.

## Vì sao việc chia token quan trọng?

Giả sử cùng một câu được chia thành 10 token thay vì 14 token.

Điều đó có thể ảnh hưởng tới số bước mô hình phải xử lý, độ dài ngữ cảnh, lượng bộ nhớ cần dùng, thời gian xử lý và cách các mảnh văn bản tương tác với nhau.

Vì vậy việc chia token không chỉ là “cắt câu cho đẹp”.

Nó là một phần của cách văn bản bước vào thế giới số của mô hình.

## Nhưng đừng suy diễn quá xa

Một lỗi khác cũng dễ mắc:

> “Nếu biết cách tokenizer chia câu, ta đã biết mô hình hiểu câu thế nào.”

Không.

Ta mới chỉ biết **cách câu được chia thành đơn vị đầu vào**.

Cách những đơn vị đó được biểu diễn và biến đổi về sau là một câu chuyện khác.

~~~text
cách chia token
= cách chia đầu vào

hiểu ý nghĩa
≠ chỉ riêng cách chia đầu vào
~~~

## Câu hỏi nhỏ nhưng rất quan trọng

Quay lại câu:

~~~text
Hà Nội là thủ đô của Việt Nam.
~~~

Nếu token “Hà” hay “Hà Nội” xuất hiện, nó có mang theo tri thức “Hà Nội là thủ đô của Việt Nam” hay không?

Nếu câu trả lời là “có”, tri thức đó nằm ở đâu trong token?

Nếu câu trả lời là “không”, tại sao mô hình vẫn có thể hoàn thành câu “Hà Nội là thủ đô của ...” bằng “Việt Nam”?

Đây là câu hỏi của Chương 2.

### Nhớ 3 điều

1. **Token là một mảnh văn bản mà mô hình xử lý, nhưng không nhất thiết là một từ.**
2. **Bộ tách và mã hóa văn bản (tokenizer) quyết định cách văn bản được chia và gắn mã token.**
3. **Biết token là gì chưa có nghĩa biết “tri thức nằm ở đâu”.** Token mới chỉ là đơn vị đầu vào của câu chuyện.

**Tiếp theo: [Chương 2 — Token cũng không phải là tri thức](02-token-khong-phai-la-tri-thuc.md)**
