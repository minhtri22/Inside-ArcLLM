# Inside ArcLLM

**Từ con số 0 đến một runtime mà ta hiểu được từng lớp bên trong.**

Inside ArcLLM là một cuốn sách tiếng Việt dành cho người bắt đầu từ số 0 — không giả định biết AI, hệ thống máy tính hay lập trình.

Cuốn sách dùng hành trình xây dựng ArcLLM như một câu chuyện thật để theo đuổi hai mục tiêu:

1. **Hiểu nền tảng AI** — từ ứng dụng AI, model, token, runtime, tensor, CPU/GPU và bộ nhớ tới cách một runtime được xây dựng, kiểm tra và tối ưu.
2. **Học cách làm việc và quản trị cùng AI** — đặt câu hỏi, khóa phạm vi, yêu cầu bằng chứng, phân biệt PASS/FAIL, giữ lịch sử và duy trì quyền quyết định của con người.

## Trạng thái

Bản đang viết công khai.

Nội dung chỉ được đưa vào repository này sau khi đã qua review biên tập. Repository nghiên cứu và mã nguồn ArcLLM không nằm trong repository sách này.

## Đọc sách

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

## Nguyên tắc biên tập

- Viết cho người không biết code.
- Thuật ngữ kỹ thuật được giải thích bằng tiếng Việt trước hoặc ngay tại điểm sử dụng.
- PASS và FAIL đều là kiến thức.
- Không biến lỗi hạ tầng thành kết luận khoa học.
- Không tuyên bố hiệu năng ngoài bằng chứng.
- Không kể ngược lịch sử từ kiến trúc cuối cùng.
- Quản trị nghiên cứu, thực thi và hạ tầng được phân biệt rõ.
- Code là phương tiện thực thi, không phải điều kiện để hiểu sách.

## Về nguồn nghiên cứu

Các số liệu và mốc kỹ thuật trong sách được biên tập từ một research source-of-truth riêng. Repository công khai này chỉ chứa nội dung sách đã được phê duyệt, không chứa source code runtime, raw artifacts hay lịch sử nghiên cứu nội bộ.

Khi nguồn nghiên cứu gốc được công khai trong tương lai, các liên kết provenance có thể được bổ sung mà không thay đổi lịch sử biên tập của cuốn sách.
