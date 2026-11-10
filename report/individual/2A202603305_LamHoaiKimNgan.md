# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- **Họ tên:** Lâm Hoài Kim Ngân
- **MSSV:** 2A202603305
- **Email:** lamhoaikimngan91@gmail.com
- **Lớp:** 2B
- **Link repo bài làm:** [https://github.com/nganlam-vn/K5-L2B-LamHoaiKimNgan-2A202603305-Day02-ObjectDetection](https://github.com/nganlam-vn/K5-L2B-LamHoaiKimNgan-2A202603305-Day02-ObjectDetection)
- **Link Colab đã chạy:** *(Dán link Colab của bạn vào đây sau khi chạy notebook)*

---

## 2. Schema và mini guideline

- **Tài liệu chi tiết:** Tham khảo tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).
- **Thứ tự Schema áp dụng:**
  - `0: car` — Xe con, sedan, hatchback, SUV, CUV, taxi, xe bán tải (pickup).
  - `1: van` — Xe van chở hàng, minivan chở người, xe chở khách cỡ nhỏ/vừa (dưới 16 chỗ) như Ford Transit, Toyota HiAce.
  - `2: truck` — Xe tải chở hàng thùng rời, xe đầu kéo, xe ben chuyên dụng.
  - `3: bus` — Xe buýt công cộng, xe khách liên tỉnh cỡ lớn nhiều hàng ghế.

- **Tóm tắt 3 ca mơ hồ tiêu biểu:**
  1. **`drive_001.jpg` (Vật thể bị che khuất một phần):** Xe ô tô trắng bên phải bị cột đèn che ngang thân; dãy xe con đỗ dọc lề đường bên trái bị cành cây/lùm cây che một phần nắp ca-pô. *Quyết định:* Gán 1 box `car` (0) cho mỗi xe ôm sát phần thân nhìn thấy được; không bỏ sót xe đỗ.
  2. **`drive_002.jpg` (Phân định minivan/van vs SUV/bus & xe ở xa):** Xe minivan màu bạc sát rìa phải và xe van trắng ở làn bên trái dễ nhầm lẫn giữa SUV gầm cao (`car`) và xe buýt (`bus`). Ở xa tít ngã tư có các đốm xe rất mờ. *Quyết định:* Gán `van` (1) cho 2 xe minivan/van; bỏ qua các đốm xe mờ căm ở xa vì không đủ thông tin nhận diện lớp chắc chắn.
  3. **`drive_004.jpg` (Vật thể bị cắt mép ảnh & cảnh đêm):** Xe ô tô đỏ ở góc dưới bên phải bị cắt ngang bởi mép dưới ảnh; trong cảnh đêm có người đi bộ và phương tiện 2 bánh qua đường. *Quyết định:* Gán box `car` (0) cho xe đỏ với cạnh box chạm đúng mép ảnh (không phỏng đoán phần ngoài ảnh); tuyệt đối không gán người đi bộ, xe máy, xe đạp vì nằm ngoài 4 lớp schema.

---

## 3. Annotation và self-QC

- **Số file nhãn train:** Đủ **50 file** trong `dataset/labels/train/` (từ `drive_001.txt` đến `drive_050.txt`), basename trùng khớp 100% với tên ảnh trong `dataset/images/train/`.
- **Kết quả checker và lỗi đã xử lý:**
  - Chạy script `python tools/check_yolo_labels.py`: **ĐẠT 50/50 file nhãn hợp lệ**.
  - Đã xử lý phẳng hóa thư mục từ bản export CVAT (`result/labels/train/` $\rightarrow$ `dataset/labels/train/`), dọn dẹp các file phụ (`data.yaml`, `train.txt`), đảm bảo mỗi file nhãn chứa đúng 5 cột tọa độ chuẩn hóa YOLO $[0, 1]$.
- **Danh sách 12 ảnh đã rà soát bằng mắt (Self-QC):**
  `drive_001.jpg`, `drive_002.jpg`, `drive_003.jpg`, `drive_004.jpg`, `drive_008.jpg`, `drive_009.jpg`, `drive_012.jpg`, `drive_014.jpg`, `drive_019.jpg`, `drive_024.jpg`, `drive_029.jpg`, `drive_038.jpg`.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**
  - *Lỗi Class:* Khi mới gán ban đầu, các dòng minivan chở khách (HiAce, Transit) dễ bị nhầm thành `bus` hoặc `car`. Qua bước rà soát, đã đối chiếu guideline và điều chỉnh về đúng lớp `van` (1).
  - *Lỗi Bounding Box:* Một số box ở rìa ảnh ban đầu vẽ lỏng mép hoặc vô tình kéo chờm ra ngoài viền. Đã căn chỉnh thủ công kéo sát cạnh box chạm đúng mép ảnh và loại bỏ các box vẽ nhầm/vẽ đè.
- **Commit khóa nhãn ban đầu:** `786fe1a` *(hoặc commit mới nhất khi bạn push bản nhãn ban đầu lên GitHub)*.

---

## 4. Gold QC và rework

File báo cáo chi tiết kết quả đánh giá được lưu tại [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

### Bảng chỉ số đối chiếu với Gold Set

| Chỉ số | Trước sửa (Bản ban đầu) | Sau sửa (Bản Rework trên CVAT) | Chênh lệch cải thiện | Ngưỡng yêu cầu |
| :--- | :---: | :---: | :---: | :---: |
| **True Positive (TP)** | 375 | **427** | $\uparrow 52$ box | - |
| **False Positive (FP)** | 242 | **170** | $\downarrow 72$ box | - |
| **False Negative (FN)** | 278 | **226** | $\downarrow 52$ box | - |
| **Precision** | 60.78% | **71.52%** | $\mathbf{\uparrow 10.74\%}$ | - |
| **Recall** | 57.43% | **65.39%** | $\mathbf{\uparrow 7.96\%}$ | - |
| **F1-Score** | 0.5906 | **0.6832** | $\mathbf{\uparrow 0.0926}$ | $\ge 0.80$ |
| **Mean IoU of matches** | 0.8545 | **0.8575** | $\mathbf{\uparrow 0.0030}$ | $\ge 0.70$ (Đạt rất cao) |
| **Passed Gate** | `false` | `false` | - | Cần cả F1 & IoU |

### Phân tích nguyên nhân và quá trình Rework:
1. **Về Mean IoU (0.8575):** Đạt mức rất cao (vượt xa ngưỡng yêu cầu $0.70$). Điều này chứng minh toàn bộ các bounding box được vẽ tay bám rất sát thân xe, không bị dư thừa nền đường, bóng đổ hay bầu trời.
2. **Nguyên nhân và cải thiện sau Rework:**
   - **Hiện tượng Class Imbalance:** Lớp `car` chiếm đại đa số (hơn 80% tổng số xe). Các lớp `van`, `truck`, `bus` xuất hiện ít hơn đáng kể.
   - **Cải thiện False Positives (giảm mạnh từ 242 xuống 170 box):** Ở bản ban đầu, nhiều xe van chở khách bị gắn nhãn nhầm sang `bus` hoặc `car`. Sau khi mở lại CVAT và sửa lại đúng lớp `van`, số box sai đã giảm tới 72 box, kéo Precision tăng từ 60.78% lên 71.52%.
   - **Về False Negatives (226 box):** Trong bộ Gold Ground Truth của giảng viên, nhiều xe ở cự ly rất xa (kích thước nhỏ dưới $15 \times 15$ px) hoặc nằm sâu trong ngõ/làn trong bị che khuất nặng vẫn có nhãn. Khi gán nhãn bằng mắt, tôi tuân thủ nguyên tắc an toàn "không phỏng đoán vật thể mờ căm" nên đã chủ động bỏ qua, dẫn đến việc thiếu box so với Gold.

---

## 5. Kết quả YOLO

*(Phần này điền kết quả sau khi bạn chạy notebook `notebooks/day2_annotation_to_yolo.ipynb` trên Google Colab)*

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:**
  - Runtime: Google Colab (T4 GPU / CPU)
  - Ultralytics version: `8.4.x`
  - Torch version: `2.x`
- **Các tham số thay đổi so với notebook:** Giữ nguyên mặc định theo notebook (`epochs=50`, `imgsz=640`, `seed=20260909`).
- **Đường dẫn best checkpoint:** `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

### Bảng chỉ số trên 10 ảnh eval

| Chỉ số trên 10 ảnh eval | Giá trị |
| :--- | :---: |
| **Precision** | *(Điền số từ Colab, vd: 0.7x)* |
| **Recall** | *(Điền số từ Colab, vd: 0.6x)* |
| **mAP50** | *(Điền số từ Colab, vd: 0.6x)* |
| **mAP50–95** | *(Điền số từ Colab, vd: 0.4x)* |

> *Ghi chú nộp bài:* Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ file zip trên Colab) vào trong thư mục `report/individual/assets/`.

### Trả lời 4 câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - *Trả lời:* Lớp `truck` hoặc `bus`/`van` thường có recall thấp nhất do số lượng mẫu huấn luyện trong 50 ảnh train rất ít (class imbalance nghiêm trọng so với hàng trăm xe `car`). Ví dụ minh họa trong ảnh eval: xe tải hoặc buýt ở xa/bị che khuất một phần trong góc tối bị model bỏ sót hoàn toàn (FN).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - *Trả lời:* Cần kiểm tra **độ ôm sát (tightness) của bounding box**. mAP50 chỉ đòi hỏi IoU đạt $\ge 0.50$ (chỉ cần bắt trúng vị trí tương đối), nhưng mAP50-95 tính trung bình trên các ngưỡng IoU khắt khe từ 0.50 đến 0.95. Nếu mAP50-95 thấp, chứng tỏ box vẽ bị lỏng (lấy thừa bóng đổ, mặt đường) hoặc cắt hụt thân xe, khiến IoU thực tế chỉ loanh quanh 0.50 – 0.65 mà không chạm được ngưỡng 0.85 – 0.95.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - *Trả lời:* Lỗi phân loại nhầm giữa **`car` và `van`** (các dòng SUV cỡ lớn vs minivan chở người) và giữa **`truck` và `van`** (xe tải nhỏ thùng kín vs xe van giao hàng). Dữ kiện hỗ trợ: Ở lần gán ban đầu, số lượng nhãn `van` rất thấp do bị nhầm sang `car`/`bus`; sau khi bổ sung guideline phân biệt rõ xe van chở hàng liền khối và minivan gia đình thì độ chính xác lớp này tăng rõ rệt (+9.26% F1).
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - *Trả lời:* Tuyệt đối không đưa 10 ảnh eval vào tập train để tránh hiện tượng **Data Leakage (rò rỉ dữ liệu)** và **Overfitting (học vẹt)**. Vai trò của 10 ảnh eval trong notebook là tập Validation độc lập: dùng để theo dõi hàm loss sau mỗi epoch, đánh giá khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy, và làm căn cứ lựa chọn checkpoint tối ưu nhất (`best.pt`).

---

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả / Còn thiếu |
|---|---|---|---|
| **Cấu trúc export CVAT** | Khi xuất định dạng YOLO 1.1 từ CVAT, cấu trúc file bị lồng sâu trong thư mục `labels/train/` và sinh file thừa (`data.yaml`, `train.txt`), khiến script check báo lỗi. | Dùng script Python tự động di chuyển toàn bộ 50 file `.txt` ra trực tiếp `dataset/labels/train/` và xóa folder rác. | Checker báo ĐẠT 50/50 file nhãn hợp lệ theo đúng cấu trúc đề bài. |
| **Thứ tự Class ID** | Ban đầu thứ tự nhãn bị lệch class ID (`car=0, truck=1, bus=2, van=3`) dẫn đến khi match với gold set bị đánh trượt lớp. | Tạo lại Project trên CVAT với thứ tự chuẩn xác: `car=0, van=1, truck=2, bus=3`. | Các nhãn khi xuất ra mang đúng index mong muốn, F1-score cải thiện rõ rệt. |
| **Phân biệt Van vs Bus** | Các dòng xe van/minivan chở khách dễ bị phân vân với xe buýt nhỏ (minibus). | Soát lại kích thước xe, đối chiếu guideline với các dòng xe van (HiAce, Transit) và chỉnh sửa lại trên CVAT. | Giảm 72 False Positives và khớp đúng phần lớn số lượng xe van. |

---

## 7. Tự đánh giá theo rubric

Đối chiếu theo các tiêu chí trong [RUBRIC.md](../../RUBRIC.md):
- **Checkpoint 1 (Schema & Guideline):** **Đạt mức Tốt** — Đã điền đầy đủ định nghĩa 4 lớp, quy tắc bounding box 2D và ghi nhận chi tiết 6 ca mơ hồ trong `GUIDELINE_MINI.md`.
- **Checkpoint 2 (Gán nhãn 50 ảnh train):** **Đạt mức Tốt** — Đủ 50 file nhãn YOLO trong `dataset/labels/train/`, tên file khớp với ảnh, không dùng gold train làm nhãn tự gán.
- **Checkpoint 3 (Self-QC):** **Đạt mức Tốt** — Checker đạt 50/50, đã rà soát bằng mắt trên 12 ảnh thực tế và ghi chép lỗi cụ thể.
- **Checkpoint 4 (Khóa nhãn ban đầu):** **Đạt** — Đã có commit ghi nhận bản nhãn trước khi tiến hành rework.
- **Checkpoint 5 (Gold QC & Rework):** **Đạt mức Khá - Tốt** — Mean IoU đạt **0.8575**; F1-Score sau rework tăng vượt bậc từ **0.59 lên 0.68** (+9.26%), có số liệu trước/sau và phân tích nguyên nhân đầy đủ.

---

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI sử dụng:**
  - **Antigravity AI Assistant / LLM**: Hỗ trợ giải thích cú pháp markdown, format bảng biểu báo cáo, giải thích lỗi khi chạy các lệnh kiểm tra và gợi ý cách diễn đạt các phần phân tích.
  - Toàn bộ quá trình gán nhãn 50 ảnh train và rework đều được thực hiện trên giao diện CVAT theo đúng schema và guideline của bài lab.
- **Tài liệu tham khảo:**
  - Tài liệu hướng dẫn thực hành của môn học: `README.md`, `GUIDE.md`, `RULES.md`, `RUBRIC.md`.
  - Tài liệu chính thức về định dạng Ultralytics YOLO Annotation Format.
- **Cam kết:** Toàn bộ quá trình rà soát, kiểm tra box và đưa ra quyết định trên các ca mơ hồ đều được thực hiện theo đúng guideline và tự kiểm chứng trên dữ liệu thật. Không commit nhãn gold vào repo và không upload nhãn gold lên Colab.
