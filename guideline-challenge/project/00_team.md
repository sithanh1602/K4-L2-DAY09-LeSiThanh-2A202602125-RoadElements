# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** Team04
- **Nhóm peer test bài của mình:** Nhóm2
- **Nhóm mình test bài của:** Nhóm2
- **Problem family:** Gán nhãn Traffic Sign, Traffic Light trên dữ liệu ảnh giao thông
- **Nguồn ảnh:** `data/dataset_vn`

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
|  Nguyễn Đăng Huân |  huandrart | Spec owner — viết bài toán và hướng dẫn gắn nhãn; cập nhật guideline và revision log |  `01_problem_statement.md`, `02_guideline.md`, `08_revision_log.md` |
| Lê Sĩ Thành | sithanh1602  | CVAT owner — cấu hình ontology, labels và task CVAT; chọn và chia bộ ảnh | `03_ontology_and_cvat_setup.md`, `03_cvat_labels.json`, `00_team.md`, `09_cvat_export_or_task_reference.txt` |
| Nguyễn Quý Toàn | ToJunn | Gold owner — viết tình huống khó, lập đáp án chuẩn và thực hiện freeze | `04_edge_cases/edge_case_cards.md`, `04_edge_cases/gold_decisions.csv` |
| Nguyễn Đức Văn | noichducvan-del | QA owner — lập kế hoạch kiểm tra chất lượng, tổng hợp calibration và kết quả blind test | `05_qa_plan.md`, `06_calibration_report.csv`, `07_blind_handoff/clarification_log.csv`, `07_blind_handoff/peer_feedback.md` |

## Phối hợp thực hiện

- Mỗi file có một người sửa chính; các thành viên khác góp ý và cung cấp bằng chứng.
- Cả 4 thành viên gắn nhãn độc lập cùng bộ calibration, sau đó gửi export mang tên mình để QA owner tổng hợp vào `06_calibration_exports/`.
- CVAT owner chốt sample pack trước khi freeze. Gold owner là người duy nhất chạy freeze để tạo `FREEZE.txt`.
- QA owner chạy công cụ để tạo `06_calibration_measure.csv`, tổng hợp bất đồng; Spec owner cập nhật guideline v2 và revision log.
- QA owner lưu export của peer vào `07_blind_handoff/peer_output/`, điều phối chấm `transfer_score.csv` và chạy tính `gts_summary.md`; các thành viên cùng đối chiếu với gold đã freeze.
- Sau blind test, Spec owner cập nhật guideline v3 và revision log từ bằng chứng do cả nhóm cung cấp.
- Các file kết quả do công cụ sinh ra sẽ xuất hiện khi thực hiện bước tương ứng; không cần tự tạo trước.
- Nguyễn Đức Văn bổ sung username hoặc đường dẫn hồ sơ GitHub thay cho email ở cột GitHub.
