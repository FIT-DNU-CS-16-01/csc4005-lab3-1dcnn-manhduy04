# CSC4005 Lab 3 Report – UrbanSound8K with 1D-CNN

## 1. Thông tin sinh viên
- Họ tên: Nguyễn Mạnh Duy
- Mã sinh viên: 19912745
- Lớp: CSC4005
- Link GitHub repo:https://github.com/FIT-DNU-CS-16-01/csc4005-lab3-1dcnn-manhduy04
- Link W&B run/project:https://wandb.ai/duy19912745-dainam-vietnam/csc4005-lab3-urbansound-1dcnn

---

## 2. Mục tiêu thí nghiệm

Mục tiêu của bài lab là xây dựng hệ thống phân loại âm thanh môi trường sử dụng mô hình 1D-CNN trên dataset UrbanSound8K.

Các nhiệm vụ chính gồm:

Tiền xử lý tín hiệu audio từ UrbanSound8K.
Trích xuất đặc trưng MFCC và log-mel spectrogram.
Xây dựng mô hình 1D-CNN để học chuỗi đặc trưng theo thời gian.
Theo dõi quá trình huấn luyện bằng Weights & Biases (W&B).
Đánh giá mô hình bằng accuracy và confusion matrix.
So sánh các pipeline MFCC, log-mel và raw waveform.

## 3. Dữ liệu và tiền xử lý

### 3.1. Dataset

Dataset: UrbanSound8K
Số lớp: 10
Các lớp:
air_conditioner
car_horn
children_playing
dog_bark
drilling
engine_idling
gun_shot
jackhammer
siren
street_music
Fold dùng để train: Fold 1–8
Fold dùng để validation: Fold 9
Fold dùng để test: Fold 10
### 3.2. Tiền xử lý audio

Điền cấu hình đã dùng:

| Thành phần      | Giá trị                  |
| --------------- | ------------------------ |
| Sample rate     | 16000 Hz                 |
| Duration        | 4 giây                   |
| Feature type    | MFCC / log-mel           |
| n_mfcc / n_mels | 32                       |
| n_fft           | 1024                     |
| hop_length      | 512                      |
| Augmentation    | Random noise, time shift |


Giải thích ngắn: vì sao cần đưa audio về cùng sample rate và cùng độ dài?

Việc đưa toàn bộ audio về cùng sample rate và cùng độ dài giúp dữ liệu đầu vào có kích thước thống nhất để mô hình neural network xử lý hiệu quả. Nếu các audio có độ dài khác nhau thì tensor đầu vào sẽ không đồng nhất và khó batch training. Đồng thời, chuẩn hóa sample rate giúp đảm bảo các đặc trưng tần số được tính toán nhất quán trên toàn bộ dataset.

## 4. Mô hình 1D-CNN

Mô tả kiến trúc mô hình:
Input feature sequence
→ Conv1D block 1
→ Conv1D block 2
→ Conv1D block 3
→ Global Average Pooling
→ Dense classifier
→ Softmax
Bảng cấu hình:
| Thành phần      | Giá trị        |
| --------------- | -------------- |
| model_name      | 1D-CNN         |
| hidden_channels | 64 / 128 / 256 |
| dropout         | 0.3            |
| optimizer       | Adam           |
| learning rate   | 0.001          |
| weight decay    | 1e-4           |
| batch size      | 32             |
| epochs          | 30             |
| patience        | 5              |


## 5. Kết quả thực nghiệm

### 5.1. Kết quả chính

| Metric                   |   Giá trị |
| ------------------------ | --------: |
| Best validation accuracy |    0.3267 |
| Test accuracy            |    0.2867 |
| Average epoch time       | 15.35 sec |
| Total parameters         |    78,490 |
| Trainable parameters     |    78,490 |

### 5.2. Learning curves
c:\Users\Admin\OneDrive\Pictures\Ảnh chụp màn hình\Screenshot 2026-05-14 024056.png

Nhận xét:

Train loss và validation loss giảm theo từng epoch.
Validation accuracy tăng dần trong quá trình training.
Chưa xuất hiện overfitting rõ ràng do mô hình còn khá nhỏ.
Early stopping chưa xảy ra trong lần chạy debug này.
### 5.3. Confusion matrix

c:\Users\Admin\OneDrive\Pictures\Ảnh chụp màn hình\Screenshot 2026-05-14 024157.png

Nhận xét:

Các lớp như dog_bark và siren dễ phân loại hơn do có đặc trưng âm thanh nổi bật.
Các lớp như engine_idling, air_conditioner, và drilling dễ bị nhầm lẫn do phổ tần số tương đối giống nhau.
Một số lỗi phân loại đến từ nhiễu nền và độ dài clip âm thanh không đồng nhất.
---

## 6. W&B tracking

link W&B:https://wandb.ai/duy19912745-dainam-vietnam/csc4005-lab3-urbansound-1dcnn

Dashboard trên W&B bao gồm:

Learning curves (loss và accuracy)
Final metrics
Hyperparameter configuration
Confusion matrix
Runtime và training statistics
---

## 7. Phân tích và thảo luận
1. Vì sao dùng 1D-CNN thay vì MLP cho chuỗi đặc trưng audio?

1D-CNN có khả năng học quan hệ cục bộ theo thời gian giữa các frame âm thanh, trong khi MLP chỉ xử lý vector phẳng và không tận dụng được cấu trúc chuỗi của tín hiệu audio.

2. Kernel 1D trong bài này đang trượt theo chiều nào?

Kernel 1D trượt theo chiều thời gian của chuỗi đặc trưng MFCC hoặc log-mel.

3. MFCC giúp mô hình học dễ hơn raw waveform ở điểm nào?

MFCC đã trích xuất các đặc trưng quan trọng về phổ tần số và loại bỏ nhiều nhiễu không cần thiết, giúp mô hình học nhanh và ổn định hơn so với raw waveform.

4. Mô hình hiện tại còn hạn chế gì?
Accuracy còn thấp.
Chưa tận dụng fully temporal dependency.
Dataset có nhiều lớp âm thanh tương tự nhau.
Mô hình còn nhỏ và số epoch hạn chế.
5. Có thể cải thiện kết quả bằng cách nào?
Tăng số epoch.
Dùng data augmentation mạnh hơn.
Sử dụng log-mel spectrogram.
Thử ResNet hoặc CRNN.
Fine-tune learning rate.
Sử dụng GPU để train lâu hơn.

## 8. Bài mở rộng nếu có

Nếu làm raw waveform hoặc log-mel, điền bảng sau:

| Pipeline    | Feature/Input         |   Test accuracy | Nhận xét                          |
| ----------- | --------------------- | --------------: | --------------------------------- |
| Baseline    | MFCC + 1D-CNN         |          28.67% | Chạy nhanh, dễ huấn luyện         |
| Extension 1 | log-mel + 1D-CNN      | Đang thử nghiệm | Giữ nhiều thông tin phổ hơn MFCC  |
| Extension 2 | raw waveform + 1D-CNN | Đang thử nghiệm | Input trực tiếp nhưng khó học hơn |

---

## 9. Kết luận

Qua bài lab này, em đã hiểu quy trình xây dựng hệ thống phân loại âm thanh môi trường bằng Deep Learning. Em đã học được cách:

Tiền xử lý dữ liệu audio và trích xuất đặc trưng MFCC/log-mel.
Xây dựng mô hình 1D-CNN cho dữ liệu chuỗi.
Theo dõi thí nghiệm bằng W&B.
Đánh giá mô hình bằng accuracy và confusion matrix.
Phân tích ưu nhược điểm giữa các loại đặc trưng audio khác nhau.
