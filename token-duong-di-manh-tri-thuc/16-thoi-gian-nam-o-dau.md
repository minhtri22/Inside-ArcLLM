# Chương 16 — Thời gian nằm ở đâu?

> **Mức đọc: Nghiên cứu**
>
> **Ta đã có:**
>
> ~~~text
> dispatch
> + timestamp
> + danh tính logic
>       ↓
> [ cộng thời gian theo từng họ phép tính ]
>       ↓
> điểm nóng
> ~~~
>
> Chương này trả lời một câu hỏi rất cụ thể: **trong một bước sinh token được đo, phần nào đang chiếm nhiều thời gian nhất?**

Hãy tưởng tượng một ngày làm việc dài 8 giờ.

Nếu muốn tối ưu thời gian, ta có thể ghi:

~~~text
họp          3 giờ
viết báo cáo 2 giờ
đọc email    1 giờ
di chuyển    1 giờ
khác         1 giờ
~~~

Ta lập tức biết nơi thời gian tập trung.

Trong hệ thống tính toán, các vùng như vậy được gọi là **điểm nóng (hotspot)**.

## Từ 469 dispatch tới vài nhóm dễ hiểu hơn

Trace ở Chương 15 có 469 dispatch.

Đọc từng dispatch riêng lẻ sẽ rất khó.

Nhưng khi quy chiếu chúng về các họ phép tính logic, ta có thể cộng lại.

Trong phép đo đó, bốn họ lớn nhất là:

~~~text
LM head   63,9109%
FFN-down  16,2659%
FFN-up     6,0136%
FFN-gate   5,9812%
~~~

Tổng cộng:

~~~text
92,1715%
~~~

Các tỷ lệ trên là tỷ lệ trong **tổng thời gian dispatch đã đo** của trace đó.

Không phải tuyên bố về mọi lần chạy hay mọi mô hình.

## Nhìn bằng một biểu đồ ASCII

Ta có thể làm tròn để nhìn trực giác:

~~~text
LM head    |████████████████████████████████| ~63,9%
FFN-down   |████████                        | ~16,3%
FFN-up     |███                             | ~ 6,0%
FFN-gate   |███                             | ~ 6,0%
khác       |████                            | ~ 7,8%
~~~

Bức tranh rất rõ:

> **Thời gian không phân bố đều.**

Một nhóm rất nhỏ các họ phép tính chiếm phần lớn tổng thời gian dispatch.

## Đây gọi là định vị chi phí

Việc tìm:

> thời gian đang nằm ở đâu?

có thể gọi là **định vị chi phí (localization)**.

Ta có thể viết:

~~~text
toàn bộ đường sinh token
        ↓
đo thời gian
        ↓
nhóm theo operation
        ↓
xếp theo tỷ trọng
        ↓
hotspot
~~~

Đây là một thành tựu quan trọng.

Nếu không biết hotspot ở đâu, ta dễ tối ưu nhầm.

## Nhưng 63,9% có nghĩa gì?

Nó có nghĩa:

> Trong trace cụ thể này, các dispatch được quy chiếu về LM head chiếm khoảng 63,9% tổng dispatch time.

Nó **không** tự động có nghĩa:

> LM head luôn chiếm 63,9% trên mọi máy.

Nó cũng không có nghĩa:

> LM head chậm vì bộ nhớ.

Và không có nghĩa:

> Nếu xóa LM head thì hệ thống nhanh hơn đúng 63,9%.

Con số mô tả một phép đo cụ thể.

## Hotspot có phải nguyên nhân không?

Không.

Đây là điểm trung tâm của chương.

Giả sử bạn thấy:

~~~text
đường X thường xuyên tắc xe
~~~

Bạn đã **định vị** được nơi có vấn đề.

Nhưng nguyên nhân có thể là:

- đèn giao thông;
- công trường;
- đường hẹp;
- tai nạn;
- giờ cao điểm;
- nhiều xe rẽ trái.

Biết “tắc ở đường X” chưa cho biết “vì sao đường X tắc”.

Tương tự:

~~~text
LM head = 63,9%
~~~

cho biết nơi thời gian tập trung.

Nó chưa nói cơ chế gây chi phí.

## Vì sao localization vẫn rất có giá trị?

Bởi nó thu hẹp không gian tìm kiếm.

Trước trace:

~~~text
hàng trăm công việc
↓
không biết nên nhìn đâu
~~~

Sau trace:

~~~text
một vài họ chiếm ~92,17%
↓
có thể ưu tiên điều tra
~~~

Đó là khác biệt rất lớn.

Ta không có nguyên nhân.

Nhưng ta có một **bản đồ ưu tiên**.

## Amdahl xuất hiện ở đây

Có một trực giác rất hữu ích.

Nếu một phần chỉ chiếm 1% tổng thời gian, dù tối ưu nó vô hạn, tổng hệ thống cũng chỉ cải thiện rất ít.

Ngược lại, nếu một phần chiếm phần lớn thời gian, nó có nhiều **dư địa ảnh hưởng** tới toàn hệ hơn.

Ý tưởng này liên quan tới **định luật Amdahl (Amdahl's law)**.

Ta không cần công thức đầy đủ ở đây.

Chỉ cần nhớ:

> **Tối ưu một phần chỉ có thể giúp toàn hệ trong phạm vi tỷ trọng thời gian mà phần đó thực sự chiếm.**

Nhưng ngay cả hotspot lớn cũng không bảo đảm tối ưu được dễ dàng.

## Điểm nóng có thể thay đổi

Giả sử ta tối ưu LM head rất mạnh.

Bức tranh sau tối ưu có thể trở thành:

~~~text
trước:
LM head   64%
FFN-down  16%
khác      20%

sau:
LM head   25%
FFN-down  35%
khác      40%
~~~

Nút thắt cũ đã thay đổi.

Vì vậy một hotspot map có **thời hạn**.

Sau thay đổi lớn, phải đo lại.

## Từ “ở đâu” sang “vì sao”

Bây giờ ta có một mục tiêu cụ thể:

~~~text
LM head
FFN-down
...
~~~

Muốn hiểu cơ chế, ta có thể hỏi phần cứng:

- có đang đọc nhiều dữ liệu không?
- cache có trúng không?
- đơn vị tính toán có bận không?
- có đang chờ phụ thuộc không?
- tần số có ổn định không?

Một nguồn trả lời là **bộ đếm phần cứng (hardware counters)**.

Nhưng các bộ đếm này cũng có thể thất bại.

Thậm chí chúng có thể trả về số 0.

Đó là câu chuyện của Chương 17.

### Nhớ 3 điều

1. **Điểm nóng (hotspot) là nơi thời gian tập trung; trong trace cụ thể này, LM head ~63,91%, FFN-down ~16,27%, FFN-up ~6,01%, FFN-gate ~5,98%, tổng bốn họ ~92,17% dispatch time.**
2. **Định vị (localization) trả lời “ở đâu”, không trả lời “vì sao”.** Một hotspot lớn chưa phải bằng chứng về cơ chế gây chậm.
3. **Bản đồ hotspot có thể trở nên lỗi thời sau khi hệ thống thay đổi; muốn tiếp tục tối ưu phải đo lại.**

**Tiếp theo: [Chương 17 — Bộ đếm phần cứng: con số đo được có nghĩa gì?](17-bo-dem-phan-cung.md)**
