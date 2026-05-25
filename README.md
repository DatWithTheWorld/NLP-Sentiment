# Vietnamese Sentiment Analysis - Final Project (NLP)

Hệ thống phân tích cảm xúc tiếng Việt cho đồ án cuối kỳ môn Xử Lý Ngôn Ngữ Tự Nhiên.

**Input ví dụ:** `"Sản phẩm rất tốt, giao hàng nhanh"`
**Output ví dụ:** `Positive (96%)`

---

## Cấu trúc nhóm (3 người)

| Người | Hướng tiếp cận | Notebook chính |
|-------|----------------|----------------|
| **Người 1** | Machine Learning truyền thống (Naive Bayes, Logistic Regression, SVM) | `notebooks/01_ml_baseline.ipynb` |
| **Người 2** | Deep Learning (LSTM, BiLSTM, GRU) | `notebooks/02_dl_baseline.ipynb` |
| **Người 3** | **Hybrid Ensemble** (SVM + LSTM + PhoBERT - Voting) | `notebooks/03_hybrid_ensemble.ipynb` |

---

## Cấu trúc thư mục

```
NLPF/
├── README.md                      # File này
├── requirements.txt               # Dependencies
├── .gitignore
├── data/
│   ├── raw/                       # UIT-VSFC raw files (sau khi tải)
│   ├── processed/                 # Data đã preprocess (CSV)
│   └── README.md                  # Hướng dẫn tải dataset
├── notebooks/                     # TẤT CẢ code nằm ở đây (notebooks-only)
│   ├── 00_data_preparation.ipynb  # Shared: tải data + tiền xử lý
│   ├── 01_ml_baseline.ipynb       # Người 1 — ML
│   ├── 02_dl_baseline.ipynb       # Người 2 — DL
│   ├── 03_hybrid_ensemble.ipynb   # Người 3 — main (self-contained)
│   └── 04_dashboard.ipynb         # Gradio demo dashboard
├── models/                        # Saved trained models (.pkl, .pt)
├── results/                       # Metrics + plots
└── report/
    └── REPORT_PERSON3.md          # Báo cáo nghiên cứu Người 3
```

> **Quy ước:** dự án chỉ dùng `.ipynb`, không có thư mục `src/`. Mỗi notebook tự
> chứa đủ code để chạy độc lập (chấp nhận lặp ~20 dòng tiền xử lý giữa notebooks
> để đổi lấy tính độc lập).

---

## Dataset: UIT-VSFC (+ tuỳ chọn augment)

**Vietnamese Students' Feedback Corpus** – 16.175 câu phản hồi sinh viên, gán nhãn 3 lớp:
- `0` = Negative
- `1` = Neutral
- `2` = Positive

Đã chia sẵn train/dev/test. Xem `data/README.md` để biết chi tiết.

> Nguồn: Nguyen et al. (2018) – UIT-VSFC: Vietnamese Students' Feedback Corpus for Sentiment Analysis.

**Notebook `00_data_preparation.ipynb` tự tải tự động** theo thứ tự ưu tiên:
1. File raw có sẵn trong `data/raw/UIT-VSFC/` (nếu đã tải tay).
2. HuggingFace Hub: `uitnlp/vietnamese_students_feedback` rồi `ura-hcmut/UIT-VSFC`.
3. Sample 300 dòng (fallback khi không có mạng — chỉ dùng smoke-test).

### Muốn nhiều data hơn?

Trong notebook 00 có **section 2.5 — Data Augmentation (optional)**. Đặt biến `USE_AUGMENT = True` để gộp thêm Vietnamese subset từ `clapAI/MultiLingualSentiment` (~127k câu, 3 lớp). Có thể nâng `train` lên ~100k câu.

**Lưu ý:** augment chỉ thêm vào `train`; `dev/test` luôn giữ thuần UIT-VSFC để metric phản ánh đúng task.

---

## Cài đặt nhanh

**Yêu cầu Python: 3.10 – 3.13.** Project đã test trên 3.13.

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1      # Windows PowerShell
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name nlpf --display-name "NLPF (venv)"
jupyter lab
```

Sau đó chạy lần lượt:
1. `00_data_preparation.ipynb` – tải + tiền xử lý data
2. `03_hybrid_ensemble.ipynb` – train 3 model thành phần + ensemble
3. `04_dashboard.ipynb` – chạy dashboard Gradio để demo

### Khắc phục lỗi cài đặt phổ biến

**Lỗi `numpy` build from source (Windows, không có MSVC):**
```
ERROR: Unknown compiler(s): [['icl'], ['cl'], ['cc'], ['gcc'], ...]
```
Nguyên nhân: phiên bản numpy bị pin cũ → không có wheel cho Python của bạn → pip phải build từ C source code.

Cách xử lý:
1. Đảm bảo dùng requirements.txt mới nhất (đã nới `numpy>=1.26`, không còn pin `<2.0`).
2. Nâng cấp pip trước: `python -m pip install --upgrade pip`.
3. Nếu vẫn lỗi: kiểm tra version Python bằng `python --version`. Nếu là 3.14+ (quá mới) → tạo venv với Python 3.11 hoặc 3.12.

**Lỗi `jupyter: The term ... is not recognized`:**
Bạn quên kích hoạt venv. Chạy `.venv\Scripts\Activate.ps1` trước. Nếu PowerShell chặn script: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

**PhoBERT tải chậm / fail mạng:**
Notebook 03 đã có fallback — nếu không load được PhoBERT, ensemble vẫn chạy với 2 thành phần (SVM + BiLSTM).

---

## Pipeline tổng thể

```
        ┌──────────────┐
        │  UIT-VSFC    │
        └──────┬───────┘
               ▼
        ┌──────────────┐
        │ Preprocess   │  (lowercase, segment, remove stopwords, ...)
        └──────┬───────┘
               ▼
   ┌───────────┼───────────┐
   ▼           ▼           ▼
┌──────┐  ┌──────┐  ┌──────────┐
│ ML   │  │ DL   │  │ Hybrid   │
│ NB   │  │ LSTM │  │ SVM+     │
│ LR   │  │BiLSTM│  │ LSTM+    │
│ SVM  │  │ GRU  │  │ PhoBERT  │
└──┬───┘  └──┬───┘  └────┬─────┘
   └─────────┴────────────┘
               ▼
       ┌──────────────┐
       │  Evaluation  │  Accuracy / Precision / Recall / F1
       └──────┬───────┘
              ▼
       ┌──────────────┐
       │ Gradio Demo  │
       └──────────────┘
```

---

## Tài liệu tham khảo chính
1. Nguyen et al., 2018 – *UIT-VSFC*
2. Nguyen & Tuan Nguyen, 2020 – *PhoBERT: Pre-trained language models for Vietnamese*
3. Hochreiter & Schmidhuber, 1997 – *LSTM*
4. Cortes & Vapnik, 1995 – *Support-Vector Networks*
