## Xây dựng hệ thống phát hiện bạo lực học đường thời gian thực bằng học sâu

### Cấu trúc dữ liệu

Cấu trúc chuẩn dạng "clips + metadata"

```
Text
dataset/
    raw_videos/ (Video gốc CCTV của bạn)
        clip_001.mp4
        ...
    annotations/ (File JSON chứa Metadata - Giữ nguyên làm Master Data)
        train.json
        val.json

    # ĐÂY LÀ PHẦN MỚI
    processed_skeletons/
        train/
            clip_001_person_1.npy  <-- File chứa chuỗi tọa độ (T frames x 17 points x 3 channels)
            clip_001_person_2.npy
            ...
        val/
```

Annotations định dạng JSON mẫu:

```json
{
  "violent/clip_0001.mp4": {
    "violence_binary": 1,
    "intent": "intentional",
    "activity_type": ["punching"],
    "location": "hallway"
  },
  "non_violent/clip_0007.mp4": {
    "violence_binary": 0,
    "intent": "playful",
    "activity_type": ["running", "laughing"],
    "location": "playground"
  }
}
```

Loader đọc:
`video_path = os.path.join(root, video_key)`

# Hướng dẫn gán nhãn dữ liệu phát hiện bạo lực

## 1. Mục tiêu

Giúp model phân biệt bạo lực thực sự và các trường hợp đùa giỡn, tập võ, thể thao, va chạm tình cờ.

## 2. Cấu trúc nhãn (Taxonomy)

### Nhãn chính

- **violence_binary**: {0, 1} - Có bạo lực vật lý hay không
- **violence_intensity**: {none, low, medium, high} - Mức độ bạo lực
- **intent_estimate**: {accidental, playful, intentional, unknown} - Ý định

### Loại hành vi (activity_type)

fighting, pushing, shoving, slapping, kicking, choking, weapon_use, group_attack, bullying_nonphysical, fighting_sports, training/practice, play_fooling, trip/fall, crowd_situation

### Thông tin bổ sung

- **actor_roles**: {aggressor, victim, passerby, bystander, intervenor}
- **weapon_present**: {none, improvised, knife, stick, bottle, other}
- **age_group_est**: {child, teen, adult, unknown}
- **location_type**: {classroom, playground, hallway, canteen, gym, bus, outdoor_campus, other}
- **camera_view**: {static_cctv, handheld, phone, mobile_pov}
- **occlusion_level**: {none, partial, heavy}

## 3. Quy tắc gán nhãn

### Phân biệt Playful vs Intentional

**Playful** (đùa giỡn):

- Có cười, cử chỉ vui vẻ
- 2 bên đồng thuận
- Không tiếp tục sau đó
- Có ngữ cảnh training/coaching

**Intentional** (có ý định):

- Nạn nhân né tránh, la hét, bỏ chạy
- Hành vi tấn công liên tục
- Có dấu hiệu gây thương tích

### Training/Sport

- Có đồng phục võ, khu vực gym
- Có coach giám sát
- Tag: `activity_type=training/practice`, `intent_estimate=playful`

### Weapon & Group Attack

- Tag weapon ngay cả khi chưa gây thương tích
- Group attack: >2 người tấn công 1 người

## 4. Metadata bắt buộc

timestamp, camera_id, resolution, FPS, day/night, audio_available, source, presence_of_supervisor, presence_of_sports_equipment

## 5. Kích thước dataset đề xuất

- **Violent clips**: ≥2,000 (đa dạng kịch bản)
- **Non-violent clips**: ≥3,000-10,000 (bao gồm: normal, play, sports, training)
- **Phân chia**: 70% train / 15% val / 15% test

## 6. Công cụ annotation

- **CVAT**: Video + bounding boxes + tracking (khuyên dùng)
- **VGG VIA**: Đơn giản, offline

## 7. Datasets tham khảo

- **RWF-2000**: 2,000 CCTV clips (violent/non-violent) - baseline
- **Violent-Flows**: Crowd violence (~246 clips)
- **Hockey Fight Dataset**: Fight/non-fight trong thể thao
- **Kaggle/Zenodo**: Nhiều datasets có thể mix

## 8. Quy trình annotation

### Bước 1: Chuẩn bị

- Thiết kế guideline 10-15 trang với ví dụ minh họa
- Train annotators 1 tuần + quiz (Cohen's kappa > 0.7)

### Bước 2: Gán nhãn

- 2 annotators độc lập gán nhãn
- Nếu disagree → reviewer thứ 3 quyết định
- Confidence < 0.6 → gửi reviewer

### Bước 3: Kiểm tra chất lượng

- Random audit 5% dataset
- Lưu versioning cho nhãn

## 9. Xử lý imbalance

- Oversample violent clips
- Dùng focal loss
- Augmentation: random crop, time jitter, flip, brightness/noise, synthetic occlusion

## 10. Mô hình & Training

### Kiến trúc đề xuất

- 2-stream (RGB + optical flow)
- 3D CNN nhẹ: I3D, C3D, SlowFast
- Lightweight 3D CNN + temporal transformer

### Multi-task learning

Predict cùng lúc: violence_binary, intent_estimate, activity_type

### Post-processing rules

- `violence_binary=1` + `intent_estimate=playful` + `weapon_present=none` → Informational alert
- `high intensity` hoặc `weapon_present` → High alert

## 11. Metrics đánh giá

- Precision, Recall, F1 cho violence_binary
- mAP cho activity_type
- **False alarm rate per hour** (quan trọng nhất)
- Latency (≤200ms cho real-time)
- ROC + confusion matrix theo context

## 12. Tips giảm false positives

### Multi-modal

- Video + audio (tiếng la hét, âm thanh va chạm)
- Cẩn thận với privacy

### Pose & Sequence features

- Đùa giỡn: chuyển động đều, pose lặp
- Bạo lực: chuyển động đột ngột, forceful

### Context tags

- `location_type=gym` + `presence_of_coach=true` → Tăng prior cho training

### Hard negative mining

- Thu thập nhiều clip "đùa giỡn" làm negative examples

## 13. Đạo đức & Privacy

- Xin phép phụ huynh + nhà trường
- Anonymization: mờ mặt, xóa audio
- Lưu audit trail
- Alert không tự động can thiệp → Thông báo nhân sự
- Kiểm tra luật địa phương về bảo mật hình ảnh học sinh

### Xây dựng bộ dữ liệu chuẩn

#### Intent chuẩn (4 loại):

**1. intentional_harm**

> Hành vi có chủ đích gây hại, tấn công, uy hiếp.

Ví dụ:

```
đấm, đá, bóp cổ, túm cổ áo
cố ý xô ngã mạnh
đuổi đánh
tấn công liên tục (đẩy → đấm → kéo áo…)
```

**2. playful**

> Hai bên vui vẻ, tự nguyện, không có biểu hiện sợ hãi.

Ví dụ:

```
trêu ghẹo, cù nhau
đấm nhẹ kiểu đùa
rượt đuổi trong sân trường khi đang cười
cảnh bạn bè giỡn mạnh nhưng không có ý gây thương tích
```

**3. training_practice**

> Tập luyện thể thao/võ thuật hoặc luyện tập có kiểm soát.

Ví dụ:

```
tập quyền, tập đá với giáp hoặc thảm
học võ trong CLB
giáo viên đứng cạnh chỉnh động tác
đấu đối kháng nhẹ có bảo hộ
```

**4. accidental / unintentional**

> Va chạm không cố ý, không có ngữ cảnh tấn công.

Ví dụ:

```
chen chúc xô nhẹ khi ra chơi
va quệt khi chạy trong hành lang
trượt chân ngã vào người khác
té ngã tự nhiên
```

- Đủ phân biệt bạo lực thật vs đùa vs tập luyện vs vô tình
  phù hợp mô hình multi-task và giảm false positive

#### Trường ACTIVITY_TYPE (hoạt động cụ thể)

Bạn có thể dùng theo dạng multi-label, nghĩa là một clip có thể gán nhiều hoạt động.
Dưới đây là danh sách chuẩn được thiết kế riêng cho trường học.

A. Hoạt động có tính bạo lực thật (true violence)

```
1. punching – đấm
2. kicking – đá
3. slapping – tát
4. pushing_shoving – xô đẩy mạnh
5. grabbing_pulling – túm kéo áo, lôi đi
6. choking – bóp cổ
7. wrestling_fighting – vật lộn
8. throwing_object – ném vật vào người khác
9. weapon_use / improvised_weapon – dùng gậy, ghế, chai…
```

B. Hoạt động không phải bạo lực (Non-violent)

```
10. playful_fooling
Giỡn, cù, chọc phá.

11. tag_running / chasing_play
Đuổi bắt khi vui chơi.

12. friendly_push / light_touch
Chạm nhẹ, đẩy nhẹ vui vẻ.

13. sports_contact
Va chạm tự nhiên khi chơi bóng đá, bóng rổ, cầu lông…

14. martial_arts_training
Tập võ có hướng dẫn hoặc đấu tập.

15. PE_training
Giáo dục thể chất – chạy, bật nhảy, tập bài.

16. dance / choreography_practice
Tập nhảy/biểu diễn.
```

C. Hoạt động vô tình (Accidental)

```
17. accidental_collision
Va chạm khi đi bộ, chạy.

18. loss_of_balance
Trượt chân, té.

19. crowd_pressure
Chen lấn trong hành lang, cổng trường.

20. object_drop / fall
Làm rơi đồ → tiếng động mạnh nhưng không bạo lực.

D. Hoạt động bối cảnh & bổ trợ (contextual)
21. verbal_argument
Cãi vã – không bạo lực nhưng là tín hiệu sớm.

22. bullying_non_physical
Trêu chọc, bao vây, đe doạ nhưng không đánh.

23. bystander_intervention
Người can ngăn.

24. fleeing_escape
Một học sinh chạy trốn (gợi ý có đe doạ phía sau).

25. teacher_intervene
Giáo viên vào dừng hành vi.
```

D. Hoạt động bối cảnh & bổ trợ (contextual)

```
21. verbal_argument
Cãi vã – không bạo lực nhưng là tín hiệu sớm.

22. bullying_non_physical
Trêu chọc, bao vây, đe doạ nhưng không đánh.

23. bystander_intervention
Người can ngăn.

24. fleeing_escape
Một học sinh chạy trốn (gợi ý có đe doạ phía sau).

25. teacher_intervene
Giáo viên vào dừng hành vi.
```

Activity_type (10 – 12 loại cốt lõi - tham khảo):

```
punching
kicking
pushing_shoving
grabbing_pulling
wrestling_fighting
playful_fooling
tag_running
martial_arts_training
sports_contact
accidental_collision
verbal_argument
bullying_non_physical
```

### Hướng dẫn xây dựng dữ liệu và hệ thống

Dữ liệu video bạo lực học đường thường không có sẵn do tính nhạy cảm và đạo đức nghiên cứu. Dưới đây là cách bạn có thể tự xây dựng dữ liệu:

1. **Thu thập dữ liệu**:
   - Quay video mô phỏng các tình huống bạo lực học đường với sự tham gia của bạn bè hoặc diễn viên.
   - Sử dụng các nguồn video công khai có sẵn trên internet, đảm bảo tuân thủ các quy định về bản quyền và đạo đức.
2. **Ghi chú và gán nhãn**:
   - Sử dụng công cụ gán nhãn video như CVAT hoặc Labelbox để đánh dấu các hành vi bạo lực trong video.
   - Gán nhãn các khung hình hoặc đoạn video cụ thể để phân biệt giữa hành vi bạo lực và không bạo lực.
3. **Xử lý dữ liệu**:
   - Cắt video thành các đoạn ngắn để dễ dàng xử lý và huấn luyện mô hình.
   - Chuyển đổi định dạng video nếu cần thiết để phù hợp với yêu cầu của mô hình học sâu.
   - Áp dụng các kỹ thuật tăng cường dữ liệu như xoay, lật, thay đổi độ sáng để làm phong phú thêm tập dữ liệu.
4. **Bảo vệ danh tính**:
   - Sử dụng các biện pháp như làm mờ khuôn mặt hoặc sử dụng khẩu trang để bảo vệ danh tính của những người tham gia trong video.
   - Đảm bảo tuân thủ các quy định về đạo đức nghiên cứu khi thu thập và sử dụng dữ liệu.
5. **Chia tách dữ liệu**:
   - Chia tập dữ liệu thành các phần huấn luyện, kiểm tra và xác nhận để đánh giá hiệu suất của mô hình.
   - Đảm bảo rằng mỗi phần có sự đa dạng về tình huống bạo lực và không bạo lực để mô hình học tốt hơn.

### Thiết kế hệ thống phát hiện bạo lực dựa trên dữ liệu xương (skeleton data)

#### Quy trình tiền xử lý (Preprocessing Pipeline)

**Bước chuyển đổi** (Offline Preprocessing).
Mục tiêu: Chuyển đổi từ Video (Pixels) -> Skeleton Sequence (Coordinates).

Bước 1: Trích xuất khung xương (Extraction Script)
Một script Python sử dụng YOLOv8-Pose để chạy qua toàn bộ thư mục raw_videos.
Input: Video .mp4.
Process:

- Dùng YOLOv8-Pose detect từng frame.
- Dùng thuật toán Tracking (tích hợp sẵn trong Ultralytics như BoT-SORT) để theo dõi từng ID người (id_1, id_2...). Điều này cực quan trọng để tạo ra chuỗi hành động liên tục của 1 người.
- Lưu lại tọa độ 17 điểm khớp (Keypoints) của mỗi ID qua các frame. Định dạng mỗi frame là vector: (x, y, confidence score).
  Output:` Với mỗi video, bạn sẽ lưu ra nhiều file .npy (NumPy), mỗi file ứng với một người trong video đó.`
  Ví dụ: Video có 2 học sinh đánh nhau -> Tạo ra clip_001_id1.npy và clip_001_id2.npy.

Bước 2: Gán nhãn từ JSON (Labeling Strategy)
Lúc này, đọc file train.json để gán nhãn cho các file .npy vừa tạo. Đây là lúc logic Hard Negative của bạn tỏa sáng:
Tạo một file Index (ví dụ train_data.pkl) chứa danh sách các file xương và nhãn tương ứng:

```
data_list = []

for video_name, info in json_data.items():
    # 1. Xác định nhãn hành động (Action Label)
    if info['violence_binary'] == 1:
        label = 0 # VIOLENCE
    elif info['intent'] == 'playful':
        label = 2 # PLAYING / ROUGH_PLAY (Quan trọng để phân biệt gia tốc)
    else:
        label = 1 # NORMAL (Đi lại, chạy nhảy)

    # 2. Tìm các file xương tương ứng với video này
    # Lưu ý: Bạn có thể lọc chỉ lấy những người tham gia hành động chính
    skeleton_files = find_skeleton_files(video_name)

    for skel_file in skeleton_files:
        data_list.append({
            'path': skel_file, # Đường dẫn đến file .npy
            'label': label
        })
```

#### Quy trình Huấn luyện (Training Pipeline)

Bây giờ bạn sẽ train model (ST-GCN hoặc LSTM) dựa trên dữ liệu tọa độ.
Model Input:

> Tensor kích thước (Batch_Size, Channels, Frames, Vertices, Person).
> Channels: 3 (tọa độ x, y và độ tin cậy score).
> Frames: Ví dụ 30 frames (bạn cần padding hoặc sampling nếu video dài ngắn khác nhau).
> Vertices: 17 (số khớp xương của COCO format).
> Model Output: Class probabilities (0: Violence, 1: Normal, 2: Playing).

Tại sao cách này ưu việt hơn X3D cho bài toán?
Model sẽ học được rằng: Class 0 (Đánh thật) có sự thay đổi tọa độ (x, y) ở tay cực nhanh và dứt khoát. Class 2 (Đùa) cũng vung tay nhưng velocity thấp hơn và trajectory (quỹ đạo) cong hơn.

#### Hệ thống Real-time thực tế (Inference Flow)

Khi triển khai (Deploy) ra thực tế, luồng xử lý sẽ như sau:

**Camera Stream**: Đọc từng frame.
**YOLOv8-Pose + Tracking:**

- Detect người và xương.
- Gán ID (ví dụ: Học sinh A là ID 101).

**Buffer** (Bộ đệm):
Hệ thống sẽ giữ một hàng đợi (Queue) cho ID 101.
Cứ mỗi frame, đẩy tọa độ xương vào Queue.
Khi Queue đủ 30 frames (khoảng 1 giây), lấy cụm dữ liệu đó ra.

**Skeleton Model Classifier**:
Đưa cụm 30 frames tọa độ vào model ST-GCN/LSTM.
Model trả về: "Đang nô đùa" (Class 2).

**Decision**:
Nếu là Class 2 -> Không cảnh báo.
Nếu là Class 0 -> Gửi cảnh báo "BẠO LỰC".

## Tóm lại thay đổi chính so với phương án cũ

| Đặc điểm                   | Phương án cũ (X3D)                         | Phương án mới (Skeleton)           |
| -------------------------- | ------------------------------------------ | ---------------------------------- |
| **Dữ liệu đã xử lý**       | Các file video .mp4 ngắn (Crop từng người) | Các file .npy chứa tọa độ xương    |
| **Dung lượng data**        | Rất nặng (GB đến TB)                       | Siêu nhẹ (MB)                      |
| **Đầu vào Model**          | Pixels (Hình ảnh màu)                      | Coordinates (Số liệu toán học)     |
| **Khả năng phân biệt đùa** | Trung bình (Dựa trên hình ảnh)             | Rất tốt (Dựa trên gia tốc/vận tốc) |
| **Tính riêng tư**          | Thấp (Lộ mặt)                              | Cao (Chỉ thấy xương)               |
