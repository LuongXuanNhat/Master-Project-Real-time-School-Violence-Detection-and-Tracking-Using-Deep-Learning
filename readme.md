## 🎓 1. Tên đề tài

### (VN) Xây dựng hệ thống phát hiện bạo lực học đường thời gian thực bằng học sâu

## 📚 2. Đề cương luận văn

### 📘 CHƯƠNG 1 – GIỚI THIỆU

1.1. Bối cảnh bạo lực học đường tại Việt Nam và thế giới
1.2. Hạn chế của hệ thống camera truyền thống
1.3. Mục tiêu luận văn
1.4. Bài toán nghiên cứu

> Phát hiện người
> Theo dõi đối tượng
> Nhận dạng được hành vi bạo lực
> Cảnh báo thời gian thực

1.5. Phạm vi nghiên cứu
1.6. Đóng góp chính của luận văn
1.7. Cấu trúc luận văn

### 📘 CHƯƠNG 2 – TỔNG QUAN LÝ THUYẾT

2.1. Học sâu và thị giác máy tính
2.2. Phát hiện đối tượng: YOLO, RT-DETR
2.3. Theo dõi đối tượng: DeepSORT, ByteTrack
2.4. Nhận dạng hành động trong video

> CNN + LSTM
> I3D
> SlowFast
> X3D
> ViViT / Video Swin Transformer

2.5. Anomaly detection trong video giám sát
2.6. Các chỉ số đánh giá

> mAP (object detection)
> MOTA/MOTP (tracking)
> Accuracy, F1, ROC-AUC (violence detection)

2.7. Các nghiên cứu liên quan trong nước và quốc tế

### 📘 CHƯƠNG 3 – PHƯƠNG PHÁP ĐỀ XUẤT & THIẾT KẾ HỆ THỐNG

3.1. Mô hình tổng thể hệ thống
3.2. Module 1 – Object Detection

> Lý do chọn YOLOv8/YOLOv10 (hoặc RT-DETR)
> Tối ưu chạy real-time

3.3. Module 2 – Object Tracking

> So sánh DeepSORT vs ByteTrack
> Chọn mô hình phù hợp cho camera trường học

3.4. Module 3 – Violence Detection

> Hai lựa chọn:
> (A) Action Recognition-based (SlowFast/ViViT/I3D)
> (B) Violence Binary Classification (RWF-2000)

3.5. Kết hợp tracking + violence recognition
3.6. Cơ chế cảnh báo sự kiện
3.7. Công nghệ sử dụng
Python, OpenCV, PyTorch
GPU: RTX 3060/3090 hoặc T4
3.8. Tóm tắt chương

### 📘 CHƯƠNG 4 – THỰC NGHIỆM & ĐÁNH GIÁ

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

### 📘 CHƯƠNG 5 – KẾT LUẬN & HƯỚNG PHÁT TRIỂN

5.1. Tóm tắt đóng góp
5.2. Hiệu quả đạt được
5.3. Các vấn đề còn tồn tại
5.4. Hướng mở rộng:

> Multi-camera tracking
> Ứng dụng transformer-based video
> Edge computing (Jetson Nano)
> Privacy-preserving AI

# ⚙️ 3. Pipeline kỹ thuật A → Z

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

**Cách A** – `Dùng Video Action Recognition (chất lượng cao)`
Input: 16–32 frames của đối tượng đang nghi ngờ
Model:

SlowFast (best)
ViViT (transformer-based)
X3D (nhẹ + real-time)

Output:
`"Fight" / "Non-Fight"`

**Cách B** – `Dùng Video-level Classification (dễ làm)`

Dataset: RWF-2000
Backbone: ResNet50 + LSTM / 3D CNN

# 🗂️ 4. Mô hình nên dùng (tối ưu nhất)

Nhiệm vụ Mô hình nên dùng Lý do
Phát hiện người YOLOv8s hoặc YOLOv10n Nhẹ, FPS cao, chính xác
Tracking ByteTrack Ổn định, tốt hơn DeepSORT
Violence detection (real-time) X3D hoặc SlowFast chính xác cao nhất
Violence detection (dễ làm) ResNet50 + LSTM Training đơn giản

**YOLOv8** (Ultralytics):
Lý do chọn:

> Hỗ trợ Pose Estimation (Quan trọng nhất): YOLOv8 có phiên bản yolov8-pose. Nó không chỉ phát hiện người mà còn vẽ được khung xương (khớp vai, khuỷu tay, cổ tay...). Để phân biệt Đùa giỡn và Đánh nhau, bạn cần phân tích sự di chuyển của các khớp xương này chứ không chỉ là cái khung bao quanh người (Bounding Box).
> Tích hợp sẵn Tracking: YOLOv8 tích hợp sẵn BoT-SORT và ByteTrack. Bạn cần cái này để theo dõi một học sinh cụ thể qua các khung hình liên tiếp xem họ đang di chuyển nhanh hay chậm.
> Cộng đồng cực lớn: Khi làm luận văn, gặp lỗi (bug) là chuyện thường. YOLOv8 có cộng đồng hỗ trợ lớn nhất, dễ tìm cách fix lỗi nhất.
> Khuyên dùng: yolov8n-pose (bản nano) hoặc yolov8s-pose (bản small) để đạt tốc độ thời gian thực cao nhất trên thiết bị hạn chế.

**YOLOv10** (Tsinghua University) - Lựa chọn cho tốc độ (Speed)
Nếu tiêu chí của bạn là "Real-time" trên các thiết bị yếu (như Jetson Nano, Raspberry Pi hay Laptop không có GPU xịn), YOLOv10 là ứng cử viên số 1.
Lý do chọn:

> Loại bỏ NMS (Non-Maximum Suppression): Các bản YOLO cũ tốn thời gian để lọc bớt các khung hình trùng nhau. YOLOv10 bỏ bước này, giúp độ trễ (latency) giảm đáng kể.
> SOTA về hiệu năng: Ở cùng một mức độ chính xác, YOLOv10 chạy nhanh hơn và tốn ít tài nguyên tính toán hơn v8 hay v9.
> Tốt cho môi trường đông đúc: Trong trường học giờ ra chơi, học sinh đứng chen chúc. YOLOv10 xử lý hiện tượng chồng lấn (occlusion) khá tốt.
> Nhược điểm: Hiện tại tập trung mạnh vào Object Detection, phần hỗ trợ Pose Estimation chưa mạnh mẽ và "mì ăn liền" bằng hệ sinh thái của Ultralytics (v8/v11).

#### So sánh 3 ứng cử viên: ResNet+LSTM, SlowFast, X3D

Để phân biệt được `"Đánh thật"` (lực mạnh, tốc độ cao, gia tốc lớn) và `"Đùa giỡn"` (lực nhẹ, ngập ngừng), mô hình cần khả năng hiểu sâu về Temporal (Thời gian/Chuyển động).
**ResNet50 + LSTM**:
Cơ chế: ResNet trích xuất đặc trưng từng ảnh, LSTM xâu chuỗi lại.
Đánh giá: Lỗi thời (Outdated). Mô hình này tách biệt không gian và thời gian quá rạch ròi. Nó rất yếu trong việc cảm nhận sự thay đổi tinh tế về tốc độ (ví dụ: cú đấm nhanh vs cú vung tay chậm).
Kết luận: Không nên dùng nếu muốn độ chính xác cao cho bài toán khó này.
**SlowFast** (Facebook AI):
Cơ chế: 2 nhánh song song. Nhánh "Chậm" nhìn chi tiết hình ảnh, nhánh "Nhanh" nhìn chuyển động.
Đánh giá: Rất chính xác. Nó là chuẩn mực để phát hiện hành động trong vài năm trước. Khả năng phân biệt đánh/đùa rất tốt.
Nhược điểm: Nặng. Nó ngốn tài nguyên tính toán lớn, khó chạy Real-time nếu phần cứng không rất mạnh.
**X3D** (Facebook AI - Bản nâng cấp của SlowFast):
Cơ chế: Mạng 3D ConvNet được tối ưu hóa cực đại (Efficient). Nó mở rộng mô hình theo nhiều chiều (frame rate, duration, resolution...) một cách thông minh.
Đánh giá: Chân ái cho Real-time. X3D nhẹ hơn SlowFast rất nhiều lần nhưng độ chính xác tương đương (thậm chí cao hơn trong một số bộ dữ liệu).

> **Kết luận: Nên chọn X3D. Đây là SOTA (State-of-the-Art) cho các bài toán nhận diện hành động hiệu quả (Efficient Action Recognition)**.

###### Phương án thay thế nhẹ hơn: Pose-based (Gợi ý thêm)

Nếu máy tính của bạn chạy X3D vẫn bị lag (FPS thấp), hãy xem xét phương án dùng Skeleton (Bộ xương):
Mô hình: YOLOv8-Pose + ST-GCN (Spatial Temporal Graph Convolutional Network).
Lý do:
X3D xử lý video (pixel RGB) nên vẫn khá nặng.
ST-GCN chỉ xử lý tọa độ các điểm khớp xương (file text tọa độ) -> Siêu nhẹ, siêu nhanh.
Để phân biệt "Đùa" và "Đánh", gia tốc của khớp cổ tay và khuỷu tay là đặc trưng rõ nhất. ST-GCN học đặc trưng này cực tốt.

## So sánh Skeleton Model vs X3D cho phát hiện bạo lực học đường

### Skeleton Model thắng áp đảo

**Kiến trúc đề xuất:**

- Detector: YOLOv8-Pose (lấy tọa độ xương + bounding box)
- Classifier: ST-GCN hoặc LSTM

---

## 4 lý do Skeleton Model phù hợp với trường học

### 1. Privacy (Bảo vệ quyền riêng tư)

**X3D:**

- Sử dụng hình ảnh RGB trực tiếp
- Mặt mũi, quần áo, không gian lớp đều vào mô hình
- Nhạy cảm với trẻ vị thành niên

**Skeleton:**

- Chỉ xử lý tọa độ điểm (khuỷu tay, đầu gối, vai...)
- Có thể vứt bỏ hình ảnh gốc sau khi detect
- Chỉ lưu metadata "xương que củi"

**→ Tính nhân văn và khả thi cao cho luận văn**

### 2. Chống nhiễu nền (Background Robustness)

**X3D:**

- Học trên pixel → dễ bị "lừa" bởi phông nền
- Trường học lộn xộn: bàn ghế, sách vở, tranh ảnh
- Có thể học thuộc góc tường thay vì hành vi

**Skeleton:**

- Loại bỏ hoàn toàn nhiễu nền
- Chuyển động xương giống nhau dù đánh ở đâu (lớp, hành lang, sân)
- Generalization tốt hơn

### 3. Phân biệt "Lực" (Đùa vs Thật) - Yêu cầu cốt lõi

**X3D:**

- CNN khó cảm nhận gia tốc/lực từ pixel màu sắc

**Skeleton:**

- Có tọa độ (x, y) theo thời gian t
- Tính toán trực tiếp:
  - **Vận tốc**: Khoảng cách di chuyển giữa 2 frame
  - **Gia tốc**: Độ thay đổi vận tốc
- Đấm thật: gia tốc cực lớ, giật cục
- Đấm đùa: đều đều, chậm hơn

**→ ST-GCN cực kỳ nhạy bén với đặc trưng vật lý này**

### 4. Góc quay CCTV (High angle view)

**X3D:**

- Pre-train trên Kinetics-400 (quay ngang: phim/youtube)
- Góc trên cao: đầu to chân bé → mô hình "ngáo"
- Dễ bị biến dạng

**Skeleton:**

- Quan hệ giữa khớp xương không đổi (tay-vai, đầu-cổ)
- YOLOv8-Pose train trên COCO đa dạng góc độ
- Bắt xương tốt hơn bắt pixel

---

## Nhược điểm duy nhất của Skeleton

### Tương tác với đồ vật

- Học sinh dùng ghế/sách ném nhau:
  - **X3D**: Nhìn thấy ghế/sách → Nhận diện tốt
  - **Skeleton**: Chỉ thấy người múa tay → Nhận diện kém

### Khắc phục

- Giới hạn phạm vi: "Bạo lực cơ học (tay chân)"
- Train thêm YOLO để detect vật nguy hiểm (ghế, gậy)

---

## Lý do chọn Skeleton Model

✅ **Nhẹ hơn**: Real-time mượt trên máy cấu hình vừa

✅ **Chính xác hơn**: Phân loại hành động giả vờ nhờ tính gia tốc

✅ **Thân thiện môi trường học đường**: Privacy + chống nhiễu nền

# 🧪 5. Dataset phù hợp

Dataset bạo lực trong video giám sát:

- WF-2000 (2000 video camera giám sát → phù hợp nhất)
- Hockey Fight Dataset
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

# 📈 6. Baseline để so sánh trong chương 4

Thành phần Baseline Mô hình đề xuất
Detection YOLOv5s YOLOv8s / YOLOv10n
Tracking DeepSORT ByteTrack
Violence 3D-CNN SlowFast/X3D

# 📅 7. Lộ trình 3–6 tháng

| Tháng       | Mục tiêu chính                       | Công việc chi tiết                                                           |
| ----------- | ------------------------------------ | ---------------------------------------------------------------------------- |
| **Tháng 1** | Nghiên cứu lý thuyết                 | • Đọc tài liệu<br>• Chọn mô hình<br>• Thu dữ liệu                            |
| **Tháng 2** | Xây dựng module detection + tracking | • Train/finetune YOLO<br>• Kết hợp ByteTrack<br>• Đạt 20–30 FPS              |
| **Tháng 3** | Xây dựng module phát hiện bạo lực    | • Chuẩn bị RWF-2000<br>• Train SlowFast/X3D<br>• Benchmark accuracy          |
| **Tháng 4** | Tích hợp hệ thống                    | • Dây pipeline<br>• Test với video dài<br>• Thử nghiệm camera 720p/1080p     |
| **Tháng 5** | Viết luận văn                        | • Chương 1–3<br>• Chương 4 thực nghiệm<br>• Chương 5 kết luận                |
| **Tháng 6** | Chỉnh sửa & chuẩn bị bảo vệ          | • Hoàn thiện luận văn<br>• Chuẩn bị slide thuyết trình<br>• Tập luyện bảo vệ |
