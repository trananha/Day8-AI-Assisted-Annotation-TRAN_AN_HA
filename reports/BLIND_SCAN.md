# Quét độc lập trước khi xem pre-label

Frame: `outputs/to_label/round1/images/train/frame_0107.jpg`

Số xe nhìn thấy bằng mắt: 27

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe:
- Vị trí 1: Hai xe phía xa ở làn đường trên cùng, bên trái, có đèn xe rất mờ và nằm trên nền tối nên dễ bị bỏ sót.
- Vị trí 2: Một nhóm xe ở giữa làn đường trung tâm, ánh đèn và phản quang khiến các xe sát nhau dễ bị vẽ sai khung hoặc nhầm thành một xe.

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
