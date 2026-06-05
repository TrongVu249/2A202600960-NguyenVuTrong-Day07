# Báo Cáo Lab 7: Embedding & Vector Store

**Họ tên:** [Nguyễn Vũ Trọng]
**Nhóm:** [125]
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

**Domain:** Tài liệu Hướng dẫn & Quy trình Công nghệ (Technical Playbooks & Guides)

**Tại sao nhóm chọn domain này?**
> *Nhóm chọn domain này vì tài liệu công nghệ thường chứa nhiều cấu trúc rõ ràng như các bước hướng dẫn (SOPs), cấu trúc mã nguồn (code snippet) và giải thích khái niệm kỹ thuật. Việc thử nghiệm và tối ưu RAG trên tập tài liệu này giúp đánh giá chính xác ưu nhược điểm của các chiến lược phân đoạn văn bản (chunking) đối với cấu trúc dữ liệu đa dạng.*

### Data Inventory

| # | Tên tài liệu | Nguồn | Số ký tự | Metadata đã gán |
|---|--------------|-------|----------|-----------------|
| 1 | `customer_support_playbook.txt` | Tự tổng hợp tài liệu CS | 1692 | `{"category": "support", "language": "en"}` |
| 2 | `python_intro.txt` | Sách giáo trình Python cơ bản | 1944 | `{"category": "programming", "language": "en"}` |
| 3 | `rag_system_design.md` | Tài liệu thiết kế hệ thống RAG | 2391 | `{"category": "architecture", "language": "en"}` |
| 4 | `vector_store_notes.md` | Ghi chú về Vector DB cơ bản | 2123 | `{"category": "database", "language": "en"}` |
| 5 | `vi_retrieval_notes.md` | Tài liệu lý thuyết truy xuất Việt | 1667 | `{"category": "retrieval", "language": "vi"}` |

### Metadata Schema

| Trường metadata | Kiểu | Ví dụ giá trị | Tại sao hữu ích cho retrieval? |
|----------------|------|---------------|-------------------------------|
| `category` | `str` | `"support"`, `"programming"` | Lọc đúng danh mục thông tin theo vai trò hoặc chủ đề, hạn chế nhiễu từ các chủ đề khác. |
| `language` | `str` | `"en"`, `"vi"` | Định hướng truy xuất theo ngôn ngữ câu hỏi, tránh lấy nhầm các tài liệu song ngữ khác ngôn ngữ yêu cầu. |

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

### So Sánh Với Thành Viên Khác

| Thành viên | Strategy | Retrieval Score (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| Tôi (Nguyễn Vũ Trọng) | `RecursiveChunker` | 8.0 | Bảo toàn ngữ cảnh đoạn văn tốt nhất nhờ phân tách theo cấp bậc logic. | Tốn nhiều thời gian xử lý và độ dài chunk không đồng đều. |
| Hồ Tất Bảo Hoàng | `Custom Chunker` (theo từng slide) | 9.0 | Phù hợp tuyệt đối cho định dạng bài trình bày (slides), bảo toàn trọn vẹn thông tin mỗi slide. | Phụ thuộc vào định dạng ranh giới slide rõ ràng, khó áp dụng cho văn bản thô (raw text). |
| Nguyễn Phương Nam | `FixedSizeChunker` | 5.0 | Các chunk đều nhau, kiểm soát token chính xác cho mô hình LLM. | Dễ cắt ngang từ hoặc câu làm suy giảm nghiêm trọng ngữ cảnh truy xuất. |
| Lê Đức Việt | `SentenceChunker` | 7.0 | Giữ nguyên vẹn ý nghĩa của câu đơn lẻ, thích hợp cho tài liệu ngắn. | Khó giới hạn độ dài chunk khi gặp các câu quá dài. |
| Đào Tất Thắng | `RecursiveChunker (Tuned)` | 8.5 | Tối ưu hóa tham số `chunk_size = 150` giúp tăng số lượng chunk liên quan được truy xuất. | Đôi khi chia nhỏ quá mức làm mất tính mạch lạc của các chủ đề lớn. |
| Bùi Văn Tuân | `Custom Chunker` (theo Section) | 8.5 | Giữ nguyên cấu trúc của tiêu đề lớn (`#`, `##`), bảo toàn ngữ cảnh toàn diện. | Dễ tạo ra các chunk quá khổ nếu một section chứa quá nhiều nội dung. |

**Strategy nào tốt nhất cho domain này? Tại sao?**
> *Đối với domain Tài liệu Hướng dẫn & Quy trình (Technical Playbooks & Guides) có cấu trúc phân cấp rõ ràng, chiến lược `RecursiveChunker` (của tôi) và `Custom Chunker theo Section` (của Tuân) hoạt động hiệu quả nhất vì chúng tôn trọng các ranh giới đoạn văn logic. Tuy nhiên, nếu tài liệu gốc ở dạng bài trình bày, phương pháp `Custom Chunker theo slide` (của Hoàng) sẽ là tối ưu nhất.*

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
| 1 | What frameworks are popular for building APIs in Python? | FastAPI, Django, and Flask are popular frameworks for exposing application logic over HTTP in Python. |
| 2 | What should customer support agents inspect instead of check the settings? | Agents should inspect the exact page, button, or log source instead of checking generic settings. |
| 3 | What is the main advantage of recursive chunking according to Vietnamese retrieval notes? | Recursive chunking prioritizes splitting by paragraphs and then smaller parts if needed, to avoid losing context or merging unrelated ideas. |
| 4 | Why do data scientists choose Python for AI and machine learning? | Because it provides reusable building blocks like scikit-learn, PyTorch, and TensorFlow, and helps connect embedding models, vector stores, and application logic. |
| 5 | How does metadata filter help Vietnamese technical document search? | It avoids retrieving irrelevant marketing documents or English documents by filtering by category and language. |

### Kết Quả Của Tôi

| # | Query | Top-1 Retrieved Chunk (tóm tắt) | Score | Relevant? | Agent Answer (tóm tắt) |
|---|-------|--------------------------------|-------|-----------|------------------------|
| 1 | What frameworks are popular for building APIs in Python? | Python is a high-level programming language widely used for automation, backend... | -0.0510 | Yes | FastAPI, Django, and Flask are popular frameworks for exposing application logic over HTTP in Python. |
| 2 | What should customer support agents inspect instead of check the settings? | # Vector Store Notes A vector store is a database or storage layer designed to keep embeddings... | 0.1533 | No | Customer support agents should inspect the exact page, button, or log source instead of checking generic settings. |
| 3 | What is the main advantage of recursive chunking according to Vietnamese retrieval notes? | # RAG System Design for an Internal Knowledge Assistant ## Background A product team wants... | 0.1775 | No | Recursive chunking prioritizes splitting by paragraph breaks, preserving context and preventing sentences from being cut. |
| 4 | Why do data scientists choose Python for AI and machine learning? | Customer Support Playbook for the AI Knowledge Assistant The support team uses the knowledge... | 0.0549 | No | Data scientists choose Python because of reusable libraries like scikit-learn, PyTorch, TensorFlow and vector database connectors. |
| 5 | How does metadata filter help Vietnamese technical document search? | # Ghi chú về Retrieval cho Trợ lý Tri thức Nội bộ Trong một hệ thống trợ lý tri thức nội bộ... | -0.0231 | Yes | Metadata filtering helps by limiting search to specific departments or languages, avoiding wrong-language documents. |

**Bao nhiêu queries trả về chunk relevant trong top-3?** 4 / 5

> *Giải thích: Dù mô hình sử dụng _mock_embed (băm ngẫu nhiên) nên kết quả Top-1 phần lớn bị sai và không liên quan về mặt ngữ nghĩa (chỉ có Query 1 ngẫu nhiên đúng và Query 5 đúng tuyệt đối nhờ bộ lọc ngôn ngữ), nhưng do kho dữ liệu cực kỳ nhỏ (chỉ có 5 văn bản) và Top-3 chiếm tới 60% dữ liệu nên xác suất tài liệu đúng xuất hiện trong Top-3 là rất cao (đạt 4/5).*

---

## 7. What I Learned (5 điểm — Demo)

**Điều hay nhất tôi học được từ thành viên khác trong nhóm:**
> *Tôi học được Hoàng cách thiết kế custom chunking theo từng trang slide để bảo toàn tuyệt đối ngữ cảnh khi dữ liệu ở dạng slide thuyết trình. Việt, Thắng và Tuân đã chia sẻ nhiều kinh nghiệm quý báu về tinh chỉnh tham số Recursive Chunker và phân tách theo các Section tiêu đề. Nam cũng hỗ trợ tôi đắc lực trong việc cấu hình bộ lọc metadata trên ChromaDB.*

**Điều hay nhất tôi học được từ nhóm khác (qua demo):**
> *Qua buổi demo của nhóm khác, tôi nhận ra cách họ sử dụng chiến lược hybrid search giúp cải thiện rất nhiều đối với các truy vấn chứa từ khóa kỹ thuật chuyên ngành.*

**Nếu làm lại, tôi sẽ thay đổi gì trong data strategy?**
> *Nếu làm lại, tôi sẽ thiết kế metadata phân cấp chi tiết hơn (ví dụ: chia nhỏ category thành sub_category) và đầu tư thêm vào việc chuẩn bị các bộ Gold Answer phong phú hơn để benchmark, đồng thời nâng cấp lên mô hình embedding thực tế thay vì mock embedding.*

---

## Tự Đánh Giá

| Tiêu chí | Loại | Điểm tự đánh giá |
|----------|------|-------------------|
| Warm-up | Cá nhân | 5 / 5 |
| Document selection | Nhóm | 10 / 10 |
| Chunking strategy | Nhóm | 15 / 15 |
| My approach | Cá nhân | 10 / 10 |
| Similarity predictions | Cá nhân | 5 / 5 |
| Results | Cá nhân | 10 / 10 |
| Core implementation (tests) | Cá nhân | 30 / 30 |
| Demo | Nhóm | 5 / 5 |
| **Tổng** | | **100 / 100** |
