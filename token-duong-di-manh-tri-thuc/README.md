# TOKEN — Đường đi của mảnh tri thức

> **Bản thảo độc lập**
>
> Cuốn sách này được viết để có thể đọc từ đầu mà không cần một cuốn sách khác làm điều kiện tiên quyết.

Ta chọn **một token** và đi cùng nó.

Câu hỏi xuyên suốt là:

> **Khi một mảnh văn bản đi vào mô hình, điều gì thật sự được mang theo, điều gì được tạo ra trên đường đi, và tới đâu ta mới có quyền gọi thứ mình quan sát là “tri thức”?**

Tên sách cố ý dùng cụm **“mảnh tri thức”** như một câu hỏi, không phải một kết luận.

Ngay từ đầu, sách sẽ làm rõ:

~~~text
token ≠ từ
token ≠ ý nghĩa cố định
token ≠ một viên tri thức được đóng gói sẵn
~~~

## Cách cuốn sách được viết

Cuốn sách dùng cùng một kỷ luật biên tập xuyên suốt:

- **Tiếng Việt là ngôn ngữ chính.**
- Thuật ngữ chuyên ngành được giới thiệu theo dạng **tiếng Việt (English)** khi có cách dịch rõ ràng, ví dụ **cách biểu diễn (representation)**, **chương trình GPU (kernel)**.
- Với những từ đã trở thành tên gọi phổ biến như **token**, sách giải thích nghĩa trước rồi giữ nguyên từ chuyên ngành để người đọc nhận ra nó khi đọc tài liệu khác.
- **Ý tưởng trước, tên gọi sau.**
- **Ví dụ gần gũi trước mô hình trừu tượng.**
- **Không dùng trước khi dạy.**
- Có thể bắt đầu bằng một cách hiểu đơn giản nhưng đúng hướng, sau đó mới sửa dần cho chính xác hơn.
- Mỗi chương chỉ mở thêm một số ít khái niệm mới.
- Sơ đồ đầu chương chỉ chứa những gì người đọc đã đủ nền để hiểu.
- Cuối mỗi chương có liên kết sang chương tiếp theo.
- Số liệu thực tế chỉ được dùng đúng phạm vi bằng chứng.
- Quan sát không được viết thành nguyên nhân nếu chưa có bằng chứng nhân quả.
- Kết quả **KHÔNG ĐẠT** hoặc **CHƯA KẾT LUẬN ĐƯỢC** vẫn có giá trị nếu chúng làm rõ giới hạn của điều ta biết.

## Cuốn sách có tự đứng được không?

Có.

Khi một khái niệm như mô hình, Transformer, CPU, GPU, khối số hay hệ thực thi xuất hiện lần đầu ở nơi nó thật sự cần thiết, sách phải giải thích đủ để người đọc tiếp tục mà không phải mở một tài liệu khác.

Một cuốn khác của cùng tác giả có thể đi sâu hơn vào lớp hệ thực thi, nhưng đó chỉ là tài liệu đọc thêm, **không phải điều kiện để hiểu cuốn này**.

## Bản đồ của cuốn sách

Bản đồ này chỉ cho thấy hướng đi tổng thể. Các chương sẽ mở từng phần một, không ném toàn bộ thuật ngữ vào người đọc ngay từ đầu.

~~~text
Văn bản
   ↓
token
   ↓
mã token
   ↓
biểu diễn bằng số
   ↓
ngữ cảnh làm biểu diễn thay đổi
   ↓
nhiều lớp Transformer
   ↓
trạng thái cuối
   ↓
điểm cho token tiếp theo
   ↓
token mới

Song song:

phép tính logic
   ↓
công việc vật lý trên phần cứng
   ↓
dấu vết thực thi
   ↓
phép đo
   ↓
bằng chứng
   ↓
diễn giải
   ↓
điều ta thật sự biết
~~~

## Lộ trình dự kiến

### Phần I — Một token thực sự là gì?
**Mức đọc: Nền tảng**

1. **Token không phải là một từ**
2. **Token cũng không phải là tri thức**
3. **Từ mã token tới một biểu diễn bằng số**
4. **Cùng một token, ngữ cảnh khác, biểu diễn khác**

### Phần II — Điều gì xảy ra khi token đi qua mô hình?
**Mức đọc: Đi sâu**

5. **Dòng tín hiệu đi xuyên Transformer**
6. **Cơ chế chú ý: token nhìn những token khác như thế nào?**
7. **Nhánh biến đổi tín hiệu: điều gì xảy ra sau cơ chế chú ý?**
8. **Đường cộng tắt: vì sao thông tin cũ không biến mất hoàn toàn?**
9. **Một lớp chưa biết cả câu — vậy nhiều lớp làm được gì?**
10. **Từ trạng thái cuối tới token tiếp theo**

### Phần III — Khi phép toán biến thành công việc thật
**Mức đọc: Đi sâu**

11. **Một phép tính logic không phải một chương trình GPU**
12. **Một phép tính có thể trở thành nhiều lần giao việc cho phần cứng**
13. **Khi nhiều phép tính được gộp thành một công việc**
14. **Bộ nhớ cũng nằm trên đường đi của token**

### Phần IV — Ta thật sự quan sát được gì?
**Mức đọc: Nghiên cứu**

15. **Dấu vết thực thi: những dấu chân một token để lại**
16. **Thời gian nằm ở đâu?**
17. **Bộ đếm phần cứng: con số đo được có nghĩa gì?**
18. **Khi một khác biệt quá nhỏ vẫn đáng để hỏi**

### Phần V — Từ quan sát tới tri thức
**Mức đọc: Nâng cao**

19. **Một khuôn mẫu đẹp vẫn chưa phải nguyên nhân**
20. **Từ phép đo tới điều ta thật sự biết**

## Nguồn của các ví dụ thực nghiệm

Cuốn sách sử dụng các **kết quả khoa học đã được kiểm tra** từ những thí nghiệm thật của tác giả để minh họa khi cần.

Ví dụ có thể gồm:

- một bước sinh token được tách thành hàng trăm công việc vật lý;
- phần lớn thời gian có thể tập trung vào một số ít họ phép tính;
- một bộ đếm phần cứng có thể trả về 0 nhưng kênh đo vẫn không đủ để kết luận đại lượng vật lý thật sự bằng 0.

Các kết quả này chỉ được dùng để dạy một nguyên lý tổng quát.

Cuốn sách **không công bố tên công cụ nghiên cứu, kiến trúc sản phẩm, lịch sử phát triển, nhánh nghiên cứu hay cơ chế nội bộ** đứng sau việc thu thập bằng chứng.

## Bản thảo hiện có

**Lời mở đầu và Chương 1–20 đã được viết đầy đủ theo bản thảo vòng 1.**

- [Chương 4 — Cùng một token, ngữ cảnh khác, biểu diễn khác](04-cung-token-ngu-canh-khac.md)
- [Chương 5 — Dòng tín hiệu đi xuyên Transformer](05-dong-tin-hieu-di-xuyen-transformer.md)
- [Chương 6 — Cơ chế chú ý: token nhìn những token khác như thế nào?](06-co-che-chu-y.md)
- [Chương 7 — Nhánh biến đổi tín hiệu: điều gì xảy ra sau cơ chế chú ý?](07-nhanh-bien-doi-tin-hieu.md)
- [Chương 8 — Đường cộng tắt: vì sao thông tin cũ không biến mất hoàn toàn?](08-duong-cong-tat.md)
- [Chương 9 — Một lớp chưa biết cả câu — vậy nhiều lớp làm được gì?](09-nhieu-lop-lam-duoc-gi.md)
- [Chương 10 — Từ trạng thái cuối tới token tiếp theo](10-tu-trang-thai-cuoi-toi-token-tiep-theo.md)
- [Chương 11 — Một phép tính logic không phải một chương trình GPU](11-phep-tinh-logic-khong-phai-kernel.md)
- [Chương 12 — Một phép tính có thể trở thành nhiều lần giao việc cho phần cứng](12-mot-phep-tinh-nhieu-lan-giao-viec.md)
- [Chương 13 — Khi nhiều phép tính được gộp thành một công việc](13-khi-nhieu-phep-tinh-duoc-gop.md)
- [Chương 14 — Bộ nhớ cũng nằm trên đường đi của token](14-bo-nho-tren-duong-di-token.md)
- [Chương 15 — Dấu vết thực thi: những dấu chân một token để lại](15-dau-vet-thuc-thi.md)
- [Chương 16 — Thời gian nằm ở đâu?](16-thoi-gian-nam-o-dau.md)
- [Chương 17 — Bộ đếm phần cứng: con số đo được có nghĩa gì?](17-bo-dem-phan-cung.md)
- [Chương 18 — Khi một khác biệt quá nhỏ vẫn đáng để hỏi](18-khac-biet-nho-van-dang-hoi.md)
- [Chương 19 — Một khuôn mẫu đẹp vẫn chưa phải nguyên nhân](19-khuon-mau-dep-chua-phai-nguyen-nhan.md)
- [Chương 20 — Từ phép đo tới điều ta thật sự biết](20-tu-phep-do-toi-dieu-ta-biet.md)

- [Kế hoạch biên tập đã dùng để triển khai Chương 4–20](DRAFT_PLAN.md)
