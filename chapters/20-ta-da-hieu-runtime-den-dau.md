# Chương 20 — Ta đã hiểu runtime đến đâu?

> **Câu hỏi của chương:** Sau tất cả những PASS, FAIL, phép đo, cơ chế và lớp trừu tượng đã đi qua, ArcLLM thực sự đã trở thành một runtime tới mức nào — và điều gì ta vẫn chưa được phép tuyên bố?

Chúng ta bắt đầu cuốn sách bằng một câu hỏi rất đơn giản:

> **Bên dưới một câu trả lời AI thực sự có gì?**

Lúc đó chưa có runtime.

Chỉ có một file model.

Rồi từng lớp xuất hiện.

```text
GGUF
↓
tensor
↓
Vulkan
↓
các phép tính nền tảng
↓
một decoder layer
↓
28 layer
↓
KV cache
↓
sinh nhiều token
↓
đo với đối chứng
↓
tìm cơ chế gây chi phí
↓
PASS và FAIL
↓
cách biểu diễn dữ liệu phục vụ thực thi
↓
vòng đời dữ liệu
↓
một mô hình runtime tổng quát hơn
```

Nhưng đến đây vẫn còn một câu hỏi rất quan trọng.

Tất cả những thứ vừa xây có thực sự trở thành **một runtime**, hay chúng vẫn chỉ là một tập hợp chương trình thí nghiệm được nối với nhau?

ArcLLM phải trả lời câu hỏi đó trước khi cuốn sách có thể kết thúc.

## Một chương trình thí nghiệm chạy được chưa chắc đã là runtime

Trong quá trình nghiên cứu, rất nhiều chương trình được tạo ra cho một mục đích cực kỳ cụ thể.

Một chương trình chỉ để:

```text
chạy 4 nhánh của thí nghiệm 2×2
```

Một chương trình khác chỉ để:

```text
so ứng viên với đối chứng
```

Một chương trình khác nữa chỉ để:

```text
kiểm tra một tải công việc đã khóa
```

Những chương trình này hoàn toàn hợp lệ.

Chúng là công cụ để trả lời câu hỏi khoa học.

Nhưng chúng mang theo rất nhiều thứ chỉ phục vụ phòng thí nghiệm:

```text
dữ liệu kiểm thử cố định

hash đầu ra mong đợi

logic PASS / FAIL

thứ tự chạy các nhánh

ngưỡng hiệu năng

các kiểm tra dành riêng cho thí nghiệm
```

Một runtime thực không nên cần biết:

> “Đây là W-S.”

Hay:

> “Kết quả đúng phải có hash này.”

Hay:

> “Hôm nay ta đang chạy nhánh B của thí nghiệm.”

Nó phải nhận một yêu cầu rồi thực hiện công việc của runtime.

Vì vậy bước hội tụ cuối cùng không phải một thí nghiệm khoa học mới.

Nó là:

> **Tách những cơ chế đã được xác nhận ra khỏi vỏ thí nghiệm đã tạo ra chúng.**

## Tách cỗ máy ra khỏi phòng thí nghiệm

Đến thời điểm này, một số mảnh quan trọng đã đứng vững.

Có cơ chế Split-K32 cho gate/up đã sống sót tới model thật.

Có cách biểu diễn EXEC148 cho Q4-down.

Có mô hình sáu chiều để quản lý:

```text
danh tính

khả dụng thực thi

sẵn sàng thực thi

trạng thái cư trú

quá trình thu nhận

vòng đời
```

Có lớp Vulkan thực hiện những quyết định đó mà không cần lén thêm luật riêng cho Q4.

Nhưng các kết quả nghiên cứu này vẫn phải hội tụ thành một đường chạy chung.

Mục tiêu lúc này là tạo một:

> **runtime chuẩn đã hội tụ (canonical runtime)**.

Từ “chuẩn” ở đây không có nghĩa:

> “Đây là kiến trúc cuối cùng và tốt nhất có thể.”

Nó chỉ có nghĩa:

> **Đây là đường runtime hiện tại được chọn làm mốc chính thức sau khi những cơ chế đã được kiểm tra và hội tụ.**

Người gọi chỉ cần cung cấp:

```text
các token đầu vào

+

số token muốn sinh
```

Runtime chịu trách nhiệm cho phần còn lại.

Người gọi không cần biết:

```text
W-S là gì

W-C là gì

nhánh A hay B là gì

thí nghiệm nào đã sinh ra kernel đang chạy
```

Đây là một ranh giới rất quan trọng.

> **Nghiên cứu tạo ra cơ chế. Runtime sử dụng cơ chế, nhưng không mang theo phòng thí nghiệm bên trong nó.**

## Nhưng dọn kiến trúc cũng có thể làm hỏng bằng chứng cũ

Tách code ra khỏi vỏ thí nghiệm nghe giống một việc thuần kỹ thuật.

Nhưng nếu làm sai, ta có thể vô tình thay đổi hành vi đã được xác nhận.

Vì vậy runtime mới phải chạy lại các **đối chứng đã đóng băng**.

Ở hai hồ sơ lịch sử, nó vẫn phải giữ đúng:

```text
441 dispatch ở prefill

469 dispatch cho mỗi bước decode

31 bước decode
```

Với đường Q4-down, mỗi lượt vẫn có:

```text
31 bước dùng đường B

1 lần tạo EXEC148

1 lần loại EXEC148
```

Chuỗi token đầu ra cũng phải giữ nguyên.

Cả hai hồ sơ đều PASS.

Nói cách khác:

> **Ta đã thay hình thức kiến trúc mà không làm thay đổi hành vi đã được xác nhận.**

Nhưng nếu chỉ chạy lại hai phép thử lịch sử thì vẫn còn một nghi ngờ.

Có thể runtime mới vẫn chỉ là một chương trình được viết riêng để vượt hai bài kiểm tra đó.

Vì vậy cần thêm một câu hỏi.

## Nếu đưa một đầu vào khác thì sao?

Một đầu vào không thuộc hai trường hợp lịch sử được đưa vào:

```text
[1, 42, 314, 2718]
```

và yêu cầu sinh:

```text
2 token
```

Runtime trả:

```text
[2718, 2718]
```

Kết quả là các số hữu hạn.

Đường chạy hoàn tất.

Không có logic W-S/W-C trong runtime.

Không có hash kết quả cố định được nhúng vào đường thực thi.

Không có logic phân xử hiệu năng của thí nghiệm nằm bên trong runtime.

Những chương trình nghiên cứu lịch sử cũng không còn nằm trên đường biên dịch đang hoạt động.

Điều này **không** chứng minh rằng:

> `[2718, 2718]` là một câu trả lời ngôn ngữ tốt.

Thậm chí yêu cầu này nằm ngoài phạm vi khoa học đã được xác nhận trước đó.

Phép thử chỉ cho phép một kết luận hẹp hơn:

> **Runtime thật sự có thể nhận dữ liệu do người gọi cung cấp, thay vì chỉ phát lại những trường hợp thí nghiệm được viết sẵn.**

Đây là một PASS về kiến trúc.

Không phải PASS về chất lượng model.

## Vậy ArcLLM đã thật sự là một runtime chưa?

Trong nghĩa thực dụng mà cuốn sách này đặt ra:

> **Có.**

Nó có thể thực hiện một chuỗi hoàn chỉnh:

```text
nạp model
↓
prefill
↓
tạo và sử dụng KV cache
↓
decode nhiều token
↓
dùng các kernel đã được xác nhận
↓
chọn đường thực thi
↓
tạo cách biểu diễn dữ liệu khi cần
↓
xác minh cách biểu diễn đó
↓
quản lý vòng đời
↓
trả token cho người gọi
```

Quan trọng hơn, đường runtime này đã được tách khỏi vỏ thí nghiệm từng tạo ra các cơ chế bên trong nó.

Đây chính là thứ Chương 1 chưa có.

Nhưng từ:

> **“ArcLLM đã trở thành một runtime.”**

không được nhảy thành:

> **“Bài toán runtime LLM đã được giải quyết.”**

Khoảng cách giữa hai câu rất lớn.

## Ta đã chứng minh được những gì?

Có thể nhìn toàn bộ hành trình như một chuỗi câu hỏi ngày càng khó hơn.

Ban đầu:

```text
có đọc đúng model không?
```

Rồi:

```text
có tính đúng không?
```

Rồi:

```text
có chạy được một decoder layer không?
```

Rồi:

```text
có chạy được cả model không?
```

Rồi:

```text
có nhớ những token trước bằng KV cache không?
```

Rồi:

```text
có sinh được nhiều token liên tiếp không?
```

Rồi:

```text
đứng ở đâu trước một runtime trưởng thành?
```

Rồi:

```text
phần nào thật sự gây chi phí?
```

Rồi:

```text
một cơ chế nhanh ở phép thử nhỏ
có còn nhanh trong model thật không?
```

Rồi:

```text
cách biểu diễn dữ liệu
có thể là một biến độc lập với cách thực thi không?
```

Rồi:

```text
cách biểu diễn được tạo,
giữ và loại như thế nào?
```

Và cuối cùng:

```text
những cơ chế khác nhau đó
có thể cùng sống trong một runtime
mà không cần lõi biết tên từng cơ chế không?
```

Mỗi câu hỏi cần một loại bằng chứng khác nhau.

Đó có lẽ là kết quả quan trọng hơn bất kỳ một con số tăng tốc riêng lẻ nào.

## Một PASS về tính đúng không phải PASS về hiệu năng

Cuốn sách liên tục buộc ta giữ các tầng kết luận tách biệt.

Q6 là ví dụ rõ nhất.

Một chương trình có thể:

```text
biên dịch
→ PASS
```

nhưng:

```text
tính đúng
→ FAIL
```

Khi tính đúng FAIL:

```text
hiệu năng
→ không được đo
```

Ở một nơi khác, một cơ chế có thể:

```text
hiệu năng thành phần
→ PASS
```

nhưng:

```text
toàn hệ thống
→ chưa biết
```

Rồi Q4 gate/up sống sót qua bước chuyển vào model thật:

```text
decode
≈ 2,20×

E2E
≈ 2,00×
```

Nhưng phép đối chứng mới sau đó lại cho thấy:

```text
nhanh hơn ArcLLM trước đó
≠
đã bắt kịp runtime trưởng thành
```

Trong các trường hợp đã đo, khoảng cách với llama.cpp vẫn vào khoảng:

```text
decode latency
≈ 10,38×

E2E latency
≈ 9,97×
```

Hai nhóm kết quả không mâu thuẫn.

Chúng trả lời hai câu hỏi khác nhau.

Một cơ chế có thể tạo **giá trị thật bên trong ArcLLM** trong khi ArcLLM **vẫn còn khoảng cách lớn với đối chứng bên ngoài**.

## Một lớp trừu tượng PASS cũng không phải PASS về hiệu năng

Chương 17 đến 19 chuyển trọng tâm khỏi việc chỉ viết kernel nhanh hơn.

Ta có sáu chiều:

```text
danh tính

khả dụng thực thi

sẵn sàng thực thi

trạng thái cư trú

quá trình thu nhận

vòng đời
```

Mô hình này giữ nguyên:

```text
114.688 / 114.688
```

quyết định của họ đầu tiên.

Nó còn biểu diễn được:

```text
cách biểu diễn tùy chọn

cách biểu diễn bắt buộc

cơ chế thực thi trực tiếp
không cần cách biểu diễn phụ
```

và một lớp Vulkan thật có thể tuân theo cùng các ranh giới đó.

Đó là bằng chứng có giá trị về **cấu trúc runtime**.

Nhưng không có phép suy luận:

```text
v4 PASS
→ ArcLLM nhanh hơn
```

Không có bằng chứng đó.

Lớp trừu tượng được tạo ra để runtime mô tả và quản lý đúng những cơ chế mà nghiên cứu đã tìm thấy.

Nó không phải một thủ thuật tăng token/giây.

Đây là một loại tiến bộ khác:

> **Ta hiểu và kiểm soát cỗ máy tốt hơn, ngay cả khi bước đó không tạo thêm một con số tăng tốc.**

## Có những lúc bước đúng tiếp theo là không xây gì cả

Một trong những thay đổi lớn nhất trong cách ArcLLM được nghiên cứu là:

> **Không còn coi mọi phần cứng chưa dùng là một cơ hội phải triển khai.**

Máy nghiên cứu còn có NPU.

Một phản xạ rất tự nhiên sẽ là:

> “GPU vẫn còn chậm. Hãy chuyển một phần sang NPU.”

Nhưng trước khi viết một đường NPU hoàn chỉnh, ArcLLM hỏi:

```text
NPU có thực sự tồn tại và gọi được không?

phép tính nào nó thực hiện được?

dữ liệu cần ở dạng nào?

chuyển dữ liệu mất bao nhiêu?

phần đó hiện chiếm bao nhiêu thời gian?

nếu NPU nhanh vô hạn,
toàn hệ còn đủ chỗ để cải thiện không?
```

Đây chính là định luật Amdahl quay lại lần nữa.

Không phải để tính một con số đẹp.

Mà để quyết định:

> **Có đáng xây không?**

Có một ranh giới bằng chứng cần khóa ngay trước các con số tiếp theo:

> **Các số NPU dưới đây là phép chiếu phân tích từ evidence exact-target hiện có của ArcLLM kết hợp với timing của NPU provider. Chúng không phải một fresh full-model benchmark có NPU, và ở thời điểm này chưa có NPU backend được tích hợp vào canonical runtime.**

Cụ thể, phần current-canonical được ước tính bằng cách lấy evidence hậu-I002 rồi áp tỷ lệ B/0 đã đo của Q4-down vào phần Q4_K FFN-down trước khi chuẩn hóa lại các family share. Vì vậy những con số này dùng để quyết định **có đáng mở một bounded transfer study hay không**, không phải để tuyên bố production speedup.

## Cơ chế từng thành công lớn có thể trở thành nơi không đáng chuyển tiếp

Gate/up từng là chiến thắng quan trọng.

Nhưng sau khi cơ chế GPU hiện tại đã cải thiện nó mạnh, phép phân tích NPU cho họ này chỉ còn khoảng:

```text
W-S
≈ 1,00×

W-C
≈ 1,04×
```

ở cấp họ phép tính theo đường nhà cung cấp đã đo.

Tức gần như không còn ngân sách đáng kể để mở một nhánh chuyển NPU cho gate/up.

Đây là một bài học đẹp.

> **Một nút thắt từng đúng không có nghĩa nó vẫn là nút thắt sau khi hệ thống thay đổi.**

Thành công của chính ArcLLM đã làm một câu hỏi cũ hết hạn.

## Nhưng FFN-down vẫn để lại một câu hỏi mở

FFN-down cho tín hiệu khác.

Theo phân tích trên runtime hiện tại, ngân sách thời gian còn lại ước tính khoảng:

```text
W-S
≈ 66,66 ms / token

W-C
≈ 24,20 ms / token
```

sau khi tính tới đường NPU đã đo ở mức nhà cung cấp.

Con số này đủ để cho phép mở:

> **một nghiên cứu NPU có giới hạn cho FFN-down.**

Chỉ FFN-down.

Không phải toàn model.

Không phải mọi phép GEMM.

Không phải:

> “ArcLLM giờ sẽ dùng NPU.”

Đó vẫn chỉ là một câu hỏi đủ tốt để đáng tiêu thêm bằng chứng.

## Và ngay câu hỏi đó cũng bị ràng buộc bởi vòng đời

Để FFN-down đi qua đường NPU đang khả thi, trọng số lượng tử hóa cần được tạo thành một cách biểu diễn FP16 ở thời điểm nạp model hoặc từ một bản đã được lưu sẵn.

Tổng dữ liệu FP16 cho 28 layer FFN-down vào khoảng:

```text
3,54 GiB
```

Chi phí nhập đồ thị đã biên dịch, nếu nhìn theo phép đo tuần tự hiện tại cho cả 28 layer, lên tới khoảng:

```text
12,6 giây
```

Điểm hòa vốn ước tính vì vậy vào khoảng:

```text
W-S
≈ 189 token

W-C
≈ 521 token
```

Điều này nói rằng:

> **Hiện chưa có cơ sở để tuyên bố một yêu cầu ngắn, bắt đầu từ trạng thái lạnh, sẽ hưởng lợi từ NPU.**

Cơ hội chỉ còn hợp lý hơn khi:

```text
phiên sử dụng đủ dài

hoặc

đồ thị và cách biểu diễn dữ liệu
được giữ lại để tái sử dụng
```

Ta đã đi một vòng rất xa để quay lại bài học Chương 18.

Hiệu năng không chỉ là:

> **thiết bị nào nhân ma trận nhanh hơn?**

Nó còn là:

> **Dữ liệu phải đổi dạng không? Đổi khi nào? Chi phí ban đầu là bao nhiêu? Và thứ vừa tạo sẽ sống bao lâu?**

## Vì vậy NPU vẫn đứng ngoài runtime chuẩn

Điều bằng chứng cho phép nói là:

> **FFN-down đủ điều kiện để mở một nghiên cứu chuyển sang NPU có giới hạn.**

Điều bằng chứng chưa cho phép nói:

```text
NPU đã làm ArcLLM nhanh hơn
```

hay:

```text
toàn model nên chuyển sang NPU
```

hay:

```text
yêu cầu ngắn sẽ có lợi
```

hay:

```text
NPU đã được tích hợp vào runtime chuẩn
```

Không có kết luận nào trong số đó.

Đây là một điểm kết rất phù hợp.

Một runtime trưởng thành không phải runtime nhét mọi khả năng phần cứng vào bên trong.

Nó phải biết:

> **Thứ gì đã đủ bằng chứng để trở thành một phần của cỗ máy, và thứ gì vẫn chỉ là một câu hỏi nghiên cứu.**

## Vậy điều gì vẫn chưa được chứng minh?

Runtime chuẩn đã PASS việc tách khỏi vỏ thí nghiệm.

Nhưng chính bằng chứng đóng băng vẫn giữ những ranh giới rất cụ thể.

### Chất lượng trên đầu vào tùy ý

Phép thử ngoài bộ dữ liệu kiểm thử cố định cho thấy giao diện runtime thực sự nhận được token do người gọi cung cấp.

Nó không chứng minh:

> **chất lượng ngôn ngữ trên mọi prompt tùy ý đã được xác nhận.**

Hai chuyện khác nhau.

### Phiên model sống lâu

Runtime hiện tại chưa chứng minh một mô hình dịch vụ trong đó model được giữ sống lâu và phục vụ nhiều yêu cầu như một phiên bền vững.

Điều này đặc biệt quan trọng với những cách biểu diễn có chi phí tạo lớn.

### NPU

NPU chưa được tích hợp vào runtime chuẩn.

Mới chỉ có một nhánh nghiên cứu giới hạn được cho phép mở.

### Đối chứng bên ngoài sau lần tách runtime cuối cùng

Việc tách runtime chuẩn không tạo một kết luận hiệu năng mới.

Vì vậy không được lấy nó làm bằng chứng rằng khoảng cách với llama.cpp đã thay đổi.

### Tính phổ quát của sáu chiều

Mô hình v4 đã sống sót qua những họ hiện có.

Không có bằng chứng rằng sáu chiều đó sẽ đủ cho mọi cơ chế tương lai.

Một phản ví dụ mới vẫn có quyền phá nó.

Đó không phải điểm yếu.

Đó chính là cách mô hình này được sinh ra từ đầu.

## Ta cũng chưa hiểu hết phần cứng

Qua nhiều chương, ta biết được rằng:

```text
thay đổi A
→ độ trễ giảm
```

hay:

```text
thay đổi B
→ độ trễ giảm
```

Nhưng không phải lúc nào các bộ đếm phần cứng cũng trả lời đầy đủ:

> **Vì sao ở cấp vật lý hiệu ứng đó xuất hiện?**

Ở thí nghiệm Q4-down, ba trong bốn kênh bộ đếm cần thiết không cung cấp tín hiệu đủ dùng.

Ta có bằng chứng cho:

```text
hiệu ứng
```

nhưng chưa có bằng chứng đầy đủ cho:

```text
toàn bộ cơ chế nhân quả
ở tầng phần cứng
```

Đây là một ranh giới quan trọng.

Ta hiểu cỗ máy sâu hơn rất nhiều so với Chương 1.

Nhưng:

> **“Hiểu sâu hơn” không có nghĩa “đã biết mọi thứ xảy ra bên trong.”**

## Nếu chỉ giữ PASS, cuốn sách sẽ kể sai lịch sử

Ta có thể viết lại ArcLLM thành một câu chuyện rất đẹp:

```text
xây runtime
↓
tìm nút thắt
↓
tối ưu Q4
↓
tạo EXEC148
↓
xây v4
↓
hội tụ
```

Nhưng đó không phải lịch sử thật.

Lịch sử thật còn có:

```text
kiến trúc đầu tiên
→ không chứng minh được lợi thế
```

```text
Q6 Split-K
→ FAIL về tính đúng
```

```text
một kiến trúc kế tiếp
→ decode tốt hơn
→ E2E tốt hơn
→ nhưng FAIL vì TTFT
```

```text
A tốt
B tốt
A+B
→ không cộng lợi ích như dự đoán
```

```text
bộ đếm phần cứng
→ một phần không đủ thông tin
```

```text
mô hình thu nhận đầu tiên
→ bị một họ mới phá
```

```text
sẵn sàng thực thi bị gắn với trạng thái cư trú
→ bị cơ chế trực tiếp phá
```

```text
lần gắn lớp kết nối phần cứng đầu tiên
→ dùng nhầm công cụ thí nghiệm
→ FAIL
```

Không thất bại nào trong số này là rác.

Chúng tạo thành bản đồ dẫn tới kiến trúc hiện tại.

Nếu xóa chúng, ta chỉ còn:

> **đáp án cuối.**

Ta mất:

> **lý do vì sao đáp án đó tồn tại.**

## Có lẽ đây mới là sản phẩm thật của toàn bộ hành trình

Tên cuốn sách là:

> **Inside ArcLLM — Xây dựng một runtime LLM từ những nguyên lý đầu tiên**
>
> *Building an LLM Runtime from First Principles*

Ta thực sự đã xây một runtime.

Nhưng thứ có giá trị không chỉ là mã nguồn của nó.

Ta còn giữ được con đường:

```text
không biết
↓
đặt câu hỏi
↓
đo
↓
để bằng chứng loại bớt khả năng
↓
khóa một cơ chế
↓
chấp nhận PASS hoặc FAIL
↓
chuyển kết quả sang hệ thống thật
↓
đo lại
↓
chỉ tạo lớp trừu tượng
khi thực tế buộc phải tạo
```

Đây cũng là nơi vai trò giữa con người và AI trở nên rõ nhất.

AI có thể:

```text
đọc mã nguồn

đề xuất nhiều giả thuyết

viết phần triển khai

tạo công cụ kiểm tra

chạy QA

so sánh hàng nghìn trạng thái

tổng hợp bằng chứng
```

Nhưng con người vẫn phải quyết định:

```text
câu hỏi nào đáng hỏi?

bằng chứng nào đủ?

điều gì phải khóa trước khi thấy kết quả?

một FAIL có được giữ nguyên không?

một PASS được phép nói tới đâu?

bằng chứng cũ đã hết hạn chưa?

và khi nào phải dừng?
```

Khi AI làm cho việc tạo phương án trở nên rẻ hơn, những quyết định này không bớt quan trọng.

Chúng trở nên quan trọng hơn.

## Cỗ máy đã được xây — nhưng ta có thể nhìn nó theo cách khác không?

Ở Chương 1, ArcLLM gần như là một hộp đen.

Tới đây, ta đã biết rất nhiều thứ mà trước đó chỉ nằm sau một lệnh “generate”:

```text
token đi qua đâu

tensor nào được đọc

phép tính nào xảy ra

kernel nào thực thi

KV cache thay đổi thế nào

cách biểu diễn nào tồn tại

cách biểu diễn nào được tạo

đường thực thi nào được chọn

dữ liệu sống bao lâu

và nhiều phần chi phí nằm ở đâu
```

Nhưng khi đã nhìn thấy đủ nhiều lớp bên trong, một câu hỏi khác tự nhiên xuất hiện.

Cho tới nay, ta chủ yếu làm như sau:

```text
nghi ngờ một vùng
↓
đo vùng đó
↓
thay đổi có kiểm soát
↓
xem kết quả
```

Nếu thay vì chỉ đo từng vùng mà ta đã biết phải nhìn, ta muốn quan sát **phản ứng của cả hệ thống khi nó đang hoạt động** thì sao?

Không chỉ hỏi:

> “Kernel này mất bao nhiêu mili-giây?”

Mà hỏi:

> **“Khi một token đi xuyên qua cỗ máy, toàn bộ hệ thống đang biến đổi như thế nào?”**

Đây không phải một câu hỏi cần được giải quyết để hoàn thành ArcLLM trong cuốn sách này.

Nó là một cánh cửa khác.

Và cánh cửa đó chỉ xuất hiện bởi cỗ máy đã được xây đủ sâu để ta biết mình đang muốn quan sát điều gì.

### Nhớ 3 điều

1. **ArcLLM đã hội tụ thành một runtime tách khỏi vỏ thí nghiệm.** Các đối chứng đóng băng, cấu trúc dispatch và hành vi vòng đời được giữ nguyên, đồng thời runtime có thể nhận token đầu vào ngoài những bộ dữ liệu kiểm thử lịch sử.
2. **“Có runtime” không có nghĩa “đã giải xong runtime”.** Khoảng cách với đối chứng trưởng thành vẫn lớn; chất lượng trên đầu vào tùy ý, phiên model sống lâu, NPU và tính phổ quát của mô hình v4 đều còn những ranh giới chưa được chứng minh.
3. **Kết quả quan trọng không chỉ nằm ở những PASS.** Những FAIL đã loại các giả thuyết yếu, xác định biên giới và buộc kiến trúc chỉ xuất hiện khi bằng chứng thực sự yêu cầu nó.

---

## Inside ArcLLM kết thúc tại đây

Ta bắt đầu với một file model và một câu hỏi:

> **“Bên dưới một câu trả lời AI có gì?”**

Ta không trả lời câu hỏi đó bằng cách đọc một sơ đồ kiến trúc rồi học thuộc tên các thành phần.

Ta mở cỗ máy ra.

Từng lớp một.

Ta xây chúng.

Làm chúng sai.

Đo chúng.

Loại bỏ những cách giải thích không đứng vững.

Giữ cả những con đường thất bại.

Và cuối cùng đưa những phần còn sống sót trở lại thành một runtime thật.

Câu trả lời cuối cùng vì vậy không phải một sơ đồ.

Nó là một bản đồ:

```text
cái gì tồn tại

↓

cái gì đã được chứng minh

↓

cái gì từng có vẻ đúng nhưng đã FAIL

↓

cái gì vẫn chưa biết
```

Cỗ máy đã được xây.

Nhưng chính vì bây giờ ta có thể nhìn vào bên trong nó, một câu hỏi mới bắt đầu hiện ra:

> **Nếu không chỉ nhìn đầu vào và đầu ra, ta có thể quan sát chính hệ thống đang biến đổi bên trong như thế nào?**

Phần Bonus sau cuốn sách sẽ bắt đầu từ câu hỏi đó.
