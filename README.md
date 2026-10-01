# Inside ArcLLM — Xây dựng hệ thực thi cho mô hình ngôn ngữ lớn từ những nguyên lý đầu tiên

*Building an LLM Runtime from First Principles*

**Từ con số 0 đến một cỗ máy mà ta hiểu được từng lớp bên trong.**

Inside ArcLLM là một cuốn sách tiếng Việt dành cho người bắt đầu từ số 0. Sách không giả định bạn đã biết trí tuệ nhân tạo, phần cứng máy tính hay lập trình.

Trong sách, **mô hình ngôn ngữ lớn (Large Language Model, LLM)** là loại mô hình tạo văn bản mà chúng ta thường gặp trong các ứng dụng AI. Cuốn sách dùng hành trình xây ArcLLM như một câu chuyện thật để theo đuổi hai mục tiêu:

1. **Hiểu AI từ bên trong bằng ngôn ngữ dễ tiếp cận.** Ta đi từ văn bản, token, các khối số, mô hình, bộ nhớ, CPU và GPU tới cách một hệ thực thi thật sự đưa các phép tính của mô hình xuống phần cứng.
2. **Học cách làm việc cùng AI mà vẫn giữ quyền phán đoán của con người.** Ta đặt câu hỏi, khóa điều kiện trước khi đo, giữ cả kết quả đạt lẫn không đạt, và chỉ kết luận trong phạm vi bằng chứng cho phép.

Cuốn sách hiện có **Lời nói đầu, Phần 0, Chương 1–20 và Bonus**.

## Bắt đầu đọc

- [Lời nói đầu — Thư gửi người đọc](chapters/00-loi-noi-dau.md)
- [Phần 0 — Bản đồ trấnh lạc lối](chapters/00-ban-do-truoc-khi-vao-rung.md)
- [Chương 1 — Bên dưới một câu trả lời AI có gì?](chapters/01-khoa-target-truoc-khi-toi-uu.md)
- [Chương 2 — Bên trong tệp mô hình có gì? (GGUF)](chapters/02-gguf-tensor-store.md)
- [Chương 3 — Làm thế nào để giao việc cho GPU? (Vulkan)](chapters/03-vulkan-runtime-core.md)
- [Chương 4 — Từng phép tính nhỏ trước, cả mô hình sau](chapters/04-tung-phep-tinh-truoc-model-sau.md)
- [Chương 5 — Ghép các phép tính thành một lớp giải mã](chapters/05-mot-decoder-layer-hoan-chinh.md)
- [Chương 6 — Giữ toàn bộ các lớp xử lý sẵn trong bộ nhớ GPU](chapters/06-full-decoder-residency.md)
- [Chương 7 — Bộ nhớ giúp mô hình không phải tính lại từ đầu (KV cache)](chapters/07-kv-cache-model-bat-dau-nho-token-truoc.md)
- [Chương 8 — Từ “chạy được” tới một đường chạy thực tế](chapters/08-production-path-khong-den-tu-mot-kernel-than-ky.md)
- [Chương 9 — Muốn biết nhanh hay chậm, phải có một mốc để so](chapters/09-benchmark-phai-co-doi-chung.md)
- [Chương 10 — Tự xây được vẫn chưa có nghĩa là tốt hơn](chapters/10-khi-tu-build-duoc-van-chua-du.md)
- [Chương 11 — Từ một thất bại tới câu hỏi đúng hơn](chapters/11-tu-that-bai-sang-mot-cau-hoi-dung-hon.md)
- [Chương 12 — Nhiều cách đều có lý, nhưng chỉ thực tế mới trả lời](chapters/12-nhieu-kien-truc-nhung-chi-thuc-te-moi-tra-loi-duoc.md)
- [Chương 13 — Nhanh nhưng sai thì vẫn là sai](chapters/13-khi-correctness-noi-khong.md)
- [Chương 14 — Từ một phép tính tốt tới cả hệ thống thật](chapters/14-tu-mot-co-che-tot-toi-he-thong-that.md)
- [Chương 15 — Chỉ có ích khi toàn hệ thực sự được lợi](chapters/15-mot-kien-truc-chi-thang-khi-toan-he-duoc-loi.md)
- [Chương 16 — Tách cách sắp dữ liệu khỏi cách thực hiện phép tính](chapters/16-experiment-2x2-tach-representation-khoi-execution.md)
- [Chương 17 — Khi một cách sắp dữ liệu trở thành một phần của kiến trúc](chapters/17-exec148-khi-bang-chung-buoc-mot-lop-truu-tuong-moi-xuat-hien.md)
- [Chương 18 — Dữ liệu có mặt chưa đủ: nó phải sẵn sàng đúng lúc](chapters/18-du-lieu-o-trong-bo-nho-van-chua-du.md)
- [Chương 19 — Từ ArcLLM tới một cách mô tả hệ thực thi tổng quát hơn](chapters/19-tu-arcllm-cu-the-toi-mo-hinh-runtime-tong-quat-hon.md)
- [Chương 20 — Ta đã hiểu cỗ máy đến đâu?](chapters/20-ta-da-hieu-runtime-den-dau.md)
- [Bonus — Từ xây cỗ máy tới lắng nghe cỗ máy](chapters/bonus-tu-xay-co-may-toi-lang-nghe-co-may.md)

## Lộ trình của cuốn sách

Sách đi theo bốn mức. Đây không phải điểm số, mà chỉ để người đọc biết độ sâu đang tăng dần.

```text
Nền tảng
   ↓
Đi sâu
   ↓
Nghiên cứu
   ↓
Nâng cao
```

### Phần 0 — Bản đồ tránh lạc lối

**Mức đọc: Nền tảng**

Phần này dành riêng cho người chưa biết AI hoạt động ra sao. Ta bắt đầu từ trải nghiệm quen thuộc nhất: gõ một câu vào ô chat và nhận câu trả lời. Từ đó mới mở dần các khái niệm: mô hình, huấn luyện, suy luận, token, khối số, CPU, GPU, bộ nhớ, Transformer và hệ thực thi.

Mục tiêu không phải học thuộc thuật ngữ. Mục tiêu là có một bức tranh đủ đơn giản để sang Chương 1 không còn cảm giác “mỗi câu lại xuất hiện một từ mới”.

### Phần I — Xây cỗ máy

**Mức đọc: Đi sâu**

**Mục tiêu:** đi từ tệp mô hình nằm yên trên ổ đĩa tới một hệ thống có thể thật sự sinh token.

- **Chương 1:** phân biệt ứng dụng AI, mô hình, hệ thực thi và phần cứng; sau đó khóa đúng mô hình và điều kiện máy trước khi bắt đầu.
- **Chương 2:** mở tệp GGUF và hiểu những khối số chứa bên trong.
- **Chương 3:** xây đường giao tiếp với GPU bằng Vulkan.
- **Chương 4:** kiểm tra từng phép tính nhỏ trước khi ghép chúng lại.
- **Chương 5:** ghép các phép tính thành một lớp giải mã.
- **Chương 6:** đưa toàn bộ các lớp xử lý và trọng số cần thiết vào vùng bộ nhớ GPU có thể dùng.
- **Chương 7:** thêm bộ nhớ đệm khóa–giá trị (KV cache) để không phải tính lại toàn bộ phần quá khứ.
- **Chương 8:** ghép mọi thứ thành đường sinh nhiều token và bắt đầu tối ưu có đo đạc.

### Phần II — Để bằng chứng phán xét

**Mức đọc: Nghiên cứu**

**Mục tiêu:** không tự khen hệ thống của mình bằng cách chỉ so nó với chính nó.

- **Chương 9:** đặt ArcLLM cạnh một hệ thực thi trưởng thành trong cùng điều kiện.
- **Chương 10:** chấp nhận khi bằng chứng chưa cho thấy lợi thế thực tế.
- **Chương 11:** dùng thất bại để đặt một câu hỏi tốt hơn thay vì tiếp tục tối ưu mù quáng.

### Phần III — Một ý tưởng chỉ có giá trị khi sống sót qua thực tế

**Mức đọc: Nghiên cứu**

**Mục tiêu:** một phép tính nhỏ chạy nhanh chưa có nghĩa cả hệ thống sẽ tốt hơn.

- **Chương 12:** chọn một cơ chế đáng kiểm tra giữa nhiều ý tưởng hợp lý.
- **Chương 13:** tính đúng phải đi trước tốc độ.
- **Chương 14:** đưa cơ chế đã đạt ở phép thử nhỏ vào dữ liệu và mô hình thật.
- **Chương 15:** đo lại toàn hệ và đối chứng bên ngoài.

### Phần IV — Khi bằng chứng buộc kiến trúc phải thay đổi

**Mức đọc: Nâng cao**

Phần này dành cho người muốn đi tiếp từ “làm cho chạy nhanh” tới câu hỏi kiến trúc: cùng một dữ liệu có thể được sắp xếp theo nhiều cách, ai tạo nó, nó nằm ở đâu, khi nào sẵn sàng và nên giữ nó bao lâu.

- **Chương 16:** tách cách sắp dữ liệu khỏi cách GPU chia công việc.
- **Chương 17:** khi một cách sắp dữ liệu có lợi ích và chi phí riêng, hệ thực thi phải coi nó là một đối tượng riêng.
- **Chương 18:** “đang nằm trong bộ nhớ” chưa đồng nghĩa với “có thể dùng ngay”.
- **Chương 19:** rút các câu hỏi đó thành một mô hình mô tả hệ thực thi tổng quát hơn.
- **Chương 20:** tổng kết điều đã chứng minh, điều đã thất bại và điều vẫn chưa biết.

## Cách cuốn sách được viết

- **Tiếng Việt là ngôn ngữ chính, thuật ngữ gốc là chiếc cầu tra cứu.** Khi một khái niệm kỹ thuật được giới thiệu, sách ưu tiên dạng **tiếng Việt (English)**, ví dụ **hệ thực thi (runtime)**, **tín hiệu hoàn thành (fence)**, **chương trình GPU (kernel)**. Không dịch mất từ gốc trong ngoặc; người đọc cần nhận ra đúng thuật ngữ khi gặp tài liệu chuyên môn sau này.
- **Ý tưởng phải được hiểu trước khi học tên.** Một khái niệm mới bắt đầu bằng hình dung gần gũi hoặc cách hiểu tạm thời, rồi mới đi dần tới định nghĩa chính xác hơn.
- **Không dùng trước khi dạy.**
- **Không yêu cầu biết lập trình.**
- **Mọi con số quan trọng đều phải có ngữ cảnh.**
- **ĐẠT và KHÔNG ĐẠT đều có giá trị.** Trong phần kỹ thuật, sách có thể ghi thêm PASS/FAIL để giữ nguyên ngôn ngữ của thí nghiệm.
- **Không biến lỗi kỹ thuật thành kết luận khoa học.**
- **Không tuyên bố hiệu năng vượt quá bằng chứng đã đo.**
- **Không kể lại lịch sử như thể tác giả đã biết đáp án từ đầu.**

## Source code: https://github.com/minhtri22/Inside-ArcLLM

## Phần bổ sung

Bonus mở ra một câu hỏi rộng hơn: khi đã biết cách nhìn vào một cỗ máy AI, liệu ta có thể quan sát sự thay đổi của một hệ thống phức tạp theo thời gian thay vì chỉ nhìn đầu vào và đầu ra hay không?

Đây chỉ là một hướng gợi mở. Sách không tuyên bố rằng những khả năng đó đã được chứng minh ngoài phạm vi bằng chứng được trình bày.

## Về nguồn nghiên cứu

Các con số và kết luận kỹ thuật trong sách được biên tập từ những phép thử thật của ArcLLM.

Kho mã này chỉ chứa nội dung sách đã xuất bản. Mã nguồn hệ thực thi, dữ liệu thí nghiệm thô và tài liệu nghiên cứu chi tiết nằm ngoài kho sách.

Mục tiêu của *Inside ArcLLM* không chỉ là kể rằng một hệ thực thi đã được xây như thế nào. Quan trọng hơn là giữ lại **vì sao một hướng được chọn, vì sao một hướng bị loại, và bằng chứng nào đã buộc câu hỏi tiếp theo phải thay đổi**.
