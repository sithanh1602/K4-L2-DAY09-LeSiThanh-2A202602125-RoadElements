# Annotation guideline — Traffic Sign & Traffic Light

**Version:** v3

Hướng dẫn thực hành trên CVAT cho bài toán gán nhãn **Traffic Sign** và **Traffic Light**.

Nếu guideline chính thức của batch/customer có quy định khác, guideline của batch/customer được ưu tiên.

## 1. Phạm vi và dữ liệu sử dụng

- **Thư mục ảnh:** `images`
- **Định dạng ảnh:** `.jpg`
- **Công cụ:** CVAT
- **Chỉ annotate:** `Traffic Sign` và `Traffic Light`.
- **Không annotate:** pedestrian, rider, car, truck, bus, train, motorcycle, bicycle, lane marking, drivable area và object instance khác.
- **Nguyên tắc:** không đoán; chỉ gán class/attribute trong scope. Case không rõ phải tạo Issue/comment để review.
- **Không tự tạo class mới**, không đổi tên class và không gộp class theo cảm tính.

## 2. Taxonomy và loại shape bắt buộc

| Nhóm nhãn | CVAT shape | Class / attribute áp dụng |
|---|---|---|
| Traffic Sign | Rectangle / Bounding Box | `traffic_sign` + `sign_status` |
| Traffic Light | Rectangle / Bounding Box / Track | `traffic_light` + `signalType` + `state` |

**Quan trọng:**

- Traffic Sign chỉ bao **mặt biển**. Không lấy cột, giá đỡ hoặc background.
- Traffic Light chỉ bao **đầu đèn / cụm đèn**. Không lấy cột, cần đèn hoặc cụm biển bên cạnh.

## 3. Traffic Sign

### 3.1. Quy tắc Bounding Box

- Một mặt biển độc lập = một Bounding Box.
- Box sát mặt biển, không lấy cột.
- Hai panel/biển độc lập trên cùng cột = hai annotation riêng.
- Biển bị che một phần vẫn gán nếu còn đủ bằng chứng để xác định là biển báo; dùng `occluded` nếu schema có.
- Biển bị cắt bởi mép ảnh vẫn gán nếu nhận diện được; dùng `truncated` nếu schema có.
- Biển quá nhỏ/mờ: không đoán loại; dùng `unknown` hoặc tạo Issue/comment.

### 3.2. Thuộc tính `sign_status`

Mỗi Traffic Sign cần gán thuộc tính phân nhóm biển báo:

| `sign_status` | Ý nghĩa | Ví dụ nhận biết |
|---|---|---|
| `prohibitory` | Biển cấm / hạn chế | Cấm đi, cấm rẽ, giới hạn tốc độ, cấm dừng/đỗ |
| `warning` | Biển cảnh báo / nguy hiểm | Cảnh báo giao nhau, người đi bộ, đường cong, công trường |
| `mandatory` | Biển hiệu lệnh | Bắt buộc đi thẳng, rẽ trái/phải, vòng xuyến, hướng phải đi |
| `direction_indication` | Biển chỉ dẫn / chỉ đường | Hướng đi, địa danh, làn/hướng, bãi đỗ, đường một chiều |
| `supplementary` | Biển phụ | Bổ sung phạm vi, khoảng cách, thời gian, đối tượng áp dụng |
| `unknown` | Không xác định chắc nhóm | Biển quá nhỏ/mờ hoặc hình thức không đủ bằng chứng |

**Rule `sign_status`:**

- Chỉ chọn nhóm khi hình dạng/nội dung biển cung cấp đủ bằng chứng.
- Không gán `warning`, `prohibitory`, `mandatory`, `direction_indication`, `supplementary` chỉ dựa vào vị trí hoặc màu nhìn không rõ.
- Nếu không chắc nhóm biển: dùng `unknown` và tạo Issue/comment nếu cần reviewer xác nhận.

## 4. Traffic Light

### 4.1. Quy tắc Bounding Box / Track

- Mỗi đầu/cụm đèn độc lập = một Bounding Box hoặc một Track nếu task dùng tracking.
- Box sát housing của đèn.
- Không lấy cột, cần đèn, dây treo, biển bên cạnh hoặc quá nhiều background.
- Không suy ra trạng thái đèn từ hành vi xe/người; chỉ đọc đúng tín hiệu nhìn thấy trong frame hiện tại.
- Nếu `state` không rõ do xa, lóa hoặc che khuất: dùng `unknown` / `off` theo schema hoặc tạo Issue/comment.

### 4.2. Thuộc tính `signalType`

| `signalType` | Áp dụng | Ghi chú |
|---|---|---|
| `vehicle` | Đèn điều khiển phương tiện | Đèn tròn/mũi tên cho luồng xe |
| `pedestrian` | Đèn dành cho người đi bộ | Biểu tượng người đứng/đi hoặc cụm tín hiệu pedestrian |

### 4.3. Thuộc tính `state`

| `state` | Vehicle | Pedestrian |
|---|---|---|
| `red` | Dừng | Người đi bộ không sang đường |
| `yellow` | Chuẩn bị chuyển pha / cảnh báo | Không dùng nếu pedestrian signal không có yellow |
| `green` | Được phép đi theo tín hiệu | Người đi bộ được sang đường |
| `off` | Đèn không sáng | Đèn không sáng |
| `unknown` | Không xác định được trạng thái | Không xác định được trạng thái |

**Rule `state`:**

- Gán trạng thái theo tín hiệu trong frame hiện tại.
- Không đoán trạng thái dựa vào dòng xe, người đi bộ hoặc logic giao lộ.
- Nếu đèn quá xa, bị lóa, bị che hoặc không rõ màu: dùng `unknown` và tạo Issue/comment nếu cần.
- Nếu nhận ra housing đèn nhưng bóng đèn không sáng: dùng `off`.

## 5. Quy tắc khi class / attribute không rõ

| Issue type | Khi nào dùng |
|---|---|
| `UNCERTAIN_CLASS` | Không chắc object là Traffic Sign hay object ngoài scope |
| `UNCERTAIN_SIGN_STATUS` | Không chắc biển là cấm / cảnh báo / hiệu lệnh / chỉ dẫn / biển phụ |
| `UNCERTAIN_BOUNDARY` | Không rõ extent do occlusion, blur, crop hoặc ảnh quá tối |
| `ATTRIBUTE_CHECK` | Không chắc `signalType`, `state` hoặc attribute cần reviewer kiểm tra |
| `UNCERTAIN_SCOPE` | Không chắc object có thuộc Traffic Sign / Traffic Light hay không |

**Nguyên tắc xử lý:**

1. Đủ bằng chứng về class và attribute -> annotate bình thường.
2. Đủ bằng chứng về class nhưng không chắc attribute -> gán `unknown` và tạo Issue/comment nếu cần.
3. Không đủ bằng chứng là Traffic Sign / Traffic Light -> không annotate hoặc tạo Issue nếu thật sự cần review.
4. Không giải thích bằng miệng; rule nào cần peer làm theo phải nằm trong guideline.

## 6. Checklist chất lượng trước khi Submit

- [ ] Chỉ có 2 nhóm: Traffic Sign và Traffic Light.
- [ ] Traffic Sign: BBox sát mặt biển, không lấy cột.
- [ ] Traffic Sign: `sign_status` đã gán đúng (`prohibitory` / `warning` / `mandatory` / `direction_indication` / `supplementary` / `unknown`).
- [ ] Traffic Light: BBox/Track sát đầu đèn, không lấy cột/cần đèn.
- [ ] Traffic Light: `signalType = vehicle` hoặc `pedestrian` đúng đối tượng.
- [ ] Traffic Light: `state` đúng frame hiện tại; không đoán màu.
- [ ] Không tự tạo class/attribute ngoài schema.
- [ ] Case chưa chắc có Issue/comment.
- [ ] Đã Save và tự review toàn bộ job trước Validation.

## 7. Tiêu chí Reviewer kiểm tra

| Hạng mục | Reviewer kiểm tra |
|---|---|
| Taxonomy | Chỉ có Traffic Sign và Traffic Light; không có lane/drivable/người/xe |
| Geometry | BBox sát mặt biển/đầu đèn; không lấy cột hoặc background thừa |
| Sign status | Phân đúng `prohibitory`, `warning`, `mandatory`, `direction_indication`, `supplementary` hoặc `unknown` |
| Signal type | Phân đúng `vehicle` và `pedestrian` |
| Light state | `red` / `yellow` / `green` / `off` / `unknown` đúng bằng chứng của frame |
| Consistency | Case tương tự được annotate và gán attribute nhất quán |

## 8. Quick Reference

| Tình huống | Làm gì | Không làm | Cần review? |
|---|---|---|---|
| Biển cấm rõ | BBox + `sign_status=prohibitory` | Chỉ ghi `traffic_sign` rồi bỏ status | Không |
| Biển cảnh báo rõ | BBox + `sign_status=warning` | Đoán nếu pictogram mờ | Không |
| Biển hiệu lệnh rõ | BBox + `sign_status=mandatory` | Nhầm với biển chỉ dẫn khi không đủ bằng chứng | Không |
| Biển chỉ dẫn/chỉ đường | BBox + `sign_status=direction_indication` | Gộp nhiều panel | Không |
| Biển phụ | BBox + `sign_status=supplementary` | Gộp vào biển chính nếu là panel riêng | Không |
| Biển không rõ nhóm | BBox + `sign_status=unknown` / Issue | Tự đoán nhóm | Có |
| Đèn xe | `signalType=vehicle` + `state` | Lấy cả cột | Nếu state không rõ |
| Đèn người đi bộ | `signalType=pedestrian` + `state` | Gán `yellow` nếu tín hiệu không có yellow | Nếu state không rõ |
| Object ngoài scope | Không annotate | Tạo class mới | Nếu không chắc scope |

## 9. Lỗi thường gặp

| Lỗi thường gặp | Cách sửa |
|---|---|
| Lấy cả cột biển vào box | Chỉ box mặt biển |
| Lấy cần/cột đèn vào box | Chỉ box đầu/cụm đèn |
| Gộp nhiều panel biển vào một box | Mỗi panel độc lập = một annotation |
| Gán `sign_status` theo màu nhìn mờ | Dùng `unknown` nếu không đủ bằng chứng |
| Gán trạng thái đèn theo hành vi xe | Chỉ đọc tín hiệu trong frame hiện tại |
| Bỏ qua pedestrian traffic light | Vẫn annotate nếu là Traffic Light; gán `signalType=pedestrian` |
| Tạo thêm class ngoài schema | Chỉ dùng `traffic_sign` và `traffic_light` |

## 10. Nguyên tắc cuối cùng

Chỉ gán **Traffic Sign** và **Traffic Light**. Không đủ bằng chứng để xác định class hoặc attribute thì dừng suy đoán, tạo Issue/comment và escalate cho reviewer.
