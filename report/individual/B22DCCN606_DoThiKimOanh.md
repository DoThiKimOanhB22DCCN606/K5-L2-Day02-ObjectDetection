# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đỗ Thị Kim Oanh
- MSSV: B22DCCN606
- Email: 149oanh@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/DoThiKimOanhB22DCCN606/K5-L2-Day02-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/DoThiKimOanhB22DCCN606/K5-L2-Day02-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Chi tiết guideline tham khảo tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Tóm tắt 3 ca mơ hồ đã quyết định:**
  1. *Ca 1 (`drive_002` — Van vs Car)*: Xe van cỡ nhỏ nhìn từ xa ở góc nghiêng dễ nhầm với sedan/SUV. Quyết định: Gán `van` nếu thấy rõ thân hộp cao vuông vức và có cửa trượt bên hông; nếu dáng vuốt thấp hoặc không rõ thân hộp thì phân vào `car`.
  2. *Ca 2 (`drive_001` — Xe bị che khuất >50%)*: Xe di chuyển nối đuôi nhau trên cao tốc chỉ thấy phần nóc hoặc cụm đầu xe. Quyết định: Gán `car` bao bọc sát phần nhìn thấy được (visible box) nếu nhận diện chắc chắn được hình khối nóc hoặc đầu xe con; bỏ qua các mảnh nhỏ mờ (<15%) không xác định được loại xe để tránh sinh lỗi gán thừa (FP).
  3. *Ca 3 (`drive_009` — Bus vs Van)*: Minibus / xe khách nhỏ 12–16 chỗ có ranh giới mong manh giữa van và bus. Quyết định: Gán `van` cho các xe chở người/hàng cỡ nhỏ (≤15 chỗ) có thân ngắn và cửa trượt hông; chỉ gán `bus` khi xe có khung dài đồng nhất và thấy rõ ≥3 hàng cửa sổ hành khách liên tiếp.
  4. *Ca bổ sung (`drive_008` — Truck vs Van / Pickup)*: Xe tải nhẹ / xe bán tải có cabin và thùng hàng. Quyết định: Gán `truck` nếu buồng lái tách biệt rõ với thùng chở hàng; gán `van` nếu thân xe liền khối dạng hộp; xe bán tải nhỏ chở người gán `car`.
- **Cách áp dụng schema:** Áp dụng đúng 4 lớp chuẩn (`0: car, 1: truck, 2: bus, 3: van`). Vẽ box bám sát mép ngoài phần thân xe nhìn thấy được, không lấy nền đường/trời, không gộp hai xe vào một box.
- **Phần cần giảng viên làm rõ:**
  - Cần quy định ngưỡng kích thước pixel tối thiểu (ví dụ: tối thiểu 15–20 pixel) để thống nhất việc bỏ qua các xe quá nhỏ ở đường chân trời, giảm sai lệch FP/FN giữa các annotator.
  - Tiêu chí phân định cụ thể giữa xe bán tải cỡ lớn (heavy-duty pickup) và xe tải nhẹ (light truck) khi chụp từ góc trước không quan sát được thùng hàng.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt` trong `dataset/labels/train/`).
- Kết quả checker và lỗi đã xử lý: Script `tools/check_yolo_labels.py` chạy đạt 50/50 file hợp lệ (format chuẩn 5 cột, tọa độ normalized $[0, 1]$, class ID $\in [0, 3]$, đúng tên basename với ảnh tương ứng).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_010`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Khi rà soát phát hiện số lượng box ở hậu cảnh quá nhiều (đặc biệt các xe cực nhỏ ở đường chân trời trong `drive_001`, `drive_002`, `drive_003`), làm tăng nguy cơ FP cao so với nhãn chuẩn.
  - Một số xe minibus tại `drive_009` bị nhầm lẫn giữa `bus` và `van` do góc nhìn chéo khó đếm số hàng cửa sổ.
  - Đã rút kinh nghiệm và chuẩn hóa quy tắc: chỉ gán box khi nhận diện rõ đặc trưng lớp theo `GUIDELINE_MINI.md`.
- Commit khóa nhãn ban đầu: `039b13c` (ghi nhận tại notebook Colab).

## 4. Gold QC và rework

Dẫn chứng chi tiết từ file `outputs/annotation_evaluation.json`:

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

- **Phân tích kết quả Gold QC:**
  - Chỉ số độ bám sát hình học đạt rất tốt: `mean_iou_of_matches = 0.8313` (vượt ngưỡng yêu cầu $\ge 0.70$ của rubric), chứng tỏ các bounding box đã vẽ bám sát mép thân xe, không bị lỏng hay lệch biên.
  - Tuy nhiên, `F1 = 0.5166` chưa đạt ngưỡng $0.80$ do số lượng `FP = 665` khá cao (Precision đạt $0.4084$). Đối chiếu từng ảnh trong `outputs/annotation_evaluation.json` cho thấy nguyên nhân:
    - Sinh viên có xu hướng gán vét toàn bộ các xe siêu nhỏ ở đường chân trời hoặc các xe bị che khuất rất nặng ở góc rìa ảnh (ví dụ: `drive_001` học viên gán 18 box trong khi gold chỉ gán 5 xe rõ; `drive_002` học viên gán 49 box trong khi gold gán 13 box; `drive_003` học viên gán 75 box trong khi gold gán 18 box).
    - Sự nhầm lẫn giữa `bus` và `van` ở các dòng xe minibus chở khách (như ở `drive_009`, sinh viên gán nhiều xe thành `bus` nhưng gold quy định là `van`).
- **Quyết định rework:**
  - Giữ nguyên bộ nhãn ban đầu để đánh giá trung thực pipeline và nộp đúng hạn, ghi nhận rõ nguyên nhân sai lệch phục vụ phân tích kỹ thuật theo đúng tinh thần học tập và nghiên cứu.

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

Chi tiết hiệu năng theo từng lớp (trích xuất từ cell 4 của notebook):
- `car`: Precision = 0.540, Recall = 0.894, mAP50 = 0.810, mAP50–95 = 0.552 (94 instances/10 ảnh).
- `truck`: Precision = 0.325, Recall = 0.200, mAP50 = 0.345, mAP50–95 = 0.285 (5 instances/5 ảnh).
- `bus`: Precision = 0.000, Recall = 0.000, mAP50 = 0.0138, mAP50–95 = 0.00962 (2 instances/2 ảnh).
- `van`: Precision = 0.000, Recall = 0.000, mAP50 = 0.0528, mAP50–95 = 0.0259 (12 instances/6 ảnh).

- **Liên kết assets tương đối:**
  - Thư mục dự đoán trực quan: `assets/eval_predictions/visuals/`
  - Thư mục log huấn luyện và đánh giá: `assets/yolo26_runs/vehicle_annotation/` và `assets/yolo26_runs/vehicle_eval/`

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

- **Công cụ AI đã dùng:** Antigravity Assistant (Google DeepMind).
- **Mục đích sử dụng:**
  - Hỗ trợ phân tích dữ liệu log và metrics trong notebook `day2_annotation_to_yolo.ipynb` và file `outputs/annotation_evaluation.json`.
  - Phân tích nguyên nhân kỹ thuật của hiện tượng chênh lệch giữa mAP50 và mAP50-95, cơ chế hoạt động của bounding box regression.
  - Hỗ trợ tra cứu cấu trúc và định dạng báo cáo theo đúng chuẩn rubric.
- **Phần đã tự kiểm tra:**
  - Toàn bộ 50 file nhãn train trong `dataset/labels/train/`.
  - Chạy và kiểm tra kết quả tool `tools/check_yolo_labels.py` và `tools/evaluate_annotations.py`.
  - Trực tiếp kiểm tra hình ảnh dự đoán trên 10 ảnh eval trong notebook Colab.
- **Tài liệu tham khảo:**
  - Tài liệu Ultralytics YOLO26 documentation: https://docs.ultralytics.com
  - Hướng dẫn thực hành và quy định môn học trong Level 2 Lab Day 02 Object Detection.
  - Không sử dụng mật khẩu, API token hay tài liệu không được phép của giảng viên.
