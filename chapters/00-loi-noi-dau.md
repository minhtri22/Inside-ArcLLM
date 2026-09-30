# Lời nói đầu — Thư gửi người đọc

Có lẽ bạn đã từng làm việc này:

```text
gõ một câu hỏi
↓
nhấn gửi
↓
vài giây sau
↓
AI trả lời
```

Nhìn từ bên ngoài, mọi thứ rất đơn giản.

Nhưng chính sự đơn giản đó dễ khiến chúng ta quên rằng phía sau ô chat là một cỗ máy rất phức tạp: dữ liệu, hàng tỷ con số đã học, bộ nhớ, những phép tính lặp đi lặp lại và phần cứng thật sự phải thực hiện chúng.

Cuốn sách này bắt đầu từ một mong muốn rất giản dị:

> **Nếu không coi AI là một chiếc hộp bí ẩn, ta có thể tự mở nó ra và hiểu từng lớp bên trong không?**

Tôi chọn một cách khá cực đoan để trả lời câu hỏi đó: thử tự xây một **hệ thực thi** cho mô hình ngôn ngữ lớn.

“Hệ thực thi” là gì thì bạn chưa cần biết ngay. Chúng ta sẽ đi tới nó từng bước. Ở đây chỉ cần hình dung rằng một tệp mô hình nằm trên ổ đĩa không thể tự chạy. Phải có một chương trình biết cách đọc tệp đó, chuẩn bị bộ nhớ, yêu cầu CPU hoặc GPU làm phép tính, rồi ghép kết quả lại để sinh ra token tiếp theo.

Chương trình làm công việc ấy chính là thứ ArcLLM cố gắng xây.

## Cuốn sách này không dành riêng cho người biết lập trình

Bạn sẽ gặp những từ như CPU, GPU, token, khối số hay Vulkan.

Nhưng tôi không muốn bạn phải biết trước chúng.

Khi một khái niệm mới xuất hiện, sách sẽ cố gắng làm ba việc theo đúng thứ tự:

```text
cho bạn một hình dung đơn giản
↓
đưa một ví dụ gần gũi
↓
sau đó mới gọi tên kỹ thuật
```

Có những cách giải thích ban đầu chưa phải toàn bộ sự thật. Điều đó là có chủ ý.

Khi một đứa trẻ học rằng Trái Đất “tròn”, ta không bắt đầu ngay bằng bán kính xích đạo, độ dẹt và mô hình địa cầu chính xác. Một cách hiểu đơn giản nhưng đúng hướng giúp ta đứng vững trước khi đi sâu hơn.

Cuốn sách này cũng vậy.

## Tại sao lại tự xây khi đã có phần mềm tốt?

Nếu mục đích chỉ là tải một mô hình về máy và trò chuyện với nó, có rất nhiều phần mềm tốt hơn việc tự xây từ đầu.

ArcLLM không được tạo ra vì thế giới thiếu một công cụ chạy mô hình.

Nó được tạo ra vì một lý do khác:

> **Muốn hiểu cỗ máy, hãy thử tự xây cỗ máy.**

Khi tự xây, ta buộc phải trả lời những câu hỏi mà người dùng thông thường không cần nghĩ tới:

- tệp mô hình thực sự chứa gì;
- những con số đó nằm ở đâu trong bộ nhớ;
- GPU nhận công việc bằng cách nào;
- một token phải đi qua bao nhiêu phép tính;
- vì sao một thay đổi có thể làm một phép tính nhanh hơn nhưng cả hệ thống lại không nhanh hơn;
- làm sao biết một kết quả tốt là thật, chứ không phải do ta vô tình thay điều kiện đo.

Đó là phần kỹ thuật của câu chuyện.

Nhưng có một phần khác còn quan trọng hơn.

## AI làm cho việc thử một ý tưởng trở nên rẻ hơn

Trước đây, biến một ý tưởng phần mềm thành một phiên bản chạy được thường tốn nhiều thời gian. Cần người viết mã lệnh máy tính, kiểm tra lỗi, chạy thử, đọc bản ghi hoạt động của hệ thống, rồi sửa tiếp.

AI có thể hỗ trợ rất nhiều việc trong chuỗi đó.

Nó có thể viết mã.

Đọc mã.

Tạo phép thử.

Tìm lỗi.

Tổng hợp kết quả.

Đề xuất hàng chục hướng tiếp theo.

Điều này làm cho việc tạo ra **phương án** trở nên rẻ hơn.

Nhưng khi phương án trở nên rẻ, một thứ khác lại trở nên quan trọng hơn:

> **Ta chọn phương án nào để tin?**

Một câu trả lời AI có thể nghe rất hợp lý mà vẫn sai.

Một chương trình có thể chạy nhanh hơn mà vẫn tính sai.

Một biểu đồ có thể rất đẹp nhưng được tạo từ một phép đo không phù hợp.

Một thay đổi có thể tốt trong bài thử nhỏ nhưng làm cả hệ thống chậm hơn.

Vì vậy hành trình ArcLLM không chỉ là hành trình viết phần mềm.

Nó còn là hành trình học cách hỏi:

```text
Ta đang kiểm tra điều gì?

Điều kiện nào phải giữ nguyên?

Bằng chứng nào đủ?

Kết quả nào được coi là đạt?

Nếu không đạt, ta có chấp nhận nó không?

Ta có đang đổi câu hỏi sau khi đã nhìn thấy kết quả không?
```

## Những thất bại trong sách không phải phần thừa

Trong nhiều câu chuyện kỹ thuật, người ta chỉ kể lại con đường cuối cùng đã thành công.

Cuốn sách này cố tình không làm vậy.

Bạn sẽ gặp những nhánh:

- chạy được nhưng quá chậm;
- nhanh hơn nhưng sai;
- tốt ở một phép tính nhỏ nhưng không giúp cả hệ thống;
- nghe rất hợp lý trên giấy nhưng không sống sót qua phép đo;
- có dữ liệu quan sát nhưng chưa đủ bằng chứng để nói về nguyên nhân.

Những kết quả **KHÔNG ĐẠT (FAIL)** đó được giữ lại.

Bởi vì nếu xóa chúng, ta chỉ còn đáp án cuối cùng.

Ta sẽ mất lý do vì sao đáp án đó phải có hình dạng như hiện tại.

## Nếu bạn bắt đầu từ con số 0

Ngay sau lời nói đầu là **Phần 0 — Bản đồ trước khi vào rừng**.

Phần đó bắt đầu từ ô chat quen thuộc và lần lượt trả lời:

```text
AI là gì?
↓
mô hình là gì?
↓
mô hình đã "học" cái gì?
↓
token là gì?
↓
CPU và GPU làm gì?
↓
tại sao cần một hệ thực thi?
```

Bạn không cần học thuộc.

Chỉ cần đọc tới đâu hiểu được một chút tới đó.

Các chương sau sẽ quay lại từng khái niệm và mở nó sâu hơn khi thật sự cần.

Nếu sau khi đọc xong, bạn không chỉ hiểu cỗ máy AI bên trong hơn mà còn biết cách **đòi hỏi bằng chứng trước khi tin một kết luận**, thì cuốn sách đã đạt mục tiêu quan trọng nhất của nó.

> **Khi việc tạo ra một phương án trở nên rẻ hơn, khả năng lựa chọn đúng phương án trở nên quý hơn.**

**Tiếp theo: [Phần 0 — Bản đồ trước khi vào rừng](00-ban-do-truoc-khi-vao-rung.md)**
