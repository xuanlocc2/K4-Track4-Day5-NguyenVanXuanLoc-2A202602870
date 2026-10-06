# Tổng hợp lab Kalman Filter & Probabilistic Fusion

Học viên: Nguyễn Văn Xuân Lộc — MSSV `2A202602870`.

Notebook nộp: [Lab/kalman_fusion_lab_2A202602870.ipynb](Lab/kalman_fusion_lab_2A202602870.ipynb). Bản làm việc: [kalman_fusion_lab_STUDENT.ipynb](Lab/kalman_fusion_lab_STUDENT.ipynb). Python 3.11 trong `.venv`; chạy local bằng VS Code/Jupyter. Notebook đã được chạy bằng kernel mới và lưu output.

## Các phần đã hoàn thành

| Phần | Nội dung | Kết quả |
|:--|:--|:--|
| 1–4 | Moving average, Gaussian fusion, KF 1D, NIS | Chạy code có sẵn và các check |
| 5.1 | F 4×4 và H 2×4 | Passed |
| 5.2 | KalmanFilter.predict/update | Passed, khớp KF 1D |
| Thử thách Q | So sánh Q = 0.1, 2, 20 | Có bảng, biểu đồ và nhận xét |
| 6.1 | Hợp nhất phép đo theo timestamp | Passed, xử lý timestamp trùng |
| Bài A/B | Mất tín hiệu LiDAR và sương mù | Có output/biểu đồ |
| 7.1/7b | Chi-square gating và timestamp/latency | Passed, có demo |
| 9.1/9.2 | Chẩn đoán UWB và sửa bằng gate | Nhãn đã điền, NIS sau sửa < 8 |
| Báo cáo | Bốn mục bằng số liệu thực | Đã điền trong notebook |
| 8.1 | Hàm đo range/bearing và Jacobian EKF | Passed finite-difference check và demo |
| Sau giờ | Cửa sổ 0–20 s, mất tín hiệu, câu hỏi suy ngẫm, histogram NIS | Đã chạy/điền |

## Các kết quả chính

- KF hình số 8: RMSE vị trí **4.22 → 1.57 m**; vận tốc **1.63 m/s**. Q = 2 tốt nhất trong ba giá trị thử. Tương quan sai số vị trí–vận tốc tại 30 s khoảng **0.69**.
- Fusion: LiDAR **0.254 m**, radar **0.234 m**, camera **0.968 m**, hợp nhất **0.109 m** trên cùng lưới đầu ra.
- LiDAR blackout 25–31 s: LiDAR-only **1.10 m**, fused **0.13 m**.
- Gate: bắt **30/30** ghost, loại nhầm **9/570** phép đo tốt; RMSE **3.46 → 0.28 m**.
- Latency 0.3 s: RMSE **1.83 → 0.45 m** khi dùng thời điểm thu thập và ngoại suy tới thời điểm đầu ra.
- EKF: RMSE radar chuyển tọa độ trực tiếp **1.11 m**, EKF **0.56 m**.

## Mission Lynx-07

Log riêng cho MSSV gồm 450 phép đo GPS và 900 UWB trong 90 s. Chẩn đoán từ dữ liệu công khai: **UWB / outlier_burst**; chọn **UWB / gate**. Cụm spike khoảng 54–62 s, mean NIS UWB rất cao nhưng median và residual trung bình tương đối bình thường. Không gọi chế độ tiết lộ ground truth/đáp án giảng viên.

| Cảm biến | Mean NIS | Median NIS | Residual trung bình [x, y] (m) |
|:--|--:|--:|:--|
| GPS | 3.6065 | 1.3369 | [0.0192, -0.0489] |
| UWB | 15.5212 | 1.6283 | [-0.0871, 0.1533] |

Pooled mean NIS **11.5497 → 2.1280**, đạt ngưỡng < 8. Chấp nhận **1268** phép đo, gate **82** phép đo UWB. NIS sau gate chỉ phản ánh các phép đo được chấp nhận; nhiệm vụ không có ground truth công khai để xác nhận RMSE vị trí.

Tại dòng cuối **t = 89.8 s**, vị trí **[22.9153, -20.7016] m**; `Pxx = Pyy = 0.076003 m²`. σ từng trục **0.275686 m**, độ bất định vị trí tổng hợp **0.389879 m**. Bán kính miền Gaussian 95% **0.674810 m**, với giả định P đúng; không đồng nhất độ bất định này với sai số thực.

20 s đầu chưa lộ outlier (mean NIS GPS/UWB 1.71/2.05). Sau 20 s, mean NIS được chấp nhận **2.1877**; cấu hình được chọn từ toàn log nên đây không phải holdout độc lập.

## Ý cần nhớ

Trạng thái `[x, y, vx, vy]` chứa cả vận tốc dù phép đo chỉ có vị trí. F và hiệp phương sai chéo cho phép cập nhật vận tốc từ vị trí. Predict dùng `x = F @ x`, `P = F @ P @ F.T + Q`; update dùng innovation, S và K để cân bằng dự đoán/phép đo. Hợp nhất đa cảm biến dùng chung bộ lọc với H/R riêng và đúng timestamp.

Q nhỏ dễ gây trễ khi đổi động học, Q lớn nhạy nhiễu; R cần phản ánh chất lượng thực. Mean NIS kỳ vọng bằng số chiều phép đo. Bias cần sửa offset; nhiễu đánh giá thấp cần tăng R; outlier cần gate. Gate có thể loại nhầm chuyển động thật hoặc bị lock-out, nên hệ thống thực cần quản lý chất lượng cảm biến và track.

## Nộp bài và điểm

Nộp notebook `.ipynb` đã lưu output và báo cáo bên trong. Bản tổng hợp này dùng để ôn lại. Các check học viên đều đã chạy; repo không có script/key giảng viên để xác nhận điểm chính thức cho nhãn Phần 9. Báo cáo tối đa 20 điểm và bonus tối đa +5 do giảng viên chấm; không coi các check hợp lệ là chứng nhận điểm cuối.

Bản nộp có tên đúng MSSV và cell cuối kiểm tra lại các bài tập trước khi in `✅ Lab Lynx-07 Complete`.
