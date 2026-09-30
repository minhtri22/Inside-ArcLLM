# Chương 17 — Khi một cách sắp dữ liệu trở thành một phần của kiến trúc

> **Mức đọc: Nâng cao**
>
> **Bạn đang mở câu hỏi nào?**
>
> ```text
> Dữ liệu logic
>      ↓
> nhiều cách sắp khác nhau
>      ↓
> [ lợi ích + chi phí riêng ]
>      ↓
> hệ thực thi phải quản lý
> ```


> **Câu hỏi của chương:** Khi cùng một khối số có thể được biểu diễn theo nhiều cách để phục vụ những đường thực thi khác nhau, hệ thực thi cần hiểu điều gì ngoài việc “khối số này đang nằm trong bộ nhớ”?

Chương 16 kết thúc ở một nơi mà ban đầu ArcLLM không định đi tới.

Ta xuất phát từ một câu hỏi rất cụ thể:

> **FFN-down Q4_K đang chậm vì cách chia công việc hay vì cách dữ liệu được bố trí?**

Thí nghiệm 2×2 trả lời rằng cả hai đều có giá trị độc lập.

Split-K32 thay đổi **cách thực thi (execution)** — tức cách GPU chia công việc — và giảm độ trễ rõ rệt.

EXEC148 thay đổi **cách biểu diễn dữ liệu (representation)** — tức cách cùng dữ liệu Q4_K được sắp xếp để đường thực thi đọc nó — và cũng giảm độ trễ rõ rệt.

Nhưng khi ghép hai thay đổi lại, lợi ích không cộng đẹp với nhau.

Kết quả đó tạo ra một vấn đề kiến trúc mới.

Trước đây, hệ thực thi có thể nghĩ đơn giản:

```text
khối số
↓
các byte của khối số
↓
chương trình GPU đọc trực tiếp các byte đó
```

Sau EXEC148, hình ảnh ấy không còn đủ nữa.

Cùng một khối số logic có thể có:

```text
dạng lưu trữ gốc
↓
và
dạng biểu diễn phục vụ thực thi
```

Cả hai cùng biểu diễn **một trọng số**.

Nhưng chúng có kích thước khác nhau, chi phí tạo khác nhau, tốc độ thực thi khác nhau và có thể chỉ phù hợp trong những điều kiện sử dụng khác nhau.

Đó là lúc một chi tiết từng nằm bên trong chương trình GPU bắt đầu đòi quyền trở thành một khái niệm riêng của hệ thực thi.

## EXEC148 thực chất là gì?

Tên `EXEC148` nghe giống một mã nội bộ khó hiểu.

Ý nghĩa của nó lại khá đơn giản.

Một block Q4_K gốc dùng:

```text
144 byte
```

EXEC148 dùng:

```text
148 byte
```

Nó không thay đổi trọng số logic.

Nó cũng không bung toàn bộ trọng số Q4_K thành FP16 hay F32.

Thay vào đó, những thông tin mà chương trình GPU cần liên tục được sắp xếp lại thành một **bố cục dữ liệu (layout)** thuận tiện hơn cho việc tính toán.

Block 148 byte có thể hình dung thành ba vùng:

```text
2 byte đầu
→ d, giữ nguyên raw FP16 bits từ Q4_K nguồn

2 byte tiếp
→ dmin, cũng giữ nguyên raw FP16 bits

16 byte tiếp
→ 8 cặp hệ số scale/min được đặt trực tiếp dưới dạng uint8

128 byte còn lại
→ các giá trị Q4 được sắp lại theo thứ tự K
```

Ở dạng lưu trữ gốc, một số thông tin được đóng gói chặt để tiết kiệm dung lượng.

Ở EXEC148, một phần thông tin được làm trực tiếp hơn để chương trình GPU bớt phải giải mã cấu trúc phức tạp trong lúc đang tính.

Đây là một sự đánh đổi rất quen thuộc:

> **dùng thêm một ít không gian để giảm bớt công việc lặp lại khi thực thi.**

Nhưng điểm quan trọng hơn là:

> **EXEC148 không phải một mô hình mới. Nó là một cách biểu diễn khác của cùng dữ liệu logic.**

## Làm sao biết dữ liệu thực sự vẫn là một?

Nếu cách biểu diễn thay đổi, ta không thể chỉ nhìn đầu ra cuối rồi nói:

> “Có vẻ giống nhau.”

ArcLLM dùng một kiểm tra chặt hơn.

Dạng dữ liệu nguồn và EXEC148 đều được giải mã về một dạng logic chung gồm:

```text
d
dmin
8 cặp hệ số scale/min
256 giá trị q
```

Sau đó từng block được so byte-for-byte ở dạng logic này.

Không phải vài block mẫu.

Mà toàn bộ block của cả:

```text
14 khối số Q4_K
```

trong họ FFN-down đang nghiên cứu.

Kết quả:

> **toàn bộ 14 khối số khớp chính xác về nội dung logic.**

Đây là một phân biệt rất quan trọng:

```text
cách biểu diễn khác nhau
≠
khối số logic khác nhau
```

Ta đã thay **cách mang dữ liệu**.

Không thay **dữ liệu mà phép toán có nghĩa là sử dụng**.

## Vì sao không cứ giấu EXEC148 bên trong chương trình GPU?

Nếu EXEC148 chỉ là một mẹo giúp chương trình GPU nhanh hơn, ta có thể nhét nó vào shader rồi kết thúc câu chuyện.

Nhưng bằng chứng ở Chương 16 khiến cách làm đó không còn đủ.

Arm B cho thấy EXEC148 tự nó có giá trị.

Ở W-S:

```text
Serial-K + Q4_K gốc
≈ 100,58 ms

Serial-K + EXEC148
≈ 24,19 ms
```

Ở W-C:

```text
Serial-K + Q4_K gốc
≈ 213,33 ms

Serial-K + EXEC148
≈ 31,41 ms
```

Điều đó có nghĩa cách biểu diễn dữ liệu không chỉ là một chi tiết trang trí quanh cách thực thi.

Nó có **giá trị hiệu năng độc lập**.

Nhưng nó cũng có **chi phí độc lập**.

Thời gian tạo biểu diễn một lần:

```text
≈ 231,64 ms
```

Vùng dữ liệu thực thi bổ sung trong thí nghiệm chiếm:

```text
549.527.552 byte
≈ 550 MB
```

Khi một thứ vừa có lợi ích riêng vừa có chi phí riêng, hệ thực thi cần có khả năng nói về nó như một đối tượng riêng.

Nếu không, ta không thể trả lời những câu hỏi như:

> EXEC148 đã tồn tại chưa?

> Ai phải tạo nó?

> Nó nằm ở đâu?

> Có nên giữ lại sau lượt sử dụng hiện tại không?

> Khi thiếu bộ nhớ, có nên bỏ nó?

> Một đường thực thi yêu cầu cách biểu diễn nào?

Những câu hỏi đó không còn thuộc riêng shader.

Chúng thuộc kiến trúc hệ thực thi.

## Bằng chứng tạo ra hai khối chức năng nền tảng khác nhau

Sau thí nghiệm 2×2, ArcLLM không chọn:

```text
A + B
```

làm kiến trúc mặc định.

Sự tương tác giữa hai cơ chế là đối kháng và B nhanh hơn AB trong cả hai **bài đo**.

Thay vào đó, bằng chứng tạo ra hai **khối chức năng nền tảng** độc lập.

Thứ nhất:

> **A — khối song song hóa tính toán**, dùng Split-K32 để thay cách GPU chia phép tính.

Trong phạm vi đã kiểm tra, A trở thành lựa chọn mặc định khi không muốn tạo thêm một dạng biểu diễn dữ liệu phụ.

Thứ hai:

> **B — khối biểu diễn dữ liệu phục vụ thực thi**, tức EXEC148 như một cách biểu diễn có giá trị hiệu năng riêng.

Hai khối này trả lời hai câu khác nhau:

```text
A:
GPU nên chia và thực hiện công việc như thế nào?

B:
Dữ liệu nên ở hình thức nào khi đường thực thi sử dụng nó?
```

Đây là một ranh giới mà hệ thực thi trước đó chưa cần biểu diễn rõ.

Bằng chứng đã buộc nó xuất hiện.

## Nhưng B nhanh hơn — sao không luôn chọn B?

Chương 16 đã cho ta điểm hòa vốn theo số token.

Ở W-S, với cách tạo EXEC148 đang được đo:

```text
khoảng 16,3 token
```

là điểm mà lợi ích độ trễ của B bắt đầu bù được thời gian tạo biểu diễn.

Ở W-C:

```text
khoảng 36,1 token
```

Nhưng điểm hòa vốn đó chỉ xét một phần của chi phí.

Một hệ thực thi thật còn phải tính tới khoảng 550 MB bộ nhớ bổ sung.

Và nhiều thứ khác.

Hãy viết bài toán tổng quát hơn.

Với A:

```text
Total_A(H)
=
H × L_A
```

Trong đó `H` là số token tương lai sẽ tiếp tục dùng cùng dữ liệu đã chuẩn bị.

Với B:

```text
Total_B(H)
=
C_create
+
C_transfer
+
C_maintenance
+
C_memory
+
H × L_B
```

Nói bằng tiếng Việt:

```text
tổng chi phí B
=
chi phí tạo biểu diễn
+
chi phí chuyển giao dữ liệu
+
chi phí duy trì
+
chi phí do chiếm bộ nhớ
+
chi phí tính toán qua các token
```

B chỉ thực sự đáng chọn khi:

```text
Total_B < Total_A
```

và mọi điều kiện về tính đúng, dung lượng bộ nhớ và khả năng truy cập dữ liệu đều thỏa mãn.

## Thử bằng số

Tạm bỏ qua chi phí chuyển giao, duy trì và áp lực bộ nhớ để nhìn phần đơn giản nhất.

Ở W-S:

```text
L_A ≈ 38,37 ms/token

L_B ≈ 24,19 ms/token

C_create ≈ 231,64 ms
```

Nếu chỉ có:

```text
H = 10 token
```

thì A mất khoảng:

```text
10 × 38,37
=
383,7 ms
```

B mất:

```text
231,64
+
10 × 24,19

=
473,54 ms
```

Lúc này A tốt hơn.

Nhưng với:

```text
H = 30 token
```

A:

```text
30 × 38,37
=
1.151,1 ms
```

B:

```text
231,64
+
30 × 24,19

=
957,34 ms
```

B bắt đầu có tổng thời gian thấp hơn.

Cùng một cách biểu diễn.

Cùng một máy.

Nhưng quyết định khác nhau chỉ vì:

> **ta dự kiến tái sử dụng nó bao lâu.**

Đây là lúc cách biểu diễn dữ liệu bắt đầu kéo theo khái niệm:

> **vòng đời (lifetime)**, tức dữ liệu đó cần tồn tại bao lâu trước khi có thể bỏ hoặc phải tạo lại.

Ta sẽ đi sâu vào điều này ở Chương 18.

## “Ai tạo?” và “nằm ở đâu?” không phải cùng một câu hỏi

Khi nghĩ về EXEC148, một cách nói rất dễ xuất hiện là:

> “Đây là dữ liệu của CPU.”

Hoặc:

> “Đây là dữ liệu của GPU.”

Nhưng cách gọi này trộn hai chuyện khác nhau.

Một cách biểu diễn có:

> **miền tính toán tạo ra nó (creator domain)**

và:

> **vùng bộ nhớ nơi nó cư trú khi được sử dụng (residency domain)**.

Hai thứ không nhất thiết giống nhau.

Trong cách triển khai đang được đo, CPU tạo EXEC148.

Nhưng CPU ghi trực tiếp vào một **vùng nhớ Vulkan** mà cả CPU lẫn GPU đều có thể truy cập trên máy này.

Máy đang dùng:

> **kiến trúc bộ nhớ hợp nhất (Unified Memory Architecture, UMA)**, tức CPU và GPU dùng chung hệ thống bộ nhớ vật lý thay vì luôn có hai kho bộ nhớ hoàn toàn tách biệt.

Vì vậy:

```text
CPU tạo dữ liệu
```

không tự động có nghĩa:

```text
dữ liệu nằm trong một vùng chỉ CPU dùng
rồi bắt buộc phải sao chép sang một vùng khác cho GPU
```

Tương tự, nếu sau này GPU tạo EXEC148, điều đó cũng không nhất thiết có nghĩa cách biểu diễn dữ liệu sẽ nằm ở một vùng bộ nhớ vật lý khác.

Đây là một thay đổi tư duy quan trọng.

Hệ thực thi không nên hỏi một câu mơ hồ:

> “EXEC148 thuộc CPU hay GPU?”

Nó phải tách ra:

```text
ai tạo?
↓
tạo ở vùng bộ nhớ nào?
↓
đường thực thi đọc từ đâu?
↓
có cần sao chép hay bàn giao dữ liệu không?
```

## Một cách biểu diễn bắt đầu có “hồ sơ” riêng

Từ bằng chứng đó, bài toán bố trí được mô tả như một **bộ thuộc tính (tuple)**:

```text
P
=
nơi tạo
×
nơi cư trú
×
đường chuyển giao
×
vòng đời
×
chính sách duy trì
```

Ta tách từng phần.

**Nơi tạo:**

> tài nguyên nào tạo ra dạng biểu diễn này.

Có thể là CPU.

Có thể là GPU.

Về lý thuyết có thể là một bộ xử lý khác nếu nó thực sự có khả năng làm phép biến đổi.

**Nơi cư trú:**

> sau khi tạo, dữ liệu nằm ở vùng bộ nhớ nào.

**Đường chuyển giao:**

> dữ liệu phải đi qua con đường nào từ nguồn tới nơi phép tính có thể sử dụng.

**Vòng đời:**

> dữ liệu cần sống trong bao lâu.

**Chính sách duy trì:**

> khi nào giữ, khi nào bỏ, khi nào phải tạo lại.

Đây chính là dấu hiệu của một **lớp trừu tượng (abstraction)** mới.

Trước đây:

```text
khối số
→ con trỏ tới các byte
```

là đủ cho nhiều trường hợp.

Bây giờ hệ thực thi bắt đầu cần:

```text
khối số logic
↓
có thể có nhiều cách biểu diễn
↓
mỗi cách có danh tính và chi phí riêng
↓
đường thực thi yêu cầu một cách biểu diễn phù hợp
```

Chưa cần thiết kế lớp mã hay giao diện cụ thể để thấy nhu cầu đó.

Bằng chứng đã tạo ra nhu cầu trước.

mã sẽ phải đi theo sau.

## CPU, GPU hay NPU không phải câu hỏi đầu tiên

Khi thấy bước tạo biểu diễn mất khoảng 232 ms, phản xạ tự nhiên có thể là:

> “Đẩy nó sang GPU.”

Hoặc:

> “Máy có NPU, dùng NPU đi.”

Nhưng đây chính là kiểu suy nghĩ ArcLLM đã học cách tránh.

Một tài nguyên đang nhàn rỗi không tự động là một tài nguyên hữu ích.

Trước khi đo hiệu năng một phương án, phải hỏi:

```text
nó có đọc được Q4_K nguồn không?

nó có thực hiện được chính xác phép biến đổi cần thiết không?

kết quả của nó có đến được nơi Vulkan sử dụng không?

có cần sao chép thêm không?

chi phí bàn giao là bao nhiêu?

nó có tranh tài nguyên với giai đoạn xử lý đầu vào hoặc sinh token không?
```

Nếu ngay cả **giới hạn lạc quan nhất (optimistic bound)** cũng không thể làm phương án đó có lợi, thì không cần triển khai.

Đây lại là triết lý:

> **loại trước khi xây (kill before build)** — dừng trước khi triển khai nếu giới hạn vật lý đã đủ để trả lời.

Một lớp trừu tượng tốt không tồn tại chỉ để làm hệ thống “trừu tượng hơn”.

Nó phải giúp hệ thực thi đặt đúng những câu hỏi này.

## Có thể tạo cách biểu diễn dữ liệu trước cả khi suy luận bắt đầu

Một hướng khác cũng xuất hiện.

Tại sao phải tạo EXEC148 khi phiên suy luận bắt đầu?

Nếu EXEC148 chỉ phụ thuộc vào:

```text
mô hình
+
kiểu lượng tử hóa
+
phiên bản của cách biểu diễn
```

thì về nguyên tắc có thể tạo nó từ trước.

Ví dụ:

```text
Mô hình Q4_K
+
tệp EXEC148 đi kèm
↓
nạp dữ liệu
↓
xác minh đúng mô hình và đúng phiên bản
↓
thực thi
```

Một **tệp dữ liệu phụ** đi kèm tệp chính thường được gọi là:

> **tệp phụ (sidecar)**.

Như vậy chi phí biến đổi không còn nằm trên:

> **đường thời gian quan trọng của suy luận (critical path)**.

Nhưng ta lại phải trả bằng:

- dung lượng lưu trữ;
- thời gian nạp;
- việc quản lý phiên bản tương thích;
- khả năng lưu lại để tái sử dụng;
- và việc bảo đảm **tệp phụ** đúng với mô hình đang chạy.

Điểm đáng chú ý không phải phương án này chắc chắn tốt.

Chưa có bằng chứng đó.

Điểm đáng chú ý là:

> **Ngay khi cách biểu diễn dữ liệu trở thành một khái niệm độc lập, không gian kiến trúc lập tức mở rộng vượt ra ngoài chương trình GPU.**

Ta có thể thay nơi tạo.

Thời điểm tạo.

Nơi lưu.

Thời gian sống.

Không cần đổi phép toán logic.

## Lớp trừu tượng không được sinh ra chỉ vì mã “đẹp”

Đây là một trong những nguyên tắc quan trọng nhất của cuốn sách.

Nếu ta bắt đầu ArcLLM từ đầu bằng cách nói:

> “Hãy thiết kế một hệ thống tổng quát với lớp quản lý cách biểu diễn khối số, chính sách bố trí và bộ quản lý vòng đời…”

ta có thể tạo ra một sơ đồ rất đẹp.

Nhưng ta chưa biết những lớp đó giải quyết vấn đề thật nào.

Trong hành trình thực tế, thứ tự ngược lại:

```text
Q4-down chậm
↓
tách cách thực thi khỏi cách biểu diễn
↓
thí nghiệm 2×2
↓
A có giá trị
B có giá trị
AB tương tác đối kháng
↓
B có độ trễ tốt hơn
nhưng có chi phí tạo + bộ nhớ
↓
quyết định phụ thuộc vào mức tái sử dụng và trạng thái tài nguyên
↓
Hệ thực thi buộc phải hiểu cách biểu diễn như một đối tượng riêng
```

Lớp trừu tượng không xuất hiện từ sở thích thiết kế.

Nó xuất hiện vì mô hình cũ không còn đủ để diễn tả bằng chứng.

Đó là một khác biệt rất lớn.

## Một lớp trừu tượng tốt phải giữ được những thất bại trước đó

Cũng cần tránh một cái bẫy.

Khi lớp trừu tượng mới xuất hiện, ta rất dễ viết lại lịch sử như thể:

> “Ngay từ đầu hệ thống nên được thiết kế theo cách này.”

Không.

Nếu không có arm B, ta chưa biết cách biểu diễn dữ liệu đáng được tách riêng.

Nếu không có AB, ta chưa biết việc kết hợp hai cơ chế tạo tương tác đối kháng.

Nếu không tính chi phí tạo biểu diễn và bộ nhớ, ta có thể tưởng B là lựa chọn tốt nhất trong mọi trường hợp.

Vì vậy lớp trừu tượng mới phải giữ lại nguồn gốc của chính nó:

```text
A
→ khối thực thi song song

B
→ khối biểu diễn dữ liệu phục vụ thực thi

AB
→ không phải lựa chọn mặc định

0
→ mốc đối chứng lịch sử
```

> **Thất bại và những đánh đổi không biến mất sau khi có kiến trúc đẹp hơn.**

Chúng chính là lý do kiến trúc đó phải tồn tại.

## Từ “khối số đang ở trong bộ nhớ” tới một câu hỏi khó hơn

Ở Chương 6, một bước tiến rất lớn là:

> **giữ toàn bộ trọng số mô hình trong vùng bộ nhớ mà GPU có thể truy cập.**

Khi đó câu hỏi chính là:

> khối số có ở đó không?

EXEC148 làm câu hỏi này không còn đủ.

Q4_K gốc có thể đang ở trong bộ nhớ.

Nhưng đường thực thi B lại cần:

```text
EXEC148
```

Nếu EXEC148 chưa tồn tại thì sao?

Nếu đã tạo nhưng vừa bị loại khỏi bộ nhớ thì sao?

Nếu cách biểu diễn dữ liệu đúng mô hình nhưng sai phiên bản thì sao?

Nếu một đường thực thi khác chỉ cần Q4_K gốc thì sao?

Ta bắt đầu thấy ba trạng thái khác nhau:

```text
dữ liệu logic tồn tại

≠

cách biểu diễn cần thiết tồn tại

≠

cách biểu diễn đó sẵn sàng cho phép tính ngay lúc này
```

Đây chính là bước dẫn sang chương tiếp theo.

### Nhớ 3 điều

1. **EXEC148 không thay đổi khối số logic; nó thay đổi cách cùng dữ liệu Q4_K được chuẩn bị cho phép tính.** Block tăng từ `144` lên `148 byte`, và toàn bộ 14 khối số đã được kiểm tra tương đương chính xác ở dạng logic.
2. **Khi một cách biểu diễn có lợi ích và chi phí riêng, hệ thực thi phải hiểu nó như một đối tượng kiến trúc.** EXEC148 có độ trễ tốt hơn trong phạm vi đã đo, nhưng phải trả chi phí tạo một lần và khoảng `550 MB` vùng dữ liệu bổ sung trong thí nghiệm.
3. **Lớp trừu tượng xuất hiện sau bằng chứng, không phải trước bằng chứng.** Thí nghiệm 2×2 buộc ArcLLM tách cách thực thi khỏi cách biểu diễn; từ đó mới nảy sinh các câu hỏi về ai tạo, nằm ở đâu, sống bao lâu và khi nào nên giữ hoặc bỏ.

**Tiếp theo: [Chương 18 — Dữ liệu có mặt chưa đủ: nó phải sẵn sàng đúng lúc](18-du-lieu-o-trong-bo-nho-van-chua-du.md)**

Ta từng nghĩ một câu hỏi lớn của hệ thực thi là:

> “khối số đã ở trong bộ nhớ chưa?”

Bây giờ câu hỏi phải dài hơn:

> **“Cách biểu diễn mà đường thực thi cần đã tồn tại chưa, lấy từ đâu, được tạo lúc nào, sẵn sàng khi nào, sống bao lâu và ai chịu trách nhiệm khi nó không còn hợp lệ?”**

Đó là lúc chỉ biết dữ liệu “đang ở trong bộ nhớ” không còn đủ để mô tả trạng thái của nó.
