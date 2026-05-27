# Báo cáo — Người 2: Deep Learning Baseline

**Đồ án cuối kỳ — Xử lý ngôn ngữ tự nhiên**  
**Đề tài:** Phân tích cảm xúc tiếng Việt (Vietnamese Sentiment Analysis)  
**Dataset:** UIT-VSFC (16.175 câu, 3 lớp: Negative / Neutral / Positive)

---

## 1. Vai trò trong nhóm

| Người             | Hướng tiếp cận                                          | File chính                           |
| ----------------- | ------------------------------------------------------- | ------------------------------------ |
| Người 1           | ML truyền thống (Naive Bayes, Logistic Regression, SVM) | `notebooks/01_ml_baseline.ipynb`     |
| **Người 2 (tôi)** | **Deep Learning (LSTM, BiLSTM, GRU)**                   | **`notebooks/02_dl_baseline.ipynb`** |
| Người 3           | Hybrid Ensemble (SVM + BiLSTM + PhoBERT — Voting)       | `notebooks/03_hybrid_ensemble.ipynb` |

---

## 2. Mục tiêu theo NLP Skeleton (phần DL)

1. **Word Embedding**: dùng embedding trainable; hỗ trợ nạp pretrained Word2Vec/FastText nếu có.
2. **Sequence Models**: triển khai **LSTM**, **BiLSTM**, **GRU** để so sánh.
3. **Training**: thiết lập epoch, learning rate, dropout, batch size; theo dõi overfitting.
4. **Visualization**: vẽ **loss/accuracy curves** và **confusion matrix**.

---

## 3. Pipeline Deep Learning

```
Raw text
  ↓
Preprocess (lowercase, tokenize, segment, stopwords)
  ↓
Vocab + Embedding (trainable / pretrained optional)
  ↓
Sequence Model (LSTM / BiLSTM / GRU)
  ↓
Classifier (Linear → Softmax)
  ↓
Evaluation (Accuracy / Precision / Recall / Macro-F1)
  ↓
Visualization (curves + confusion matrix)
```

---

## 4. Thực nghiệm

### 4.1. Cấu hình chạy

- Python 3.10+
- PyTorch 2.x
- sklearn 1.4+
- Underthesea / PyVi (tokenize & segment)

### 4.2. Kết quả (đã lưu trong `results/dl_baseline.json`)

| Model  | Accuracy | Precision | Recall | F1 macro |
| ------ | -------: | --------: | -----: | -------: |
| BiLSTM |   0.8913 |    0.7955 | 0.6901 |   0.7117 |
| GRU    |   0.8806 |    0.7639 | 0.6827 |   0.7012 |
| LSTM   |   0.8828 |    0.7559 | 0.6798 |   0.6975 |

> **Ghi chú:** Macro-F1 được ưu tiên do dữ liệu lệch lớp (Neutral ~5%).

### 4.3. Output đã sinh ra

- Model + vocab:
  - `models/dl_baseline.pt`
  - `models/dl_baseline_vocab.json`
- Metrics:
  - `results/dl_baseline.json`
  - `results/dl_baseline.csv`
- Visualization:
  - `results/dl_curves.png`
  - `results/dl_confusion_matrix.png`

---

## 5. Nhận xét nhanh

- **BiLSTM** cho kết quả tốt nhất trong 3 mô hình DL baseline.
- Các mô hình đều ổn định nhưng **neutral class** khó do dữ liệu ít.
- Embedding pretrained (FastText/Word2Vec) đã hỗ trợ trong code, nhưng chưa bật do chưa có file embedding local.

---

## 6. Hạn chế & hướng mở rộng

1. **Không dùng pretrained embedding** → có thể tăng F1 nếu nạp FastText.
2. **Chưa thử tuning sâu** (embedding dim, hidden size, số layer).
3. **Chưa thử attention** hoặc transformer nhỏ (DistilPhoBERT).

---

## 7. Reproducibility

Chạy theo thứ tự:

1. `notebooks/00_data_preparation.ipynb`
2. `notebooks/02_dl_baseline.ipynb`
3. (Tuỳ chọn) `notebooks/04_dashboard.ipynb`

Seed mặc định = 42.
