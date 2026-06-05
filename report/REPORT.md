# Báo Cáo Lab 7: Embedding & Vector Store

**Họ tên:** [Nguyễn Vũ Trọng]
**Nhóm:** [C1]
**Ngày:** [05/06/2026]

---

## 1. Warm-up (5 điểm)

### Cosine Similarity (Ex 1.1)

**High cosine similarity nghĩa là gì?**
> *High cosine similarity nghĩa là hai đoạn văn bản có sự tương đồng rất lớn về mặt ý nghĩa và ngữ cảnh cốt lõi trong không gian vector, bất kể chúng có độ dài khác nhau hay sử dụng từ vựng khác nhau.*

**Ví dụ HIGH similarity:**
- Sentence A: "Chiếc ô tô màu đỏ lao rất nhanh trên đường cao tốc."
- Sentence B: "Một phương tiện bốn bánh màu hồng lựu đang di chuyển với tốc độ cao trên lộ trình liên bang."
- Tại sao tương đồng: AI nhận diện được các cặp từ đồng nghĩa như "ô tô" = "phương tiện bốn bánh" và "lao rất nhanh" = "di chuyển với tốc độ cao", khiến hai vector chỉ về cùng một hướng ý nghĩa.

**Ví dụ LOW similarity:**
- Sentence A: "Ngân hàng Trung ương quyết định tăng lãi suất để kiềm chế lạm phát."
- Sentence B: "Một lát bánh táo ăn cùng kem vani thì thật là tuyệt vời."
- Tại sao khác: Hai câu thuộc hai trường từ vựng và chủ đề hoàn toàn tách biệt (kinh tế đối lập với ẩm thực), khiến các vector của chúng nằm vuông góc và xa rời nhau.

**Tại sao cosine similarity được ưu tiên hơn Euclidean distance cho text embeddings?**
> *Cosine similarity được ưu tiên vì nó chỉ đo góc giữa các vector để so sánh ý nghĩa và hoàn toàn loại bỏ được sự ảnh hưởng của độ dài văn bản (vốn là điểm yếu khiến Euclidean distance đánh giá sai khi một văn bản quá ngắn còn văn bản kia quá dài).*

### Chunking Math (Ex 1.2)

**Document 10,000 ký tự, chunk_size=500, overlap=50. Bao nhiêu chunks?**
> *Trình bày phép tính:*
*Gọi $N$ là tổng số ký tự ($10,000$), $C$ là kích thước chunk ($500$), và $O$ là độ trùng lặp ($50$). Kích thước thực tế tăng thêm của mỗi chunk tiếp theo sau chunk đầu tiên là $C - O = 500 - 50 = 450$.Số lượng chunk được tính theo công thức:$$\text{Số chunks} = \left\lceil \frac{N - O}{C - O} \right\rceil = \left\lceil \frac{10,000 - 50}{500 - 50} \right\rceil = \left\lceil \frac{9,950}{450} \right\rceil = \lceil 22.11 \rceil = 23$$*
> *Đáp án: 23 chunks*

**Nếu overlap tăng lên 100, chunk count thay đổi thế nào? Tại sao muốn overlap nhiều hơn?**
> *Khi overlap tăng lên 100, số lượng chunk sẽ tăng lên thành 25 do khoảng cách dịch chuyển của mỗi bước nhỏ lại (400 ký tự). Người ta muốn tăng overlap để tránh việc các thông tin quan trọng hoặc ngữ cảnh của câu bị cắt đôi ở ranh giới giữa hai chunk, giúp AI hiểu tài liệu liền mạch hơn.*

---

## 2. Document Selection — Nhóm (10 điểm)

### Domain & Lý Do Chọn

**Domain:** [ví dụ: Customer support FAQ, Vietnamese law, cooking recipes, ...]

**Tại sao nhóm chọn domain này?**
> *Viết 2-3 câu:*

### Data Inventory

| # | Tên tài liệu | Nguồn | Số ký tự | Metadata đã gán |
|---|--------------|-------|----------|-----------------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

### Metadata Schema

| Trường metadata | Kiểu | Ví dụ giá trị | Tại sao hữu ích cho retrieval? |
|----------------|------|---------------|-------------------------------|
| | | | |
| | | | |

---

## 3. Chunking Strategy — Cá nhân chọn, nhóm so sánh (15 điểm)

### Baseline Analysis

Chạy `ChunkingStrategyComparator().compare()` trên 2-3 tài liệu:

| Tài liệu | Strategy | Chunk Count | Avg Length | Preserves Context? |
|-----------|----------|-------------|------------|-------------------|
| `customer_support_playbook.txt` | FixedSizeChunker (`fixed_size`) | 11 | 199.27 | Không (Bị cắt ngang từ hoặc câu ở ranh giới chunk) |
| | SentenceChunker (`by_sentences`) | 4 | 421.00 | Có (Giữ nguyên cấu trúc câu hoàn chỉnh) |
| | RecursiveChunker (`recursive`) | 11 | 152.09 | Có (Giữ nguyên câu/đoạn văn logic nhờ độ ưu tiên dấu câu) |
| `python_intro.txt` | FixedSizeChunker (`fixed_size`) | 13 | 195.69 | Không (Bị cắt ngang từ hoặc câu ở ranh giới chunk) |
| | SentenceChunker (`by_sentences`) | 5 | 387.00 | Có (Giữ nguyên cấu trúc câu hoàn chỉnh) |
| | RecursiveChunker (`recursive`) | 12 | 160.08 | Có (Giữ nguyên câu/đoạn văn logic nhờ độ ưu tiên dấu câu) |
| `rag_system_design.md` | FixedSizeChunker (`fixed_size`) | 16 | 196.31 | Không (Bị cắt ngang từ hoặc câu ở ranh giới chunk) |
| | SentenceChunker (`by_sentences`) | 5 | 476.00 | Có (Giữ nguyên cấu trúc câu hoàn chỉnh) |
| | RecursiveChunker (`recursive`) | 16 | 147.56 | Có (Giữ nguyên câu/đoạn văn logic nhờ độ ưu tiên dấu câu) |

### Strategy Của Tôi

**Loại:** [FixedSizeChunker / SentenceChunker / RecursiveChunker / custom strategy]

**Mô tả cách hoạt động:**
> *Viết 3-4 câu: strategy chunk thế nào? Dựa trên dấu hiệu gì?*

**Tại sao tôi chọn strategy này cho domain nhóm?**
> *Viết 2-3 câu: domain có pattern gì mà strategy khai thác?*

**Code snippet (nếu custom):**
```python
# Paste implementation here
```

### So Sánh: Strategy của tôi vs Baseline

| Tài liệu | Strategy | Chunk Count | Avg Length | Retrieval Quality? |
|-----------|----------|-------------|------------|--------------------|
| | best baseline | | | |
| | **của tôi** | | | |

### So Sánh Với Thành Viên Khác

| Thành viên | Strategy | Retrieval Score (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| Tôi | | | | |
| [Tên] | | | | |
| [Tên] | | | | |

**Strategy nào tốt nhất cho domain này? Tại sao?**
> *Viết 2-3 câu:*

---

## 4. My Approach — Cá nhân (10 điểm)

Giải thích cách tiếp cận của bạn khi implement các phần chính trong package `src`.

### Chunking Functions

**`SentenceChunker.chunk`** — approach:
> *Viết 2-3 câu: dùng regex gì để detect sentence? Xử lý edge case nào?*

**`RecursiveChunker.chunk` / `_split`** — approach:
> *Viết 2-3 câu: algorithm hoạt động thế nào? Base case là gì?*

### EmbeddingStore

**`add_documents` + `search`** — approach:
> *Viết 2-3 câu: lưu trữ thế nào? Tính similarity ra sao?*

**`search_with_filter` + `delete_document`** — approach:
> *Viết 2-3 câu: filter trước hay sau? Delete bằng cách nào?*

### KnowledgeBaseAgent

**`answer`** — approach:
> *Viết 2-3 câu: prompt structure? Cách inject context?*

### Test Results

```
# Paste output of: pytest tests/ -v
```

**Số tests pass:** __ / __

---

## 5. Similarity Predictions — Cá nhân (5 điểm)

| Pair | Sentence A | Sentence B | Dự đoán | Actual Score | Đúng? |
|------|-----------|-----------|---------|--------------|-------|
| 1 | | | high / low | | |
| 2 | | | high / low | | |
| 3 | | | high / low | | |
| 4 | | | high / low | | |
| 5 | | | high / low | | |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn nghĩa?**
> *Viết 2-3 câu:*

---

## 6. Results — Cá nhân (10 điểm)

Chạy 5 benchmark queries của nhóm trên implementation cá nhân của bạn trong package `src`. **5 queries phải trùng với các thành viên cùng nhóm.**

### Benchmark Queries & Gold Answers (nhóm thống nhất)

| # | Query | Gold Answer |
|---|-------|-------------|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |

### Kết Quả Của Tôi

| # | Query | Top-1 Retrieved Chunk (tóm tắt) | Score | Relevant? | Agent Answer (tóm tắt) |
|---|-------|--------------------------------|-------|-----------|------------------------|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

**Bao nhiêu queries trả về chunk relevant trong top-3?** __ / 5

---

## 7. What I Learned (5 điểm — Demo)

**Điều hay nhất tôi học được từ thành viên khác trong nhóm:**
> *Viết 2-3 câu:*

**Điều hay nhất tôi học được từ nhóm khác (qua demo):**
> *Viết 2-3 câu:*

**Nếu làm lại, tôi sẽ thay đổi gì trong data strategy?**
> *Viết 2-3 câu:*

---

## Tự Đánh Giá

| Tiêu chí | Loại | Điểm tự đánh giá |
|----------|------|-------------------|
| Warm-up | Cá nhân | / 5 |
| Document selection | Nhóm | / 10 |
| Chunking strategy | Nhóm | / 15 |
| My approach | Cá nhân | / 10 |
| Similarity predictions | Cá nhân | / 5 |
| Results | Cá nhân | / 10 |
| Core implementation (tests) | Cá nhân | / 30 |
| Demo | Nhóm | / 5 |
| **Tổng** | | **/ 100** |
