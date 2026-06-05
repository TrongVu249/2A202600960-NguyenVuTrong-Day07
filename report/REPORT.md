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
*Gọi N là tổng số ký tự (10,000), C là kích thước chunk (500), và O là độ trùng lặp (50). Kích thước thực tế tăng thêm của mỗi chunk tiếp theo sau chunk đầu tiên là C - O = 500 - 50 = 450.Số lượng chunk được tính theo công thức:{Số chunks} = \left\lceil \frac{N - O}{C - O} \right\rceil = \left\lceil \frac{10,000 - 50}{500 - 50} \right\rceil = \left\lceil \frac{9,950}{450} \right\rceil = \lceil 22.11 \rceil = 23*
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

**Loại:** `RecursiveChunker`

**Mô tả cách hoạt động:**
- `RecursiveChunker` chia nhỏ văn bản đệ quy dựa trên một danh sách các ký tự phân tách theo độ ưu tiên giảm dần: đoạn văn (`\n\n`), dòng (`\n`), câu kết thúc bằng dấu chấm (`. `), khoảng trắng từ (` `), và ký tự rỗng (`""`).
- Nếu đoạn văn bản hiện tại lớn hơn `chunk_size` (ở đây cấu hình là 200), nó sẽ phân tách bằng ký tự phân tách có ưu tiên cao nhất, sau đó đệ quy xuống các phần nhỏ hơn bằng các ký tự phân tách tiếp theo.
- Sau khi phân tách, các phần nhỏ sẽ được gộp lại với nhau một cách tối đa mà không vượt quá `chunk_size` nhằm bảo toàn toàn bộ ngữ cảnh một cách tự nhiên nhất.

**Tại sao tôi chọn strategy này cho domain nhóm?**
- Tài liệu nhóm thuộc domain **Tài liệu hướng dẫn & Quy trình (Technical Docs & SOPs)**. Các văn bản này chứa nhiều tiêu đề, danh sách gạch đầu dòng và các đoạn văn logic hoàn chỉnh.
- Sử dụng `RecursiveChunker` giúp giữ nguyên các khối thông tin logic (như một bước hướng dẫn hoặc đoạn code ví dụ) thay vì cắt ngang ở giữa câu như `FixedSizeChunker`, giúp mô hình ngôn ngữ (LLM) sau này nhận diện đầy đủ ngữ cảnh để trả lời chính xác.

**Code snippet:**
```python
from src import RecursiveChunker

# Khởi tạo và sử dụng chiến lược Recursive Chunker
chunker = RecursiveChunker(chunk_size=200)
chunks = chunker.chunk(text)
```

### So Sánh: Strategy của tôi vs Baseline

| Tài liệu | Strategy | Chunk Count | Avg Length | Retrieval Quality? |
|-----------|----------|-------------|------------|--------------------|
| `customer_support_playbook.txt` | `fixed_size` (Baseline) | 11 | 199.27 | Trung bình kém (Cắt ranh giới từ hoặc câu khiến thông tin bị khuyết thiếu khi truy vấn) |
| | `recursive` (Của tôi) | 11 | 152.09 | Tốt (Các chunk kết thúc ở khoảng trắng hoặc ranh giới câu hợp lý, đầy đủ ý nghĩa) |
| `python_intro.txt` | `fixed_size` (Baseline) | 13 | 195.69 | Trung bình (Các cấu trúc code Python và diễn giải cú pháp bị cắt vụn nửa chừng) |
| | `recursive` (Của tôi) | 12 | 160.08 | Rất tốt (Giữ trọn vẹn ngữ nghĩa câu và toàn bộ khối code Python nhỏ) |


---

## 4. My Approach — Cá nhân (10 điểm)

Giải thích cách tiếp cận của bạn khi implement các phần chính trong package `src`.

### Chunking Functions

**`SentenceChunker.chunk`** — approach:
- Sử dụng hàm `re.split` với biểu thức chính quy lookbehind: `(?<=\. |! |\? |\.\n)` để phát hiện ranh giới câu mà không loại bỏ các ký tự kết thúc câu (`.`, `!`, `?`).
- Sau khi loại bỏ khoảng trắng thừa ở mỗi câu bằng `.strip()`, gom nhóm các câu lại theo kích thước tối đa là `max_sentences_per_chunk` và ghép chúng lại bằng khoảng trắng `" "`.

**`RecursiveChunker.chunk` / `_split`** — approach:
- Triển khai thuật toán chia để trị đệ quy sử dụng danh sách dấu phân tách theo độ ưu tiên mặc định: `["\n\n", "\n", ". ", " ", ""]`.
- Với mỗi bước đệ quy, nếu độ dài văn bản hiện tại nhỏ hơn `chunk_size` thì trả về chính nó (Base case). Ngược lại, chia nhỏ bằng dấu phân tách hiện tại và đệ quy sâu hơn với các phần tử quá khổ bằng dấu phân tách kế tiếp. Cuối cùng, gộp tối đa các mảnh nhỏ liền kề để tạo thành chunk tối ưu nhất.

### EmbeddingStore

**`add_documents` + `search`** — approach:
- **Lưu trữ**: Triển khai lưu trữ in-memory đơn giản bằng một danh sách các từ điển lưu cặp `(id, content, metadata, embedding)` tạo bởi `_make_record`, hỗ trợ chuyển đổi sang ChromaDB nếu được kích hoạt.
- **Tìm kiếm**: Tính toán vector embedding của câu truy vấn (`query`), so sánh độ tương đồng cosine với tất cả các vector đã lưu trữ thông qua hàm `compute_similarity`, rồi sắp xếp giảm dần theo điểm số để trả về `top_k` kết quả.

**`search_with_filter` + `delete_document`** — approach:
- **Filter**: Thực hiện pre-filtering (lọc trước) các bản ghi trong danh sách lưu trữ bằng cách so khớp chính xác tất cả các khoá-giá trị trong `metadata_filter` trước khi tiến hành tính toán độ tương đồng cosine.
- **Delete**: Sử dụng list comprehension để lọc và giữ lại tất cả các bản ghi có `id` khác `doc_id` đồng thời không chứa `doc_id` trong trường metadata.

### KnowledgeBaseAgent

**`answer`** — approach:
- Gọi hàm tìm kiếm tương đồng của `EmbeddingStore` để lấy ra các chunk văn bản liên quan nhất làm ngữ cảnh.
- Ghép nối nội dung các chunk này và nhúng vào một prompt mẫu được thiết kế sẵn theo cấu trúc: `Context:\n{context}\n\nQuestion: {question}\nAnswer:` trước khi gọi hàm gọi mô hình ngôn ngữ `llm_fn`.

### Test Results

```
============================= test session starts =============================
platform win32 -- Python 3.13.13, pytest-9.0.3, pluggy-1.6.0 -- C:\Users\Admin\AppData\Local\Programs\Python\Python313\python.exe
cachedir: .pytest_cache
rootdir: D:\Project\Vin_AI\Lab 7\2A202600960-NguyenVuTrong-Day07
plugins: anyio-4.13.0
collecting ... collected 42 items

tests/test_solution.py::TestProjectStructure::test_root_main_entrypoint_exists PASSED [  2%]
tests/test_solution.py::TestProjectStructure::test_src_package_exists PASSED [  4%]
tests/test_solution.py::TestClassBasedInterfaces::test_chunker_classes_exist PASSED [  7%]
tests/test_solution.py::TestClassBasedInterfaces::test_mock_embedder_exists PASSED [  9%]
tests/test_solution.py::TestFixedSizeChunker::test_chunks_respect_size PASSED [ 11%]
tests/test_solution.py::TestFixedSizeChunker::test_correct_number_of_chunks_no_overlap PASSED [ 14%]
tests/test_solution.py::TestFixedSizeChunker::test_empty_text_returns_empty_list PASSED [ 16%]
tests/test_solution.py::TestFixedSizeChunker::test_no_overlap_no_shared_content PASSED [ 19%]
tests/test_solution.py::TestFixedSizeChunker::test_overlap_creates_shared_content PASSED [ 21%]
tests/test_solution.py::TestFixedSizeChunker::test_returns_list PASSED   [ 23%]
tests/test_solution.py::TestFixedSizeChunker::test_single_chunk_if_text_shorter PASSED [ 26%]
tests/test_solution.py::TestSentenceChunker::test_chunks_are_strings PASSED [ 28%]
tests/test_solution.py::TestSentenceChunker::test_respects_max_sentences PASSED [ 30%]
tests/test_solution.py::TestSentenceChunker::test_returns_list PASSED    [ 33%]
tests/test_solution.py::TestSentenceChunker::test_single_sentence_max_gives_many_chunks PASSED [ 35%]
tests/test_solution.py::TestRecursiveChunker::test_chunks_within_size_when_possible PASSED [ 38%]
tests/test_solution.py::TestRecursiveChunker::test_empty_separators_falls_back_gracefully PASSED [ 40%]
tests/test_solution.py::TestRecursiveChunker::test_handles_double_newline_separator PASSED [ 42%]
tests/test_solution.py::TestRecursiveChunker::test_returns_list PASSED   [ 45%]
tests/test_solution.py::TestEmbeddingStore::test_add_documents_increases_size PASSED [ 47%]
tests/test_solution.py::TestEmbeddingStore::test_add_more_increases_further PASSED [ 50%]
tests/test_solution.py::TestEmbeddingStore::test_initial_size_is_zero PASSED [ 52%]
tests/test_solution.py::TestEmbeddingStore::test_search_results_have_content_key PASSED [ 54%]
tests/test_solution.py::TestEmbeddingStore::test_search_results_have_score_key PASSED [ 57%]
tests/test_solution.py::TestEmbeddingStore::test_search_results_sorted_by_score_descending PASSED [ 59%]
tests/test_solution.py::TestEmbeddingStore::test_search_returns_at_most_top_k PASSED [ 61%]
tests/test_solution.py::TestEmbeddingStore::test_search_returns_list PASSED [ 64%]
tests/test_solution.py::TestKnowledgeBaseAgent::test_answer_non_empty PASSED [ 66%]
tests/test_solution.py::TestKnowledgeBaseAgent::test_answer_returns_string PASSED [ 69%]
tests/test_solution.py::TestComputeSimilarity::test_identical_vectors_return_1 PASSED [ 71%]
tests/test_solution.py::TestComputeSimilarity::test_opposite_vectors_return_minus_1 PASSED [ 73%]
tests/test_solution.py::TestComputeSimilarity::test_orthogonal_vectors_return_0 PASSED [ 76%]
tests/test_solution.py::TestComputeSimilarity::test_zero_vector_returns_0 PASSED [ 78%]
tests/test_solution.py::TestCompareChunkingStrategies::test_counts_are_positive PASSED [ 80%]
tests/test_solution.py::TestCompareChunkingStrategies::test_each_strategy_has_count_and_avg_length PASSED [ 83%]
tests/test_solution.py::TestCompareChunkingStrategies::test_returns_three_strategies PASSED [ 85%]
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::test_filter_by_department PASSED [ 88%]
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::test_no_filter_returns_all_candidates PASSED [ 90%]
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::test_returns_at_most_top_k PASSED [ 92%]
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::test_delete_reduces_collection_size PASSED [ 95%]
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::test_delete_returns_false_for_nonexistent_doc PASSED [ 97%]
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::test_delete_returns_true_for_existing_doc PASSED [100%]

============================= 42 passed in 0.07s ==============================
```

**Số tests pass:** 42 / 42

---

## 5. Similarity Predictions — Cá nhân (5 điểm)

| Pair | Sentence A | Sentence B | Dự đoán | Actual Score | Đúng? |
|------|-----------|-----------|---------|--------------|-------|
| 1 | "The dog chases the cat." | "A hound runs after the kitty." | High (Semantic similarity) | 0.0782 | Không (Do mock embedder gán ngẫu nhiên bằng MD5) |
| 2 | "The dog chases the cat." | "The cat chases the dog." | High (Similar word usage) | -0.0466 | Không (MD5 hash thay đổi hoàn toàn khi đổi vị trí từ) |
| 3 | "Python is a popular programming language." | "The quick brown fox jumps over the lazy dog." | Low (Unrelated concepts) | 0.2131 | Không (Mã băm ngẫu nhiên có độ tương đồng ngẫu nhiên cao) |
| 4 | "I love coding in Python." | "I hate writing software in Python." | Medium/High (Opposite sentiment, same subject) | 0.0402 | Không (Mã băm MD5 trực giao không học ngữ nghĩa) |
| 5 | "Vector databases store embeddings for similarity search." | "Vector databases store embedding representations for similarity searching." | High (Almost identical) | 0.1068 | Không (MD5 thay đổi toàn bộ vector khi chuỗi sai lệch nhỏ) |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn nghĩa?**
- Kết quả bất ngờ nhất là cặp câu không liên quan (Pair 3) lại có điểm tương đồng cao nhất (0.2131), trong khi các cặp đồng nghĩa hoặc gần giống nhau lại có điểm gần bằng 0.
- Điều này chứng minh rằng `_mock_embed` (dựa trên thuật toán băm MD5 để tạo vector ngẫu nhiên) hoàn toàn không có khả năng hiểu ngữ nghĩa văn bản. Trong thực tế, các mô hình embedding ngữ nghĩa (như SentenceTransformers hay OpenAI) được huấn luyện để chuyển hóa các mối quan hệ ngữ nghĩa thành khoảng cách hình học, từ đó các câu có ý nghĩa giống nhau sẽ luôn có vector nằm gần nhau và điểm cosine similarity cao.

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
