# Chương 3 — Xây phần lõi Vulkan

> **Câu hỏi của chương:** Ta đã biết dữ liệu của model nằm ở đâu. Làm thế nào để GPU thật sự có một nơi nhận dữ liệu, nhận lệnh và báo lại rằng công việc đã hoàn thành?

Ở cuối Chương 2, ArcLLM đã biến file GGUF từ một “cục dữ liệu” thành một kho tensor có bản đồ rõ ràng. Runtime biết tensor nào nằm ở đâu, dài bao nhiêu byte và đang được lưu theo kiểu nào. Q4_K vẫn có thể được giữ nguyên dạng đóng gói thay vì bung toàn bộ thành F32.

Nhưng lúc này GPU vẫn chưa thật sự làm việc.

Biết một kiện hàng nằm ở ngăn nào trong kho không có nghĩa dây chuyền sản xuất đã có thể lấy nó ra và sử dụng. Ta còn cần đường vận chuyển, khu vực chứa nguyên liệu, một chỗ để làm việc, cách gửi phiếu việc xuống máy và một tín hiệu báo rằng máy đã làm xong.

Đó là nhiệm vụ của P2.

## Vulkan là cây cầu xuống GPU

Ở Chương 1, Vulkan đã xuất hiện rất ngắn. Bây giờ ta có thể nhìn nó rõ hơn.

**Vulkan** là một giao diện lập trình cấp thấp cho phép chương trình làm việc khá trực tiếp với GPU. “Cấp thấp” ở đây không có nghĩa kém hơn. Nó có nghĩa Vulkan để lộ nhiều chi tiết mà những lớp phần mềm cao hơn thường che đi.

Điều đó khiến Vulkan khó dùng hơn, nhưng lại hợp với ArcLLM: ta muốn nhìn thấy ai cấp bộ nhớ, ai giữ dữ liệu, ai gửi công việc và khi nào GPU hoàn thành.

Có thể hình dung:

```text
ArcLLM
   ↓
Vulkan
   ↓
driver
   ↓
GPU
```

**Driver — trình điều khiển —** là phần mềm nối hệ điều hành và chương trình với phần cứng cụ thể. ArcLLM không nói trực tiếp với từng mạch điện trong GPU. Nó dùng Vulkan; Vulkan và driver chịu trách nhiệm chuyển yêu cầu đó xuống phần cứng.

P2 chưa cố chạy model.

Mục tiêu thấp hơn nhiều:

> **Dựng được một “xưởng” tối thiểu nơi dữ liệu có thể ở lại, GPU có thể nhận một công việc, thực hiện nó và cho runtime biết công việc đã xong.**

## Device: trước hết phải biết ta đang nói chuyện với GPU nào

Trong Vulkan, một trong những khái niệm đầu tiên là **device — thiết bị**.

Máy tính có thể có nhiều thiết bị hoặc nhiều khả năng khác nhau. Runtime không thể chỉ nói chung chung “hãy chạy trên GPU”. Nó phải mở một kết nối làm việc với thiết bị cụ thể mà Vulkan cung cấp.

Trong P2, ArcLLM lấy được **một Vulkan device — một thiết bị Vulkan mà runtime có thể dùng để làm việc với GPU**.

Con số “1” ở đây chỉ có nghĩa runtime đã chọn và mở được một thiết bị Vulkan cho đường thực thi này.

Nhưng có device vẫn chưa đủ.

Ta cần một nơi để gửi công việc.

## Queue: hàng chờ nhận việc của GPU

Vulkan dùng khái niệm **queue — hàng đợi công việc**.

Hãy tưởng tượng một xưởng có một cửa nhận phiếu việc. Bạn không chạy vào giữa dây chuyền và tự tay ra lệnh cho từng bộ phận. Bạn chuẩn bị một phiếu rõ ràng rồi đặt nó vào hàng chờ. Xưởng lấy các phiếu theo cơ chế của nó và xử lý.

P2 cần một **compute queue — hàng đợi có khả năng nhận công việc tính toán**.

Kết quả: ArcLLM lấy được queue đó.

Tới đây ta có:

```text
ArcLLM
   ↓
Vulkan device
   ↓
compute queue
   ↓
GPU
```

Nhưng vẫn còn một câu hỏi lớn: dữ liệu ở đâu?

## Không thể mỗi lần cần tensor lại đi lấy từ đầu

Ở P1, GGUF được ánh xạ để runtime biết và truy cập trực tiếp từng vùng byte. Đó là một nền móng tốt cho phía file.

P2 chuyển sang câu hỏi phía GPU:

> Nếu model sắp phải dùng cùng những trọng số rất nhiều lần, có nên mỗi lần lại chuẩn bị chúng từ đầu hay không?

Câu trả lời của thiết kế P2 là giữ dữ liệu cần thiết trong những vùng bộ nhớ tồn tại lâu hơn.

Từ **persistent** có thể hiểu đơn giản là **được giữ lại trong thời gian dài hơn thay vì tạo rồi bỏ ngay**.

Thay vì:

```text
tạo buffer
→ dùng một lần
→ xóa
→ lần sau tạo lại
```

P2 muốn:

```text
tạo vùng nhớ
→ đưa dữ liệu vào
→ giữ nó sống
→ nhiều công việc sau có thể tiếp tục dùng
```

Trong giai đoạn này, ta có thể gọi đó là dữ liệu **resident — cư trú, tức đang nằm sẵn trong vùng bộ nhớ phục vụ GPU**. Ở P2, nghĩa rất đơn giản: vùng nhớ đã được tạo, dữ liệu đã được đặt vào và runtime chưa giải phóng nó.

## Tại sao phải chia thành bốn arena?

Payload đóng gói của GGUF mà P2 cần giữ có kích thước:

```text
980.097.536 byte
```

Đổi sang MiB:

```text
1 MiB = 1.048.576 byte

980.097.536 / 1.048.576
≈ 934,7 MiB
```

P2 chọn chia vùng chứa trọng số thành các **arena — vùng chứa lớn được runtime quản lý**, mỗi arena không lớn hơn 256 MiB.

Thay vì tạo hàng trăm vùng nhỏ cho từng tensor, runtime có một số khoang lớn để giữ cả payload.

Ta thử tính:

```text
3 × 256 MiB = 768 MiB
```

768 MiB không đủ cho khoảng 934,7 MiB dữ liệu.

Còn:

```text
4 × 256 MiB = 1.024 MiB
```

thì đủ sức chứa.

Vì vậy P2 giữ toàn bộ payload trong **4 persistent weight arenas — bốn vùng lưu giữ trọng số tồn tại lâu dài**, mỗi arena không vượt quá giới hạn 256 MiB mà thiết kế đã đặt ra.

Điểm quan trọng ở đây: 256 MiB là cách P2 tổ chức bộ nhớ cho bước này; không nên đọc nó thành “GPU chỉ cấp được tối đa 256 MiB”.

Bốn arena là quyết định của runtime trong contract P2.

## Một bàn làm việc riêng: scratch memory

Trọng số của model giống nguyên liệu hoặc dụng cụ được giữ lâu dài.

Nhưng khi tính toán, chương trình còn cần chỗ để đặt kết quả tạm thời.

Ví dụ rất đời thường: trong bếp, tủ đựng gạo và gia vị có thể tồn tại lâu dài; nhưng bạn vẫn cần mặt bàn để thái rau, đặt bát và chuẩn bị món ăn. Mặt bàn không phải nguyên liệu. Nó là không gian làm việc.

Runtime cũng cần một vùng như vậy.

P2 tạo một **scratch arena — vùng nhớ làm việc tạm**, dung lượng 64 MiB, và cũng giữ vùng này tồn tại để sẵn sàng cho các công việc tiếp theo.

```text
4 vùng lưu trọng số
+ 1 vùng scratch làm việc tạm
-----------------------------
= 5 buffer đang sống
```

Đây chính là năm buffer được ghi nhận còn tồn tại khi lệnh GPU được thực hiện trong P2.

Một lần nữa, P2 chưa dùng scratch để chạy attention hay nhân ma trận của model. Nó chỉ đang chứng minh rằng hạ tầng bộ nhớ tồn tại và có thể được GPU đụng tới đúng cách.

## Buffer là gì?

Từ **buffer** xuất hiện rất nhiều trong lập trình hệ thống.

Trong ngữ cảnh hiện tại, có thể hiểu **buffer — vùng chứa dữ liệu đã được thiết lập để chương trình/GPU sử dụng**.

Điều đáng nhớ không phải tên API, mà là bức tranh:

```text
GGUF payload
     ↓
┌────────────┐
│ weight #1  │
├────────────┤
│ weight #2  │
├────────────┤
│ weight #3  │
├────────────┤
│ weight #4  │
└────────────┘

┌────────────┐
│ scratch    │
└────────────┘
```

Bốn vùng phía trên giữ dữ liệu model.

Vùng phía dưới là bàn làm việc.

## Nhưng làm sao “giao việc” cho GPU?

Có device, queue và buffer rồi vẫn chưa đủ. Runtime phải mô tả công việc cần làm.

Vulkan dùng **command buffer — vùng ghi các lệnh sẽ gửi cho GPU**.

Command buffer không phải kho chứa trọng số. Nó giống một tờ phiếu công việc:

```text
làm việc A
sau đó làm việc B
sau đó làm việc C
```

Các command buffer thường được cấp từ một **command pool — nơi quản lý các command buffer**.

P2 dựng được command pool và command buffer cần thiết, rồi gửi command buffer vào compute queue.

Nhưng một vấn đề khác xuất hiện.

GPU làm việc bất đồng bộ với CPU: CPU có thể giao việc rồi tiếp tục chạy. Vậy làm sao CPU biết lúc nào GPU đã hoàn thành?

## Fence: tấm biển “đã làm xong”

P2 dùng một **fence — tín hiệu đồng bộ cho biết công việc GPU đã hoàn thành**.

Hãy tưởng tượng bạn gửi một đơn xuống xưởng rồi nhận một số thứ tự. Bạn không thể kiểm tra sản phẩm trước khi xưởng báo hoàn tất.

Fence đóng vai trò tương tự.

Chuỗi đơn giản là:

```text
CPU chuẩn bị command
        ↓
submit vào queue
        ↓
GPU thực hiện
        ↓
fence được báo hoàn thành
        ↓
CPU mới kiểm tra kết quả
```

Nếu không có ranh giới này, CPU có thể đọc vùng nhớ khi GPU còn chưa viết xong và tưởng nhầm đó là kết quả sai.

## P2 chưa chạy model — nó chạy một phép thử nhỏ hơn nhiều

Đây là điểm rất dễ bị hiểu nhầm.

P2 không cần một phép nhân ma trận lớn để chứng minh đường GPU hoạt động.

Nếu mục tiêu chỉ là kiểm tra:

> “Ta có thật sự gửi được một lệnh xuống GPU, GPU có chạm được vào vùng nhớ và ta có xác nhận được kết quả sau khi nó hoàn thành không?”

thì một phép thử nhỏ sẽ tốt hơn.

P2 yêu cầu GPU **fill — điền một giá trị vào vùng scratch**.

Sau khi command được submit và fence báo hoàn thành, phía CPU kiểm tra lại vùng nhớ có mang đúng dấu vết mong đợi hay không.

Kết quả:

```text
command submit : PASS
fence wait     : PASS
scratch fill   : VERIFIED
```

Nói bằng tiếng Việt bình thường:

> ArcLLM đã đưa được một công việc thật xuống GPU, chờ GPU làm xong và xác nhận vùng nhớ đã thay đổi đúng như yêu cầu.

Đây chưa phải phép tính AI, nhưng “nhà máy” đã thật sự nhận một phiếu việc và phản hồi.

## Làm sao biết dữ liệu trọng số không bị sai khi đưa vào các arena?

P2 còn kiểm tra **fingerprint — dấu nhận dạng rút gọn của dữ liệu** cho các arena trọng số.

Ý tưởng giống việc niêm phong bốn kiện hàng trước khi vận chuyển rồi kiểm tra lại dấu nhận dạng sau khi chúng đã được đặt vào kho mới.

Kết quả là cả bốn arena đều vượt qua kiểm tra fingerprint.

Điều đó cho P2 bằng chứng rằng payload vẫn nhất quán trong các vùng bộ nhớ mới. Nhưng đây chưa phải bằng chứng mọi tensor sẽ tính đúng: P2 chưa thực hiện phép toán model.

Nó chỉ chứng minh **đường đưa và giữ dữ liệu đã hoạt động theo contract — tức hợp đồng kỹ thuật về cách các lớp phải phối hợp với nhau: ArcLLM ↔ Vulkan ↔ CPU/GPU**.

> **Ghi chú nhỏ:** “Contract” trong sách không phải hợp đồng pháp lý. Nó là tập những điều đã được xác định trước rằng mỗi lớp phải làm đúng: ArcLLM yêu cầu gì, Vulkan chuyển và đồng bộ yêu cầu đó thế nào, CPU/GPU phải tạo ra dấu hiệu/kết quả nào để phép thử được coi là đạt.

## P2 PASS thực sự có nghĩa gì?

Đến cuối P2, ArcLLM có:

```text
1 Vulkan device
→ 1 thiết bị Vulkan mà ArcLLM đã mở để làm việc với GPU

1 compute queue
→ 1 hàng đợi nhận công việc tính toán

980.097.536 byte dữ liệu GGUF vẫn ở dạng đóng gói (packed GGUF payload)
→ khoảng 934,7 MiB
→ cư trú (resident) trong 4 vùng lưu giữ trọng số
  (persistent weight arenas)

64 MiB vùng nhớ làm việc tạm
(persistent scratch)

4 vùng lưu trọng số (weight buffers)
+ 1 vùng làm việc tạm (scratch buffer)
= 5 buffer sống trong lúc thực thi

command submit — gửi lệnh xuống queue
/ fence — tín hiệu chờ GPU hoàn tất
→ PASS

GPU scratch fill
— GPU điền dữ liệu thử vào vùng scratch
→ VERIFIED
```

P2 vì vậy được đóng với kết quả PASS.

Nhưng PASS ở đây có phạm vi rất cụ thể.

P2 **không chứng minh model chạy đúng**.

P2 không chứng minh Q4_K được GPU giải mã đúng.

P2 chưa có attention.

Chưa có softmax.

Chưa có một decoder layer.

Và hoàn toàn chưa có benchmark hiệu năng model.

P2 chỉ nói:

> **ArcLLM đã có một lõi Vulkan đủ để giữ payload, có vùng làm việc, gửi được công việc tới GPU và xác nhận việc đó hoàn thành.**

Khi nền móng này đứng vững, câu hỏi tiếp theo thay đổi.

Ta không còn phải hỏi:

> “GPU có nhận lệnh của ArcLLM không?”

Ta có thể hỏi:

> **“GPU có thực hiện đúng những phép toán thật mà model cần không?”**

Đó là lúc ArcLLM đi từ hạ tầng sang toán học của model.

Ở P3, từng phép tính sẽ được đưa lên GPU một cách có kiểm soát: chuẩn hóa, nhân với trọng số Q4_K đóng gói, xoay vị trí, attention, softmax và những phần khác.

Nhưng có một nguyên tắc sẽ giữ nguyên:

> **Đừng xây cả model rồi mới hy vọng nó đúng. Hãy chứng minh từng viên gạch trước.**

### Nhớ 3 điều

1. **Vulkan cho ArcLLM một con đường có kiểm soát tới GPU:** device — thiết bị để làm việc với GPU; queue — hàng đợi để giao việc; fence — tín hiệu để biết khi nào việc đã hoàn thành.
2. **P2 tách bộ nhớ thành phần giữ lâu và phần làm việc:** khoảng 934,7 MiB payload cư trú trong bốn vùng lưu trọng số; 64 MiB scratch là vùng làm việc tạm.
3. **P2 PASS chưa phải model PASS.** Nó chỉ chứng minh hạ tầng bộ nhớ và đường gửi lệnh GPU đã hoạt động đủ để bắt đầu kiểm tra những phép toán thật.

**Chương 4 — Từng phép tính trước, model sau**

Ta đã có kho dữ liệu và đã dựng được nhà máy.

Bây giờ đến câu hỏi quan trọng hơn: nếu đưa một phép toán thật của model xuống GPU, làm sao biết GPU tính đúng?

ArcLLM sẽ chưa chạy cả model.

Nó sẽ bắt đầu với từng viên gạch.
