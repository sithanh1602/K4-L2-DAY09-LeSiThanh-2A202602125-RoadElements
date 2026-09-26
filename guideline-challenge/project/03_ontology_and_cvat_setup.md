# Ontology + CVAT setup — Team04

## Phạm vi và tài liệu tham chiếu

- Bài toán: Gán nhãn Traffic Sign, Traffic Light trên dữ liệu ảnh giao thông.
- Người phụ trách CVAT: Lê Sĩ Thành — GitHub: sithanh1602.
- Tài liệu tham chiếu: `_GUIDELINE V3.pdf` do nhóm cung cấp.
- Schema đối chiếu: `03_cvat_labels.json` hiện tại.
- Chỉ gán biển báo và đèn giao thông; không gán người, xe, lane marking hoặc drivable area.
- Đây là phần chuẩn bị cấu hình. Chưa có thông tin xác nhận task và setup test thực tế.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| traffic_sign | rectangle | class | Không áp dụng | Không áp dụng | Không áp dụng | Một mặt biển độc lập là một object; box sát mặt biển, không lấy cột. |
| sign_status (thuộc traffic_sign) | Kế thừa rectangle | attribute | prohibitory, warning, mandatory, direction_indication, supplementary, unknown | unknown | false | Phân nhóm cùng loại object biển báo; nhóm biển không thay đổi theo frame của cùng object. |
| traffic_light | rectangle | class | Không áp dụng | Không áp dụng | Không áp dụng | Một đầu/cụm đèn độc lập là một object; box sát housing, không lấy cột/cần đèn. |
| signalType (thuộc traffic_light) | Kế thừa rectangle | attribute | vehicle, pedestrian | vehicle | false | Phân biệt đèn điều khiển xe và đèn người đi bộ; đặc tính cố định của đầu đèn. |
| state (thuộc traffic_light) | Kế thừa rectangle | attribute | red, yellow, green, off, unknown | unknown | true | Trạng thái có thể thay đổi theo frame nếu dùng tracking; ảnh tĩnh chỉ đọc tín hiệu nhìn thấy trong ảnh. |

Tất cả attribute dùng input_type=select. Tên label, tên attribute, giá trị, default và mutable ở bảng này khớp JSON hiện tại.

## Class hay attribute

Traffic Sign và Traffic Light là hai loại object khác nhau nên dùng hai class. Nhóm biển (sign_status), loại tín hiệu (signalType) và trạng thái (state) là các thuộc tính của object nên không tách thành class riêng.

Ý nghĩa sign_status:

- prohibitory: biển cấm / hạn chế.
- warning: biển cảnh báo / nguy hiểm.
- mandatory: biển hiệu lệnh.
- direction_indication: biển chỉ dẫn / chỉ đường.
- supplementary: biển phụ.
- unknown: nhận diện được là biển báo nhưng không đủ bằng chứng xác định nhóm.

signalType gồm vehicle (đèn điều khiển phương tiện) và pedestrian (đèn dành cho người đi bộ).

state gồm red, yellow, green, off và unknown. off dùng khi xác định đèn không sáng; unknown dùng khi không đủ bằng chứng đọc trạng thái do xa, mờ, lóa hoặc che khuất. Không suy ra trạng thái từ hành vi xe/người. Không gán yellow nếu đèn người đi bộ không có tín hiệu yellow.

## Default, mutable và nguy cơ gán sai

PDF không quy định default hoặc mutable; các giá trị trong bảng là lựa chọn cấu hình của schema hiện tại, cần được người viết guideline ghi thống nhất trước calibration.

- sign_status mặc định unknown: tránh tự gán một nhóm biển khi chưa xem đủ bằng chứng. Annotator vẫn phải kiểm tra và đổi sang nhóm cụ thể khi xác định được.
- state mặc định unknown: tránh tự tạo nhãn màu đèn khi quên chọn. unknown không có nghĩa là đèn tắt.
- signalType mặc định vehicle vì schema chỉ có vehicle/pedestrian. Annotator phải kiểm tra từng đầu đèn và đổi sang pedestrian nếu phù hợp; bỏ qua bước này có thể gán nhầm đèn người đi bộ thành đèn xe.
- Nếu không xác định được signalType, tạo Issue để review; không coi default vehicle là kết luận và không coi object đã được xử lý xong.
- PDF ghi off / none; JSON thống nhất dùng off, không dùng thêm none.
- JSON không có __undefined__.
- state mutable=true không bắt buộc dùng Track. Bộ ảnh tĩnh độc lập dùng Shape.

## Geometry và ca chưa rõ

- Một mặt biển độc lập = một rectangle. Hai panel độc lập cùng cột = hai annotation.
- Box biển sát mặt biển, không lấy cột hoặc giá đỡ.
- Mỗi đầu/cụm đèn độc lập có một rectangle sát housing, không bao cột/cần đèn hoặc biển bên cạnh.
- Biển bị che một phần hoặc cắt mép ảnh vẫn gán nếu còn đủ bằng chứng nhận diện là biển báo.
- Không đoán nhóm biển hoặc trạng thái đèn khi ảnh không đủ bằng chứng.
- Các loại Issue trong PDF: UNCERTAIN_CLASS, UNCERTAIN_SIGN_STATUS, UNCERTAIN_BOUNDARY, ATTRIBUTE_CHECK, UNCERTAIN_SCOPE.
- Không tự thêm class hoặc attribute ngoài schema. JSON hiện không có attribute truncated hoặc thuộc tính review riêng.

TODO — Chốt geometry tolerance và cách đặt box khi bị che/cắt với người phụ trách guideline.

TODO — Kiểm tra cách thể hiện escalation trong export phục vụ lab. Không mặc định Issue/comment có trong annotations.xml. Nếu cần mở rộng schema để chấm được, nhóm phải thống nhất và cập nhật guideline trước khi triển khai.

## CVAT

- **Phiên bản CVAT:** TODO — kiểm bằng `python -X utf8 lab9.py cvat` trên máy chạy CVAT.
- **Tên task calibration dự kiến:** `Team04-calib-v1-Thanh`.
- **Tên và ID task calibration thực tế:** TODO — bổ sung sau khi tạo/xác nhận task.
- **Guide của task đã dán 02_guideline.md?** TODO — chưa xác nhận. File trong repo hiện là v0; cần hoàn thiện bản dùng cho calibration trước khi dán.
- **Shape hay Track:** Shape cho bộ ảnh tĩnh độc lập. Nếu chuyển sang chuỗi frame và tracking thì phải bổ sung quy tắc temporal trước khi làm.
- **Ảnh upload:** gom từ sample_pack.csv bằng `python -X utf8 lab9.py pack calibration`; kết quả nằm trong build/calibration/.
- **Labels:** dán toàn bộ 03_cvat_labels.json vào Labels → Raw, kiểm lại trong Constructor.
- **Export cho task ảnh tĩnh:** CVAT for images 1.1; tắt Save images; lưu annotation trước khi export.
- **Calibration:** mỗi thành viên tạo task riêng với cùng ảnh, labels và guideline, rồi label độc lập.

## Setup test

TODO — Một thành viên chưa tham gia setup mở task và ghi kết quả thực tế:

| Mục cần ghi | Kết quả |
|---|---|
| Người test và ngày test | TODO |
| Chọn đúng label và công cụ rectangle? | TODO |
| Hiểu sign_status, signalType, state và kiểm tra default? | TODO |
| Biết khi nào dùng unknown/off và khi nào tạo Issue? | TODO |
| Chỗ vấp và cách xử lý | TODO |

## Phần còn phải đồng bộ

TODO — Người phụ trách guideline đưa taxonomy, default, quy tắc unknown/off và cách xử lý ca chưa rõ vào 02_guideline.md. Tên V3 của PDF không thay thế các mốc v1/v2/v3 và bằng chứng calibration/blind test của lab.
