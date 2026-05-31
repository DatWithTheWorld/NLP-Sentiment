# Guide trình bày — Person 2 (Deep Learning Baseline)

Tài liệu này là **bản trình bày đầy đủ** dựa trên kết quả trong `results/` và dashboard `notebooks/04_dashboard.ipynb`.  
Thời lượng khuyến nghị: **4–5 phút**.

---

## 1) Mở đầu (30–40s)

**Mục tiêu nói:** giới thiệu vai trò, dataset, phạm vi công việc.

**Script trình bày:**

> “Em là Người 2, phụ trách **Deep Learning baseline**. Em so sánh ba kiến trúc LSTM, BiLSTM và GRU trên bộ dữ liệu UIT‑VSFC để tạo baseline cho nhóm.”
>
> “UIT‑VSFC là bộ dữ liệu phản hồi sinh viên với 3 lớp cảm xúc: Negative, Neutral, Positive, tổng 16.175 câu.”

**Điểm nhấn:**

- Dataset: UIT‑VSFC (16.175 câu, 3 lớp).
- Lý do chọn 3 model: các RNN sequence phổ biến, dễ so sánh và phù hợp text ngắn.

---

## 2) Pipeline & thiết kế mô hình (50–60s)

**Mục tiêu nói:** mô tả pipeline đúng theo NLP Skeleton + nhấn mạnh tính công bằng khi so sánh.

**Script trình bày:**

> “Pipeline theo skeleton gồm 5 bước: preprocess → embedding → sequence model → evaluation → visualization.”
>
> “Ở bước embedding, em dùng **trainable embedding** để giữ đơn giản, đồng thời có hỗ trợ nạp FastText/Word2Vec nếu có file.”
>
> “Ba mô hình LSTM, BiLSTM, GRU đều dùng **cùng khung dữ liệu, cùng preprocessing, cùng metric** để so sánh công bằng.”

**Điểm nhấn:**

- Embedding: trainable (hỗ trợ pretrained nếu có FastText/Word2Vec).
- Sequence models: LSTM / BiLSTM / GRU chung pipeline.
- Evaluation: Accuracy / Precision / Recall / **Macro‑F1** (ưu tiên vì dữ liệu lệch lớp).

---

## 3) Dashboard kết quả (2 phút)

> Mở `04_dashboard.ipynb` và trình bày theo 3 phần: **bảng metrics → biểu đồ → curves & confusion matrix**.

### 3.1. Bảng metrics (CSV)

**Nguồn:** `results/dl_baseline.csv`

**Script trình bày:**

> “Bảng metrics thể hiện 4 chỉ số: Accuracy, Precision, Recall và Macro‑F1. Em đọc từng model theo thứ tự, rồi mới kết luận tổng quát.”
>
> “**LSTM**: Accuracy ~0.883, Precision ~0.756, Recall ~0.680, Macro‑F1 ~0.697. Điều này cho thấy LSTM học ổn định nhưng vẫn bỏ sót khá nhiều mẫu ở lớp ít (Neutral), nên Recall và F1 chưa cao.”
>
> “**GRU**: Accuracy ~0.881, Precision ~0.764, Recall ~0.683, Macro‑F1 ~0.701. GRU nhỉnh hơn LSTM ở Precision và F1, tức là dự đoán đúng hơn một chút trên các lớp khó.”
>
> “**BiLSTM**: Accuracy ~0.891, Precision ~0.796, Recall ~0.690, Macro‑F1 ~0.712. BiLSTM cao nhất ở cả Precision và F1, chứng tỏ mô hình nắm ngữ cảnh tốt hơn.”
>
> “Em dùng **Macro‑F1** vì dữ liệu lệch lớp, đặc biệt lớp Neutral ít. Macro‑F1 phản ánh tốt hơn hiệu năng trên từng lớp thay vì chỉ nhìn Accuracy.”

**Điểm cần nhấn:**

- BiLSTM tốt nhất trong 3 mô hình.
- Macro‑F1 phù hợp hơn accuracy khi dữ liệu lệch lớp.

### 3.2. Biểu đồ so sánh

**Nguồn:** dashboard (plot metrics)

**Script trình bày:**

> “Biểu đồ cột giúp nhìn rõ **độ chênh giữa 3 model** theo từng metric.”
>
> “Ở trục Accuracy, BiLSTM cao nhất nhưng không cách biệt quá lớn → nghĩa là cả 3 model đều học được mặt bằng chung.”
>
> “Ở trục Precision và Macro‑F1, BiLSTM nhỉnh hơn rõ hơn → đây mới là điểm quyết định vì phản ánh chất lượng dự đoán ở lớp khó.”
>
> “GRU thường nằm giữa LSTM và BiLSTM → có cải thiện nhẹ so với LSTM nhưng chưa vượt BiLSTM.”

### 3.3. Training curves + Confusion matrix

**Nguồn:** `results/dl_curves.png`, `results/dl_confusion_matrix.png`

**Script trình bày:**

> “**Training curves** cho thấy loss giảm đều và dev‑accuracy tăng ổn định → mô hình học tốt, không có dấu hiệu overfit mạnh.”
>
> “**Confusion matrix** của model tốt nhất (BiLSTM) cho thấy lớp **Neutral** thường bị nhầm sang Positive/Negative. Đây là lý do Recall và Macro‑F1 không quá cao dù Accuracy tốt.”
>
> “Từ đó có thể kết luận: điểm yếu chung của cả 3 model là lớp Neutral. BiLSTM cải thiện tốt nhất nhưng vẫn chịu ảnh hưởng vì dữ liệu Neutral ít.”

---

## 4) Kết luận (30s)

**Script trình bày:**

> “Kết luận: BiLSTM là baseline DL tốt nhất trong 3 mô hình. Em sẽ dùng BiLSTM làm đại diện khi so sánh với ML và Hybrid.”

---

## 5) Hạn chế & hướng mở rộng (30–40s)

**Script trình bày:**

> “Hiện tại em chưa dùng pretrained embedding và chưa tune sâu hyper‑params.”
>
> “Hướng mở rộng là nạp FastText/Word2Vec, tăng hidden size hoặc thêm attention để cải thiện F1.”

---

## 6) Tóm tắt 1 câu (kết thúc)

> “Person 2 cung cấp baseline DL đáng tin cậy, với BiLSTM đạt kết quả tốt nhất trên UIT‑VSFC và sẵn sàng cho dashboard tổng hợp.”

---

## 7) Câu hỏi dự phòng (Q&A ngắn)

- **Tại sao dùng macro‑F1?**  
  Vì dữ liệu lệch lớp, accuracy có thể cao nhưng dự đoán sai Neutral.

- **Vì sao BiLSTM tốt hơn?**  
  Vì đọc ngữ cảnh hai chiều, phù hợp câu tiếng Việt ngắn.

- **Nếu có thêm thời gian sẽ làm gì?**  
  Nạp FastText/Word2Vec, tune hidden/dropout, thử attention.

---

## Gợi ý slide (7 trang)

1. Vai trò & mục tiêu
2. Pipeline DL
3. Bảng metrics
4. Biểu đồ so sánh
5. Curves + Confusion matrix
6. Kết luận & hướng mở rộng
7. Q&A
