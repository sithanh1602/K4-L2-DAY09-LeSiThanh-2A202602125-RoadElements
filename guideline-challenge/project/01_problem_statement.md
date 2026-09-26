# Problem statement + downstream contract

## Bài toán

Gắn nhãn **Traffic Sign Taxonomy + Traffic Light State** trong ảnh giao thông. Nhiệm vụ chỉ tập trung vào hai nhóm đối tượng:

- **Traffic Sign:** vẽ bounding box sát mặt biển và gán thuộc tính `sign_status`.
- **Traffic Light:** vẽ bounding box/track sát đầu hoặc cụm đèn và gán thuộc tính `signalType`, `state`.

Mục tiêu là tạo ground-truth rõ ràng để module perception của hệ thống ADAS/AV học cách nhận diện biển báo và đèn giao thông trong từng frame, đặc biệt với các trường hợp nhỏ, xa, mờ, bị che một phần, ánh sáng yếu hoặc trạng thái đèn không rõ.

## Problem family

**Traffic Sign Taxonomy + Traffic Light State Classification**

Chủ đề này kết hợp hai family:

- `Traffic sign taxonomy`: phân nhóm biển báo theo chức năng.
- `Traffic light state`: phân loại loại đèn và trạng thái tín hiệu trong frame hiện tại.

## Downstream contract

1. **Downstream task / model / user là ai?**

   Module perception của hệ thống ADAS/AV. Model cần phát hiện đúng biển báo và đèn giao thông để phục vụ các quyết định như đi/dừng, cảnh báo, điều chỉnh tốc độ hoặc đánh giá hành vi lái xe. User cuối là kỹ sư ML/QA cần bộ ground-truth để train, evaluate và review lỗi model.

2. **Output annotation nào thực sự cần?**

   - **Traffic Sign:** shape `rectangle` / `bounding box`, class `traffic_sign`, attribute `sign_status`.
   - **Traffic Light:** shape `rectangle` / `bounding box`; nếu task tracking thì có thể dùng `track`, class `traffic_light`, attribute `signalType`, attribute `state`.

3. **Schema / ontology chính**

   **Traffic Sign**

   | Class | Attribute | Giá trị |
   |---|---|---|
   | `traffic_sign` | `sign_status` | `prohibitory`, `warning`, `mandatory`, `direction_indication`, `supplementary`, `unknown` |

   **Traffic Light**

   | Class | Attribute | Giá trị |
   |---|---|---|
   | `traffic_light` | `signalType` | `vehicle`, `pedestrian` |
   | `traffic_light` | `state` | `red`, `yellow`, `green`, `off`, `unknown` |

4. **Failure nào gây hậu quả lớn nhất?**

   - Bỏ sót Traffic Light hoặc gán sai `state` trong frame hiện tại -> model học sai tín hiệu dừng/đi.
   - Gán sai `signalType`, ví dụ nhầm đèn người đi bộ thành đèn xe -> downstream có thể đánh giá sai luồng giao thông.
   - Bỏ sót Traffic Sign quan trọng hoặc gán sai `sign_status`, ví dụ nhầm `prohibitory` với `warning` -> model học sai ý nghĩa biển.
   - Box lấy cả cột, giá đỡ hoặc quá nhiều background -> làm nhiễu geometry ground-truth và ảnh hưởng metric.

5. **Khi ambiguity không resolve được, escalation path là gì?**

   Không đoán. Nếu không chắc class, boundary hoặc attribute thì tạo Issue/comment trong CVAT theo nhóm:

   - `UNCERTAIN_CLASS`
   - `UNCERTAIN_SIGN_STATUS`
   - `UNCERTAIN_BOUNDARY`
   - `ATTRIBUTE_CHECK`
   - `UNCERTAIN_SCOPE`

   Reviewer/QA owner tổng hợp các case này để cập nhật guideline và revision log.

## Scope

### Trong scope

- Tất cả **Traffic Sign** nhìn thấy đủ bằng chứng để xác định là biển báo.
- Tất cả **Traffic Light** nhìn thấy đủ bằng chứng để xác định là đèn giao thông.
- Đèn điều khiển phương tiện: `signalType=vehicle`.
- Đèn dành cho người đi bộ: `signalType=pedestrian`.
- Biển báo bị che hoặc bị cắt mép ảnh vẫn gán nếu còn đủ bằng chứng; dùng `occluded` / `truncated` nếu schema có.

### Ngoài scope

Không annotate các đối tượng ngoài hai nhóm trên:

- pedestrian, rider, car, truck, bus, train, motorcycle, bicycle.
- lane marking, drivable area.
- cột đèn, giá đỡ, cần đèn nếu không phải phần đầu/cụm đèn.
- cột biển, background, object instance khác.
- biển quảng cáo, object trang trí, vật thể giống biển nhưng không đủ bằng chứng là Traffic Sign.

## Geometry tolerance

- Traffic Sign: box sát mặt biển, không lấy cột.
- Traffic Light: box sát đầu/cụm đèn, không lấy cột, cần đèn hoặc background thừa.
- Đối tượng bị che/cắt mép ảnh: chỉ box phần nhìn thấy trong frame.
- Nếu boundary không rõ do blur/occlusion: tạo Issue/comment `UNCERTAIN_BOUNDARY`.

## Output chấm được

Blind test / reviewer sẽ kiểm:

- **Taxonomy:** chỉ có `traffic_sign` và `traffic_light`.
- **Geometry:** box sát đúng đối tượng, không lấy cột/background thừa.
- **Sign status:** đúng `prohibitory`, `warning`, `mandatory`, `direction_indication`, `supplementary`, `unknown`.
- **Signal type:** đúng `vehicle` hoặc `pedestrian`.
- **Light state:** đúng `red`, `yellow`, `green`, `off`, `unknown` theo bằng chứng trong frame.
- **Consistency:** case tương tự được annotate và gán attribute nhất quán.
- **Issue handling:** case không chắc phải có Issue/comment, không tự đoán.

Mỗi quyết định phải nhìn thấy được trong export CVAT: class name, attribute value, issue/comment nếu có.

## Dữ liệu và giới hạn

- **Thư mục ảnh:** `images`
- **Định dạng ảnh:** `.jpg`
- **Công cụ:** CVAT
- **Lưu ý:** Nếu guideline chính thức của batch/customer có quy định khác, guideline của batch/customer được ưu tiên.
