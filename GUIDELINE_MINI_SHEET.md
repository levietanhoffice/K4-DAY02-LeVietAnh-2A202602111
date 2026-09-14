# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** LÊ VIỆT ANH<br>
**MSSV:** 2A202602111<br>
**Hình thức:** CÁ NHÂN<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

|  Mã | Lớp              | Gán khi nhìn thấy                                       | Không gán vào lớp này                             |
| --: | ---------------- | ------------------------------------------------------- | ------------------------------------------------- |
|   0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
|   1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng      | ô tô con; thân xe buýt; xe van kín một khối       |
|   2 | `bus` (xe buýt)  | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế           | xe van nhỏ; xe tải; ô tô con                      |
|   3 | `van` (xe van)   | thân hộp nhỏ, kín, dùng chở người hoặc hàng             | thân xe buýt; khoang hàng tách biệt như xe tải    |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính                          | Giá trị                                                 | Ý nghĩa                             |
| ----------------------------------- | ------------------------------------------------------- | ----------------------------------- |
| `visibility` (mức nhìn thấy)        | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy            |
| `boundary` (quan hệ mép ảnh)        | `inside` (trong ảnh), `truncated` (bị cắt)              | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại)      | đánh dấu quyết định cần quay lại    |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: drive_008, xe màu vàng di chuyển ở làn ngoài cùng bên trái.
- Dấu hiệu nhìn thấy: Thân xe dài dạng khối lớn, vóc dáng cao hơn hẳn các xe con xung quanh, dọc sườn có nhiều khung kính hành khách liên tiếp.
- Quy tắc áp dụng: Lớp bus áp dụng cho thân xe khách dài, có nhiều cửa sổ hoặc nhiều hàng ghế; lớp van chỉ áp dụng cho thân xe hộp nhỏ, kín.
- Quyết định: Gán lớp bus (class_id = 2).
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Giữ nguyên bounding box ôm sát phần thân nhìn thấy, tạm đặt review_state = needs_review, phóng to 100% kiểm tra tỷ lệ chiều dài/chiều cao trục bánh và đối chiếu với giáo viên hướng dẫn (Lab Coach) trước khi xuất nhãn.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: drive_033, xe bán tải (pickup) màu trắng đi ở làn trung tâm.
- Dấu hiệu nhìn thấy: Đầu xe và cabin có kiểu dáng bo tròn giống SUV/xe con, phía sau có thùng hở nhỏ chở đồ gia đình, không mang biển hiệu công vụ hay ben tải nặng
- Quy tắc áp dụng: Xe bán tải phục vụ đi lại cá nhân hoặc dùng như xe con thuộc lớp car; chỉ gán truck khi có thùng/ben/sàn hàng hoặc thiết bị công vụ rõ ràng.
- Quyết định: Gán lớp car (class_id = 0).
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Kiểm tra độ tách biệt giữa cabin lái và khoang hàng; nếu không có kết cấu chassis tách rời chuyên chở hàng hóa thương mại thì không suy diễn sang truck mà giữ nhãn car.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: drive_038, phần đầu một chiếc sedan màu bạc ở sát góc dưới mép phải ảnh.
- Dấu hiệu nhìn thấy khi phóng 100%: Chỉ thấy rõ nắp capo, một bên đèn pha và một phần bánh trước; phần thân và đuôi xe đã nằm ngoài khung hình do mép ảnh cắt ngang.
- Giá trị visibility: clear (phần nằm trong khung hình nhìn rất sắc nét, không bị xe khác đè lên).
- Giá trị boundary: truncated (vật thể bị đường viền mép ảnh cắt ngang).
- Trạng thái review_state: confident (các đường nét đèn pha và nắp capo đủ đặc trưng để xác định là xe con).
- Lý do: Vật thể chạm mép ảnh vẫn được gán hộp khi đã đủ bằng chứng phân lớp, đặt boundary = truncated và chỉ vẽ bounding box khép kín phần điểm ảnh thực tế hiển thị.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 52 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
