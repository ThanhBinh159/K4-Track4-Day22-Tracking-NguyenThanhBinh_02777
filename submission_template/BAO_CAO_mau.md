# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** BBQ **Thành viên:** Nguyễn Thanh Bình

Detector cố định: `yolo26n.pt`, ảnh 640 px, lớp người, Re-ID `osnet_x0_25_msmt17.pt`.

## 1. Cấu hình đã chọn

Các nhận xét video dựa trên preview; chỉ `video_1` có ground truth. Thống kê số dòng/ID trong các lần thử ngoài `video_1` chỉ dùng để so sánh cấu hình, không phải điểm chất lượng.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---:|---:|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | BoTSORT | 0.30 | 0.70 | Hộp xuất hiện quanh người rõ ở các mốc 16/76/136; người ở xa vẫn dễ bị bỏ sót. | BoTSORT 0.30/0.50: HOTA 29.460, thấp hơn 29.969 của cấu hình chọn. ByteTrack 0.30/0.50: HOTA 26.912. |
| video_2 (phố đêm, tĩnh, rất đông) | ByteTrack | 0.15 | 0.50 | Cảnh tối và đông; mức conf thấp bắt thêm người nhỏ/tối. Trong 150 frame thử: 1.377 dòng, 15 ID, median 114 frame/ID; conf 0.30 có 1.313 dòng, 16 ID, median 108. Vẫn bỏ sót một số người ở xa. | ByteTrack 0.50/0.50: 1.122 dòng, median 79 frame/ID; giảm độ bao phủ trong cảnh thiếu sáng. |
| video_3 (camera di động, ảnh nhỏ) | BoTSORT | 0.15 | 0.50 | Người gần camera có hộp rất lớn do bị cắt sát khung; người nhỏ phía xa khó bắt ổn định. Trong 150 frame thử, conf 0.15 có 848 dòng, median 22 frame/ID; conf 0.30 có 765 dòng, median 15. | BoTSORT 0.50/0.50: 569 dòng, median 12 frame/ID; bỏ nhiều người nhỏ hơn. |
| video_4 (trong nhà, camera di chuyển, kính phản chiếu) | BoTSORT | 0.50 | 0.50 | Người rõ ở tiền cảnh còn hộp ở các mốc 16/76/136. Mức conf 0.50 giảm các hộp yếu; người nhỏ hoặc phản chiếu mờ có thể bị bỏ sót. | BoTSORT 0.15/0.50: 1.037 dòng, 16 ID; có nhiều hộp yếu hơn trong cảnh phản chiếu. Không có nhãn để xác nhận đó là hộp giả. |
| video_5 (trên xe bus, giao lộ đông, rung lắc) | ByteTrack | 0.30 | 0.50 | Góc nhìn rộng và rung; người ở xa có hộp thưa, một số người nhỏ khó phát hiện. | ByteTrack 0.15/0.50: chỉ thêm 20 dòng trong 150 frame, số ID và median độ dài track vẫn là 20 và 22 frame. |

## 2. Số liệu video_1

TrackEval trên cấu hình nộp `BoTSORT`, `conf=0.30`, `iou=0.70`:

| HOTA | MOTA | IDF1 | IDSW |
|---:|---:|---:|---:|
| 29.969 | 19.025 | 29.703 | 33 |

Chỉ số được tính trên `video_1`; không có số HOTA/MOTA/IDF1 cho `video_2`–`video_5`.

## 3. Phân tích

Với `video_2`, camera đứng yên nên ByteTrack có thể dựa vào chuyển động giữa các frame. Cảnh tối làm người nhỏ và ít sáng khó phát hiện; `conf=0.15` thu được nhiều dòng track hơn mức 0.30 trong lần thử. Đây là lựa chọn ưu tiên độ bao phủ; vì video không có nhãn, các hộp tăng thêm chưa thể xác nhận đều là người thật.

Với `video_3`, camera chuyển động và ảnh nhỏ làm liên kết chỉ dựa trên vị trí kém ổn định. Mình chọn BoTSORT để bổ sung đặc trưng ngoại hình; `conf=0.15` bắt được nhiều khung hơn mức 0.30 trong 150 frame đầu. Người đi sát camera vẫn tạo hộp rất lớn và người xa có thể mất ID.

Với `video_4`, BoTSORT phù hợp để thử liên kết người khi góc nhìn tiến về phía trước. Mình chọn `conf=0.50` vì cảnh có kính phản chiếu và preview cho thấy người rõ vẫn được phát hiện ở ba mốc kiểm tra. Đây là lựa chọn thận trọng với hộp yếu; có thể bỏ sót người nhỏ hoặc tối.

## 4. Nếu có thêm thời gian

Xin nhãn đánh giá cho `video_2`–`video_5` để kiểm tra các lựa chọn bằng HOTA, MOTA và IDF1 thay vì chỉ xem bằng mắt. Có GPU thì chạy lại BoTSORT cho `video_3` và `video_4` để giảm thời gian xử lý.

**Ghi chú:** Gói `LAB_DATA` ban đầu thiếu `video_1/eval_config.json` mà script chấm yêu cầu. Mình đã thêm file cấu hình này dựa trên `seqinfo.ini`; ảnh và nhãn gốc không bị sửa.
