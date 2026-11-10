# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đỗ Thị Kim Oanh
- MSSV: 2A202604034
- Email: 26ai.oanhdtk@vinuni.edu.vn
- Lớp: 2B-LAB-E403
- Link repo bài làm: [repo bài làm](https://github.com/DoThiKimOanhB22DCCN606/K5-L2-DAY02-DoThiKimOanh-2A202604034-ObjectDetection)
- Link Colab đã chạy: [Colab](https://colab.research.google.com/github/DoThiKimOanhB22DCCN606/K5-L2-DAY02-DoThiKimOanh-2A202604034-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb)

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](https://github.com/DoThiKimOanhB22DCCN606/K5-L2-DAY02-DoThiKimOanh-2A202604034-ObjectDetection/blob/main/GUIDELINE_MINI.md).

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_002` | Xe van cỡ nhỏ nhìn từ xa ở góc nghiêng, khó phân biệt với xe con sedan/SUV | Gán `van` vì thấy rõ thân hộp cao và có cửa trượt bên | Van có thân hộp vuông vức và cao hơn car sedan/SUV |
| `drive_001` | Xe di chuyển nối đuôi bị che khuất >50% bởi xe phía trước, chỉ thấy phần nóc/đầu | Gán `car` vì nhận ra được nóc/đầu xe con; bỏ qua các mảnh nhỏ <15% không xác định được lớp để tránh lỗi gán thừa (FP) | Quy tắc: gán box quanh phần nhìn thấy khi đủ nhận diện lớp, không đoán phần khuất |
| `drive_009` | Minibus / xe khách nhỏ ~12–16 chỗ có ranh giới mong manh giữa van và bus | Gán `van` cho xe chở người/hàng cỡ nhỏ (≤15 chỗ) có thân ngắn; gán `bus` khi xe có khung dài và thấy ≥3 hàng cửa sổ hành khách liên tục | Bus phân biệt bởi hàng cửa sổ dài liên tục và thân xe dài đồng nhất |

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt` trong `dataset/labels/train/`).
- Kết quả checker và lỗi đã xử lý: Script `tools/check_yolo_labels.py` chạy đạt 50/50 file hợp lệ (format chuẩn 5 cột, tọa độ normalized $[0, 1]$, class ID $\in [0, 3]$, đúng tên basename với ảnh tương ứng).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_010`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Khi rà soát phát hiện số lượng box ở hậu cảnh quá nhiều (đặc biệt các xe cực nhỏ ở đường chân trời trong `drive_001`, `drive_002`, `drive_003`), làm tăng nguy cơ FP cao so với nhãn chuẩn.
  - Một số xe minibus tại `drive_009` bị nhầm lẫn giữa `bus` và `van` do góc nhìn chéo khó đếm số hàng cửa sổ.
  - Đã rút kinh nghiệm và chuẩn hóa quy tắc: chỉ gán box khi nhận diện rõ đặc trưng lớp theo `GUIDELINE_MINI.md`.
- Commit khóa nhãn ban đầu: `039b13c`.

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 459 | - |
| FP | 665 | - |
| FN | 194 | - |
| Precision | 0.4084 | - |
| Recall | 0.7029 | - |
| F1 | 0.5166 | - |
| mean_iou_of_matches | 0.8313 | - |
| passed | false | - |

Dẫn tới `outputs/annotation_evaluation.json`: [annotation_evaluation.json](https://github.com/DoThiKimOanhB22DCCN606/K5-L2-DAY02-DoThiKimOanh-2A202604034-ObjectDetection/blob/main/outputs/annotation_evaluation.json)
Không có rework.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
  - Nền tảng: Google Colab, GPU Tesla T4 (14913 MiB), CUDA:0.
  - Ultralytics: 8.4.208.
  - PyTorch: 2.14.1+cu130.
  - Python: 3.13.16.
- Các tham số thay đổi so với notebook nếu có và lý do:
  - Giữ nguyên cấu hình chuẩn của notebook: `model = YOLO('yolo26n.pt')`, `epochs = 50`, `imgsz = 640`, `batch = -1` (autobatch), `device = DEVICE`, `seed = 20260909`, `plots = True`.
- Đường dẫn best checkpoint trong phiên chạy:
  `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.2162 |
| Recall | 0.2734 |
| mAP50 | 0.3053 |
| mAP50–95 | 0.2181 |

Link tương đối: 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` 
[report/individual/assets/eval_predictions](https://github.com/DoThiKimOanhB22DCCN606/K5-L2-DAY02-DoThiKimOanh-2A202604034-ObjectDetection/tree/main/report/individual/assets/eval_predictions)
[report/individual/assets/yolo26_runs](https://github.com/DoThiKimOanhB22DCCN606/K5-L2-DAY02-DoThiKimOanh-2A202604034-ObjectDetection/tree/main/report/individual/assets/yolo26_runs)

- **Phân tích ảnh trực quan cụ thể:**
  - **Ảnh `drive_051.jpg`** (`assets/eval_predictions/visuals/drive_051.jpg`): Ground truth thực tế có 1 xe `bus` (`dataset/labels/eval/drive_051.txt` dòng 4: `2 0.828125 0.130469 0.110156 0.201563`). Tuy nhiên, kết quả dự đoán của mô hình là `9 cars, 1 van` $\rightarrow$ đối tượng xe bus duy nhất bị mô hình bỏ sót hoàn toàn (FN) hoặc gán nhầm sang van/car.
  - **Ảnh `drive_058.jpg`** (`assets/eval_predictions/visuals/drive_058.jpg`): Ground truth có 1 `bus`, 6 `vans`, và 32 `cars`. Nhưng mô hình dự đoán ra `27 cars, 6 buss, 1 van` $\rightarrow$ mô hình bị nhầm lẫn đảo ngược nghiêm trọng giữa bus và van (dự đoán 6 xe van thành 6 xe bus).

### Trả lời bốn câu hỏi cuối notebook:

#### 1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
- Hai lớp có Recall thấp nhất là **`bus`** và **`van`** đều đạt **$\text{Recall} = 0.000$** (ở ngưỡng IoU $\ge 0.50$ mặc định, mô hình không phát hiện đúng bất kỳ instance nào).
- Trong đó, **`bus`** là lớp có hiệu năng kém nhất toàn diện với $\text{mAP50} = 0.0138$ và $\text{mAP50-95} = 0.00962$ (thấp nhất trong 4 lớp), do ground truth chỉ có vỏn vẹn 2 instances trong toàn bộ 10 ảnh eval.
- **Ảnh minh họa:**
  - Ảnh `drive_051.jpg`: Ground truth có 1 bus (`2 0.828125 0.130469 0.110156 0.201563`), nhưng mô hình chỉ predict `9 cars, 1 van` (bỏ sót hoàn toàn xe bus, Recall = 0).
  - Ảnh `drive_052.jpg`: Ground truth có 1 van (`3 0.427344 0.533594 0.207031 0.655469`), nhưng mô hình predict ra `13 cars` (0 van, Recall = 0).

#### 2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
- **Bản chất kỹ thuật:** $\text{mAP50}$ chỉ yêu cầu diện tích chồng lấn $\text{IoU} \ge 0.50$ để tính là TP, trong khi $\text{mAP50-95}$ là trung bình cộng của mAP ở 10 ngưỡng IoU khắt khe từ $0.50$ đến $0.95$ (bước nhảy $0.05$).
- **Vấn đề cần kiểm tra trên box:** Hiện tượng này cho thấy mô hình phát hiện và phân loại đúng đối tượng (classification tốt), nhưng **độ chính xác định vị ranh giới box (box localization / regression) rất kém**. Cần kiểm tra:
  1. *Độ bám sát mép (Tighness):* Box có bị lỏng (loose box), thừa nhiều nền đường/trời xung quanh xe hay không.
  2. *Độ lệch tâm và cắt lẹm (Alignment):* Box có bị xê dịch hoặc cắt lẹm vào thân xe hay không.
  - Khi tăng ngưỡng IoU lên $\ge 0.70$ hoặc $0.85$, các box lỏng/lệch này không đạt ngưỡng, bị chuyển từ TP sang FP/FN, làm sụt giảm mạnh mAP.
  - Thực tế lớp `car` trong bài đạt $\text{mAP50} = 0.810$ nhưng $\text{mAP50-95}$ chỉ đạt $0.552$ (giảm tới 31.8%) do 50 ảnh train là quá ít để mạng học hồi quy tọa độ box cực sát.

#### 3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
- **Lỗi nhầm lẫn giữa `bus` và `van`:**
  - *Liên quan guideline:* Ca mơ hồ 3 quy định minibus $\ge 3$ hàng cửa sổ gán `bus`, $\le 2$ hàng cửa sổ và có cửa trượt hông gán `van`. Trên ảnh camera 640x640 mờ và chụp góc xéo/từ xa, đặc trưng này bị mất chi tiết, khiến mô hình không học được ranh giới.
  - *Dữ kiện hỗ trợ:* Tại ảnh `drive_058.jpg`, ground truth có 6 vans và 1 bus, nhưng mô hình predict ra 6 buss và 1 van (nhầm toàn bộ van thành bus). Tại `drive_053.jpg`, ground truth có 2 vans và 0 bus, nhưng mô hình predict ra 2 buss.
- **Lỗi nhầm lẫn giữa `van` và `car`:**
  - *Liên quan guideline:* Ca mơ hồ 1 quy định van cỡ nhỏ nhìn từ xa gán `van` nếu thấy thân hộp và cửa trượt. Do góc chụp chính diện/sau đuôi không thấy cửa trượt, cộng với class imbalance (`car` 94 instances vs `van` 12 instances), mô hình thiên lệch đoán van thành car.
  - *Dữ kiện hỗ trợ:* Tại ảnh `drive_052.jpg`, xe van lớn chiếm giữa ảnh bị mô hình gán thành `car`.
- **Lỗi gán thừa (FP) từ tập train:**
  - *Liên quan guideline:* Việc không tuân thủ chặt ca 2 (bỏ qua xe mờ $<15\%$) dẫn tới $\text{FP} = 665$ trong `outputs/annotation_evaluation.json`, đưa nhiễu vào mô hình.

#### 4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
- **Vì sao không dùng 10 ảnh eval để train:**
  - *Tránh rò rỉ dữ liệu (Data Leakage) & Overfitting:* Trong học máy, nếu đưa tập kiểm thử vào huấn luyện (cập nhật gradient), mô hình sẽ học vẹt (memorize) các ảnh này. Các chỉ số khi đó bị thổi phồng giả tạo (inflated metrics), không phản ánh năng lực tổng quát hóa (generalization) trên dữ liệu thực tế.
  - *Quy định bắt buộc:* `GUIDE.md` (dòng 122), `CHECKPOINTS.md` (dòng 53) và `RUBRIC.md` (dòng 19) quy định rõ: dùng eval để train sẽ bị hủy kết quả phần pipeline.
- **Vai trò Validation trong notebook:**
  - Trong `data.yaml`, đường dẫn `val` trỏ tới `images/eval`.
  - 10 ảnh eval đóng vai trò là **Validation Set**: sau mỗi epoch trong 50 epochs, YOLO tính toán loss và metric (fitness) trên 10 ảnh này để theo dõi tiến trình huấn luyện và trực tiếp dùng để lưu trọng số tốt nhất (`outputs/yolo26_runs/vehicle_annotation/weights/best.pt`).
  - Vì 10 ảnh eval đã tham gia vào quá trình lựa chọn checkpoint tối ưu, nó là tập validation, không phải test set độc lập, và tuyệt đối không được dùng để train.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn ban đầu (Annotation) | Rất nhiều xe nhỏ ở xa đường chân trời hoặc bị che khuất sau dải phân cách | Ban đầu cố gắng gán hết tất cả các điểm nghi là xe | Dẫn đến số lượng box rất lớn (FP=665), cần thống nhất kích thước pixel tối thiểu trong guideline |
| Phân lớp (Classification) | Khó phân biệt minibus (12–16 chỗ) với xe van chở hàng khi nhìn từ góc sau | Đọc kỹ guideline ca 3, tìm đặc trưng hàng cửa sổ và cửa trượt hông | Đã phân định được trên các xe gần, các xe ở xa vẫn có sự nhầm lẫn giữa bus và van |
| Huấn luyện YOLO | mAP của lớp `bus` và `van` rất thấp (Recall = 0) | Kiểm tra phân bố tập dữ liệu (eval chỉ có 2 bus, 12 van trong khi car có 94) | Nhận diện được nguyên nhân do mất cân bằng dữ liệu (class imbalance) trầm trọng và tập dữ liệu 50 ảnh quá nhỏ |
| Hồi quy tọa độ box | mAP50 đạt cao (0.81 ở car) nhưng mAP50-95 thấp (0.552) | So sánh box dự đoán với ground truth | Xác định lỗi do box chưa đủ sát mép ở các ngưỡng IoU khắt khe; cần tăng lượng ảnh train và độ tỉ mỉ khi vẽ box |

## 7. Tự đánh giá theo rubric

Đối chiếu theo [RUBRIC.md](../../RUBRIC.md):

| Tiêu chí | Kết quả tự đánh giá | Tự chấm |
|---|---|---:|
| Hoàn thành và định dạng YOLO | 50 file nhãn đầy đủ, đúng 5 cột, tọa độ chuẩn hóa $[0, 1]$, class ID $[0, 3]$, tool check đạt 50/50 | 10 / 10 |
| Đầy đủ vật thể | Gán đầy đủ các xe nhìn thấy rõ trong 50 ảnh train; có xu hướng gán thừa xe xa | 17 / 20 |
| Độ bám sát và hình học box | Box bám sát phần thân xe nhìn thấy; `mean_iou_of_matches = 0.8313` đạt vượt ngưỡng $\ge 0.70$ | 18 / 20 |
| Phân lớp nhất quán | Phân biệt đúng `car/truck/bus/van`; còn nhầm lẫn một số ca minibus giữa bus và van | 12 / 15 |
| Mini guideline và quyết định mơ hồ | `GUIDELINE_MINI.md` đầy đủ định nghĩa lớp, quy tắc box và 4 ca mơ hồ kèm ảnh cụ thể | 10 / 10 |
| Gold-set annotation QC | `mean_iou_of_matches = 0.8313` đạt; `F1 = 0.5166` do FP cao; có phân tích nguyên nhân kỹ thuật chi tiết theo đúng rubric | 12 / 15 |
| Pipeline YOLO và phân tích | Chạy Colab thành công, giữ đúng split eval, metrics đầy đủ, ảnh minh họa trực quan và trả lời xuất sắc 4 câu hỏi phân tích | 10 / 10 |
| **Tổng điểm tự đánh giá** | **89 / 100** |

## 8. Công cụ AI và tài liệu tham khảo
- **Tài liệu tham khảo:**
  - Tài liệu Ultralytics YOLO26 documentation: https://docs.ultralytics.com
  - Hướng dẫn thực hành và quy định môn học trong Level 2 Lab Day 02 Object Detection.
  - Không sử dụng mật khẩu, API token hay tài liệu không được phép của giảng viên.

