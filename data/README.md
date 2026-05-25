# Dataset – UIT-VSFC

**Vietnamese Students' Feedback Corpus** (UIT-VSFC) là bộ dữ liệu phản hồi của sinh viên về môn học, đã được gán nhãn cảm xúc 3 lớp.

## Thống kê

| Tập | Số câu |
|-----|--------|
| Train | 11.426 |
| Dev | 1.583 |
| Test | 3.166 |
| **Tổng** | **16.175** |

## Nhãn

| ID | Tên | Ý nghĩa |
|----|-----|---------|
| 0 | Negative | Tiêu cực |
| 1 | Neutral | Trung tính |
| 2 | Positive | Tích cực |

## Cấu trúc file gốc

Mỗi tập (train/dev/test) gồm 3 file song song:
- `sents.txt` – 1 câu / dòng
- `sentiments.txt` – 1 nhãn (0/1/2) / dòng, tương ứng với `sents.txt`
- `topics.txt` – chủ đề (không dùng trong project này)

## Cách tải

**Notebook `00_data_preparation.ipynb` tự tải dataset** theo thứ tự ưu tiên:

1. **Đã có sẵn** trong `data/raw/UIT-VSFC/` → load thẳng, skip download.
2. **HuggingFace Hub** — `uitnlp/vietnamese_students_feedback`, fallback `ura-hcmut/UIT-VSFC`. Sau khi tải lần đầu, notebook **tự lưu xuống `data/raw/UIT-VSFC/`** theo format chuẩn → lần sau không phải tải lại.
3. **SAMPLE 300 dòng** — fallback khi mất mạng (chỉ smoke-test).

Nếu cả 1+2 fail, tải thủ công từ https://nlp.uit.edu.vn/datasets/ rồi giải nén vào `data/raw/UIT-VSFC/`.

### Cấu trúc raw (sau khi tải)

```
data/raw/
├── UIT-VSFC/                       # tự tạo khi chạy notebook 00
│   ├── train/
│   │   ├── sents.txt               # 11.426 câu, 1 câu/dòng
│   │   └── sentiments.txt          # 11.426 nhãn 0/1/2
│   ├── dev/
│   │   ├── sents.txt               # 1.583 câu
│   │   └── sentiments.txt
│   └── test/
│       ├── sents.txt               # 3.166 câu
│       └── sentiments.txt
└── clapAI_vi.csv                   # (optional) cache khi bật USE_AUGMENT=True
```

> Lưu ý: `.gitignore` đã loại trừ `data/raw/*` nên các file này không bị commit lên git (đỡ phình repo).

## Citation

```bibtex
@inproceedings{nguyen2018uit,
  title={UIT-VSFC: Vietnamese students' feedback corpus for sentiment analysis},
  author={Nguyen, Kiet Van and Nguyen, Vu Duc and Nguyen, Phu X.V. and Truong, Tham T.H. and Nguyen, Ngan Luu-Thuy},
  booktitle={2018 10th International Conference on Knowledge and Systems Engineering (KSE)},
  pages={19--24},
  year={2018}
}
```
