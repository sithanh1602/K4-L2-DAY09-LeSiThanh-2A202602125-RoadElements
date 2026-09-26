# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:**nhóm 2 
- **Chọn sample theo rule nào** (random, theo tag rủi ro, theo annotator mới…): Stratified rule-based sampling kết hợp ngẫu nhiên:
  1. 100% sample có gắn cờ Issue từ Annotator trên CVAT (`UNCERTAIN_CLASS`, `UNCERTAIN_SIGN_STATUS`, `UNCERTAIN_BOUNDARY`, `ATTRIBUTE_CHECK`, `UNCERTAIN_SCOPE`).
  2. 100% sample chứa đối tượng rủi ro cao theo Guideline V3 (ảnh ban đêm lóa đèn, biển báo xa/mờ, cột gắn nhiều panel biển báo, đèn tín hiệu người đi bộ).
  3. 50% sample của annotator mới gia nhập hoặc annotator có tỷ lệ lỗi vượt ngưỡng ở đợt review gần nhất.
  4. 20% random sample trên toàn bộ số ảnh còn lại để đánh giá tổng thể chất lượng lô.
- **Issue được ghi ở đâu, đóng thế nào:**
  - Issue được ghi trực tiếp trên CVAT tại đúng bounding box hoặc vị trí frame nghi vấn bằng công cụ "Create Issue", sử dụng đúng 5 mã Issue chuẩn hóa tại Mục 5 của Guideline:
    + `UNCERTAIN_CLASS`: Nghi ngờ đối tượng ngoài scope (người, xe, biển quảng cáo...).
    + `UNCERTAIN_SIGN_STATUS`: Không chắc chắn nhóm biển báo (cấm, cảnh báo, hiệu lệnh, chỉ dẫn, biển phụ).
    + `UNCERTAIN_BOUNDARY`: Biên bao không rõ do bị che khuất (occlusion) hoặc nhòe mờ (blur).
    + `ATTRIBUTE_CHECK`: Không chắc chắn thuộc tính `signalType` hoặc `state` của đèn giao thông.
    + `UNCERTAIN_SCOPE`: Không chắc đối tượng có thuộc phạm vi Traffic Sign / Traffic Light hay không.
  - Xử lý và đóng issue: Annotator và Reviewer đối chiếu Quick Reference (Mục 8) và quy tắc Guideline V3 để giải quyết. Sau khi BBox/attribute được chỉnh sửa đúng hoặc thống nhất giải pháp, Reviewer xác nhận và chuyển trạng thái Issue sang **Resolved** rồi **Closed**. Mọi task phải đạt 0 Open Issue trước khi nghiệm thu.
- **Khi phát hiện guideline gap thì update và version ra sao:**
  - Khi phát hiện trường hợp góc cạnh hoặc mâu thuẫn mà Guideline chưa quy định rõ, Reviewer/Annotator escalate lên Guideline Owner.
  - Guideline Owner bổ sung quy tắc mới, ví dụ minh họa vào `02_guideline.md`, sau đó tăng version (v1 -> v2 sau calibration, v2 -> v3 sau blind handoff).
  - Mọi thay đổi được ghi nhận chi tiết vào `08_revision_log.md` (phiên bản, ngày cập nhật, tóm tắt thay đổi, lý do) và thông báo đồng bộ ngay lập tức cho toàn bộ đội ngũ dán nhãn.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Lỗi gây sai lệch bản chất nghiêm trọng ảnh hưởng trực tiếp đến an toàn vận hành xe tự hành (downstream perception): gán sai trạng thái đèn đỏ/xanh, bỏ sót biển báo cấm hoặc nguy hiểm, dán nhãn đối tượng ngoài scope thành trong scope. | - Bỏ sót đèn đỏ (`traffic_light` state=`red`)<br>- Gán đèn đỏ thành xanh (`red` thành `green`)<br>- Bỏ sót biển cấm (`sign_status`=`prohibitory`)<br>- Gán người/xe/biển quảng cáo thành `traffic_sign` | Recheck 100% batch của annotator; Reject batch ngay lập tức; Annotator bắt buộc Rework toàn bộ sample lỗi và nhận coaching trực tiếp từ QA Lead. |
| Major | Gán sai phân loại nhóm biển báo (`sign_status`), sai loại đèn (`signalType`), bỏ sót biển phụ/chỉ dẫn, hoặc Bounding Box sai lệch lớn (> 5px hoặc lẹm mất housing/mặt biển) làm sai lệch tính năng phát hiện vật thể. | - Biển hiệu lệnh (`mandatory`) gán nhầm thành chỉ dẫn (`direction_indication`)<br>- Đèn xe (`vehicle`) gán nhầm thành đèn người đi bộ (`pedestrian`)<br>- BBox bao trùm cả cột đèn lớn hoặc cần vươn<br>- BBox cắt lẹm mất > 10% diện tích mặt biển | Trả về cho annotator Rework; Reviewer tăng tỷ lệ sample kiểm tra của annotator đó thêm 20% ở batch kế tiếp. |
| Minor | Sai lệch dung sai hình học nhỏ (BBox lệch 2–4px so với mép thực tế), thừa khoảng trống viền nhỏ nhưng không lấy cột, quên đánh dấu thuộc tính phụ (`occluded`, `truncated`) khi biển bị che khuất nhẹ hoặc chạm mép ảnh nhưng class và status vẫn chính xác. | - BBox mặt biển báo thừa 3px mép ngoài<br>- Biển báo bị cành cây che 5% góc biển nhưng quên tick occluded<br>- BBox đèn hơi lệch 2px so với housing thực tế | Reviewer sửa trực tiếp trên CVAT hoặc ghi chú nhắc nhở annotator rút kinh nghiệm cuối ca; không bắt buộc reject cả batch. |
| Question | Trường hợp dữ liệu mơ hồ, thời tiết xấu (ban đêm lóa sáng, sương mù, mưa lớn) hoặc biển báo hư hỏng nặng khiến không đủ bằng chứng kết luận class/attribute, annotator chủ động flag issue xin hướng dẫn. | - Biển rỉ sét mờ hoàn toàn hình vẽ pictogram (`UNCERTAIN_SIGN_STATUS`)<br>- Đèn giao thông ban đêm bị lóa sáng không rõ bóng đèn sáng hay phản quang (`ATTRIBUTE_CHECK`) | Escalate lên QA Lead / Guideline Owner để quyết định; nếu không đủ bằng chứng thì áp dụng quy tắc gán `unknown` hoặc loại bỏ theo mục 5 & 8 Guideline. |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Critical Defect Escape Rate (CDER) | `(Số lỗi Critical phát hiện tại QA / Tổng số đối tượng Critical) * 100%` | Đảm bảo an toàn tuyệt đối cho hệ thống lái tự động; mục tiêu là 0% lỗi nguy hiểm lọt qua. |
| Mean Bounding Box IoU (mIoU) | `Trung bình IoU giữa BBox annotator và BBox ground truth/reviewer` | Đánh giá độ chính xác hình học ôm sát mặt biển/housing đèn, tránh bbox lấy cột hay thừa background. |
| Attribute Classification Accuracy | `(Số attribute gán đúng / Tổng số attribute cần gán) * 100%` cho sign_status, signalType, state | Đánh giá độ chính xác taxonomy đa tầng của bài toán biển báo và đèn tín hiệu. |
| Defect Density | `Tổng số lỗi (trọng số: Critical*5 + Major*2 + Minor*1) / Tổng số annotation` | Đo lường chất lượng tổng thể của từng annotator và từng batch dữ liệu dán nhãn. |

Metric high-risk tách riêng (ví dụ critical defect escape rate): Critical Defect Escape Rate bắt buộc phải bằng 0% (Zero Critical Escape Policy). Nếu phát hiện bất kỳ lỗi Critical nào lọt qua khâu Self-QC sang khâu QA Review, toàn bộ lô công việc (batch) bị đóng băng để audit.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  - Critical Defect Count = 0
  - Major Defect Rate <= 3.0%
  - Minor Defect Rate <= 5.0%
  - Mean Bounding Box IoU >= 0.85
  - 100% Issue flag trên CVAT đã được giải quyết (Closed/Resolved)

REWORK if:
  - Có đúng 1 lỗi Critical HOẶC
  - Major Defect Rate > 3.0% và <= 8.0% HOẶC
  - Mean Bounding Box IoU < 0.85 và >= 0.75 HOẶC
  - Còn Issue flag mở chưa được xử lý

REJECT / ESCALATE if:
  - Có từ 2 lỗi Critical trở lên trong một batch HOẶC
  - Major Defect Rate > 8.0% HOẶC
  - Mean Bounding Box IoU < 0.75 HOẶC
  - Phát hiện xung đột guideline nghiêm trọng giữa các annotator đòi hỏi cập nhật Guideline
```

Trade-off:
- Nhóm chấp nhận nới lỏng dung sai đối với Minor Defect (cho phép sai số 2-3px ở BBox và tỷ lệ lỗi minor đến 5%) nhằm tối ưu hóa chi phí và tốc độ gán nhãn, tránh việc annotator tốn quá nhiều thời gian căn chỉnh từng pixel không ảnh hưởng đến thuật toán object detection.
- Tuy nhiên, nhóm siết chặt tuyệt đối đối với Critical Defect (ngưỡng 0 lỗi) và siết chặt phân loại thuộc tính an toàn (state của đèn giao thông, sign_status cấm/nguy hiểm) vì hậu quả downstream của xe tự hành là tai nạn nghiêm trọng nếu nhận diện sai đèn đỏ hoặc bỏ sót biển cấm.
