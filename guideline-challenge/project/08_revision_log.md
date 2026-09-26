# Revision Log

## v3 — đồng bộ theo `_GUIDELINE V3.pdf`

### Thay đổi chính

- Chốt problem family: **Traffic Sign Taxonomy + Traffic Light State Classification**.
- Chỉ giữ 2 class annotation: `traffic_sign` và `traffic_light`.
- Traffic Sign dùng attribute `sign_status` với các giá trị:
  - `prohibitory`
  - `warning`
  - `mandatory`
  - `direction_indication`
  - `supplementary`
  - `unknown`
- Traffic Light dùng attribute:
  - `signalType`: `vehicle`, `pedestrian`
  - `state`: `red`, `yellow`, `green`, `off`, `unknown`
- Bỏ rule cũ `ego_relevant`.
- Bỏ rule cũ `legible`.
- Sửa rule pedestrian traffic light: không ignore; annotate với `signalType=pedestrian`.
- Đồng bộ nguồn ảnh theo guide V3: thư mục `images`, định dạng `.jpg`.
- Bổ sung issue types:
  - `UNCERTAIN_CLASS`
  - `UNCERTAIN_SIGN_STATUS`
  - `UNCERTAIN_BOUNDARY`
  - `ATTRIBUTE_CHECK`
  - `UNCERTAIN_SCOPE`

### Lý do

Các file markdown trước đó chưa khớp với `_GUIDELINE V3.pdf`: còn dùng ontology cũ (`ego_relevant`, `legible`, `regulatory/warning/informational`) và còn ignore pedestrian signal. Bản này lấy V3 làm chuẩn để CVAT owner, QA owner và nhóm peer test dùng thống nhất.

### Evidence

- `_GUIDELINE V3.pdf`
- `01_problem_statement.md`
- `02_guideline.md`
