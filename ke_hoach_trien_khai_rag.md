# BẢN KẾ HOẠCH TRIỂN KHAI PRODUCTION RAG PIPELINE (LAB 18)

Bản kế hoạch chi tiết giúp điều phối công việc và kiểm soát tiến độ triển khai hệ thống Production RAG Pipeline nâng cao theo chuẩn yêu cầu đề bài.

---

## 📌 THÔNG TIN TỔNG QUAN LUỒNG CHẠY HỆ THỐNG
```
[Tài liệu đầu vào] 
        ↓
M1: Chunking (Semantic / Hierarchical / Structure-Aware)
        ↓
M5: Enrichment (Summarization, HyQA, Contextual Prepend, Metadata) -> *Combined Mode*
        ↓
M2: Hybrid Search Indexing (BM25 với Underthesea + Dense Embedding với Qdrant DB)
        ↓
[Người dùng đặt câu hỏi] -> Search thô (Top-20) -> M3: Reranker (Top-3 bằng Cross-Encoder)
        ↓
[LLM sinh câu trả lời] -> M4: RAGAS Evaluation & Phân tích lỗi (Error Tree)
```

---

## 📅 LỊCH TRÌNH THỰC HIỆN CHI TIẾT (TỔNG THỜI GIAN: 2 GIỜ 30 PHÚT)

### GIAI ĐOẠN 1: THIẾT LẬP MÔI TRƯỜNG & CHẠY BASELINE (Thời gian: 0:00 - 0:10)
KHỞI TẠO MÔI TRƯỜNG ẢO .VENV
- [ ] **1.1. Khởi động Cơ sở dữ liệu:** Chạy Qdrant instance qua Docker Compose: `docker compose up -d`.
- [ ] **1.2. Cài đặt thư viện:** Cài đặt các gói phụ thuộc: `pip install -r requirements.txt`.
- [ ] **1.3. Cấu hình biến môi trường:** Sao chép `.env.example` thành `.env` và điền `OPENAI_API_KEY`.
- [ ] **1.4. Lưu điểm số Baseline:** Chạy `python naive_baseline.py` để ghi nhận điểm số RAGAS ban đầu làm mốc so sánh ($\Delta$).

### GIAI ĐOẠN 2: HIỆN THỰC HÓA MÃ NGUỒN CÁC MODULES (Thời gian: 0:10 - 1:40)
- [ ] **2.1. Hoàn thiện Module 1 - Advanced Chunking (`src/m1_chunking.py`) [Thời gian: 20 phút]**
  - Triển khai `chunk_semantic()` dùng mô hình `all-MiniLM-L6-v2`.
  - Triển khai `chunk_hierarchical()` phân tách cấp Parent (2048 ký tự) và Child (256 ký tự).
  - Triển khai `chunk_structure_aware()` bám sát cấu trúc Markdown (`#`, `##`, `###`).
  - *Kiểm thử nhanh:* Chạy lệnh `pytest tests/test_m1.py`.
- [ ] **2.2. Hoàn thiện Module 2 - Hybrid Search (`src/m2_search.py`) [Thời gian: 20 phút]**
  - Viết hàm tách từ tiếng Việt `segment_vietnamese()` tích hợp thư viện `underthesea` (lưu ý chuẩn hóa dấu gạch dưới `_` thành dấu cách).
  - Kết nối Vector Search với Qdrant qua hàm `query_points()` của thư viện Qdrant Client.
  - Triển khai thuật toán trộn kết quả xếp hạng `reciprocal_rank_fusion` (RRF).
  - *Kiểm thử nhanh:* Chạy lệnh `pytest tests/test_m2.py`.
- [ ] **2.3. Hoàn thiện Module 3 - Cross-Encoder Reranking (`src/m3_rerank.py`) [Thời gian: 15 phút]**
  - Tích hợp mô hình `BAAI/bge-reranker-v2-m3` bằng thư viện `sentence_transformers`.
  - Lọc Top-20 kết quả thô ban đầu để chọn ra Top-3 mảnh có mức độ liên quan cao nhất.
  - *Kiểm thử nhanh:* Chạy lệnh `pytest tests/test_m3.py`.
- [ ] **2.4. Hoàn thiện Module 5 - LLM-based Chunk Enrichment (`src/m5_enrichment.py`) [Thời gian: 20 phút]**
  - Triển khai hàm `_enrich_single_call()` gọi LLM sinh đồng thời: Summarization, HyQA, Contextual Prepend, và Auto Metadata trong một API call duy nhất (**Combined Mode** lấy trọn điểm thưởng).
  - *Kiểm thử nhanh:* Chạy lệnh `pytest tests/test_m5.py`.
- [ ] **2.5. Hoàn thiện Module 4 - Evaluation (`src/m4_eval.py`) [Thời gian: 15 phút]**
  - Tích hợp bộ sinh dữ liệu và kiểm thử của RAGAS với 4 chỉ số: Faithfulness, Answer Relevancy, Context Precision, Context Recall dựa trên tập mẫu `test_set.json`.
  - *Kiểm thử nhanh:* Chạy lệnh `pytest tests/test_m4.py`.

### GIAI ĐOẠN 3: THỰC THI END-TO-END & PHÂN TÍCH LỖI (Thời gian: 1:40 - 2:00)
- [ ] **3.1. Chạy tích hợp toàn luồng:** Chạy lệnh `python src/pipeline.py` (hoặc `python main.py`) để chạy thử nghiệm đầu cuối hệ thống nâng cấp.
- [ ] **3.2. Đảm bảo xuất file báo cáo:** Kiểm tra xem tệp `reports/ragas_report.json` đã được tạo ra đầy đủ với mã thoát `exit code 0` hay chưa.
- [ ] **3.3. Phân tích Error Tree:** Trích xuất 5 câu hỏi có điểm số tệ nhất (Bottom-5). Điền chi tiết thông tin chẩn đoán lỗi vào file `analysis/failure_analysis.md`.

### GIAI ĐOẠN 4: VIẾT REFLECTION & KIỂM TRA ĐỊNH DẠNG NỘP BÀI (Thời gian: 2:00 - 2:30)
- [ ] **4.1. Tạo tệp Reflection cá nhân:** Tạo file `analysis/reflections/reflection_LeBaChien.md` đúng theo cấu trúc template mẫu.
- [ ] **4.2. Hoàn thiện nội dung Reflection:**
  - *Lecture Mapping:* Ánh xạ lý thuyết lớp học vào thực tế cấu hình hệ thống (như độ tương đồng Cosine trong chunking).
  - *Troubleshooting:* Ghi nhận chi tiết các mã lỗi, lỗi thư viện, hoặc xung đột phiên bản mà bạn đã xử lý khi debug.
  - *Action Plan:* Kế hoạch hành động cụ thể để ứng dụng kiến thức Production RAG này vào các dự án AI Engineer cá nhân sắp tới.
- [ ] **4.3. Chạy script kiểm tra tự động:** Thực thi lệnh `python check_lab.py` để quét toàn bộ dự án. Đảm bảo không còn ký tự `TODO` hoặc `FIXME` nào sót lại trong các file mã nguồn và nhận được thông báo: `"🚀 Bài lab sẵn sàng để nộp!"`.

---

## 🎯 CÁC MỐC ĐIỂM THƯỞNG CẦN ĐẠT ĐƯỢC (BONUS TRACKS)
*   **Bonus 1 (+3 điểm):** Triển khai cấu trúc "Combined Mode" cho Module 5 Enrichment để nén 4 tác vụ phân tích LLM vào duy nhất 1 lượt gọi API.
*   **Bonus 2 (+4 điểm):** Điểm số kiểm thử trung bình trên bộ dữ liệu RAGAS đạt mức vượt trội ($\ge 0.75$).
*   **Bonus 3 (+3 điểm):** Tệp phân tích lỗi `failure_analysis.md` có chiều sâu, mô tả rõ sơ đồ cây chẩn đoán (Error Tree Walkthrough) cho từng câu hỏi bị lỗi.
