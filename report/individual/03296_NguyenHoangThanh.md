# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hoàng Thành
- MSSV: 2A202603296
- Email: 26ai.thanhnh4@vinuni.edu.vn
- Lớp: K5-L2
- Link repo bài làm: https://github.com/TT1172001/K5-L2-NguyenHoangThanh-2A202603296-Day02-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/16XO8e7_FCt8DM28M4JMlIaJ_lHoIqntQ

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema dùng đúng README và `data.yaml`: `0 car`, `1 truck`, `2 bus`, `3 van`. Lớp mô tả loại xe, không mô tả trạng thái, nên xe đỗ, xe bị che hoặc bị cắt khung vẫn được gán nếu nhận ra lớp.

Các ca mơ hồ đã quyết định (đủ 5 ca trong guideline):

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| `drive_014` | Xe đỏ IVECO mái cao ở mép phải | `van` | Thân một khối, một hàng cửa sổ ngắn, không dài như minibus |
| `drive_030` | MPV bạc 7 chỗ (dáng Buick GL8) | `van` | Minivan chở người, mái cao, thân một khối, khác sedan/SUV |
| `drive_014` | Xe cứu hộ có cần cẩu phía sau | `truck` | Cabin tách khỏi phần thiết bị kéo/chở, thuộc nhóm xe chở hàng |
| `drive_003` | Cụm xe rất nhỏ, mờ dưới gầm cầu | Không gán | Không tách được từng xe để xác định lớp |
| `drive_024` | Xe ba bánh chở người có mui | Không gán | Ngoài bốn lớp của schema |

Ngưỡng xe nhỏ: không gán box dưới khoảng 310 px² trên ảnh 640×640. Đây là box nhỏ nhất trong nhãn `eval`, chỉ dùng để tham khảo kích thước, không đưa eval vào train.

Phần cần giảng viên làm rõ: (1) ca xe IVECO `drive_014` nên là `van` hay `bus`; (2) thứ tự ID lớp trong gold và eval khác README (xem mục 4). Lab coach đã xác nhận ý (2) là sự cố của bộ dữ liệu và dặn cứ làm đầy đủ theo schema.

## 3. Annotation và self-QC

- **Số file nhãn train:** 50/50, cùng basename với ảnh trong `dataset/images/train/`. Tổng cộng 1168 box: car 976, truck 24, bus 108, van 60.
- **Cách gán nhãn:**
  - `drive_001`–`drive_010`: gán tay trên CVAT.
  - `drive_011`–`drive_050`: nhãn nháp sinh bằng mô hình YOLO pretrained (đã được phép dùng AI), sau đó rà bằng mắt và sửa lớp theo schema.
- **Kết quả checker:** `python tools/check_yolo_labels.py` cho `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. Trên Windows, checker in tiếng Việt bị lỗi `UnicodeEncodeError` (console cp1252); chạy với `PYTHONIOENCODING=utf-8` thì hết lỗi. Đây là lỗi hiển thị, không phải lỗi nhãn.
- **Ảnh đã rà bằng mắt (16 ảnh):** `drive_001`–`drive_010` (gán tay toàn bộ), `drive_012`, `drive_014`, `drive_018`, `drive_024`, `drive_030`, `drive_040`.
- **Lỗi phát hiện khi rà và cách sửa:**
  - Nhãn nháp YOLO chỉ biết lớp xe chung, nên phải xếp lại lớp cho từng xe: minibus kem `drive_012` sang `bus`; xe IVECO `drive_014` và MPV `drive_030` sang `van`; xe cứu hộ `drive_014` và xe ben `drive_040` sang `truck`.
  - Bỏ box thuộc vật thể ngoài schema, ví dụ xe ba bánh `drive_024`.
  - Bỏ cụm xe dính nhau ở cuối đường (`drive_003`) và mọi box nhỏ hơn khoảng 310 px².
- **Commit khóa nhãn ban đầu:** `c4ace0e` (“Khóa bộ nhãn ban đầu 50 ảnh train và mini guideline”), đã push trước khi chấm gold.

## 4. Gold QC và rework

Kết quả đầy đủ ở [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json) (commit `19bc5ad`). Lệnh chạy:

```bash
python tools/evaluate_annotations.py --student-labels dataset/labels/train --gold-labels gold_train_labels
```

| Chỉ số | Trước sửa (`c4ace0e`) | Sau sửa nếu có |
|---|---:|---:|
| TP | 473 | không rework |
| FP | 695 | — |
| FN | 180 | — |
| Precision | 0.405 | — |
| Recall | 0.724 | — |
| F1 | 0.520 | — |
| mean_iou_of_matches | 0.834 | — |
| passed | false (F1 < 0.80) | — |

Mean IoU 0.834 vượt ngưỡng 0.70, nên các box đã khớp được vẽ sát. Chỉ F1 chưa đạt. tôi không kết luận nguyên nhân từ con số FP/FN mà đã kiểm tra trực tiếp theo hai cách sau.

**1. Gold dùng thứ tự ID lớp khác README.** tôi ghép box của tôi với box gold theo vị trí (IoU ≥ 0.50, không xét lớp), rồi đếm cặp nào khác ID:

| Lớp tôi gán (ID theo README) | ID trong gold | Số box |
|---|---:|---:|
| `bus` (2) | 3 | 53 |
| `van` (3) | 1 | 32 |
| `truck` (1) | 2 | 18 |
| `car` (0) | 0 | 472 |

Kết quả nhất quán theo kiểu hoán vị: gold đang dùng thứ tự `car, van, truck, bus`. Nhãn `eval` có cùng dấu hiệu: ID 3 có 12 box, còn ID 2 chỉ có 2 box, trong khi ảnh đường phố có xe buýt nhiều hơn hẳn van. Evaluator chỉ ghép box cùng ID, nên khoảng 103 box đúng vị trí bị tính hai lần: một FP và một FN.

**2. Gold bỏ nhiều xe nhìn thấy rõ.** Gold có 653 box cho 50 ảnh, còn bộ của tôi có 1168. Ví dụ `drive_018`: tôi vẽ chồng hai bộ nhãn lên ảnh và thấy gold chỉ có 2 box (taxi trên làn đường và một xe ở góc trên phải). Gold bỏ cả hàng 8 xe đỗ rõ nét bên lề trái, dù README ghi “Xe đang đỗ vẫn là `car`”. Ở chính ảnh này tôi cũng có một lỗi thật: bỏ sót chiếc xe tối màu ở góc trên phải mà gold có gán. Các ảnh đông xe chênh lệch tương tự: `drive_002` (tôi 45 box, gold 13), `drive_006` (53 so với 21).

**Mô phỏng (không áp dụng vào nhãn nộp):**

| Giả định | F1 | mean IoU |
|---|---:|---:|
| Nhãn như đã khóa | 0.520 | 0.834 |
| Đổi ID theo thứ tự của gold | 0.632 | 0.839 |
| Đổi ID và bỏ box < 1000 px² | 0.708 | 0.847 |

**Quyết định: không rework.**

- Muốn đạt F1 ≥ 0.80 thì phải xóa từng xe đỗ mà gold không gán, tức là chép theo gold. Làm vậy trái README và trái RULES.md.
- Đổi ID sang thứ tự của gold thì nhãn sẽ sai `data.yaml`, vi phạm quy định “không đổi ID lớp” trong SUBMISSION.md.
- Lab coach xác nhận đây là sự cố của dữ liệu và dặn cứ làm đầy đủ. Vì vậy tôi giữ nguyên bộ nhãn `c4ace0e` theo schema README để train.

Hệ quả cho bước train: notebook dùng `eval` làm validation, mà nhãn eval có thể cũng bị hoán vị ID. Khi đó điểm các lớp `truck/bus/van` trên eval sẽ thấp dù model nhận đúng loại xe. Mục 5 sẽ đối chiếu ý này.

## 5. Kết quả YOLO

- **Runtime:** Colab CPU (Python 3, Google Compute Engine, không có GPU), mỗi epoch khoảng 37 giây, 50 epoch hết 2015 giây.
- **Phiên bản Ultralytics / Torch:** Ultralytics 8.4.208, Torch 2.14.1+cpu
- **Tham số:** giữ nguyên notebook: `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`. Split giữ nguyên: 50 ảnh train, 10 ảnh eval chỉ dùng làm validation.
- **Best checkpoint:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`. File `.pt` không được nộp. Theo [`results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv), best là **epoch 1** (mAP50-95 0.198). Các epoch sau precision tăng lên khoảng 0.69 nhưng recall giảm còn khoảng 0.19, nên không epoch nào vượt epoch 1. Phần giải thích ở câu 1 và 3.

| Chỉ số trên 10 ảnh eval (113 box) | Giá trị |
|---|---:|
| Precision | 0.457 |
| Recall | 0.241 |
| mAP50 | 0.275 |
| mAP50–95 | 0.197 |

**Kết quả đính kèm:**
- Ảnh dự đoán của 10 ảnh eval: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/).
- Log train: [`assets/yolo26_runs/vehicle_annotation/`](assets/yolo26_runs/vehicle_annotation/).
- Ảnh chỉ số của best checkpoint: [`assets/yolo26_runs/vehicle_eval/`](assets/yolo26_runs/vehicle_eval/), gồm confusion matrix và các đường PR, F1.

**Confusion matrix trên eval** ([ảnh](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)). Mỗi cột là lớp theo nhãn eval, mỗi ô là số box model dự đoán:

| Nhãn eval (tên theo `data.yaml`) | Số box | Model dự đoán |
|---|---:|---|
| `car` (0) | 94 | 86 car, 2 bus, 2 van, 4 bỏ sót |
| `truck` (1) | 5 | 4 car, 1 truck |
| `bus` (2) | 2 | 1 car, 1 truck |
| `van` (3) | 12 | **11 bus**, 1 bỏ sót |

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Tính theo nhãn eval, `van` và `bus` có recall bằng 0: không box nào được dự đoán đúng ID. Ngược lại, `car` đạt 86/94.
   - Ảnh minh họa là [`drive_058`](assets/eval_predictions/visuals/drive_058.jpg). Nhãn eval có 6 box ID 3 (`van`) ở ảnh này. Khi xem ảnh dự đoán, đó là các xe buýt ở làn trái, và model đã dự đoán `bus` với độ tin cậy 0.92, 0.65, 0.45... Nghĩa là model nhận đúng loại xe, nhưng bị tính sai vì nhãn eval ghi ID 3.
   - Hiện tượng này khớp với lỗi hoán vị ID đã thấy ở gold train (mục 4): gold cũng ghi bus là 3. Vậy recall thấp ở `van/bus` chủ yếu do nhãn eval lệch thứ tự lớp, không phải model bỏ sót.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - mAP50–95 thấp hơn mAP50 nghĩa là box tìm đúng vật thể nhưng chưa sát. Khi đó cần kiểm từng cạnh box ở IoU cao: xe nhỏ ở xa, xe bị che trong hàng xe dày đặc, xe bị cắt ở rìa ảnh. Cũng cần kiểm xem nhãn train và nhãn eval có cùng quy tắc vẽ box không (chỉ phần nhìn thấy hay đoán cả phần bị che).
   - Ở đây mAP50 là 0.275 và mAP50–95 là 0.197, tỉ lệ khoảng 0.72, nên box không phải vấn đề chính. Cả hai chỉ số đều thấp chủ yếu do lệch lớp (câu 1 và 3).
   - Ở bước gán nhãn, mean IoU giữa nhãn của tôi và gold là 0.834, cho thấy nhãn train vẽ box sát.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - **`bus` bị chấm thành `van`:** 11/12 box "van" của eval được model dự đoán là bus, và ảnh `drive_058` cho thấy đó đúng là xe buýt. Lỗi này nằm ở thứ tự ID trong nhãn eval, không phải ranh giới định nghĩa giữa hai lớp trong guideline.
   - **Ranh giới `van`/MPV với `car`:** 4/5 box eval ghi ID 1 bị model dự đoán là `car`. Theo thứ tự lớp của gold (`car, van, truck, bus`), ID 1 có thể là `van`. Nếu đúng vậy, đây là lỗi thật liên quan guideline: MPV và minivan nhìn gần giống SUV (ca `drive_030` trong guideline), và train chỉ có 60 box van so với 976 box car.
   - **Mất cân bằng lớp:** train có car 976, bus 108, van 60, truck 24. Model chủ yếu học `car`, nên khi không chắc, nó nghiêng về `car`.
   - Điều này cũng giải thích vì sao best là epoch 1. Train càng lâu, model càng học theo ID của README (bus = 2). Nhãn eval lại ghi bus = 3, nên chỉ số trên eval không tăng theo, dù loss train giảm đều.
   - Cột background của confusion matrix rất lớn, tức là có nhiều box dự đoán không ghép được với nhãn eval nào. Chưa thể kết luận model sai ở đây: ở gold train, nhãn chuẩn cũng bỏ nhiều xe đỗ, nên cần đối chiếu từng ảnh trước khi kết luận.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng là gì?**
   - Nếu đưa eval vào train, model sẽ thấy trước chính các ảnh dùng để chấm, và chỉ số trên eval không còn phản ánh khả năng tổng quát hóa.
   - Trong notebook, 10 ảnh eval là tập validation. Chúng được dùng để tính metric sau mỗi epoch, chọn `best.pt` theo fitness, rồi chấm lại best checkpoint. Vì đã dùng để chọn checkpoint nên đây không phải test set độc lập: kết quả có thể hơi lạc quan so với dữ liệu mới.
   - Lần chạy này cho thấy rõ vai trò đó. Nhãn eval lệch ID nên việc chọn checkpoint cũng bị lệch theo: chọn trúng epoch 1, gần như trọng số pretrained.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP2: gán nhãn 50 ảnh | Không đủ thời gian gán tay toàn bộ ảnh đông xe | Gán tay 10 ảnh, 40 ảnh còn lại dùng nhãn nháp YOLO rồi rà lớp bằng mắt | Đủ 50 file. Nhãn nháp gán cả xe đỗ ở xa nên số box nhiều hơn gold |
| CP5: chấm gold | F1 0.520, có box đúng vị trí nhưng bị tính sai lớp | Ghép box theo vị trí | Phát hiện gold và eval hoán vị ID lớp và gold bỏ xe đỗ. Đã được lab coach xác nhận là sự cố |
| CP5: rework | Không thể đạt F1 0.80 nếu không chép gold | Mô phỏng đổi ID và lọc xe nhỏ, F1 tối đa khoảng 0.71 | Không rework, giữ nhãn theo README |
| CP6: đọc kết quả | Recall chỉ 0.24, best là epoch 1 | Đọc confusion matrix và ảnh dự đoán `drive_058` | Xe buýt được nhận đúng nhưng nhãn eval ghi ID 3, cùng lỗi hoán vị với gold |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Hiện trạng |
|---|---|
| Hoàn thành và định dạng YOLO (10) | Đạt: 50/50 file, checker đạt, ID đúng `data.yaml` |
| Đầy đủ vật thể (20) | Gán cả xe đỗ, xe che và xe cắt khung theo README. Recall so với gold là 0.724, phần thiếu chủ yếu do lệch ID. Có sót thật, ví dụ xe góc trên phải `drive_018` |
| Bám sát box (20) | Mean IoU 0.834 trên 473 box khớp, trên ngưỡng 0.70 |
| Phân lớp nhất quán (15) | Theo schema README. So với gold, các lớp ngoài `car` lệch theo kiểu hoán vị đều đặn, không phải lỗi ngẫu nhiên |
| Mini guideline (10) | Có định nghĩa 4 lớp, quy tắc box, 5 ca mơ hồ có lý do |
| Gold-set QC (15) | Chưa đạt F1. Có phân tích nguyên nhân, mô phỏng và lý do không rework, đã báo labcoach |
| Pipeline YOLO (10) | Đã chạy Colab, không dùng eval để train. Có metrics, ảnh dự đoán eval, confusion matrix và phân tích lỗi trên ảnh cụ thể `drive_058` |

## 8. Công cụ AI và tài liệu tham khảo

- **Mô hình YOLO pretrained:** tạo nhãn nháp cho `drive_011`–`drive_050`. Học viên rà lớp từng ảnh và sửa theo schema. Box thuộc lớp ngoài schema và box quá nhỏ đã được bỏ.
- **Phần học viên tự kiểm:** quyết định lớp trong các ca mơ hồ; xem ảnh chồng nhãn `drive_018`; quyết định không chép gold. Số liệu trong báo cáo lấy từ `outputs/annotation_evaluation.json` và từ chạy lại script.
- Không dùng nguồn ngoài repo. Gold train không được commit.
