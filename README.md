# Inside ArcLLM

**Từ con số 0 đến một runtime mà ta hiểu được từng lớp bên trong.**

Inside ArcLLM là một cuốn sách tiếng Việt dành cho người bắt đầu từ số 0 — không giả định bạn đã biết AI, hệ thống máy tính hay lập trình.

Cuốn sách dùng hành trình xây dựng ArcLLM như một câu chuyện thật để theo đuổi hai mục tiêu:

1. **Hiểu nền tảng AI từ bên trong** — từ model, token, tensor, runtime, CPU/GPU và bộ nhớ tới cách một hệ thống suy luận thực sự được xây dựng, kiểm tra và tối ưu.
2. **Học cách làm việc cùng AI mà vẫn giữ quyền phán đoán của con người** — đặt câu hỏi, khóa phạm vi, yêu cầu bằng chứng, phân biệt PASS/FAIL, biết khi nào nên tiếp tục và khi nào phải dừng.

Sách được xuất bản tuần tự theo từng chương. Hiện đã có **Lời nói đầu và Chương 1–18**: Phần I, Phần II và Phần III đã hoàn chỉnh; Phần IV đang được xuất bản.

## Bắt đầu đọc

- [Lời nói đầu — Thư gửi người đọc](chapters/00-loi-noi-dau.md)
- [Chương 1 — Bên dưới một câu trả lời AI có gì?](chapters/01-khoa-target-truoc-khi-toi-uu.md)
- [Chương 2 — GGUF không còn là một file, nó trở thành tensor store](chapters/02-gguf-tensor-store.md)
- [Chương 3 — Xây phần lõi Vulkan](chapters/03-vulkan-runtime-core.md)
- [Chương 4 — Từng phép tính trước, model sau](chapters/04-tung-phep-tinh-truoc-model-sau.md)
- [Chương 5 — Một decoder layer hoàn chỉnh](chapters/05-mot-decoder-layer-hoan-chinh.md)
- [Chương 6 — Full decoder residency: giữ cả “tòa nhà” trên GPU](chapters/06-full-decoder-residency.md)
- [Chương 7 — KV cache: model bắt đầu nhớ token trước](chapters/07-kv-cache-model-bat-dau-nho-token-truoc.md)
- [Chương 8 — Production path không đến từ một kernel thần kỳ](chapters/08-production-path-khong-den-tu-mot-kernel-than-ky.md)
- [Chương 9 — Benchmark phải có đối chứng](chapters/09-benchmark-phai-co-doi-chung.md)
- [Chương 10 — Khi “tự build được” vẫn chưa đủ](chapters/10-khi-tu-build-duoc-van-chua-du.md)
- [Chương 11 — Từ thất bại sang một câu hỏi đúng hơn](chapters/11-tu-that-bai-sang-mot-cau-hoi-dung-hon.md)
- [Chương 12 — Nhiều kiến trúc, nhưng chỉ thực tế mới trả lời được](chapters/12-nhieu-kien-truc-nhung-chi-thuc-te-moi-tra-loi-duoc.md)
- [Chương 13 — Khi correctness nói “không”](chapters/13-khi-correctness-noi-khong.md)
- [Chương 14 — Từ một cơ chế tốt tới hệ thống thật](chapters/14-tu-mot-co-che-tot-toi-he-thong-that.md)
- [Chương 15 — Một kiến trúc chỉ thắng khi toàn hệ được lợi](chapters/15-mot-kien-truc-chi-thang-khi-toan-he-duoc-loi.md)
- [Chương 16 — Experiment 2×2: tách representation khỏi execution](chapters/16-experiment-2x2-tach-representation-khoi-execution.md)
- [Chương 17 — EXEC148: khi bằng chứng buộc một lớp trừu tượng mới xuất hiện](chapters/17-exec148-khi-bang-chung-buoc-mot-lop-truu-tuong-moi-xuat-hien.md)
- [Chương 18 — Dữ liệu ở trong bộ nhớ vẫn chưa đủ: lấy từ đâu và sống bao lâu](chapters/18-du-lieu-o-trong-bo-nho-van-chua-du.md)

## Lộ trình của cuốn sách

### Phần I — Build the Machine

**Mục tiêu:** đi từ con số 0 tới một runtime thực sự chạy được.

Phần này xây từng lớp của ArcLLM: từ cách đọc model, lưu tensor và giao tiếp với GPU bằng Vulkan, tới các phép tính nền tảng, một decoder layer hoàn chỉnh, toàn bộ decoder, KV cache, vòng lặp sinh token và cuối cùng là một production path được chọn bằng đo lường.

**Chương 1–8.**

Kết thúc Phần I, ArcLLM đã chạy được một đường suy luận hoàn chỉnh. Nhưng “chạy được” vẫn chưa trả lời câu hỏi quan trọng hơn: nó đứng ở đâu khi so với một runtime trưởng thành?

### Phần II — Để bằng chứng phán xét

**Mục tiêu:** đặt runtime trước một phép đối chứng cùng điều kiện và chấp nhận kết quả, kể cả khi kết quả đó không có lợi cho kiến trúc mình đã xây.

Phần này đi từ benchmark có đối chứng, tới một phép xác nhận bằng dữ liệu mới, rồi đóng kiến trúc hiện tại khi nó không chứng minh được lợi thế thực tế. Chương cuối của phần này mở ra một nguyên tắc mới: chỉ được nghiên cứu một kiến trúc kế tiếp khi có một cơ chế đủ rõ để kiểm tra.

**Chương 9–11.**

### Phần III — Kiến trúc chỉ có giá trị khi đi qua thực tế

**Mục tiêu:** cho thấy một ý tưởng kỹ thuật chỉ có giá trị khi nó sống sót qua tính đúng, thực nghiệm, quá trình đưa vào hệ thống lớn hơn và tác động ở cấp toàn hệ.

- [**Chương 12 — Nhiều kiến trúc, nhưng chỉ thực tế mới trả lời được**](chapters/12-nhieu-kien-truc-nhung-chi-thuc-te-moi-tra-loi-duoc.md)  
  Nhiều giả thuyết có thể cùng hợp lý trên giấy. Chương này cho thấy cách khám phá rộng nhưng chỉ khóa một cơ chế đủ rõ để đáng tiêu bằng chứng mới.
- [**Chương 13 — Khi correctness nói “không”**](chapters/13-khi-correctness-noi-khong.md)  
  Một cơ chế có thể rất nhanh ở nơi này nhưng vẫn phải dừng nếu không giữ được tính đúng khi chuyển sang miền khác.
- [**Chương 14 — Từ một cơ chế tốt tới hệ thống thật**](chapters/14-tu-mot-co-che-tot-toi-he-thong-that.md)  
  Một thành phần chạy tốt chưa có nghĩa toàn bộ runtime sẽ được lợi. Chương này theo dõi cơ chế Q4 từ fixture tới trọng số/activation thật, semantics toàn model, decode và E2E để kiểm tra carry-through.
- [**Chương 15 — Một kiến trúc chỉ thắng khi toàn hệ được lợi**](chapters/15-mot-kien-truc-chi-thang-khi-toan-he-duoc-loi.md)  
  Khép lại vòng từ giả thuyết tới giá trị end-to-end, benchmark lại với đối chứng bên ngoài và cho thấy vì sao một hệ thống đã thay đổi phải được đo lại trước khi chọn tối ưu tiếp theo.

Trong Phần III, các mode làm việc E/M/C/T được giới thiệu ngay tại những tình huống thực tế đã tạo ra nhu cầu cho chúng, thay vì tách thành một chương quản trị riêng.

### Phần IV — Từ runtime cụ thể tới abstraction tổng quát

**Mục tiêu:** rút ra những ranh giới và khái niệm tổng quát chỉ sau khi thực nghiệm cho thấy runtime thực sự cần chúng.

Dự kiến:

- [**Chương 16 — Experiment 2×2: tách representation khỏi execution**](chapters/16-experiment-2x2-tach-representation-khoi-execution.md)  
  Dùng thiết kế 2×2 để tách hiệu ứng của cách chia công việc khỏi cách biểu diễn dữ liệu, rồi kiểm tra cả tương tác và chi phí kiến trúc.
- [**Chương 17 — EXEC148: khi bằng chứng buộc một lớp trừu tượng mới xuất hiện**](chapters/17-exec148-khi-bang-chung-buoc-mot-lop-truu-tuong-moi-xuat-hien.md)  
  Cùng một tensor logic có thể có nhiều cách biểu diễn phục vụ thực thi; bằng chứng buộc runtime phải tách khái niệm này khỏi riêng kernel và bắt đầu quản lý chi phí, nơi tạo, nơi cư trú và vòng đời.
- [**Chương 18 — Dữ liệu ở trong bộ nhớ vẫn chưa đủ: lấy từ đâu và sống bao lâu**](chapters/18-du-lieu-o-trong-bo-nho-van-chua-du.md)  
  Tách trạng thái cư trú, quá trình tạo/thu nhận, trạng thái sẵn sàng thực thi và vòng đời để runtime không dùng một biến duy nhất cho nhiều câu hỏi khác nhau.
- **Chương 19 — Từ ArcLLM-specific tới runtime abstraction v4**  
  Những ranh giới đã được kiểm tra dần được gom thành một mô hình runtime tổng quát hơn.
- **Chương 20 — Ta đã hiểu runtime đến đâu?**  
  Khép lại Tập 1 bằng những gì đã được chứng minh, những gì chưa được chứng minh và những câu hỏi còn mở.

## Cách cuốn sách được viết

Cuốn sách giữ một số nguyên tắc xuyên suốt:

- **Không yêu cầu biết code.** Code là phương tiện để thực thi nghiên cứu, không phải điều kiện để hiểu câu chuyện.
- **Thuật ngữ kỹ thuật phải được giải thích tại chỗ.** Tiếng Anh được giữ như từ khóa để người đọc có thể tra cứu, nhưng nội dung tiếng Việt phải đủ để hiểu.
- **Mọi con số quan trọng phải có ngữ cảnh.** Công thức và tham số được đi kèm ví dụ đơn giản khi cần.
- **PASS và FAIL đều có giá trị.** Một nhánh thất bại có thể giúp đóng một con đường, xác định giới hạn hoặc đặt ra câu hỏi tốt hơn.
- **Không biến lỗi kỹ thuật thành kết luận khoa học.**
- **Không tuyên bố hiệu năng vượt quá bằng chứng đã đo.**
- **Không kể ngược lịch sử từ đáp án cuối cùng.** Người đọc đi qua các câu hỏi theo đúng thứ tự mà bằng chứng đã buộc dự án phải đi.
- **Một thí nghiệm không mặc nhiên trở thành một chương.** Sách chỉ giữ những bước tạo ra khái niệm mới, thay đổi niềm tin, đóng một con đường hoặc buộc kiến trúc phải thay đổi.

## Các phần bổ sung

Sau 20 chương chính, sách dự kiến có thêm:

- **Bonus — Token-XRay:** một phần đào sâu về công cụ quan sát được tạo ra trong quá trình nghiên cứu để nhìn rõ hơn đường đi và chi phí của token bên trong runtime.
- **Epilogue:** mở sang câu hỏi lớn hơn của Tập 2 — khi đã xây được một hệ thống, liệu ta có thể “lắng nghe” những gì đang xảy ra bên trong nó hay không?
- **Phụ lục A — Một người + AI:** một workflow thực hành cho người không cần biết code nhưng muốn dùng AI để biến câu hỏi thành phép thử có thể kiểm tra và truy vết.

## Về nguồn nghiên cứu

Các số liệu và kết luận kỹ thuật trong sách được biên tập từ những phép thử và tài liệu nghiên cứu của ArcLLM.

Repository này chỉ chứa **nội dung sách đã được xuất bản**. Mã nguồn runtime, dữ liệu thí nghiệm thô và các tài liệu nghiên cứu chi tiết không nằm trong repository sách.

Mục tiêu của Inside ArcLLM không phải chỉ kể rằng một runtime đã được xây như thế nào, mà còn giữ lại **vì sao một hướng được chọn, vì sao một hướng bị loại, và bằng chứng nào đã thay đổi quyết định tiếp theo**.
