# Individual Reflection — Lab 18

**Tên:** Lê Bá Chiến  
**Module phụ trách:** Toàn bộ pipeline (M1, M2, M3, M4, M5 - Bài tập cá nhân)

---

## 1. Đóng góp kỹ thuật

- **Module đã implement:** M1 (Chunking), M2 (Search), M3 (Reranking), M4 (Evaluation), M5 (Enrichment).
- **Các hàm/class chính đã viết:** 
  - `chunk_semantic()`, `chunk_hierarchical()`, `chunk_structure_aware()`
  - `HybridSearch` (kết hợp BM25 và Dense Search qua RRF)
  - `CrossEncoderReranker`
  - `evaluate_ragas()`
  - `enrich_chunks()` (cập nhật tích hợp thêm cache để tối ưu API call).
- **Số tests pass:** 37/37

## 2. Kiến thức học được

- **Khái niệm mới nhất:** Quá trình chuẩn hóa một Production RAG thực tế, đặc biệt là framework RAGAS sử dụng LLM-as-a-judge để tự động đánh giá các metric (Faithfulness, Context Precision, Context Recall, Answer Relevancy).
- **Điều bất ngờ nhất:** Sức mạnh của Hybrid Search kết hợp với Reciprocal Rank Fusion (RRF) cùng việc dùng Cross-Encoder Reranking đem lại kết quả truy xuất (retrieval) chính xác hơn vượt trội so với phiên bản Naive Baseline thông thường.
- **Kết nối với bài giảng:** Đã áp dụng thành công các kiến thức nâng cao về Document Chunking (Semantic, Hierarchical) và Contextual Prepend để giải quyết bài toán thiếu hụt bối cảnh (context loss) được đề cập trong lý thuyết.

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:** 
  1. Hết hạn mức API miễn phí (Free tier quota) của Alibaba API trong quá trình gọi 107 API calls ở bước Enrichment và Evaluation.
  2. Lỗi font chữ (UnicodeEncodeError) trên Terminal Windows khi hệ thống tự chấm điểm `check_lab.py` (do script in ra các ký tự Emoji).
- **Cách giải quyết:** 
  1. Xây dựng thêm cơ chế Cache: lưu mảng chunk sau khi enrich vào file `data/enriched_chunks_cache.json`. Ở các lần chạy sau, hệ thống sẽ tự đọc từ file thay vì gọi lại API. Thiết lập fallback (chế độ bỏ qua) để bypass các lỗi API limit.
  2. Cấu hình lại chuẩn mã hóa của môi trường bằng lệnh powershell: `$env:PYTHONIOENCODING="utf-8"` trước khi chạy `check_lab.py`.
- **Thời gian debug:** Khoảng 30-45 phút.

## 4. Nếu làm lại & Action Plan cho Project

- **Sẽ làm khác điều gì:** Sẽ nâng cấp tài khoản API để quá trình chạy RAGAS evaluation diễn ra mượt mà hơn. Sẽ tuning thêm prompt để RAGAS hiểu ngữ cảnh Tiếng Việt chính xác hơn thay vì xài prompt mặc định.
- **Module nào muốn thử tiếp:** Muốn đào sâu thêm về M5 (Enrichment) như Auto Metadata Extraction để tự động trích xuất thực thể (NER) và phân quyền truy cập, cũng như áp dụng Agentic RAG.
