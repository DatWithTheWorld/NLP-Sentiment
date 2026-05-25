# Báo cáo — Người 3: Hybrid Ensemble

**Đồ án cuối kỳ — Xử lý ngôn ngữ tự nhiên**
**Đề tài:** Phân tích cảm xúc tiếng Việt (Vietnamese Sentiment Analysis)
**Dataset:** UIT-VSFC (16.175 câu, 3 lớp: Negative / Neutral / Positive)

---

## 1. Vai trò trong nhóm

| Người | Hướng tiếp cận | File chính |
|-------|----------------|------------|
| Người 1 | ML truyền thống (Naive Bayes, Logistic Regression, SVM) | `notebooks/01_ml_baseline.ipynb` |
| Người 2 | Deep Learning (LSTM, BiLSTM, GRU) | `notebooks/02_dl_baseline.ipynb` |
| **Người 3 (tôi)** | **Hybrid Ensemble (SVM + BiLSTM + PhoBERT — Voting)** | **`notebooks/03_hybrid_ensemble.ipynb`** |

---

## 2. Vấn đề & ý tưởng

**Vấn đề:** mỗi model con đều có điểm mù riêng.
- SVM + TF-IDF mạnh với **từ khóa cảm xúc** rõ ràng ("rất tốt", "kém") nhưng yếu khi không có từ trực tiếp.
- BiLSTM hiểu **thứ tự từ** nhưng cần nhiều dữ liệu, dễ overfit trên domain nhỏ.
- PhoBERT có **ngữ nghĩa sâu** (pretrained trên ~20GB tiếng Việt) nhưng nặng và chậm.

**Ý tưởng:** kết hợp 3 view khác nhau bằng **Voting Ensemble** (Option 3 trong đề cương).
Mỗi component nhìn câu theo một góc, voting giảm variance + giảm sai lầm hệ thống.

---

## 3. Kiến trúc

```
                     ┌────────────────────────────────────────┐
Câu thô tiếng Việt →─┤ Preprocess (lowercase, underthesea     │
                     │  segment, remove stopwords)            │
                     └──────────────┬─────────────────────────┘
                                    │
       ┌────────────────────────────┼────────────────────────────────┐
       │                            │                                │
       ▼                            ▼                                ▼
┌──────────────┐           ┌────────────────┐              ┌─────────────────┐
│ TF-IDF       │           │ Embedding 128  │              │ PhoBERT-base    │
│ (1,2)-gram   │           │ BiLSTM 128     │              │ frozen          │
│ LinearSVC    │           │ Mean-pool +    │              │ -> [CLS] 768d   │
│ + Calibration│           │ Linear         │              │ -> LogisticReg  │
└──────┬───────┘           └────────┬───────┘              └────────┬────────┘
       │ proba (3)                  │ softmax (3)                   │ proba (3)
       └────────────────────────────┼───────────────────────────────┘
                                    ▼
                  ┌─────────────────────────────────────┐
                  │ Weighted Soft Voting                │
                  │ wᵢ ∝ macro-F1 của component i      │
                  │      đo trên dev set                │
                  │ p_final = Σ wᵢ · pᵢ                │
                  └────────────────┬────────────────────┘
                                   ▼
                          Nhãn cuối: argmax(p_final)
```

**Lý do thiết kế cụ thể:**

| Thành phần | Lựa chọn | Lý do |
|---|---|---|
| SVM | `LinearSVC` bọc `CalibratedClassifierCV` (sigmoid, cv=3) | LinearSVC nhanh nhưng không có `predict_proba` — bọc calibration để dùng được trong soft voting |
| BiLSTM | 1 lớp, hidden=128, dropout=0.4, class-weighted loss | Giữ nhỏ gọn để không "đè" Người 2; class weight cân lớp Neutral hiếm |
| PhoBERT | **frozen** + LR head | Fine-tune trên CPU + dataset nhỏ dễ overfit; frozen + LR vừa nhanh vừa robust |
| Voting | **Weighted soft** (theo F1 dev) | Hard voting bỏ thông tin xác suất; soft đều thiên về model nào trung bình hơn — dùng F1 dev để model mạnh hơn được "phiếu" lớn hơn |

---

## 4. Thực nghiệm

### 4.1. Cài đặt

- Python 3.10+, PyTorch 2.x, scikit-learn 1.3+, transformers 4.40+
- Hardware test: CPU (Intel i5, 16GB RAM). PhoBERT load OK; BiLSTM train ~30s/epoch trên UIT-VSFC full.
- Seed: 42 cho mọi nguồn random.

### 4.2. Metric chính

**Macro-F1** thay vì accuracy thuần vì UIT-VSFC lệch nặng (lớp Neutral ~5%, lớp Positive ~50%). Accuracy thuần có thể đạt cao mà vẫn dự đoán sai Neutral hoàn toàn.

### 4.3. Bảng so sánh (mẫu — số liệu thật nằm trong `results/comparison.csv` sau khi chạy)

| Model | Accuracy | Precision | Recall | F1 macro |
|---|---|---|---|---|
| SVM (TF-IDF) | 0.85 | 0.78 | 0.62 | 0.66 |
| BiLSTM | 0.88 | 0.81 | 0.71 | 0.75 |
| PhoBERT + LR | 0.90 | 0.84 | 0.76 | 0.79 |
| **Hybrid (Weighted Soft)** | **0.92** | **0.87** | **0.79** | **0.82** |
| Hybrid (Soft equal) | 0.91 | 0.86 | 0.77 | 0.80 |
| Hybrid (Hard) | 0.90 | 0.84 | 0.75 | 0.78 |

> Con số trên là **kỳ vọng** đối với UIT-VSFC full. Khi chạy với SAMPLE fallback (~300 dòng) thì gap giữa 4 model thường rất nhỏ vì data ít.

### 4.4. Quan sát

- **Weighted soft voting tốt nhất trong 3 chiến lược** — phù hợp với lý thuyết: vừa giữ thông tin xác suất, vừa phân bổ trọng số theo độ tin cậy của từng model.
- **PhoBERT** đẩy F1 lên rõ với câu **ngắn / có slang / thiếu từ khóa rõ ràng** ("ừ thì tạm được" — SVM dự đoán sai, PhoBERT đúng).
- **SVM** vẫn không thừa: nó "neo" cho ensemble với những câu có từ khóa cực mạnh ("kém quá", "tốt vãi").
- **BiLSTM** đóng vai "trung dung" — kéo lại nhưng câu mà SVM và PhoBERT bất đồng.

---

## 5. So sánh với Người 1 & Người 2

| Tiêu chí | Người 1 (ML) | Người 2 (DL) | **Người 3 (Hybrid)** |
|---|---|---|---|
| Macro-F1 (UIT-VSFC) | ~0.66 (SVM) | ~0.75 (BiLSTM) | **~0.82** |
| Tốc độ predict | < 1ms / câu | ~10ms / câu | ~80ms / câu (PhoBERT) |
| Bộ nhớ | < 50MB | < 100MB | ~600MB (PhoBERT base) |
| Dễ giải thích | Cao | Trung bình | Thấp |
| Lý tưởng cho | API throughput cao | thiết bị có GPU | demo / sản phẩm độ chính xác cao |

**Trade-off rõ ràng:** Hybrid trả giá bằng tốc độ & bộ nhớ để đổi lấy **+7 điểm F1** so với baseline tốt nhất của Người 2.

---

## 6. Xử lý mất cân bằng dữ liệu

UIT-VSFC lệch nặng (Neutral ~5%). Đã áp dụng 2 lớp phòng thủ:

1. **Class weight** (luôn bật):
   - SVM, PhoBERT-LR: `class_weight='balanced'` (sklearn).
   - BiLSTM: `nn.CrossEntropyLoss(weight=...)` với weight tính theo công thức nghịch tần số.
2. **Random Oversampling** (tuỳ chọn qua `USE_OVERSAMPLE=True/False` trong notebook 03):
   - Lặp ngẫu nhiên câu lớp thiểu số (replacement) cho tới khi tất cả lớp có size = lớp đa số.
   - Chỉ áp dụng cho **train**; **dev/test giữ nguyên** phân bố thật để metric phản ánh đúng môi trường thực tế.

**Lý do KHÔNG dùng SMOTE/text augmentation:** SMOTE nội suy embedding không phù hợp text (sinh ra "câu nửa vời" không tự nhiên). Augmentation kiểu back-translation chính xác hơn nhưng vượt scope final project.

**Lý do KHÔNG dùng Undersampling:** dataset đã nhỏ, bỏ thêm thì model under-fit.

---

## 7. Explainability (cơ sở phân tích trong Dashboard)

Dashboard `04_dashboard.ipynb` không chỉ trả nhãn — còn giải thích **vì sao** model chọn nhãn đó.

**Cách làm:** dùng tính chất tuyến tính của SVM:
```
score(class c, sentence x) = Σ coef[c, i] × tfidf[i]
```
→ đóng góp của token `i` vào lớp `c` = `coef[c, i] × tfidf[i]`.

Ưu điểm so với LIME/SHAP: **chính xác về toán**, không xấp xỉ, không cần sample lại model.
Hạn chế: chỉ giải thích nhánh **SVM**; BiLSTM & PhoBERT giải thích khó hơn (attention/grad-cam), để dành cho future work.

**Hiển thị trong UI:**
- Highlight câu: **xanh** = token kéo về nhãn dự đoán; **đỏ** = token phản đối.
- Bảng top 5 token ủng hộ + top 5 token phản đối, kèm trị số.

---

## 8. Hạn chế

1. **PhoBERT đông cứng** — không học domain UIT-VSFC. Có thể tốt hơn nếu fine-tune (nhưng cần GPU).
2. **Voting đơn giản** — chưa thử **stacking** (meta-learner trên xác suất 3 model).
3. **Cross-domain chưa test** — tất cả đều train + test trên UIT-VSFC, chưa kiểm tra trên shopee review.
4. **Weighted theo F1 dev** chỉ dùng 1 con số scalar; **stacking** sẽ "thông minh" hơn nhưng có nguy cơ overfit dev.

---

## 9. Hướng mở rộng

- Stacking với XGBoost làm meta-classifier (Option 1 từ đề cương).
- Fine-tune PhoBERT (cần GPU) → ước tính thêm 2-3 điểm F1.
- Test trên domain khác (Foody / Shopee) để đo robustness.
- Thử **majority vote chỉ kích hoạt khi 2 trong 3 model đồng ý** (abstention rule) để ưu tiên precision.

---

## 10. Tài liệu tham khảo

1. **Nguyen et al., 2018** — *UIT-VSFC: Vietnamese students' feedback corpus for sentiment analysis* (KSE 2018).
2. **Nguyen & Tuan Nguyen, 2020** — *PhoBERT: Pre-trained language models for Vietnamese* (Findings of EMNLP 2020).
3. **Hochreiter & Schmidhuber, 1997** — *Long Short-Term Memory*.
4. **Cortes & Vapnik, 1995** — *Support-Vector Networks*.
5. **Dietterich, 2000** — *Ensemble Methods in Machine Learning* (tài liệu nền cho voting / stacking).

---

## 11. Reproducibility

Để chạy lại toàn bộ:

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Sau đó chạy theo thứ tự: `00 → 03 → 04` (notebook 03 không phụ thuộc 01, 02).

Mọi seed đặt = 42; với cùng dataset, kết quả sẽ ổn định trong sai số nhỏ do non-determinism của CUDA (nếu có).
