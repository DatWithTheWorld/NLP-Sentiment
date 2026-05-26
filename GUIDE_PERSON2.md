# Hướng dẫn Người 2 — Deep Learning Baseline

Tài liệu này giúp bạn **hiểu và chạy** phần Deep Learning (Notebook 02) và **dashboard so sánh** các model DL.

## 1) Tổng quan pipeline (theo NLP Skeleton)

1. **Word Embedding**: Word2Vec/FastText (nếu có) hoặc trainable embedding.
2. **Sequence Models**: LSTM / BiLSTM / GRU.
3. **Training**: epoch, learning rate, dropout, batch size, early-stopping.
4. **Visualization**: loss/accuracy + confusion matrix.

## 2) Chuẩn bị môi trường

- Python: 3.10–3.13
- Cài dependencies trong `requirements.txt` (đã có `gensim` cho Word2Vec/FastText).

## 3) Chạy notebook theo thứ tự

1. **Chuẩn bị dữ liệu**
   - Mở `notebooks/00_data_preparation.ipynb` và chạy toàn bộ để tạo file trong `data/processed/`.

2. **Huấn luyện Deep Learning**
   - Mở `notebooks/02_dl_baseline.ipynb` và chạy lần lượt các cell.
   - Kết quả chính sẽ được lưu vào:
     - `results/dl_baseline.csv` + `results/dl_baseline.json`
     - `results/dl_curves.png`
     - `results/dl_confusion_matrix.png`
     - `models/dl_baseline.pt` + `models/dl_baseline_vocab.json`

3. **Dashboard so sánh**
   - Mở `notebooks/04_dashboard.ipynb` và chạy toàn bộ.
   - Chọn tab **DL Baselines** để xem bảng và biểu đồ so sánh.

## 4) Dùng Word2Vec / FastText (tuỳ chọn)

Nếu bạn muốn dùng pretrained embedding:

1. Tải embedding về máy (ví dụ FastText `.vec` hoặc `.bin`).
2. Đặt file vào đường dẫn:
   - `data/raw/embeddings/fasttext.vi.vec`
3. Notebook 02 sẽ tự nhận file nếu tồn tại và **freeze embedding 5 epoch đầu**.

> Nếu không có file pretrained, notebook sẽ tự dùng **embedding trainable**.

## 5) Lưu ý nhanh

- Nếu tab **DL Baselines** trống, hãy chắc chắn đã chạy `02_dl_baseline.ipynb`.
- Đổi các hyper-params trong cell đầu notebook 02 để cải thiện kết quả.

Chúc bạn chạy mượt! Nếu cần tinh chỉnh thêm, cứ nói mình hỗ trợ nhé.
