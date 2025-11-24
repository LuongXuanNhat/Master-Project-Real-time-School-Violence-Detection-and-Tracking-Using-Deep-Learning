Tuyệt! Mình sẽ xây dựng toàn bộ bộ tài liệu nền tảng cho đề tài luận văn thạc sĩ của bạn theo đúng trình tự:

Tên đề tài tối ưu
Đề cương 5 chương chi tiết (chuẩn trường ĐH/Viện nghiên cứu)
Pipeline kỹ thuật từ A–Z
Gợi ý mô hình mạnh nhất & nhẹ nhất
Dataset phù hợp + cách chuẩn bị dữ liệu
Các baseline để so sánh khi thực nghiệm
Kế hoạch thực hiện 3–6 tháng

🎓 1. Tên đề tài (chọn 1 trong 3 đề xuất đẹp nhất)
(VN) 1. Xây dựng hệ thống phát hiện và theo dõi hành vi bạo lực học đường thời gian thực bằng học sâu
(VN) 2. Nhận dạng và cảnh báo sớm nguy cơ bạo lực trong khuôn viên trường học sử dụng YOLO và mô hình phân tích hành vi video
(EN) 3. Real-time School Violence Detection and Tracking Using Deep Learning

Bạn thích tên nào, mình sẽ dựa vào đó để đóng khung luận văn.

📚 2. Đề cương luận văn (5 chương, bản chi tiết nhất)
📘 CHƯƠNG 1 – GIỚI THIỆU

1.1. Bối cảnh bạo lực học đường tại Việt Nam và thế giới
1.2. Hạn chế của hệ thống camera truyền thống
1.3. Mục tiêu luận văn
1.4. Bài toán nghiên cứu

Phát hiện người

Theo dõi đối tượng

Nhận dạng hành vi bạo lực

Cảnh báo thời gian thực
1.5. Phạm vi nghiên cứu
1.6. Đóng góp chính của luận văn
1.7. Cấu trúc luận văn

📘 CHƯƠNG 2 – TỔNG QUAN LÝ THUYẾT

2.1. Học sâu và thị giác máy tính
2.2. Phát hiện đối tượng: YOLO, RT-DETR
2.3. Theo dõi đối tượng: DeepSORT, ByteTrack
2.4. Nhận dạng hành động trong video

CNN + LSTM

I3D

SlowFast

X3D

ViViT / Video Swin Transformer
2.5. Anomaly detection trong video giám sát
2.6. Các chỉ số đánh giá

mAP (object detection)

MOTA/MOTP (tracking)

Accuracy, F1, ROC-AUC (violence detection)
2.7. Các nghiên cứu liên quan trong nước và quốc tế

📘 CHƯƠNG 3 – PHƯƠNG PHÁP ĐỀ XUẤT & THIẾT KẾ HỆ THỐNG

3.1. Mô hình tổng thể hệ thống
3.2. Module 1 – Object Detection

Lý do chọn YOLOv8/YOLOv10 (hoặc RT-DETR)

Tối ưu chạy real-time
3.3. Module 2 – Object Tracking

So sánh DeepSORT vs ByteTrack

Chọn mô hình phù hợp cho camera trường học
3.4. Module 3 – Violence Detection
Hai lựa chọn:

(A) Action Recognition-based (SlowFast/ViViT/I3D)

(B) Violence Binary Classification (RWF-2000)
3.5. Kết hợp tracking + violence recognition
3.6. Cơ chế cảnh báo sự kiện
3.7. Công nghệ sử dụng

Python, OpenCV, PyTorch

GPU: RTX 3060/3090 hoặc T4
3.8. Tóm tắt chương

📘 CHƯƠNG 4 – THỰC NGHIỆM & ĐÁNH GIÁ

4.1. Môi trường thực nghiệm
4.2. Chuẩn bị dữ liệu
4.3. Thực nghiệm 1 – Phát hiện người

So sánh YOLOv7, YOLOv8, YOLOv10
4.4. Thực nghiệm 2 – Theo dõi đối tượng

DeepSORT vs ByteTrack
4.5. Thực nghiệm 3 – Nhận dạng bạo lực

SlowFast vs ViViT vs ResNet + LSTM

Hoặc: RWF-2000 vs Hockey Fight Dataset
4.6. Thực nghiệm 4 – Hệ thống hoàn chỉnh

FPS

Độ trễ (latency)

Khi có nhiều học sinh
4.7. Phân tích và thảo luận kết quả
4.8. Hạn chế

📘 CHƯƠNG 5 – KẾT LUẬN & HƯỚNG PHÁT TRIỂN

5.1. Tóm tắt đóng góp
5.2. Hiệu quả đạt được
5.3. Các vấn đề còn tồn tại
5.4. Hướng mở rộng:

Multi-camera tracking

Ứng dụng transformer-based video

Edge computing (Jetson Nano)

Privacy-preserving AI

⚙️ 3. Pipeline kỹ thuật A → Z
Camera → (1) Object Detection → (2) Tracking → (3) Violence Recognition →
→ (4) Event Decision → (5) Cảnh báo → (6) Lưu video

(1) Object Detection

Model: YOLOv8n/v8s/v10n

Input: mỗi khung hình từ camera

Output: bounding boxes người

(2) Tracking

Model: ByteTrack (khuyến nghị 2025)

Input: box người + confidence

Output: ID của từng học sinh

(3) Violence Recognition

2 cách để làm:

Cách A – Dùng Video Action Recognition (chất lượng cao)

Input: 16–32 frames của đối tượng đang nghi ngờ
Model:

SlowFast (best)

ViViT (transformer-based)

X3D (nhẹ + real-time)

Output:
"Fight" / "Non-Fight"

Cách B – Dùng Video-level Classification (dễ làm)

Dataset: RWF-2000
Backbone: ResNet50 + LSTM / 3D CNN

🗂️ 4. Mô hình nên dùng (tối ưu nhất)
Nhiệm vụ	Mô hình nên dùng	Lý do
Phát hiện người	YOLOv8s hoặc YOLOv10n	Nhẹ, FPS cao, chính xác
Tracking	ByteTrack	Ổn định, tốt hơn DeepSORT
Violence detection (real-time)	X3D hoặc SlowFast	SlowFast chính xác cao nhất
Violence detection (dễ làm)	ResNet50 + LSTM	Training đơn giản
🧪 5. Dataset phù hợp
Dataset bạo lực trong video

RWF-2000 (2000 video camera giám sát → phù hợp nhất)

Hockey Fight Dataset

Surveillance Fight Dataset

UCF Crime (nhiều loại hành vi bất thường)

Dataset người / hành động (phụ trợ)

COCO (people)

CrowdHuman

Human3.6M

Dataset tự thu thập

Bạn có thể:

Để bạn bè/diễn viên mô phỏng xô đẩy nhẹ

Dùng áo khẩu trang để tránh lộ danh tính

Gắn blur mặt → đảm bảo đạo đức nghiên cứu

📈 6. Baseline để so sánh trong chương 4
Thành phần	Baseline	Mô hình đề xuất
Detection	YOLOv5s	YOLOv8s / YOLOv10n
Tracking	DeepSORT	ByteTrack
Violence	3D-CNN	SlowFast/X3D

Bạn sẽ có bảng so sánh rất đẹp trong luận văn.

📅 7. Lộ trình 3–6 tháng
Tháng 1 – Nghiên cứu lý thuyết

Đọc tài liệu

Chọn mô hình

Thu dữ liệu

Tháng 2 – Xây dựng module detection + tracking

Train/finetune YOLO

Kết hợp ByteTrack

Đạt 20–30 FPS

Tháng 3 – Xây dựng module phát hiện bạo lực

Chuẩn bị RWF-2000

Train SlowFast/X3D

Benchmark accuracy

Tháng 4 – Tích hợp hệ thống

Dây pipeline

Test với video dài

Thử nghiệm camera 720p/1080p
(Có thể mua camera: )

Tháng 5 – Viết luận văn

Chương 1–3

Chương 4 thực nghiệm

Chương 5 kết luận

Tháng 6 – Chỉnh sửa & chuẩn bị bảo vệ
🎯 Bạn muốn bước tiếp theo là gì?

Mình có thể giúp bạn:

✔ Viết toàn bộ đề cương chi tiết 5 chương (bản dài 20–30 trang)
✔ Viết phần Giới thiệu (Chương 1) đầy đủ
✔ Viết Chương 2 – Tổng quan mô hình
✔ Phác thảo code minh họa pipeline YOLO + ByteTrack
✔ Soạn slide thuyết trình bảo vệ

👉 Bạn muốn mình viết trước Chương 1, hay viết đề cương dài, hay bắt đầu từ pipeline kỹ thuật?