# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Trần An Hạ

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng đệm ở giữa, thay vì chia ngẫu nhiên, vì camera đứng ở một vị trí cố định và các frame liền kề rất giống nhau. Nếu chia ngẫu nhiên, cùng một cảnh hoặc cùng một xe sẽ xuất hiện đồng thời ở cả tập học và tập test; khi đó, mô hình có thể đạt điểm cao vì nhìn thấy cùng sự kiện nhưng không thực sự tổng quát hóa vào dữ liệu mới. Khi dữ liệu bị rò rỉ theo thời gian, AP50 sẽ bị đánh giá quá lạc quan và không phản ánh chất lượng thật của mô hình trên dữ liệu mới.

## 2. Mô hình khởi đầu lạnh (cold start)

Dòng vòng 0 trong `outputs/reports/rounds_table.md` là:

- vòng 0, model = yolov8n cold start (COCO car+bus+truck)
- ảnh train = 0
- box train = 0
- AP50 = 0.771
- P@0.25 = 0.925
- R@0.25 = 0.489
- F1 = 0.640
- R small = 0.182
- R medium = 0.547
- R large = 0.561

Dựa trên `outputs/metrics_round0.json`, mô hình khởi đầu lạnh không khớp nhãn tham chiếu tốt nhất ở các xe nhỏ và xe ở xa. Recall theo kích thước cho thấy rõ: xe nhỏ chỉ đạt 0.182, trong khi xe trung bình và lớn lần lượt là 0.547 và 0.561. Điều này cho thấy model bỏ sót nhiều xe ở xa hoặc có độ sáng thấp, đặc biệt ở đêm trên đường cao tốc. Một trường hợp cần xem lại nhãn tham chiếu trước khi kết luận mô hình sai là các xe rất xa, chỉ còn hai chấm đèn, hoặc xe bị cắt mép ảnh: ở những trường hợp đó, việc xem là “có xe” hay “không có xe” còn phụ thuộc vào quy tắc lab và chiều rộng khung đánh dấu, nên cần kiểm tra lại trước khi kết luận mô hình luôn thiếu.

## 3. Chiến lược chọn mẫu

Chiến lược chọn mẫu dựa theo score = W_U·U + W_A·A + W_D·D. Theo đó, phần lớn trọng số nằm ở độ bất định U (uncertainty), phần còn lại ở mức độ khó A và của ảnh có nhiều box mơ hồ D, còn trong cùng một cảnh cũng xét khoảng cách thời gian để tránh ảnh gần trùng. `MIN_GAP_S` giúp loại bớt các frame rất gần nhau về thời điểm, vì trong camera cố định, hai ảnh liền kề thường gần như cùng một cảnh và sửa cả hai mang lại ít giá trị học hơn so với chọn ảnh đa dạng.

Dấu hiệu rõ nhất xuất hiện ở ba frame trong `reports/SELECTION.md`:
- `frame_0182.jpg` có score cao nhất và là cảnh rất khó vì nhiều xe gần nhau và xe xa.
- `frame_0369.jpg` có score 0.9324, nhiều xe cùng lúc trong khung nhìn, rất phù hợp để bổ sung box thiếu.
- `frame_0380.jpg` có score 0.917, cảnh có xe sáng tối xen kẽ và cần chỉnh nhiều box lệch.

Một frame khác đáng để xem là `frame_0372.jpg`: dù điểm cao (0.9101), nó không được chọn vì nó gần thời điểm với `frame_0369.jpg` và cùng một cảnh. Khi hai ảnh gần như cùng một cảnh, sửa cả hai tốn công mà ít tăng giá trị học. Điều này chứng minh rằng điểm cao không đồng nghĩa với việc ảnh đó chắc chắn cải thiện mô hình; nó chỉ cho biết ảnh này có nhiều thông tin chưa chắc chắn và đáng xem xét.

## 4. Các vòng học chủ động (active learning)

Từ dữ liệu thực tế trong `outputs/metrics_round1.json` và `outputs/round1_diff.md`, vòng 1 có thông số sau:

- model: `yolov8n fine-tune vong 1..1`
- ảnh train: 12
- box train: 346
- AP50: 0.6176
- Δ AP50 so cold start: -0.154
- precision: 1.000
- recall: 0.0918
- F1: 0.1682

`round1_diff.md` cho biết khối lượng sửa nhãn gợi ý trên 12 ảnh như sau:

- model đề xuất: 169 box
- sau khi sửa: 346 box
- accepted: 112
- edited: 40
- deleted: 17
- added: 194
- accept rate: 66%

Những con số này cho thấy khối lượng sửa nhãn rất lớn và hầu hết là bổ sung xe bị AI bỏ sót: 194 box được thêm vào. Tuy nhiên, kết quả cuối cùng không cải thiện AP50; vòng 1 giảm từ 0.771 xuống 0.618, tức là mất 0.154 so với cold start. Điều này cho thấy việc sửa nhãn gợi ý chưa đủ để cải thiện mô hình trên tập test trong trường hợp này, và có thể do ảnh học quá nhỏ, tập dữ liệu quá khó, hoặc là các frame chọn không mang lại thêm độ tổng quát.

Mỗi ca sửa đã được lưu trong `reports/REVIEW_LOG.csv`:
- `frame_0099.jpg`: thêm xe tối ở mép trái do AI bỏ sót.
- `frame_0107.jpg`: xóa vệt đèn phản quang trên mặt đường, không phải xe.
- `frame_0326.jpg`: kéo khung xe giữa làn về đúng thân xe.

Các ca này phản ánh đúng lý do theo guideline: thiếu xe, không phải xe, box lệch. Điều này tách biệt rõ với quan sát độc lập trong `reports/BLIND_SCAN.md`, nơi người đánh giá nhìn bằng mắt trước khi xem nhãn AI; vì vậy, quan sát độc lập và chỉnh sửa nhãn là hai hoạt động khác nhau. Có một ca khó theo guideline là trường hợp xe rất xa ở mép ảnh hoặc dưới ánh phản quang, nơi phần thân nhìn không rõ và chỉ còn đèn hoặc mép xe; ở đây cần cân nhắc quyết định bỏ hoặc giữ tùy theo mức độ nhìn thấy của thân xe và vùng bị che.

## 5. Kết luận và giới hạn

So với cold start, model khởi đầu lạnh có AP50 0.771, trong khi vòng 1 chỉ còn 0.618. Điều này cho thấy việc fine-tune trên 12 ảnh từ lô chọn đã làm recall và F1 giảm mạnh: recall giảm từ 0.489 xuống 0.0918, còn F1 từ 0.640 xuống 0.1682. Nhóm xe nhỏ gần như không được phát hiện ở vòng 1 (recall small = 0.0), còn xe lớn vẫn giữ được một phần recall (0.5122). Đây là dấu hiệu rằng mô hình tương đối tốt trên các xe lớn nhưng mất hẳn khả năng nhận diện xe nhỏ và xe ở xa khi fine-tune trên nhãn đã chỉnh sửa không phù hợp.

Hai trường hợp còn yếu cần quan tâm cho vòng sau là:
1. xe nhỏ ở xa, chỉ còn một hoặc hai chấm đèn; chi phí rà nhãn ở đây khá thấp nhưng dễ nhầm nếu không kiểm kỹ.
2. xe gần mép ảnh hoặc bị che một phần thân; đây là trường hợp dễ cho AI bỏ sót hoặc box lệch, và nếu chọn quá nhiều ảnh như vậy có thể làm tăng chi phí mà không cải thiện rõ model.

Tập kiểm thử chỉ có 20 ảnh, và các nhãn tham chiếu cũng có quy tắc bỏ qua xe quá nhỏ. Đây là giới hạn quan trọng: khi khối lượng dữ liệu test nhỏ và có lớp xe rất nhỏ bị bỏ qua, kết luận về AP50 sẽ bị ảnh hưởng mạnh bởi những trường hợp ngoại lệ. Nếu AP50 giảm trong vòng sau, điều đầu tiên cần kiểm tra là xem liệu do lỗi của pre-label sửa sai, do ảnh gần trùng, hay do model bị overfit vào các khung quá khó mà không cải thiện các trường hợp thực tế.
