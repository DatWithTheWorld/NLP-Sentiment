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

**Notebook `00_data_preparation.ipynb` sẽ tự tải dataset.** Nếu tự động không được, tải thủ công:

1. Trang chủ: https://nlp.uit.edu.vn/datasets/
2. Hoặc mirror Github (cộng đồng): tìm "UIT-VSFC dataset"
3. Giải nén vào: `data/raw/UIT-VSFC/`

Sau khi giải nén thư mục `data/raw/` phải có dạng:

```
data/raw/UIT-VSFC/
├── train/
│   ├── sents.txt
│   ├── sentiments.txt
│   └── topics.txt
├── dev/
│   ├── sents.txt
│   ├── sentiments.txt
│   └── topics.txt
└── test/
    ├── sents.txt
    ├── sentiments.txt
    └── topics.txt
```

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
