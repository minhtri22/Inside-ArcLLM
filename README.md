# Inside ArcLLM

**Từ con số 0 đến một runtime mà ta hiểu được từng lớp bên trong.**

Inside ArcLLM là một cuốn sách tiếng Việt dành cho người bắt đầu từ số 0 — không giả định biết AI, hệ thống máy tính hay lập trình.

Cuốn sách dùng hành trình xây dựng ArcLLM như một câu chuyện thật để theo đuổi hai mục tiêu:

1. **Hiểu nền tảng AI** — từ ứng dụng AI, model, token, runtime, tensor, CPU/GPU và bộ nhớ tới cách một runtime được xây dựng, kiểm tra và tối ưu.
2. **Học cách làm việc và quản trị cùng AI** — đặt câu hỏi, khóa phạm vi, yêu cầu bằng chứng, phân biệt PASS/FAIL, giữ lịch sử và duy trì quyền quyết định của con người.

## Trạng thái

Bản đang viết công khai.

Nội dung chỉ được đưa vào repository này sau khi đã qua review biên tập. Repository nghiên cứu và mã nguồn ArcLLM không nằm trong repository sách này.

Outline chính đã được chốt ở **20 chương / 4 phần**. Chương 1–11 đã được tác giả duyệt; Chương 12–20 hiện mới là outline và chỉ được viết sau khi chương trước được duyệt.

## Cấu trúc 20 chương

### Phần I — Build the Machine

**Mục tiêu:** đi từ con số 0 tới một runtime thực sự chạy được; xây từng lớp từ dữ liệu model, Vulkan và các phép tính cơ bản tới decoder, KV cache và production path.

- [Chương 1 — Bên dưới một câu trả lời AI có gì?](chapters/01-khoa-target-truoc-khi-toi-uu.md)
- [Chương 2 — GGUF không còn là một file, nó trở thành tensor store](chapters/02-gguf-tensor-store.md)
- [Chương 3 — Xây phần lõi Vulkan](chapters/03-vulkan-runtime-core.md)
- [Chương 4 — Từng phép tính trước, model sau](chapters/04-tung-phep-tinh-truoc-model-sau.md)
- [Chương 5 — Một decoder layer hoàn chỉnh](chapters/05-mot-decoder-layer-hoan-chinh.md)
- [Chương 6 — Full decoder residency: giữ cả “tòa nhà” trên GPU](chapters/06-full-decoder-residency.md)
- [Chương 7 — KV cache: model bắt đầu nhớ token trước](chapters/07-kv-cache-model-bat-dau-nho-token-truoc.md)
- [Chương 8 — Production path không đến từ một kernel thần kỳ](chapters/08-production-path-khong-den-tu-mot-kernel-than-ky.md)

**Kết thúc Phần I:** ArcLLM đã có một production path được chọn bằng evidence, nhưng chưa được phép kết luận rằng nó cạnh tranh được với một runtime trưởng thành.

### Phần II — Để evidence phán xét

**Mục tiêu:** đặt runtime trước phép đối chứng cùng điều kiện, để bằng chứng quyết định thay vì tiếp tục bảo vệ kiến trúc đã xây.

- [Chương 9 — Benchmark phải có đối chứng](chapters/09-benchmark-phai-co-doi-chung.md)
- [Chương 10 — Khi “tự build được” vẫn chưa đủ](chapters/10-khi-tu-build-duoc-van-chua-du.md)
- [Chương 11 — Từ thất bại sang một câu hỏi đúng hơn](chapters/11-tu-that-bai-sang-mot-cau-hoi-dung-hon.md)

**Kết thúc Phần II:** matched benchmark và fresh confirmation không chứng minh được practical advantage của kiến trúc hiện tại; kiến trúc cũ đóng, và một kiến trúc kế tiếp chỉ được mở khi có một cơ chế mới đủ rõ để kiểm tra.

### Phần III — Kiến trúc chỉ có giá trị khi đi qua thực tế

**Mục tiêu:** cho thấy có thể tồn tại nhiều giả thuyết và nhiều kiến trúc hợp lý trên giấy, nhưng chỉ thực nghiệm, transfer và giá trị ở cấp toàn hệ mới quyết định thứ gì đáng giữ. Đây cũng là nơi các mode quản trị cùng AI được đưa vào đúng ngữ cảnh thực tế.

- **Chương 12 — Nhiều kiến trúc, nhưng chỉ thực tế mới trả lời được**  
  Nhiều giả thuyết, nhiều cách phân rã công việc; Mode E → Mode M giúp AI sinh phương án nhưng không tiêu fresh evidence cho mọi ý tưởng.
- **Chương 13 — Khi correctness nói “không”**  
  Giữ correctness như một contract độc lập; Mode C cho thấy nhanh hơn nhưng sai ngoài ngưỡng đã khóa vẫn là FAIL.
- **Chương 14 — Từ một cơ chế tốt tới hệ thống thật**  
  Carry-through / transfer: component PASS không tự động trở thành system PASS; Mode T kiểm tra lợi ích có sống sót khi đi vào đường thực thi lớn hơn hay không.
- **Chương 15 — Một kiến trúc chỉ thắng khi toàn hệ được lợi**  
  Amdahl, headroom, end-to-end movement, stop rule và quyền quyết định của con người trong một dự án cùng AI.

**Kết thúc Phần III:** một người + AI không nghiên cứu bằng cách để AI thử vô hạn. Con người đặt câu hỏi và ranh giới; AI giúp sinh phương án, triển khai và tổng hợp evidence; PASS/FAIL cùng stop rule quyết định con đường tiếp theo.

### Phần IV — Từ runtime cụ thể tới abstraction tổng quát

**Mục tiêu:** rút ra những abstraction chỉ xuất hiện sau khi evidence cho thấy runtime cần chúng, thay vì thiết kế một kiến trúc đẹp trên giấy từ trước.

- **Chương 16 — Experiment 2×2: tách representation khỏi execution**  
  Tách hai biến để biết chính xác thay đổi nào tạo ra hiệu ứng.
- **Chương 17 — EXEC148: khi evidence buộc abstraction mới xuất hiện**  
  Representation và execution được tách khi bằng chứng buộc runtime phải có ranh giới mới.
- **Chương 18 — Residency chưa đủ: acquisition và lifecycle**  
  Không chỉ hỏi dữ liệu có ở trong memory hay không, mà còn hỏi nó đến từ đâu, sẵn sàng khi nào, sống bao lâu và ai sở hữu.
- **Chương 19 — Từ ArcLLM-specific tới runtime abstraction v4**  
  Identity, execution availability/readiness, residency, acquisition và lifecycle được gom thành một mô hình runtime tổng quát hơn.
- **Chương 20 — Ta đã hiểu runtime đến đâu?**  
  Backend validation/integration, những gì đã được chứng minh, những gì vẫn còn mở và ranh giới của Tập 1.

Trong Phần IV, **Token-XRay chỉ xuất hiện nhẹ như một công cụ đo nội bộ được sinh ra khi chuỗi nghiên cứu cần nhìn sâu hơn vào đường đi và chi phí của token**; nó không có một chương chính riêng.

## Ngoài 20 chương chính

- [Lời nói đầu — Thư gửi người đọc](chapters/00-loi-noi-dau.md)
- **Bonus — Token-XRay**, chỉ nếu cần một phần đào sâu cho độc giả kỹ thuật hơn.
- **Epilogue — từ runtime sang “lắng nghe hệ thống”**, dẫn sang hướng SIX/RF của Tập 2.
- **Phụ lục A — Một người + AI**, biến các nguyên tắc quản trị đã xuất hiện trong Chương 12–15 thành workflow thực hành cho người không cần biết code.

## Nguyên tắc biên tập

- Viết cho người không biết code.
- Thuật ngữ kỹ thuật được giải thích bằng tiếng Việt trước hoặc ngay tại điểm sử dụng; tiếng Anh chỉ là từ khóa để tra cứu, không phải điều kiện để hiểu sách.
- Khi một thuật ngữ quay lại sau một khoảng đọc dài, nhắc lại nghĩa ngắn gọn tại chỗ nếu cần.
- Mọi công thức hoặc tham số quan trọng phải có ví dụ số đơn giản.
- PASS và FAIL đều là kiến thức.
- Không biến lỗi hạ tầng thành kết luận khoa học.
- Không tuyên bố hiệu năng ngoài bằng chứng.
- Không kể ngược lịch sử từ kiến trúc cuối cùng.
- Quản trị nghiên cứu, thực thi và hạ tầng được phân biệt rõ.
- Code là phương tiện thực thi, không phải điều kiện để hiểu sách.
- Một experiment không mặc nhiên xứng đáng thành một chương; một chương chỉ tồn tại khi nó tạo khái niệm mới, thay đổi niềm tin, đóng một con đường hoặc buộc kiến trúc phải thay đổi.

## Về nguồn nghiên cứu

Các số liệu và mốc kỹ thuật trong sách được biên tập từ một research source-of-truth riêng. Repository công khai này chỉ chứa nội dung sách đã được phê duyệt, không chứa source code runtime, raw artifacts hay lịch sử nghiên cứu nội bộ.

Khi nguồn nghiên cứu gốc được công khai trong tương lai, các liên kết provenance có thể được bổ sung mà không thay đổi lịch sử biên tập của cuốn sách.
