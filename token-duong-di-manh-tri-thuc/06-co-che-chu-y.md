# Chương 6 — Cơ chế chú ý: token nhìn những token khác như thế nào?

> **Mức đọc: Đi sâu**
>
> **Ta đang mở chiếc hộp nào?**
>
> ~~~text
> trạng thái hiện tại
>       ↓
> [ cơ chế chú ý ]
>       ↓
> trạng thái có thêm
> thông tin từ ngữ cảnh
> ~~~
>
> Chương này giải thích để bạn dễ hình dung của **cơ chế chú ý (attention)**. Công thức sẽ chỉ xuất hiện ở mức tối thiểu cần thiết để hiểu ba vai trò: truy vấn, khóa và giá trị.

Hãy đọc câu:

~~~text
Nam đặt chiếc cốc lên bàn rồi nhấc nó lên.
~~~

Khi tới từ “nó”, ta tự nhiên hỏi:

> “Nó” đang chỉ cái gì?

Trong câu này, “chiếc cốc” là ứng viên hợp lý hơn “bàn”.

Một mô hình ngôn ngữ cũng cần một cơ chế để trạng thái tại vị trí hiện tại sử dụng thông tin từ những vị trí trước.

Cơ chế chú ý là một phần quan trọng của việc đó.

## “Nhìn” chỉ là cách nói

Khi ta nói:

> token này “nhìn” token kia,

đừng hiểu là có một con mắt hay một thao tác đọc chữ theo nghĩa con người.

Thực tế là những dãy số được đưa vào các phép toán để tạo ra mức độ liên quan rồi trộn thông tin.

Từ “nhìn” chỉ giúp ta xây trực giác, dễ hình dung.

## Ba vai trò: tôi đang cần gì, ai có thể phù hợp, và họ mang gì?

Hãy tưởng tượng bạn đang ở thư viện.

Bạn cần tìm tài liệu về:

~~~text
lịch sử Hà Nội
~~~

Bạn có một câu hỏi.

Các cuốn sách có nhãn.

Và mỗi cuốn chứa nội dung.

Ta có thể dùng phép so sánh:

~~~text
câu hỏi đang tìm gì?
→ truy vấn

nhãn của từng nguồn
→ khóa

nội dung có thể lấy về
→ giá trị
~~~

Trong attention, ba vai trò đó được gọi là:

- **truy vấn (query)**;
- **khóa (key)**;
- **giá trị (value)**.

Thường viết tắt là:

~~~text
Q = query
K = key
V = value
~~~

## Quay lại câu “chiếc cốc”

Ta có:

~~~text
Nam | đặt | chiếc | cốc | lên | bàn | rồi | nhấc | nó | lên
~~~

Khi xử lý vị trí “nó”, ta có thể hình dung trạng thái hiện tại tạo ra một **truy vấn**.

Các vị trí trước có những **khóa**.

Mô hình so sánh truy vấn hiện tại với các khóa để tạo ra các điểm liên quan.

~~~text
                  vị trí hiện tại
                       [nó]
                         │
                      truy vấn
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
      [cốc]            [bàn]            [Nam]
       khóa              khóa             khóa
        │                │                │
     điểm cao?        điểm thấp?       điểm khác?
~~~

Sau đó các điểm được dùng để quyết định lượng thông tin nào từ **giá trị** của từng vị trí được đưa về trạng thái hiện tại.

## Khóa không phải giá trị

Đây là chi tiết dễ nhầm.

Một vị trí không chỉ có một dãy số duy nhất làm tất cả mọi việc trong attention.

Từ trạng thái của mỗi vị trí, mô hình tạo ra những cách nhìn khác nhau phục vụ các vai trò khác nhau.

Có thể hình dung:

~~~text
trạng thái của token
      │
      ├→ truy vấn Q
      ├→ khóa K
      └→ giá trị V
~~~

Các phép biến đổi này dùng trọng số đã học.

Truy vấn và khóa chủ yếu tham gia vào câu hỏi:

> “Mức độ liên quan giữa hai vị trí là bao nhiêu?”

Giá trị tham gia vào câu hỏi:

> “Nếu vị trí này được xem là liên quan, phần thông tin nào sẽ được đưa sang?”

## Điểm chú ý

Ta chưa cần công thức đầy đủ.

Chỉ cần biết mô hình tạo ra một **điểm chú ý (attention score)** giữa truy vấn hiện tại và các khóa có thể nhìn tới.

Ví dụ minh họa:

~~~text
"nó" so với "cốc"  →  4,2
"nó" so với "bàn"  →  1,1
"nó" so với "Nam"  →  0,5
~~~

Các số trên hoàn toàn là ví dụ.

Điều quan trọng là:

~~~text
điểm cao hơn
→ vị trí đó có thể đóng góp nhiều hơn
~~~

Nhưng điểm thô chưa phải tỷ lệ cuối cùng.

## Từ điểm số sang trọng số

Các điểm thường được đưa qua một hàm gọi là **softmax**.

Ta có thể hiểu softmax như một bước biến một nhóm điểm thành các trọng số dương có tổng bằng 1.

Ví dụ tưởng tượng:

~~~text
điểm thô

cốc   4,2
bàn   1,1
Nam   0,5

        ↓ softmax

trọng số chú ý

cốc   0,90
bàn   0,06
Nam   0,04
~~~

Sau đó các giá trị được trộn theo những trọng số này.

~~~text
0,90 × giá trị(cốc)
+
0,06 × giá trị(bàn)
+
0,04 × giá trị(Nam)
↓
thông tin được đưa về vị trí "nó"
~~~

Một lần nữa: số liệu chỉ để minh họa.

## Token hiện tại có được nhìn tương lai không?

Trong mô hình sinh văn bản kiểu chỉ dùng bộ giải mã, vị trí hiện tại thường không được phép dùng những token ở tương lai.

Nếu chuỗi hiện tại là:

~~~text
Hà Nội là thủ đô của
~~~

mô hình chưa được nhìn token “Việt” nếu “Việt” chính là thứ nó đang cần dự đoán.

Ta có thể hình dung một mặt nạ:

~~~text
vị trí hiện tại: [của]

được nhìn:
[Hà] [Nội] [là] [thủ] [đô] [của]

không được nhìn:
[Việt] [Nam] ...
~~~

Quy tắc này được gọi là **che nhân quả (causal masking)**.

Tên “nhân quả” ở đây nói về hướng thời gian trong chuỗi: token hiện tại chỉ dùng phần đã có trước đó để dự đoán phần tiếp theo.

Nó không có nghĩa mô hình đã chứng minh quan hệ nhân quả trong thế giới thật.

## Một đầu chú ý có đủ không?

Nếu chỉ có một cách đánh giá quan hệ, mô hình sẽ bị hạn chế.

Trong thực tế, attention thường có nhiều **đầu chú ý (attention heads)**.

Có thể hình dung mỗi đầu có bộ phép chiếu Q/K/V riêng và có thể học những kiểu quan hệ khác nhau.

~~~text
cùng trạng thái đầu vào
        │
        ├→ đầu 1
        ├→ đầu 2
        ├→ đầu 3
        └→ ...
              ↓
        kết hợp lại
~~~

Ta không nên gắn nhãn cứng:

> đầu 1 = ngữ pháp  
> đầu 2 = địa lý  
> đầu 3 = suy luận

Có những nghiên cứu tìm thấy các khuôn mẫu thú vị ở một số đầu, nhưng chức năng trong mô hình thật có thể phân tán và phụ thuộc ngữ cảnh.

> **Ghi chú về GQA và MQA**
>
> Cách hình dung “mỗi đầu chú ý có các phép chiếu Q/K/V riêng” phù hợp để hiểu **multi-head attention** theo dạng cơ bản, nhưng không phải mọi LLM hiện đại đều tổ chức Q, K và V theo đúng cách đó.
>
> Một số mô hình dùng **Multi-Query Attention (MQA)**, trong đó nhiều đầu truy vấn (query heads) có thể dùng chung phần khóa và giá trị (key/value). Một dạng trung gian phổ biến hơn là **Grouped-Query Attention (GQA)**, trong đó nhiều query heads được chia thành các nhóm và mỗi nhóm dùng chung một số key/value heads.
>
> Các thiết kế này giúp giảm lượng dữ liệu cần lưu và xử lý, đặc biệt đối với **KV cache** khi sinh văn bản.
>
> Cuốn sách này không đi sâu vào MQA/GQA. Khi giải thích attention, ta vẫn dùng mô hình Q/K/V cơ bản để xây trực giác; điều cần nhớ là **số query heads và số key/value heads trong một mô hình thật không nhất thiết bằng nhau**.

## Attention có phải lời giải thích cho “mô hình đang nghĩ gì” không?

Không nên nói như vậy.

Trọng số chú ý cho ta biết một phần cách thông tin được trộn trong một phép tính cụ thể.

Nhưng toàn mô hình còn có:

- nhiều đầu chú ý;
- nhiều lớp;
- FFN;
- đường cộng tắt;
- các phép chuẩn hóa;
- nhiều biến đổi khác.

Do đó:

~~~text
trọng số chú ý
≠
toàn bộ lời giải thích cho dự đoán
~~~

Một bản đồ attention đẹp có thể hữu ích.

Nhưng nó không tự động trở thành bằng chứng đầy đủ về ý nghĩa, nguyên nhân hay “suy nghĩ” của mô hình.

Đây là một ví dụ sớm cho nguyên tắc lớn của cuốn sách:

> **Quan sát được một tín hiệu chưa có nghĩa ta đã giải thích được cả cơ chế.**

## Attention làm xong thì sao?

Sau attention, trạng thái của mỗi vị trí đã nhận thêm thông tin từ những vị trí khác.

Nhưng lớp Transformer chưa kết thúc.

Trạng thái đó còn đi qua một nhánh biến đổi khác.

Nếu attention giống bước:

> “Tôi nên lấy thông tin nào từ những vị trí khác?”

thì nhánh tiếp theo gần với câu hỏi:

> “Sau khi có thông tin đó, tôi biến đổi trạng thái của vị trí này như thế nào?”

Đó là Chương 7.

### Nhớ 3 điều

1. **Cơ chế chú ý (attention) cho phép một vị trí trộn thông tin từ những vị trí khác thông qua truy vấn (query), khóa (key) và giá trị (value).**
2. **Điểm chú ý được biến thành trọng số để quyết định lượng thông tin được lấy từ từng vị trí; trong mô hình sinh văn bản, vị trí hiện tại không được nhìn token tương lai.**
3. **Trọng số chú ý không phải lời giải thích hoàn chỉnh cho “mô hình đang nghĩ gì”.** Nó chỉ là một phần của một chuỗi biến đổi lớn hơn.

**Tiếp theo: [Chương 7 — Nhánh biến đổi tín hiệu: điều gì xảy ra sau cơ chế chú ý?](07-nhanh-bien-doi-tin-hieu.md)**
