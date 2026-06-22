# Failure Analysis — Lab 18

**Tên:** Lê Bá Chiến  
**Module phụ trách:** Toàn bộ pipeline (Bài tập cá nhân)

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | NaN | 0.9157 | +0.9157 |
| Answer Relevancy | NaN | 0.8839 | +0.8839 |
| Context Precision | NaN | 0.9500 | +0.9500 |
| Context Recall | NaN | 0.8750 | +0.8750 |

## Bottom-5 Failures

### #1
- **Question:** Bao lâu phải đổi mật khẩu một lần?
- **Expected:** (Đúng theo tài liệu chính sách bảo mật nội bộ)
- **Got:** Câu trả lời không có tính xác thực cao hoặc sinh ảo giác do quá trình truy xuất không cung cấp đủ ngữ cảnh.
- **Worst metric:** Faithfulness (Score: 0.0)
- **Error Tree:** Output sai → LLM hallucinating. Root cause: API sinh ảo giác khi context không chặt chẽ hoặc do lỗi fallback API limit.
- **Suggested fix:** Tighten prompt, lower temperature.

### #2
- **Question:** Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT?
- **Expected:** Gồm nhiều vế (Người phê duyệt và các bước từ IT).
- **Got:** Truy xuất được rất nhiều đoạn văn nhưng độ liên quan của các đoạn văn này bị loãng, không chứa đúng thông tin cốt lõi.
- **Worst metric:** Context Precision (Score: 0.3333)
- **Error Tree:** Context sai → Too many irrelevant chunks. Root cause: Câu hỏi dài có nhiều ý, Vector search trả về các đoạn không chứa đúng ý hỏi.
- **Suggested fix:** Add reranking or metadata filter (Tăng top-k trước khi rerank).

### #3
- **Question:** Thâm niên bao nhiêu năm thì được cộng thêm ngày phép?
- **Expected:** Đoạn văn nói về chính sách nghỉ phép và thâm niên.
- **Got:** Kết quả tìm kiếm không đưa ra được chính xác đoạn văn quy định về số năm thâm niên.
- **Worst metric:** Context Recall (Score: 0.5)
- **Error Tree:** Context thiếu → Missing relevant chunks. Root cause: Việc cắt đoạn (chunking) chưa đủ độ mịn hoặc BM25 không bắt được từ khóa.
- **Suggested fix:** Improve chunking or add BM25.

### #4
- **Question:** Có cần kích hoạt xác thực đa yếu tố (MFA) không?
- **Expected:** Có/Không kèm quy định cụ thể.
- **Got:** Bỏ sót đoạn context chứa thông tin cụ thể về MFA.
- **Worst metric:** Context Recall (Score: 0.5)
- **Error Tree:** Context thiếu → Missing relevant chunks. Root cause: Tương tự như trên, câu hỏi quá ngắn khiến Dense vector bắt hụt nếu context không chứa từ khoá MFA rõ ràng.
- **Suggested fix:** Improve chunking or add BM25.

### #5
- **Question:** Thông tin lương thuộc cấp độ phân loại dữ liệu nào?
- **Expected:** Cấp độ Tuyệt Mật/Nội Bộ (ví dụ).
- **Got:** Thiếu context nói rõ về phân loại của thông tin lương.
- **Worst metric:** Context Recall (Score: 0.5)
- **Error Tree:** Context thiếu → Missing relevant chunks. Root cause: Vấn đề trong pha Retrieval bị trượt (recall thấp).
- **Suggested fix:** Improve chunking or add BM25.

## Case Study (presentation)

**Question:** Bao lâu phải đổi mật khẩu một lần?

**Error Tree walkthrough:**
1. Output đúng? → Không, điểm Faithfulness = 0.0.
2. Context đúng? → Context thu thập được có thể đúng nhưng bị loãng hoặc thiếu ý chính về thời hạn (Recall không hoàn hảo).
3. Query rewrite OK? → Query giữ nguyên từ người dùng, có thể cần Rewrite để rõ ý hơn (ví dụ: "thời hạn thay đổi mật khẩu định kỳ").
4. Fix ở bước: Cần cải thiện Prompt (Tighten prompt, lower temperature) ở bước Generation để bắt buộc LLM chỉ trả lời dựa trên context. Thêm Query Expansion.

**Nếu có thêm 1 giờ:**
- Sẽ cải tiến bước M5 (Enrichment) để sinh ra đa dạng câu hỏi giả định (HyQA) hơn (hiện tại n_questions=3 chưa đủ phủ hết).
- Thử nghiệm việc tích hợp Query Rewriting để cải thiện Context Recall (điểm metric tệ thứ 2) trước khi đưa vào BM25+Dense.
