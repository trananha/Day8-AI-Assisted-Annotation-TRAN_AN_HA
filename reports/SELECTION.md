# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, tôi ưu tiên năm frame sau nếu chỉ có ngân sách rà năm ảnh: 

1. `frame_0182.jpg` — score 0.9591, hạng 1. Ảnh này có độ bất định cao, bài toán bên trong là nhiều xe sáng và tối xen kẽ, đồng thời khung AI có nhiều trường hợp thiếu xe ở mép trái và mép phải. Đây là ảnh có giá trị học tốt nhất trong lô chọn.
2. `frame_0369.jpg` — score 0.9324, hạng 2. Ảnh này có nhiều xe trong cùng khung nhìn, mức độ mơ hồ cao và nhiều ô tô ở kích thước trung bình, nên việc bổ sung/điều chỉnh box sẽ cải thiện model đáng kể.
3. `frame_0380.jpg` — score 0.9170, hạng 3. Ảnh này có nhiều xe có khoảng cách gần nhau và một số box AI đang chồng nhau hoặc lệch vị trí; đây là trường hợp tốt để sửa nhãn để tăng độ chính xác khoanh xe.
4. `frame_0326.jpg` — score 0.9155, hạng 4. Ảnh có nhiều xe ở làn giữa, và AI đã bỏ sót nhiều xe gần nhau. Đây là kiểu cảnh mà thêm nhãn sẽ tăng khả năng nhận diện các xe sát nhau trong điều kiện đêm.
5. `frame_0331.jpg` — score 0.9154, hạng 5. Ảnh này có nhiều xe ở vùng sáng tối và ở mép hình, làm model dễ mắc lỗi. Vì vậy nó phù hợp để nâng chất lượng học trên các trường hợp xe bị che hoặc gần mép ảnh.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet:
- `frame_0182.jpg` — score 0.9591, nằm trong top 5 và là ảnh biểu tượng cho trường hợp có nhiều xe và nhiều lỗi bỏ sót.
- `frame_0369.jpg` — score 0.9324, là cảnh nhiều xe trong cùng một góc nhìn, rất có giá trị cho học chủ động.
- `frame_0107.jpg` — score 0.8876, nằm trong 12 ảnh chọn nhưng có mức độ bất định cao; AI đã vẽ nhiều khung lạ nhưng cũng có một số xe bị bỏ sót.

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem:
- `frame_0372.jpg` — score 0.9101, điểm cao hơn một số ảnh đã chọn nhưng không được chọn vì nó khá gần với `frame_0369.jpg`, tức là cảnh tương tự và lại rất sát thời điểm. Hai ảnh gần như cùng một cảnh nên sửa cả hai tốn công mà không mang lại thêm nhiều thông tin mới cho học.
- `frame_0270.jpg` — score 0.8878, thấp hơn top 5 nhưng vẫn thuộc 12 ảnh AI chọn; còn đáng xem vì nó có thể chứa các xe ở xa và mép ảnh, nơi model dễ nhầm do độ mờ và phản quang.

Điều phép chọn này chưa chứng minh về chất lượng mô hình:
- Kết quả chọn ảnh chỉ phản ánh độ bất định và độ khó của lô mà AI đang đánh giá, không phải là bằng chứng rằng model sẽ cải thiện ngay lập tức sau khi sửa. Một điểm cao chỉ cho thấy ảnh đó là "hữu ích để xem xét" chứ không auto-khẳng định là ảnh sẽ làm tăng AP50. Ở đây, các ảnh được chọn đều có yếu tố khó như xe xa, xe sát nhau, ánh sáng đêm và mép ảnh; những trường hợp này đúng là quan trọng nhưng vẫn cần kiểm thử thực tế trên tập test.
