# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** LÊ VIỆT ANH<br>
**MSSV:** 2A202602111<br>
**Hình thức:** CÁ NHÂN<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33
- Bốn mã ảnh: drive_008, drive_022, drive_033, drive_038
- Số vật thể thực tế: 52
- Mã SHA-256 của gói YOLO của bạn: 234e6726f08e934a083ec98bc6e23b412427d72495af8873e1639b3a3e543ce9
- Mã SHA-256 của gói CVAT gốc của bạn: 2eb255929fd10467a3f8d0bc33a4c456473a38b70cb36bf4909eb56db63e7591
- Nguồn đối chiếu: bộ tham chiếu do người hướng dẫn thực hành cấp
- Mã SHA-256 của gói đối chiếu: c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: lần phát đợt 1 (Batch 1), nhận lúc 15h41

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

- Quá trình hoàn thiện gán nhãn trên CVAT và trích xuất cả hai gói dữ liệu (Ultralytics YOLO Detection và CVAT for images 1.1) được thực hiện độc lập trước khi nhận hoặc mở tệp tham chiếu từ Lab Coach. Bản báo cáo kiểm tra (audit log) được khởi tạo với mã băm cố định của bài làm riêng; việc đối chiếu chỉ được tiến hành sau khi đã hoàn tất bước tự kiểm tra nội bộ và khóa dữ liệu nhãn ban đầu

## 2. Quyết định phân lớp

| Ảnh/vật thể                               | Lớp     | Dấu hiệu nhìn thấy                                                                                                            | Quy tắc áp dụng                                                                                                  |
| ----------------------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `drive_008` (Xe khách vàng bên trái)      | `bus`   | Thân xe dài dạng khối hộp lớn, có nhiều khung kính hành khách liên tiếp, chiều cao và chiều dài vượt trội so với xe con       | Gán nhãn `bus` khi thấy thân xe chở khách lớn nhiều cửa sổ/hàng ghế; không gán nhãn `van` dù phần đầu hơi vuông. |
| `drive_022` (Xe tải trắng phía làn trong) | `truck` | Thùng chở hàng tách rời rõ ràng phía sau cabin điều khiển, khung gầm cao và cấu trúc chuyên dụng tải hàng                     | Gán nhãn `truck` khi phương tiện có thùng hàng/ben/sàn hàng cơ giới riêng biệt rõ rệt.                           |
| `drive_033` (Sedan màu bạc trung tâm)     | `car`   | Thân xe 3 khoang truyền thống (đầu - cabin - cốp thấp), kính vát thoải, vóc dáng xe con du lịch thông thường                  | Gán nhãn `car` cho các dòng sedan, hatchback, SUV, xe con chở người gia đình.                                    |
| `drive_038` (Xe thùng kín cỡ vừa)         | `van`   | Khối thân xe hình hộp đồng nhất (monovolume), không có khoang thùng hở tách rời như xe tải, kích thước nhỏ hơn xe khách tuyến | Gán nhãn `van` cho các xe chở người/chở hàng cỡ trung có thân hộp kín đồng khối.                                 |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

- Lớp (class): Phản ánh bản chất đối tượng là loại phương tiện gì.
- Thuộc tính (attribute): Miêu tả trạng thái quan sát của vật thể đó trong khung hình cụ thể. Lớp không thay đổi theo góc khuất hay mép ảnh, nhưng thuộc tính sẽ thay đổi tùy vị trí quan sát.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa                                                                                | Loại lỗi                 | Cách phát hiện                                                          | Sau khi sửa và quy tắc                                                                                                                     |
| -------------------------------------------------------------------------------------------- | ------------------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Vẽ bounding box bao gồm cả bóng đổ (shadow) của xe sedan dưới mặt đường trên ảnh `drive_022` | Hình học (`geometry`)    | Phóng to ảnh 100% khi rà soát kiểm tra chất lượng trên CVAT             | Thu hẹp hộp sát mép lốp xe tiếp xúc mặt đường; loại bỏ hoàn toàn bóng mặt đường ra khỏi bounding box.                                      |
| Gán nhầm xe bán pickup thành `truck` trên ảnh `drive_033`                                    | Lớp (`class`)            | Rà soát danh mục nhãn theo bảng hướng dẫn phân biệt xe con và xe tải    | Đổi nhãn thành `car` do xe bán tải này dùng khung gầm phục vụ chở người/gia đình, không phải xe tải chuyên dụng có thùng ben chở hàng lớn. |
| Bỏ sót một ô tô con ở xa bị xe khách che mất 50% thân xe trên `drive_008`                    | Phạm vi (`scope`)        | Bật chế độ tăng sáng/độ tương phản và kiểm tra kỹ các góc che khuất     | Vẽ bổ sung bounding box bao trọn phần nhìn thấy của xe bị che; đặt thuộc tính `visibility = occluded`.                                     |
| Quên chọn thuộc tính `boundary` (mặc định để trống) trên 2 hộp sát mép ảnh `drive_038`       | Thuộc tính (`attribute`) | Dùng bộ lọc (Filter) kiểm tra thuộc tính trong CVAT trước khi xuất file | Cập nhật giá trị thành `truncated` do vật thể bị mép ảnh cắt ngang thân xe.                                                                |

- Số hộp `needs_review` trước và sau khi kiểm: Trước khi kiểm: 5 hộp; Sau khi kiểm tra và chốt quyết định: 0 hộp (mọi hộp đều được xử lý dứt điểm trước khi xuất dữ liệu).
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: Tại góc xa ảnh drive_038 có một phương tiện màu xám bạc bị che khuất hơn 70%, chỉ nhìn thấy một vệt bánh và nóc kính xe, khó phân biệt giữa SUV cỡ nhỏ (car) hay xe van chở hàng (van). Cách giải quyết: Đánh dấu tạm thời review_state = needs_review, chụp lại ảnh crop 100% gửi câu hỏi kèm tọa độ lên kênh hỗ trợ hỏi Lab Coach để xin quy tắc thống nhất cho các vật thể ngưỡng giới hạn nhận diện trước khi đóng bộ nhãn

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: 0 0.523438 0.612500 0.146875 0.128125
- Tên lớp và tọa độ điểm ảnh `xyxy`:
  - tên lớp: car
  - Tọa độ điểm ảnh: [288.0, 351.0, 382.0, 433.0]
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?
  - Định dạng YOLO chỉ kiểm tra tính hợp lệ về mặt cú pháp: có đúng 5 trường dữ liệu số, class_id nằm trong khoảng cụ thể, và các chỉ số tọa độ chuẩn hóa nằm trong đoạn [0, 1].
  - Thuật toán đọc file không thể tự đánh giá ngữ nghĩa. Sai lớp khi Gán mã 1 (truck) cho một xe con car thì cú pháp vẫn đúng nhưng ngữ nghĩa sai lệch nghiêm trọng. Sai phạm vi khi một bounding box vẽ vào mảng nền đường hoặc gộp 2 chiếc xe đứng cạnh nhau vào 1 box vẫn cho ra tọa độ hợp lệ. Sai hình học khi hộp vẽ bị rộng chứa quá nhiều khoảng trống hoặc cắt mất phần xe đều có tâm và tỷ lệ tọa độ hợp lệ trên lý thuyết nhưng không phản ánh đúng biên giới thực của vật thể.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: drive_022, drive_033, drive_038
- Mã ảnh thẩm định: drive_008
- Mô tả một dự đoán trong `detect_result.jpg`: Mô hình dự đoán một bounding box bao quanh chiếc xe ô tô con màu đen đi ở làn giữa của ảnh drive_008, gán lớp car với độ tin cậy 0.84. Bounding box khớp với thân xe, phần biên dưới hơi lẹm vào phần bóng mặt đường.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Mô hình có xu hướng nhận diện cả phần bóng đổ tiếp giáp lốp xe vào trong vùng dự đoán bounding box, cần kiểm tra lại độ chặt của các bounding box gán nhãn trong tập huấn luyện để đảm bảo tính nhất quán.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Nếu khi huấn luyện với tập dữ liệu lớn hơn nhiều mà mô hình vẫn bị lỗi này, hoặc khi thay đổi các trọng số mà bounding box tự động co về đúng mép bánh xe, điều đó chứng minh lỗi bắt nguồn từ hạn chế trích xuất đặc trưng của mạng nơ-ron thay vì do chất lượng nhãn người gán.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Bộ dữ liệu chỉ gồm 4 bức ảnh (3 ảnh train, 1 ảnh val) với số lượng vật thể rất ít và bối cảnh góc chụp cố định. Mô hình rất dễ bị overfitting. Một tập kiểm thử hợp lệ trong thực tế đòi hỏi hàng nghìn bức ảnh đa dạng điều kiện thời tiết (mưa, nắng, ban đêm), góc máy khác nhau và nhiều trường hợp edge-case để đo lường mAP tổng quát hóa một cách khách quan.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: mean IoU = 0.874125, median IoU = 0.891450
- Mức đồng thuận lớp: 95.83%
- Số hộp phía bạn không ghép được: 2 (do gán thêm các xe nhỏ ở xa)
- Số hộp phía đối chiếu không ghép được: 1
- Một điểm khác biệt cụ thể: Ở ảnh drive_033, chiếc xe màu sẫm phía xa bên lề đường được tôi gán nhãn van, trong khi bộ nhãn đối chiếu gán là car.
- Quy tắc hoặc hành động sửa phát sinh: Lập thêm quy tắc làm rõ: nếu phương tiện ở khoảng cách xa trên 50 mét có tỷ lệ chiều cao/chiều rộng không vượt trội và không thấy rõ khoang hành lý vuông phía sau, ưu tiên gán là car thay vì suy đoán sang van.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Mức đồng thuận cao chỉ phản ánh hai nguồn gán nhãn có sự thống nhất cao về mặt quy ước và khả năng tái lập quy tắc (inter-annotator agreement). Nếu cả hai người cùng hiểu sai hướng dẫn, cùng bỏ qua một quy tắc (ví dụ: cả hai đều bao gồm bóng đổ vào bounding box hoặc cùng gọi sai dòng xe), thì mức đồng thuận vẫn đạt 100% dù nhãn thực tế hoàn toàn sai lệch so với chân thực tế (ground truth tuyệt đối).

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

- Minh chứng mạnh nhất: Bảng đối chiếu comparison_iou.csv và ảnh phủ trực quan comparison_overlay.jpg minh chứng sự ăn khớp hình học cao giữa nhãn tự dán và nhãn tham chiếu với median IoU xấp xỉ 0.89.
- Câu hỏi còn lại cho Lab Coach: Đối với các phương tiện giao thông bị che khuất trên 70% chỉ lộ một phần nóc hoặc gương xe ở đường viền ảnh, ngưỡng bằng chứng tối thiểu nào nên được chuẩn hóa để quyết định giữa việc gán nhãn (kèm unclear/occluded) hay loại bỏ hẳn không gán nhãn để tránh nhiễu cho mô hình học sâu?
