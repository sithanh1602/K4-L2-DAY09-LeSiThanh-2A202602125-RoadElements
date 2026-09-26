# Edge-case library — Team04

Nguồn: `Team04_ground_truth.zip` và ảnh trong `data/dataset_vn/`. Tọa độ dưới đây là `(x1, y1, x2, y2)` theo XML, dùng để tìm object. Các ca ESCALATE là đề xuất review, chưa phải bằng chứng đã tạo Issue hoặc gold đã freeze. Sample dùng tên ảnh thật; cần đối chiếu catalog/sample_pack trước khi dùng cho blind test.

---

CASE ID: EC01
Sample: frame_01.jpg
Scene: Giao lộ ban đêm.
Observation: XML gán đèn xe đỏ và đèn pedestrian xanh sát nhau; object nhỏ nên dễ nhầm loại tín hiệu.
Decision: LABEL / ESCALATE nếu không đọc được biểu tượng.
Expected: Tách từng đầu đèn; kiểm tra signalType theo hình thực tế, không suy ra từ màu. Review các box pedestrian xanh (853.70,342.20,860.00,350.35) và (870.78,321.31,883.84,333.53) trước khi chốt.
Rationale: Nhầm đèn người đi bộ thành đèn xe làm sai tín hiệu đi/dừng.
Common mistake: Thấy xanh là gán vehicle hoặc giữ default vehicle.
Diversity: critical / ambiguity / small_far

---

CASE ID: EC02
Sample: frame_03.jpg
Scene: Nhiều đầu đèn tại giao lộ ban đêm.
Observation: Có các đèn đỏ và một đèn xanh bên phải trong cùng ảnh.
Decision: LABEL.
Expected: Mỗi đầu đèn một box; đọc state riêng. XML ghi đèn tại (883.72,410.75,899.90,425.28) là vehicle/green, các đầu đèn đỏ không đổi theo nó.
Rationale: Tránh truyền sai trạng thái giữa các tín hiệu điều khiển luồng khác nhau.
Common mistake: Gán tất cả đèn cùng màu hoặc suy theo dòng xe.
Diversity: conflict / critical / low_visibility

---

CASE ID: EC03
Sample: frame_03.jpg
Scene: Biển nhỏ cạnh cụm biển lớn bên phải.
Observation: Box (902.91,361.90,947.09,400.34) được gán occluded=1 và sign_status=unknown trong XML.
Decision: UNKNOWN; ESCALATE nếu cần xác nhận ranh giới.
Expected: traffic_sign, sign_status=unknown; giữ một box riêng, chỉ bao phần nhìn thấy theo quy tắc của nhóm; tạo UNCERTAIN_BOUNDARY nếu biên không rõ.
Rationale: Tránh đoán nhóm biển khi bằng chứng bị che.
Common mistake: Gán direction_indication chỉ vì ở cạnh biển xanh lớn.
Diversity: occlusion / ambiguity / escalation

---

CASE ID: EC04
Sample: frame_09.jpg
Scene: Đường phố ban đêm, tín hiệu ở xa.
Observation: XML gán vehicle/yellow cho box chỉ khoảng 12 × 11 px tại (1107.44,604.27,1119.44,615.11).
Decision: ESCALATE.
Expected: Review ATTRIBUTE_CHECK trên ảnh gốc để xác nhận là đầu đèn và màu vàng. Nếu nhận diện được đèn nhưng không đọc được state, dùng unknown; nếu chưa rõ class, review UNCERTAIN_CLASS.
Rationale: Điểm sáng nhỏ có thể bị nhầm với nguồn sáng khác, gây sai ground truth.
Common mistake: Coi mọi điểm sáng vàng là đèn giao thông hoặc xác nhận yellow chỉ từ XML.
Diversity: small_far / low_visibility / escalation

---

CASE ID: EC05
Sample: frame_11.jpg
Scene: Biển tròn xanh viền đỏ có dấu gạch chéo bên phải.
Observation: Biển có nền xanh nhưng ký hiệu cấm; XML gán prohibitory.
Decision: LABEL.
Expected: traffic_sign, sign_status=prohibitory tại (998.75,414.19,1061.69,469.91); box sát mặt biển, không lấy cột.
Rationale: Nhóm chức năng phải dựa vào ký hiệu, không chỉ màu nền.
Common mistake: Chọn direction_indication hoặc mandatory vì biển màu xanh.
Diversity: ambiguity / semantics

---

CASE ID: EC06
Sample: frame_13.jpg
Scene: Đường dưới cầu, nhiều biển phía xa.
Observation: Một biển rất nhỏ có box khoảng 11 × 9 px tại (938.90,720.50,950.20,729.89); XML gán unknown.
Decision: UNKNOWN.
Expected: traffic_sign, sign_status=unknown khi nhận diện được biển nhưng không đủ bằng chứng phân nhóm; cần review nếu cả class chưa chắc.
Rationale: Không tạo nhãn chức năng dựa trên vài pixel không rõ.
Common mistake: Đoán nhóm theo biển bên cạnh hoặc vị trí trên đường.
Diversity: small_far / ambiguity

---

CASE ID: EC07
Sample: frame_14.jpg
Scene: Cụm nhiều biển ở sát mép phải ảnh.
Observation: Biển tròn xanh có hai mũi tên nằm phía trên các panel hình phương tiện. XML gán biển tròn là direction_indication.
Decision: ESCALATE.
Expected: Review UNCERTAIN_SIGN_STATUS cho box (1747.94,501.22,1830.53,584.66). Đề xuất mandatory nếu xác nhận là biển bắt buộc đi theo hướng mũi tên; giữ từng panel riêng, không gộp cả cụm.
Rationale: Phân biệt hướng bắt buộc với thông tin chỉ dẫn, tránh giữ nhãn sai chỉ vì đã có trong ZIP.
Common mistake: Gán mọi biển nền xanh là direction_indication.
Diversity: ambiguity / multi_panel / escalation

---

CASE ID: EC08
Sample: frame_14.jpg
Scene: Đầu đèn phía trên cụm biển bên phải.
Observation: XML gán vehicle/off tại (1762.30,440.54,1813.67,487.74); đầu đèn nhìn tối trong ảnh.
Decision: ESCALATE trước khi chốt off.
Expected: Chỉ giữ off nếu xác nhận nhìn được mặt tín hiệu và đèn không sáng. Nếu không đọc được do góc nhìn/che khuất thì state=unknown và ATTRIBUTE_CHECK; box không bao biển bên dưới.
Rationale: Không nhìn thấy ánh sáng chưa đủ chứng minh đèn đang tắt.
Common mistake: Đầu đèn tối hoặc quay lệch là tự gán off.
Diversity: visibility / ambiguity / escalation
