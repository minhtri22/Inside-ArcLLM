# Chương 19 — Từ ArcLLM tới một cách mô tả hệ thực thi tổng quát hơn

> **Mức đọc: Nâng cao**
>
> **Bạn đang mở câu hỏi nào?**
>
> ```text
> Danh tính
>    ↓
> có thể thực thi?
>    ↓
> sẵn sàng chưa?
>    ↓
> đang ở đâu?
>    ↓
> lấy bằng cách nào?
>    ↓
> giữ bao lâu?
> ```


> **Câu hỏi của chương:** Sáu câu hỏi mà ArcLLM vừa phải tách ra có thể trở thành một mô hình chung cho nhiều kiểu đường thực thi khác nhau hay không — mà không nhét luật riêng của từng trường hợp vào lõi hệ thực thi?

Cuối Chương 18, ta có sáu câu hỏi.

```text
đây là gì?

đường thực thi có tồn tại không?

có chạy được ngay không?

dữ liệu cần thiết đã ở trong bộ nhớ chưa?

nếu thiếu thì lấy hoặc tạo bằng cách nào?

sau đó giữ nó tới bao giờ?
```

Chúng tương ứng với sáu chiều:

```text
danh tính

khả dụng thực thi

sẵn sàng thực thi

trạng thái cư trú

quá trình thu nhận

vòng đời
```

Nhìn riêng từng câu, không có gì quá đặc biệt.

Điều khó nằm ở chỗ:

> **Một mô hình hệ thực thi chỉ có giá trị nếu nhiều loại cơ chế rất khác nhau cùng đi qua được sáu chiều đó mà không cần lõi chính sách biết tên từng cơ chế.**

Nếu lõi phải viết:

```text
nếu là EXEC148 thì...

nếu là Split-K32 thì...

nếu là representation phân đoạn thì...
```

thì ta chưa có một lớp trừu tượng tổng quát.

Ta chỉ có một danh sách `if/else` được đặt tên đẹp hơn.

## Vì sao cần thử trên nhiều loại cơ chế?

EXEC148 đã dạy cho hệ thực thi về cách biểu diễn dữ liệu.

Nhưng nếu ta thiết kế toàn bộ mô hình chỉ từ EXEC148, rất dễ vô tình biến những đặc điểm riêng của nó thành luật chung.

Ví dụ EXEC148 có:

- một cách biểu diễn phụ;
- chi phí tạo;
- khả năng dùng đường A làm dự phòng;
- quyết định phụ thuộc vào mức tái sử dụng.

Nếu ta nhìn riêng trường hợp đó, rất dễ nghĩ:

> “Mọi đường thực thi ưu tiên đều phải có cách biểu diễn dữ liệu riêng.”

Hoặc:

> “Mọi quá trình thu nhận đều phải có ngưỡng tái sử dụng.”

Nhưng Chương 18 đã cho thấy cả hai đều sai.

Vì vậy muốn kiểm tra một lớp trừu tượng, ta cần những **phản ví dụ độc lập**.

Không phải những trường hợp được tạo ra để vừa với kiến trúc.

Mà là những cơ chế đã tồn tại trước đó, với bằng chứng riêng của chúng.

## Ba họ cơ chế rất khác nhau

Tới thời điểm này, ArcLLM có ba kiểu đủ khác nhau để thử mô hình.

### Họ thứ nhất — cách biểu diễn dữ liệu tùy chọn

Đây là câu chuyện A/B của Chương 16 và 17.

```text
A
→ chạy trực tiếp từ Q4_K gốc

B
→ cần EXEC148
```

Nếu B chưa có, A vẫn là một đường hợp lệ.

B chỉ đáng được tạo khi điều kiện sử dụng làm chi phí đó hợp lý.

Ta có:

```text
đường ưu tiên
→ B

đường dự phòng
→ A

representation phụ
→ có

tạo representation
→ tùy chọn, dựa trên mức tái sử dụng
```

### Họ thứ hai — cách biểu diễn bắt buộc trong phép kiểm tra P8 có giới hạn

Ở có giới hạn P8 case đã được kiểm tra, cách biểu diễn phân đoạn là điều kiện để đường thực thi có thể hoạt động trong đúng phạm vi evidence đó.

Không có một đường dự phòng đã được xác nhận tương đương.

Ta có:

```text
representation cần thiết đã có
→ có thể đi tiếp

representation chưa có
nhưng có thể tạo
→ tạo

representation chưa có
và hiện không thể tạo
→ NOT_READY
```

Ở đây không được hỏi:

> “Có đủ token để hoàn vốn không?”

Bởi việc tạo cách biểu diễn dữ liệu là điều kiện để phép tính khả thi.

### Họ thứ ba — không có cách biểu diễn phụ

Đây là cơ chế Split-K32 của Chương 14.

Nó chỉ thay cách chia công việc.

```text
Q4_K gốc
↓
Split-K32
```

Không tạo image phụ.

Không cần bước thu nhận dữ liệu mới.

Không có vòng đời cách biểu diễn dữ liệu riêng.

Nếu đường này sẵn sàng:

```text
chạy
```

Nếu không:

```text
đi đường dự phòng đã xác nhận
```

Ba họ này khác nhau đủ để bắt đầu thử xem sáu khái niệm có thực sự độc lập hay không.

## Phiên bản trước đã KHÔNG ĐẠT vì một giả định ẩn

Mô hình trước đó xử lý được hai họ đầu.

Nó biết:

- cách biểu diễn dữ liệu có thể có hoặc không;
- có thể có đường dự phòng hoặc không;
- việc tạo cách biểu diễn dữ liệu có thể dựa trên tái sử dụng hoặc bắt buộc để khả thi.

Nhưng khi đưa cơ chế trực tiếp của Chương 14 vào như một phép thử độc lập, mô hình KHÔNG ĐẠT (FAIL).

Lý do ta đã gặp ở Chương 18:

> **Nó vẫn ngầm gắn “sẵn sàng thực thi” với “cách biểu diễn dữ liệu đang ở trong bộ nhớ”.**

Cơ chế trực tiếp không có cách biểu diễn dữ liệu riêng.

Vì vậy không thể mô tả nó trung thực mà không giả vờ rằng một thứ không tồn tại đang `resident`.

KHÔNG ĐẠT (FAIL) đó dẫn tới một thay đổi rất nhỏ:

```text
execution_ready
```

Một giá trị đúng/sai riêng cho:

> **Đường thực thi này có thể xử lý yêu cầu hiện tại ngay bây giờ hay không?**

Chỉ một biến.

Không phải một cây trạng thái lớn.

Nhưng nó tách được hai câu hỏi trước đó bị dính vào nhau.

## Sáu chiều của mô hình v4

Sau các vòng sửa bằng phản ví dụ, ArcLLM khóa một bề mặt gồm sáu chiều.

Tên nội bộ là **v4**.

`v4` ở đây không có nghĩa toàn bộ ArcLLM đã trở thành “ArcLLM phiên bản 4”.

Nó chỉ là phiên bản thứ tư của **bề mặt khái niệm dùng để mô tả các đường thực thi và cách biểu diễn dữ liệu**.

Ta đi từng chiều.

## 1. Danh tính

Câu hỏi:

> **Ta đang nói về chính xác thứ gì?**

Một cách biểu diễn dữ liệu không chỉ cần tên:

```text
EXEC148
```

Nó còn phải gắn với đúng:

- mô hình;
- khối số;
- định dạng;
- phiên bản;
- bằng chứng đã xác nhận nó.

Nếu một vùng dữ liệu được tạo cho mô hình A nhưng hệ thực thi đang chạy mô hình B, việc nó vẫn nằm trong bộ nhớ không làm nó hợp lệ.

Vì vậy:

```text
có dữ liệu
≠
đúng dữ liệu
```

Danh tính phải được xác minh trước khi route.

## 2. Khả dụng thực thi

Câu hỏi:

> **Đường thực thi có tồn tại và về nguyên tắc có thể dùng hay không?**

Ví dụ binary chương trình GPU tương ứng có thể đã được build và lớp thực thi phần cứng biết cách gọi nó.

Khi ấy ta có thể nói:

```text
execution_available = true
```

Nhưng điều đó chưa có nghĩa:

> “Hãy chạy ngay.”

Bởi còn câu hỏi tiếp theo.

## 3. Sẵn sàng thực thi

Câu hỏi:

> **Đường đó có thể xử lý yêu cầu hiện tại ngay lúc này hay không?**

Một executor có thể tồn tại nhưng tạm thời không sẵn sàng.

Hai chiều vì vậy khác nhau:

```text
có đường
≠
đường đang dùng được ngay
```

Điều quan trọng hơn:

> **Sẵn sàng thực thi độc lập với việc có cách biểu diễn dữ liệu phụ hay không.**

Cơ chế Split-K32 trực tiếp chứng minh điều này.

Nó có thể:

```text
sẵn sàng thực thi = có
```

trong khi:

```text
representation phụ = không tồn tại
```

và hoàn toàn không có vấn đề gì.

## 4. Trạng thái cư trú

Câu hỏi:

> **Một cách biểu diễn dữ liệu được tạo riêng có đang tồn tại trong vùng bộ nhớ cần thiết hay không?**

Đây là nghĩa hẹp của **trạng thái cư trú — trạng thái cư trú trong bộ nhớ**.

Không dùng nó để biểu diễn:

- executor có tồn tại;
- executor có chạy được;
- request có hợp lệ;
- hay dữ liệu có thể được tạo.

Nó chỉ nói:

```text
representation phụ
có ở trong bộ nhớ hay không?
```

Một trạng thái — một câu hỏi.

## 5. Quá trình thu nhận

Câu hỏi:

> **Nếu cách biểu diễn dữ liệu cần thiết chưa có, hệ thực thi có con đường hợp lệ nào để làm nó xuất hiện không?**

ArcLLM hiện có bằng chứng cho ít nhất hai loại.

```text
tạo nếu tái sử dụng đủ để bù chi phí
```

và:

```text
tạo bắt buộc để phép tính khả thi
```

Loại thứ hai không được phép mang ngưỡng tái sử dụng.

Nếu không có bằng chứng cho một bước tạo cách biểu diễn dữ liệu:

```text
không được bịa ra acquisition
```

Đó chính là điều giữ cho cơ chế trực tiếp vẫn là cơ chế trực tiếp.

## 6. Vòng đời

Câu hỏi:

> **cách biểu diễn dữ liệu đang có mặt thì khi nào giữ, khi nào loại và khi nào mất hiệu lực?**

Ví dụ:

```text
model bị dỡ
→ representation đi kèm phải mất hiệu lực
```

hoặc:

```text
identity không còn đúng
→ loại
```

Nhưng:

```text
executor tạm thời chưa sẵn sàng
```

không tự động có nghĩa:

```text
xóa representation
```

Một vùng dữ liệu đắt tiền vẫn có thể hoàn toàn hợp lệ và đáng giữ để dùng lại sau.

Đó là lý do vòng đời phải độc lập với trạng thái sẵn sàng thực thi.

## Sáu chiều giúp tránh những suy luận sai nào?

Hãy lấy một trạng thái giả định:

```text
EXEC148 đúng model
→ có

EXEC148 đang trong bộ nhớ
→ có

đường B tồn tại
→ có

đường B hiện chưa sẵn sàng
→ không
```

Nếu chỉ có biến `resident`, hệ thực thi rất dễ nói:

> “Dữ liệu có rồi, dùng B.”

Sai.

Nếu cứ thấy `execution_ready = false` rồi xóa cách biểu diễn dữ liệu:

> cũng sai.

Mô hình sáu chiều cho phép:

```text
giữ EXEC148

không route B lúc này

có thể dùng A nếu A sẵn sàng

sau đó B có thể trở lại sẵn sàng
mà không phải tạo EXEC148 lại
```

Một ví dụ khác:

```text
representation bắt buộc
→ chưa có

có thể tạo
→ không
```

Nếu không có đường dự phòng:

```text
NOT_READY
```

Không phải:

```text
chạy đại một kernel khác
```

Sự tách biệt giữa các chiều làm những quyết định này trở nên rõ ràng.

## Làm sao biết mô hình mới không phá hành vi cũ?

Đây là một vấn đề rất quan trọng.

Mỗi lần thêm một khái niệm mới vào lõi chính sách, ta có nguy cơ sửa được trường hợp mới nhưng âm thầm làm đổi trường hợp cũ.

Vì vậy ArcLLM giữ một ma trận quyết định của họ A/B đầu tiên.

Tổng số tổ hợp trạng thái:

```text
114.688
```

Cần hiểu đúng con số này.

Đây **không phải**:

```text
114.688 lần chạy model
```

Nó là:

> **114.688 tổ hợp trạng thái đầu vào của logic chính sách.**

Ví dụ các trạng thái có thể khác nhau ở:

- cách biểu diễn dữ liệu đã có hay chưa;
- danh tính còn hợp lệ hay không;
- mức tái sử dụng;
- khả năng bắt đầu tạo dữ liệu;
- đường ưu tiên có sẵn hay không;
- đường dự phòng có sẵn hay không;
- các điều kiện vòng đời khác.

Sau khi thêm các khái niệm mới, toàn bộ:

```text
114.688 / 114.688
```

quyết định cũ phải giữ nguyên.

Không được sửa I002 rồi làm EXEC148 đổi hành vi.

Không được sửa cách biểu diễn dữ liệu bắt buộc rồi phá đường A/B.

Đó là một dạng:

> **regression phép kiểm tra — bộ đối chứng hồi quy**, dùng để bảo đảm lớp trừu tượng mới không viết lại những gì trước đó đã được chứng minh.

## Một biến đúng/sai có thực sự đủ không?

Sau khi thêm `execution_ready`, một câu hỏi hợp lý là:

> Có cần thiết kế một máy trạng thái phức tạp hơn không?

Ví dụ:

```text
READY
BLOCKED
WAITING
STALE
ACQUIRING
...
```

ArcLLM không thêm.

Thay vào đó, nó cố phá mô hình nhỏ nhất.

Các tình huống được thử gồm:

```text
representation hợp lệ
nhưng executor tạm thời chưa sẵn sàng
```

```text
đường ưu tiên chưa sẵn sàng
nhưng đường dự phòng sẵn sàng
```

```text
cả ưu tiên lẫn dự phòng đều chưa sẵn sàng
```

```text
đang có một acquisition
không được phát lệnh tạo lần thứ hai
```

```text
identity hết hiệu lực
trong khi readiness vẫn báo có
```

Kết quả cho thấy một giá trị đúng/sai:

```text
execution_ready
```

vẫn đủ cho các họ đã được kiểm tra.

Không cần một máy trạng thái lớn hơn.

Đây là một nguyên tắc đã lặp lại nhiều lần trong cuốn sách:

> **Không xây độ phức tạp để phòng một tương lai tưởng tượng. Chỉ thêm nó khi một phản ví dụ thật buộc ta phải làm vậy.**

## Thứ tự quyết định cũng quan trọng

Sáu chiều không chỉ cần tồn tại.

Hệ thực thi còn phải dùng chúng theo một thứ tự có nghĩa.

Một ví dụ quan trọng:

> **Nếu identity hoặc vòng đời cho biết cách biểu diễn dữ liệu đã không còn hợp lệ, phải xử lý việc đó trước khi xét đường thực thi có sẵn sàng hay không.**

Tại sao?

Giả sử:

```text
EXEC148 vẫn nằm trong bộ nhớ

execution_ready = true
```

nhưng cách biểu diễn dữ liệu đó thuộc mô hình cũ.

Nếu nhìn readiness trước, hệ thực thi có thể route vào dữ liệu sai.

Vì vậy:

```text
danh tính / vòng đời
↓
sẵn sàng thực thi
```

chứ không phải ngược lại.

Tương tự:

> **Một đường thực thi chỉ được route khi vừa khả dụng, vừa sẵn sàng.**

Nếu nó còn cần cách biểu diễn dữ liệu riêng thì cách biểu diễn dữ liệu đó còn phải:

- đang cư trú;
- đúng danh tính;
- còn hiệu lực.

Đây là cách sáu chiều trở thành một mô hình quyết định chứ không chỉ là sáu biến rời rạc.

## Đường dự phòng cũng không được mặc định là luôn sẵn sàng

Một giả định rất dễ mắc là:

> “Nếu đường ưu tiên lỗi thì cứ dùng mốc đối chứng.”

Nhưng đường dự phòng cũng là một đường thực thi thật.

Nó cũng có thể tạm thời chưa khả dụng hoặc chưa sẵn sàng.

Vì vậy mô hình không được viết:

```text
preferred không chạy
→ fallback
```

mà phải gần hơn với:

```text
preferred hợp lệ + sẵn sàng
→ dùng preferred

nếu không:
fallback hợp lệ + sẵn sàng
→ dùng fallback

nếu không có đường nào sẵn sàng
→ NOT_READY
```

Điều này nghe rất hiển nhiên sau khi đã viết ra.

Nhưng nó chỉ trở nên hiển nhiên vì những trường hợp trước đó đã buộc ta tách từng khái niệm.

## Điều quan trọng nhất: lõi không biết tên từng họ

Mục tiêu thật của bề mặt v4 là:

> **Không cần một nhánh chính sách riêng cho từng cơ chế đã nghiên cứu.**

Lõi không cần biết:

```text
đây là EXEC148
```

hay:

```text
đây là Split-K32 gate/up
```

Nó chỉ cần các mô tả như:

```text
có representation riêng không?

có đường dự phòng không?

việc tạo representation thuộc loại nào?

đường ưu tiên có sẵn sàng không?

representation có đang hợp lệ không?

điều kiện vòng đời là gì?
```

Nếu một họ mới chỉ có thể hoạt động bằng cách thêm:

```text
if family == X
```

vào lõi chung, đó là dấu hiệu lớp trừu tượng có thể vẫn thiếu một khái niệm.

## Nhưng tổng quát trên giấy vẫn chưa đủ

Tới đây ta vẫn có thể mắc một sai lầm.

Một mô hình chính sách có thể ĐẠT (PASS) mọi phép thử logic nhưng khi gắn vào Vulkan thật lại buộc lớp thực thi phần cứng phải lén làm thêm những việc mà lõi không biết.

Khi ấy lớp trừu tượng chỉ đúng trên giấy.

Vì vậy ArcLLM còn đưa v4 xuống một lớp kết nối Vulkan Q4 thật.

Các tình huống thực tế bao gồm:

```text
A được chọn
→ không tạo EXEC148
```

```text
B chưa sẵn sàng
→ không âm thầm tạo hoặc thử lại trái chính sách
```

```text
chính sách yêu cầu tạo B
→ cấp phát
→ materialize
→ xác minh identity
→ dùng B
```

```text
B đã hợp lệ trong bộ nhớ
→ dùng lại
→ không tạo lần hai
```

```text
B bị loại
→ quay về A khi A là đường hợp lệ
```

Tất cả đều ĐẠT (PASS).

Điều quan trọng là:

> **lớp thực thi phần cứng thật không cần sửa nghĩa của sáu chiều và không cần thêm một nhánh chính sách bí mật dành riêng cho Q4.**

## Một kết quả KHÔNG ĐẠT khác lại giúp phân biệt thí nghiệm với hệ thực thi

Quá trình gắn vào lớp thực thi phần cứng không ĐẠT (PASS) ngay từ lần đầu.

Có một KHÔNG ĐẠT (FAIL) đáng giữ.

Nguyên nhân không phải sáu chiều sai.

Nó đến từ việc một **harness thí nghiệm 4-arm** trước đó được dùng như thể nó là lớp thực thi phần cứng vòng đời thật.

Nhắc lại mục tiêu của thí nghiệm 2×2:

```text
0
A
B
AB
```

Để so bốn arm công bằng, harness đó chủ động chuẩn bị B theo thiết kế của Thí nghiệm.

Điều này hoàn toàn đúng cho thí nghiệm.

Nhưng hệ thực thi thật lại cần:

> **chỉ tạo B khi chính sách quyết định rằng B cần được tạo.**

Nếu lấy harness thí nghiệm rồi coi nó là hệ thực thi, B sẽ bị tạo quá sớm.

Ta sẽ vô tình biến:

```text
acquisition theo nhu cầu
```

thành:

```text
luôn tạo B
```

và phá chính semantics vừa xây.

Sau khi tách đúng lớp kết nối Vulkan theo nhu cầu, phép kiểm tra lại ĐẠT (PASS).

Bài học rất quan trọng:

> **Một công cụ thí nghiệm đúng không mặc nhiên là một kiến trúc hệ thực thi đúng.**

Harness được thiết kế để trả lời một câu hỏi khoa học.

Hệ thực thi được thiết kế để thực hiện một chính sách trong đời sống hệ thống.

Hai mục tiêu khác nhau.

## v4 đã chứng minh được gì?

Tới đây, có thể nói:

> **Một bề mặt gồm sáu chiều đã mô tả được các họ cơ chế mà ArcLLM thực sự kiểm tra mà không cần nhánh chính sách riêng cho từng họ.**

Sáu chiều là:

```text
1. danh tính

2. khả dụng thực thi

3. sẵn sàng thực thi

4. trạng thái cư trú

5. quá trình thu nhận

6. vòng đời
```

Nó giữ nguyên:

```text
114.688 / 114.688
```

quyết định của họ đầu tiên.

Nó biểu diễn được trường hợp cách biểu diễn dữ liệu bắt buộc trong có giới hạn P8 phép kiểm tra đã kiểm tra.

Nó biểu diễn được cơ chế trực tiếp không cần cách biểu diễn dữ liệu phụ.

Và một lớp kết nối Vulkan thật có thể tuân theo các ranh giới đó.

Nhưng vẫn phải giữ giới hạn.

## v4 chưa phải “kiến trúc phổ quát cho mọi hệ thực thi AI”

Tên “tổng quát” rất dễ tạo cảm giác lớn hơn evidence.

Bằng chứng hiện tại không cho phép nói:

> “Mọi primitive trong mọi mô hình, mọi GPU và mọi hệ thực thi đều chỉ cần sáu chiều này.”

Điều được chứng minh nhỏ hơn:

> **Sáu chiều này đủ cho các lớp đã có bằng chứng hiện tại.**

Một họ tương lai hoàn toàn có thể đưa ra phản ví dụ mới.

Nếu vậy, v4 phải được mở lại.

Không có gì sai với điều đó.

Thực ra chính lịch sử từ Chương 17 đến đây cho thấy:

```text
một abstraction tốt
không phải abstraction không bao giờ thay đổi

mà là abstraction biết rõ
bằng chứng nào đang chống đỡ nó
```

## Từ kiến trúc riêng của ArcLLM tới một bề mặt mở rộng

Đầu cuốn sách, việc thêm một đường thực thi mới thường có nghĩa:

```text
viết kernel
↓
gắn trực tiếp vào runtime
```

Tới đây, hình ảnh đã khác.

Một đường mới cần mô tả:

```text
danh tính
↓
executor nào tồn tại
↓
khi nào executor sẵn sàng
↓
có representation phụ không
↓
nếu thiếu thì tạo thế nào
↓
representation sống bao lâu
```

Sau đó lớp thực thi phần cứng phần cứng cụ thể thực hiện các hành động đó.

Ta bắt đầu có một ranh giới:

```text
CHÍNH SÁCH CHUNG

“nên dùng gì?”
“có cần tạo gì?”
“có được route không?”
“có cần loại dữ liệu không?”

          ↓

LỚP KẾT NỐI PHẦN CỨNG

“Vulkan thực hiện quyết định đó như thế nào?”
```

Đây chính là ý nghĩa thực tế của một **bề mặt mở rộng tổng quát**.

Không phải plugin framework đồ sộ.

Không phải kiến trúc được thiết kế trước để “sau này có thể mở rộng”.

Mà là:

> **một ranh giới tối thiểu đã đủ để những cơ chế thật khác nhau cùng đi qua mà không phá lõi.**

## Và vẫn chưa có kết luận hiệu năng mới

Có một chi tiết cần giữ thật rõ.

Các bước xây v4, kiểm tra 114.688 trạng thái và gắn vào lớp thực thi phần cứng Vulkan không phải một benchmark hiệu năng mới.

Chúng xác nhận:

- nghĩa của các trạng thái;
- quyết định định tuyến;
- quá trình tạo dữ liệu;
- vòng đời;
- tính đúng;
- và ranh giới giữa lõi chung với lớp thực thi phần cứng.

Chúng **không** xác nhận:

> “v4 làm ArcLLM nhanh hơn.”

Không có kết luận đó.

Một kiến trúc phần mềm có thể tốt hơn về khả năng mô tả và quản lý hệ thống mà chưa tạo ra một speedup mới.

Đây là một loại ĐẠT (PASS) khác.

## Lớp trừu tượng cuối cùng lại quay về cùng nguyên tắc đầu sách

Chúng ta không bắt đầu với sáu chiều.

Không ai ngồi ở Chương 1 rồi nói:

```text
ArcLLM phải có:
identity
execution availability
execution readiness
residency
acquisition
lifecycle
```

Nếu làm vậy, đó chỉ là một thiết kế đẹp chưa có lý do.

Thay vào đó:

```text
tensor phải resident
↓
EXEC148 xuất hiện
↓
representation có chi phí riêng
↓
cần acquisition và lifecycle
↓
họ bắt buộc phá logic tái sử dụng
↓
cơ chế trực tiếp phá liên kết readiness-residency
↓
phản ví dụ chuyển tiếp thử phá readiness
↓
backend thật thử phá toàn bộ mô hình
↓
sáu chiều còn đứng vững
```

Đó là lý do ta có thể tin chúng nhiều hơn một sơ đồ được nghĩ ra từ trước.

Không phải vì chúng đẹp.

Mà vì chúng đã bị thử phá.

### Nhớ 3 điều

1. **v4 tách sáu câu hỏi độc lập:** danh tính, khả dụng thực thi, sẵn sàng thực thi, trạng thái cư trú, quá trình thu nhận và vòng đời. Mỗi chiều chỉ nên trả lời một câu hỏi.
2. **Một lớp trừu tượng chỉ đáng tin khi những họ khác nhau cùng đi qua được mà không cần luật riêng cho từng họ.** Mô hình giữ nguyên 114.688 quyết định của họ đầu tiên, đồng thời biểu diễn được có giới hạn P8 mandatory-feasibility case và cơ chế trực tiếp không có cách biểu diễn dữ liệu phụ.
3. **“Tổng quát” không có nghĩa “phổ quát”.** v4 chỉ được xác nhận trong các lớp đã có bằng chứng. Một phản ví dụ tương lai có quyền mở lại kiến trúc.

**Chương 20 — Ta đã hiểu hệ thực thi đến đâu?**

Tới đây các mảnh đã hội tụ:

```text
một production path thật

các cơ chế được đo

PASS và FAIL được giữ lại

representation tách khỏi execution

acquisition tách khỏi residency

readiness tách khỏi cả hai

vòng đời có chủ thể quản lý

backend thật tuân theo cùng semantics
```

Nhưng một câu hỏi cuối vẫn còn:

> **Khi đưa toàn bộ những phần đã được xác nhận trở lại một hệ thực thi hoàn chỉnh, điều gì thực sự đã được chứng minh — và ranh giới nào vẫn phải để mở?**
