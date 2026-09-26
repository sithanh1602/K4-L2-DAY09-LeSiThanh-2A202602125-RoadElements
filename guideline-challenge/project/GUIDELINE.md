# ANNOTATION GUIDELINE V3

## Traffic Sign & Traffic Light

**Hướng dẫn thực hành trên CVAT**

| Nội dung | Quy định |
|---|---|
| Dữ liệu | Ảnh giao thông |
| Công cụ | CVAT |
| Phạm vi | Traffic Sign: Bounding Box; Traffic Light: Bounding Box / Track |
| Không gán | Người, xe, lane marking, drivable area và object instance khác |
| Nguyên tắc | Không đoán; chỉ gán class/attribute trong scope; case không rõ phải đưa review |

> Nếu guideline chính thức của batch/customer có quy định khác, guideline của batch/customer được ưu tiên.

## 1. Phạm vi và dữ liệu sử dụng

- Thư mục ảnh: `images`.
- Định dạng ảnh: `.jpg`.
- Chỉ annotate **Traffic Sign** và **Traffic Light**.
- Không annotate pedestrian, rider, car, truck, bus, train, motorcycle, bicycle, lane marking hoặc drivable area.
- Không tự tạo class mới, không đổi tên class và không gộp class theo cảm tính.

## 2. Taxonomy và loại shape bắt buộc

| Nhóm nhãn | CVAT shape | Class / attribute áp dụng |
|---|---|---|
| Traffic Sign | Rectangle / Bounding Box | `traffic_sign` + `sign_status` |
| Traffic Light | Rectangle / Bounding Box / Track | `traffic_light` + `signalType` + `state` |

> **Quan trọng:** Traffic Sign chỉ bao mặt biển. Traffic Light chỉ bao đầu/cụm đèn. Không lấy cột, giá đỡ hoặc quá nhiều background.

## 3. Traffic Sign

### 3.1. Quy tắc Bounding Box

- Một mặt biển độc lập = một Bounding Box; box sát mặt biển, không lấy cột.
- Hai panel/biển độc lập trên cùng cột = hai annotation.
- Biển bị che một phần vẫn gán nếu còn đủ bằng chứng để xác định là biển báo; dùng `occluded` nếu schema có.
- Biển bị cắt bởi mép ảnh vẫn gán nếu nhận diện được; dùng `truncated` nếu schema có.
- Biển quá nhỏ/mờ: không đoán loại; dùng `unknown` hoặc tạo Issue/comment.

### 3.1.2. Ví dụ minh họa

Các tình huống trong hình minh họa của PDF được chuyển thành bảng để tài liệu dùng độc lập trong một file Markdown:

| Tình huống trong hình | Cách xử lý minh họa |
|---|---|
| Biển STOP rõ, không bị che | Gán một box sát mặt biển. |
| Biển cảnh báo người đi bộ rõ | Gán box sát mặt biển. |
| Biển giới hạn tốc độ 50 và biển cấm đỗ trên cùng cột | Gán hai box riêng vì là hai biển độc lập. |
| Biển bị che một phần | Vẫn gán nếu nhận diện được; dùng occluded nếu schema có. |
| Biển bị cắt mép ảnh | Vẫn gán nếu nhận diện được; dùng truncated nếu schema có. |
| Biển bị che từ 50% trở lên trong ví dụ | Hình minh họa ghi gán unknown nếu không xác định được loại; không coi phần trăm che là lý do tự động chọn unknown khi vẫn đủ bằng chứng. |

*Hình 1 trong PDF: minh họa Traffic Sign rõ hoàn toàn, nhiều panel, occluded, truncated, che từ 50%, quá nhỏ/mờ và các nhóm sign_status.*

### 3.2. Thuộc tính `sign_status`

Mỗi Traffic Sign cần gán thêm thuộc tính phân nhóm biển báo:

| `sign_status` | Ý nghĩa | Ví dụ nhận biết |
|---|---|---|
| `prohibitory` | Biển cấm / hạn chế | Cấm đi, cấm rẽ, giới hạn tốc độ, cấm dừng/đỗ... |
| `warning` | Biển cảnh báo / nguy hiểm | Cảnh báo giao nhau, người đi bộ, đường cong, công trường... |
| `mandatory` | Biển hiệu lệnh | Bắt buộc đi thẳng, rẽ trái/phải, vòng xuyến, hướng phải đi... |
| `direction_indication` | Biển chỉ dẫn / chỉ đường | Hướng đi, địa danh, làn/hướng, bãi đỗ, đường một chiều... |
| `supplementary` | Biển phụ | Biển bổ sung phạm vi, khoảng cách, thời gian, đối tượng áp dụng... |
| `unknown` | Không xác định chắc nhóm | Biển quá nhỏ/mờ hoặc hình thức không đủ bằng chứng. |

> **Rule `sign_status`:** Chỉ chọn nhóm khi hình dạng/nội dung biển cung cấp đủ bằng chứng. Không gán `warning` / `prohibitory` / ... chỉ dựa vào vị trí hoặc màu nhìn không rõ.

## 4. Traffic Light

### 4.1. Quy tắc Bounding Box / Track

- Mỗi đầu/cụm đèn độc lập = một Bounding Box hoặc một Track nếu task dùng tracking.
- Box sát housing của đèn; không lấy cột, cần đèn hoặc cụm biển bên cạnh.
- Không suy ra trạng thái đèn từ hành vi xe/người; chỉ đọc đúng tín hiệu nhìn thấy ở frame hiện tại.
- Nếu `state` không rõ do xa, lóa hoặc che khuất: dùng `unknown` / `none` theo schema hoặc tạo Issue.

### 4.2. Thuộc tính `signalType`

| `signalType` | Áp dụng | Ghi chú |
|---|---|---|
| `vehicle` | Đèn điều khiển phương tiện | Đèn tròn/mũi tên cho luồng xe. |
| `pedestrian` | Đèn dành cho người đi bộ | Biểu tượng người đứng/đi hoặc cụm tín hiệu pedestrian. |

### 4.3. Thuộc tính `state`

| `state` | Vehicle | Pedestrian |
|---|---|---|
| `red` | Dừng | Người đi bộ không sang đường |
| `yellow` | Chuẩn bị chuyển pha / cảnh báo | Không dùng nếu pedestrian signal không có yellow |
| `green` | Được phép đi theo tín hiệu | Người đi bộ được sang đường |
| `off` / `none` | Đèn không sáng | Đèn không sáng |
| `unknown` | Không xác định được trạng thái | Không xác định được trạng thái |

### 4.4. Ví dụ minh họa đèn trong PDF

| Tình huống | Nhãn/thuộc tính minh họa |
|---|---|
| Đèn xe đỏ, vàng hoặc xanh | vehicle + red / yellow / green tương ứng |
| Đèn hình người đứng màu đỏ | pedestrian + red |
| Đèn hình người đi màu xanh | pedestrian + green |
| Đầu đèn không sáng | vehicle + off/none theo schema |
| Đầu đèn bị che một phần nhưng còn nhận diện được | Gán; dùng occluded nếu schema có |
| Vật thể đèn quá mờ, không xác định được trong hình cuối | Hình ghi “Không gán”; tạo Issue theo quy tắc ca chưa rõ. Phân biệt với đầu đèn đã nhận diện chắc nhưng không đọc được state, khi đó dùng unknown. |

## 5. Quy tắc khi class / attribute không rõ

| Issue type | Ví dụ |
|---|---|
| `UNCERTAIN_CLASS` | Không chắc object là traffic sign hay object ngoài scope |
| `UNCERTAIN_SIGN_STATUS` | Không chắc biển cấm / cảnh báo / hiệu lệnh / chỉ dẫn |
| `UNCERTAIN_BOUNDARY` | Không rõ extent do occlusion hoặc blur |
| `ATTRIBUTE_CHECK` | Không chắc `signalType` hoặc traffic-light `state` |
| `UNCERTAIN_SCOPE` | Không chắc object có thuộc Traffic Sign / Traffic Light |

## 6. Checklist chất lượng trước khi Submit

- [ ] Chỉ có 2 nhóm: Traffic Sign và Traffic Light.
- [ ] Traffic Sign: BBox sát mặt biển, không lấy cột.
- [ ] Traffic Sign: `sign_status` đã gán đúng (`prohibitory` / `warning` / `mandatory` / `direction_indication` / `supplementary` / `unknown`).
- [ ] Traffic Light: BBox/Track sát đầu đèn, không lấy cột/cần đèn.
- [ ] Traffic Light: `signalType` = `vehicle` hoặc `pedestrian` đúng đối tượng.
- [ ] Traffic Light: `state` đúng frame hiện tại; không đoán màu.
- [ ] Không tự tạo class/attribute ngoài schema; case chưa chắc có Issue/comment.
- [ ] Đã Save và tự review toàn bộ job trước Validation.

## 7. Tiêu chí Reviewer kiểm tra

| Hạng mục | Reviewer kiểm tra |
|---|---|
| Taxonomy | Chỉ Traffic Sign và Traffic Light; không có lane/drivable/người/xe. |
| Geometry | BBox sát mặt biển/đầu đèn; không lấy cột hoặc background thừa. |
| Sign status | Phân đúng `prohibitory`, `warning`, `mandatory`, `direction_indication`, `supplementary` hoặc `unknown`. |
| Signal type | Phân đúng `vehicle` và `pedestrian`. |
| Light state | `red` / `yellow` / `green` / `off` / `unknown` đúng bằng chứng của frame. |
| Consistency | Case tương tự được annotate và gán attribute nhất quán. |

## 8. Quick Reference

| Tình huống | Làm gì | Không làm | Cần review? |
|---|---|---|---|
| Biển cấm rõ | BBox + `sign_status=prohibitory` | Chỉ ghi `traffic_sign` rồi bỏ status | Không |
| Biển cảnh báo rõ | BBox + `sign_status=warning` | Đoán nếu pictogram mờ | Không |
| Biển chỉ dẫn/chỉ đường | BBox + `sign_status=direction_indication` | Gộp nhiều panel | Không |
| Biển không rõ nhóm | BBox + `sign_status=unknown` / Issue | Tự đoán nhóm | Có |
| Đèn xe | `signalType=vehicle` + `state` | Lấy cả cột | Nếu state không rõ |
| Đèn người đi bộ | `signalType=pedestrian` + `state` | Gán yellow nếu tín hiệu không có yellow | Nếu state không rõ |
| Object ngoài scope | Không annotate | Tạo class mới | Nếu không chắc scope |

> **Nguyên tắc cuối cùng:** Chỉ gán Traffic Sign và Traffic Light. Không đủ bằng chứng để xác định class hoặc attribute: dừng suy đoán, tạo Issue và escalate.

---

## Ghi chú áp dụng cho schema Team04

Phần này ghi rõ cách áp dụng vào JSON đã thống nhất của Team04, bổ sung cho nội dung chuyển từ PDF ở trên:

- JSON dùng `off` cho đèn không sáng và `unknown` cho trạng thái không xác định; không có giá trị `none` hoặc `__undefined__`.
- `sign_status` mặc định `unknown`; `state` mặc định `unknown`.
- `signalType` mặc định `vehicle`. Phải kiểm tra từng object và đổi sang `pedestrian` khi là đèn người đi bộ. Default không thay thế việc xác định loại đèn; không chắc thì tạo Issue để review.
- `sign_status` và `signalType` có `mutable=false`; `state` có `mutable=true`. Đây là cấu hình của nhóm, không phải default/mutable được quy định trong PDF.
- Với 25 ảnh tĩnh độc lập, dùng Rectangle → Shape.
- Các hình minh họa trong PDF được diễn giải thành bảng ví dụ ở mục 3.1.2 và 4.4; file này không nhúng ảnh minh họa.
