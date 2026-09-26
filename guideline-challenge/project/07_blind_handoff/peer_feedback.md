# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:**nhóm2


## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất?
   - Quy tắc Scope (Mục 1 & 2): Phân định rõ chỉ dán nhãn Traffic Sign và Traffic Light, bỏ qua người, xe và lane marking.
   - Bảng Quick Reference (Mục 8) và Checklist kiểm tra trước Submit (Mục 6) giúp đối chiếu nhanh chóng khi hoàn thiện nhãn.
   - Quy tắc BBox sát mặt biển/housing đầu đèn, không lấy cột (Mục 2, 3.1, 4.1).

2. Rule nào mơ hồ hoặc phải tự suy diễn?
   - Xử lý các biển báo ở xa hoặc bị mờ pictogram: ban đầu còn lúng túng giữa việc cố nhận diện loại biển hay gán `unknown`.
   - Trường hợp nhiều panel biển báo gắn trên cùng một cột: dễ nhầm lẫn giữa việc vẽ 1 box gộp hay vẽ nhiều box riêng biệt.
   - Trạng thái đèn người đi bộ khi chuyển pha: chưa rõ có được gán `yellow` hay không nếu thiết bị chỉ có đèn đỏ và xanh.

3. Sample nào khiến guideline "vỡ"?
   - Sample chụp ban đêm bị chói lóa sáng đèn giao thông (không nhìn rõ tín hiệu thực tế đang bật màu gì hay chỉ là phản quang).
   - Sample có cụm nhiều biển báo phụ (supplementary) kích thước nhỏ nằm sát nhau bên dưới biển báo chính khiến khó phân định ranh giới box.

4. Attribute / default nào trong CVAT dễ gây thao tác sai?
   - Dropdown attribute `sign_status` có nhiều giá trị dễ chọn nhầm nếu không đọc kỹ (ví dụ `mandatory` vs `direction_indication`).
   - Giá trị mặc định của `state` trong traffic_light dễ bị bỏ qua nếu annotator quên chọn lại trạng thái thực tế của frame.

5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?
   - Bổ sung bảng chuẩn hóa các mã Issue (`UNCERTAIN_CLASS`, `UNCERTAIN_SIGN_STATUS`, `UNCERTAIN_BOUNDARY`, `ATTRIBUTE_CHECK`) để annotator tự tin gắn cờ escalate khi thiếu bằng chứng thay vì phải dừng lại hỏi miệng.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Peer gộp hai biển báo trên cùng một cột vào một BBox duy nhất | guideline gap | accept + revise | Đã bổ sung mục 3.1: "Hai panel/biển độc lập trên cùng cột = hai annotation riêng biệt" |
| Peer cố đoán sign_status=prohibitory khi pictogram quá mờ | guideline gap | accept + revise | Đã cập nhật mục 3.2: "Biển quá nhỏ/mờ: dùng unknown hoặc tạo Issue; không đoán mò dựa vào màu mờ" |
| Peer vẽ BBox đèn giao thông ôm trọn cả cột đèn và cần vươn | execution error | reject with evidence | Guideline mục 2 & 4.1 nêu rõ: "Box sát housing của đèn; không lấy cột, cần đèn hoặc background thừa" |
| Đèn ban đêm bị lóa sáng không rõ màu, peer đoán màu green theo dòng xe | data ambiguity | add escalation rule | Đã bổ sung mục 4.1 & 5: "Không suy ra trạng thái từ hành vi xe; nếu không rõ tạo Issue ATTRIBUTE_CHECK" |
| Peer chọn trạng thái yellow cho đèn tín hiệu người đi bộ | guideline gap | accept + revise | Đã cập nhật bảng mục 4.3: "Pedestrian: không dùng yellow nếu pedestrian signal không có yellow" |
