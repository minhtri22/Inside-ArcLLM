# Chương 12 — Một phép tính có thể trở thành nhiều lần giao việc cho phần cứng

> **Mức đọc: Đi sâu**
>
> **Ta đang mở quan hệ này:**
>
> ~~~text
> 1 phép tính logic
>         ↓
> [ chia nhỏ để thực thi ]
>         ↓
> nhiều lần giao việc vật lý
> ~~~
>
> Chương này không hỏi phép tính “nghĩa là gì”. Nó hỏi một câu thấp hơn: **vì sao một công việc logic có thể phải được chia thành nhiều phần khi chạy thật?**

Hãy tưởng tượng bạn cần sơn một bức tường rất lớn.

Trong kế hoạch, đó là một công việc:

~~~text
Sơn bức tường phía đông.
~~~

Nhưng khi làm thật, bạn có thể chia:

~~~text
ô 1 | ô 2 | ô 3 | ô 4 | ...
~~~

Nhiều người cùng làm.

Hoặc một người làm từng phần.

Công việc logic vẫn là “sơn bức tường”.

Cách thực thi lại được chia thành nhiều phần vật lý.

## GPU cũng cần chia công việc

GPU có rất nhiều đơn vị tính toán hoạt động song song.

Để tận dụng chúng, một phép tính lớn thường được chia theo một **hình học thực thi (execution geometry)**.

Có thể hiểu đơn giản:

> **Hình học thực thi là cách ta chia một bài toán lớn thành những nhóm công việc nhỏ hơn để phần cứng xử lý.**

Ví dụ một ma trận lớn có thể được chia thành các khối:

~~~text
┌────┬────┬────┬────┐
│ A1 │ A2 │ A3 │ A4 │
├────┼────┼────┼────┤
│ B1 │ B2 │ B3 │ B4 │
├────┼────┼────┼────┤
│ C1 │ C2 │ C3 │ C4 │
└────┴────┴────┴────┘
~~~

Mỗi khối có thể trở thành một phần của công việc GPU.

Cách chia cụ thể phụ thuộc thuật toán, phần cứng, kích thước dữ liệu và cách hệ thực thi được viết.

## Lớp đầu ra là một ví dụ rất rõ

Ở Chương 10, ta đã gặp **lớp tạo điểm đầu ra (LM head)**.

Nó phải tạo điểm cho rất nhiều token trong từ vựng.

Về logic:

~~~text
trạng thái cuối
↓
LM head
↓
logits cho toàn bộ từ vựng
~~~

Trong một phép đo thật, LM head là:

~~~text
1 phép tính logic
~~~

nhưng được thực thi thành:

~~~text
19 lần giao việc vật lý
~~~

Ta có:

~~~text
LM head
  │
  ├→ dispatch 1
  ├→ dispatch 2
  ├→ dispatch 3
  │     ...
  └→ dispatch 19
~~~

Đây là bằng chứng trực tiếp cho một quan hệ một-nhiều.

## Có phải vì từ vựng bị chia thành 19 phần?

Không nên tự suy ra điều đó chỉ từ con số 19.

Có nhiều lý do một phép tính có thể được chia:

- giới hạn kích thước mỗi lần xử lý;
- cách chia hàng hoặc cột;
- cách thuật toán phân công công việc;
- cách hệ thực thi tổ chức bộ nhớ;
- giới hạn hoặc lựa chọn của kernel;
- cách tối ưu cho phần cứng cụ thể.

Nếu chỉ nhìn dấu vết và thấy:

~~~text
1 phép tính logic
→
19 dispatch
~~~

ta biết **quan hệ ánh xạ**.

Ta chưa chắc đã biết **nguyên nhân thiết kế** tạo ra ánh xạ đó.

Đây là một ví dụ rất quan trọng:

> **Nhìn thấy cấu trúc thực thi chưa đồng nghĩa đã giải thích được vì sao cấu trúc đó tồn tại.**

## Chia nhỏ có phải lúc nào cũng xấu?

Không.

Nhiều dispatch hơn không tự động nghĩa chậm hơn.

Chia nhỏ có thể giúp:

- tận dụng song song tốt hơn;
- phù hợp giới hạn phần cứng;
- giảm kích thước vùng làm việc;
- cho phép xử lý dữ liệu lớn hơn;
- tạo điều kiện tái sử dụng tài nguyên.

Nhưng chia quá nhỏ cũng có thể tạo thêm chi phí:

- nhiều lần gửi công việc;
- nhiều điểm đồng bộ;
- nhiều lần đọc / ghi dữ liệu;
- nhiều khoảng trống giữa các công việc.

Ta không thể kết luận chỉ từ số lượng dispatch.

## Một ví dụ bằng vận chuyển

Giả sử cần chuyển 10 tấn hàng.

Phương án A:

~~~text
1 xe rất lớn
× 1 chuyến
~~~

Phương án B:

~~~text
5 xe nhỏ
× 2 chuyến
~~~

Không thể nhìn:

~~~text
1 chuyến < 10 chuyến
~~~

rồi kết luận phương án A chắc chắn nhanh hơn.

Còn phụ thuộc tải trọng, đường, thời gian xếp hàng, khả năng chạy song song và chi phí chờ.

GPU cũng vậy.

Số dispatch là một phần của bức tranh, không phải phán quyết.

## Phân rã vật lý

Ta có thể gọi việc biến một phép tính logic thành nhiều công việc thực thi là **phân rã vật lý (physical decomposition)**.

~~~text
PHÉP TÍNH LOGIC
      │
      │ một danh tính
      ↓
┌────────────────────────┐
│ phân rã để thực thi    │
└────────────────────────┘
      │
      ├→ công việc 1
      ├→ công việc 2
      ├→ công việc 3
      └→ ...
~~~

Điều này giúp ta hiểu vì sao cần giữ hai loại danh tính riêng.

Nếu không, ta có thể nhìn 19 dispatch và tưởng có 19 phép tính logic khác nhau.

## Khi cộng thời gian thì cộng thế nào?

Giả sử một phép tính logic được chia thành ba dispatch:

~~~text
dispatch A: 2 ms
dispatch B: 3 ms
dispatch C: 1 ms
~~~

Nếu chúng chạy tuần tự và tất cả đều thuộc riêng phép tính đó, ta có thể nghĩ tới tổng:

~~~text
2 + 3 + 1 = 6 ms
~~~

Nhưng hệ thống thật có thể có chồng lấn, đồng bộ hoặc nhiều cách đo khác nhau.

Vì vậy việc “cộng thời gian” cũng cần một quy tắc rõ.

Trong phép đo dùng làm ví dụ ở cuốn sách này, các dispatch mục tiêu được gắn dấu thời gian và quy chiếu về phép tính logic theo một quy tắc đo đã khóa trước.

Điều đó cho phép ta nói thời gian được **quy chiếu** về đâu.

Nó vẫn không biến mọi khoảng trống hệ thống thành thời gian của riêng phép tính đó.

## Và chiều ngược lại?

Ta vừa thấy:

~~~text
1 phép tính logic
→
nhiều dispatch
~~~

Nhưng liệu có thể có:

~~~text
nhiều phép tính logic
→
1 dispatch
~~~

Có.

Khi hệ thống **gộp phép tính (fusion)**, nhiều công việc logic có thể được thực hiện trong cùng một chương trình GPU.

Đó là Chương 13.

### Nhớ 3 điều

1. **Một phép tính logic lớn có thể được phân rã thành nhiều lần giao việc vật lý để phù hợp với cách GPU thực thi.**
2. **Trong một phép đo thật, một LM head logic được thực thi bằng 19 dispatch; con số này chứng minh quan hệ một-nhiều trong hệ thống đó, không phải quy luật cho mọi mô hình.**
3. **Nhiều dispatch hơn không tự động nghĩa chậm hơn, và thấy một phép tính bị chia chưa tự động giải thích vì sao hệ thực thi chọn cách chia đó.**

**Tiếp theo: [Chương 13 — Khi nhiều phép tính được gộp thành một công việc](13-khi-nhieu-phep-tinh-duoc-gop.md)**
