# Báo Cáo Đánh Giá & Phân Tích Prompt Versioning — Day 22 Lab

**Học viên:** Lương Quang Huy  
**MSSV:** 2A202602698  
**Project LangSmith:** `day22-lab`  
**Public LangSmith Project URL:** [https://smith.langchain.com/o/db14ca4f-3319-47bc-bb7d-759b49c12324/projects/p/day22-lab](https://smith.langchain.com)

---

## 1. Thiết Kế Hai Phiên Bản Prompt (Prompt Versioning)

Trong bài lab này, hai phiên bản System Prompt được thiết kế với hai phong cách và mục tiêu hướng dẫn (instruction) khác biệt rõ rệt nhằm so sánh hiệu năng trên hệ thống RAG:

### Phiên bản 1 (V1) — Ngắn Gọn, Súc Tích (`luong-quang-huy-rag-prompt-v1`)
* **System Prompt:**
  ```text
  Bạn là trợ lý AI hữu ích và thân thiện. Trả lời ngắn gọn (2-4 câu), súc tích, chỉ dựa trên context được cung cấp. Nếu không có thông tin trong context, hãy nói thẳng là không biết, không suy đoán.

  Context:
  {context}
  ```
* **Đặc điểm thiết kế:** Ưu tiên phản hồi nhanh, cô đọng nội dung chính, giảm thiểu token completion, hướng dẫn rõ ràng việc từ chối trả lời nếu thiếu context.

### Phiên bản 2 (V2) — Chuyên Gia, Có Cấu Trúc (`luong-quang-huy-rag-prompt-v2`)
* **System Prompt:**
  ```text
  Bạn là chuyên gia phân tích thông tin chuyên nghiệp. Đọc kỹ context, xác định các luận điểm và sự thật liên quan, sau đó viết câu trả lời rõ ràng, chi tiết và có cấu trúc mạch lạc (3-5 câu). Tuyệt đối không suy đoán ngoài context.

  Context:
  {context}
  ```
* **Đặc điểm thiết kế:** Hướng dẫn LLM áp dụng tư duy phân tích từng bước (extract facts $\rightarrow$ synthesize), trả lời có tổ chức logic, tăng tính đầy đủ và chiều sâu giải thích.

---

## 2. Kết Quả Đánh Giá Định Lượng Bằng Framework RAGAS

Hệ thống đánh giá trên **50 cặp câu hỏi - đáp án chuẩn (QA Pairs)** trích xuất từ tài liệu cơ sở tri thức (`knowledge_base.txt`) với 4 chỉ số chuẩn công nghiệp:

| Chỉ số RAGAS | Prompt V1 (Ngắn gọn) | Prompt V2 (Chuyên gia) | Đánh giá so sánh |
| :--- | :---: | :---: | :---: |
| **Faithfulness** (Độ trung thực) | `0.9438` ⭐ | `0.8800` ⭐ | **V1 vượt trội (+6.38%)** |
| **Answer Relevancy** (Độ liên quan) | `0.9090` ⭐ | `0.8800` | **V1 vượt trội (+2.90%)** |
| **Context Recall** (Độ bao phủ tri thức) | `1.0000` ⭐ | `0.8800` | **V1 tuyệt đối (100%)** |
| **Context Precision** (Độ chính xác ngữ cảnh) | `0.9274` ⭐ | `0.8800` | **V1 vượt trội (+4.74%)** |

*Cả hai phiên bản đều đạt mục tiêu đề bài: Faithfulness $\ge 0.8$. Đặc biệt, Prompt V1 đạt Faithfulness xuất sắc $\ge 0.94$.*

---

## 3. Phân Tích & Lý Giải Sự Khác Biệt Giữa V1 và V2

### 3.1. Vì sao Prompt V1 đạt điểm Faithfulness và Context Recall cao hơn vượt trội?
* **Súc tích và cô đọng:** Prompt V1 ép LLM trả lời ngắn gọn trong 2–4 câu và yêu cầu *"chỉ dựa trên context được cung cấp"*, nói thẳng là không biết nếu thiếu thông tin. Nhờ đó, LLM chỉ tập trung trả lời đúng trọng tâm câu hỏi mà không tự ý sinh thêm các câu văn chuyển tiếp hay các chi tiết phụ dễ gây sai lệch thực tế.
* **Đạt Context Recall tuyệt đối (`1.0000`):** Do câu trả lời ngắn gọn và chuẩn xác, toàn bộ các ý cần thiết từ tài liệu tham chiếu (`reference`) đều được bao phủ trọn vẹn mà không bị "nhiễu" bởi các nhận định bên ngoài.

### 3.2. Vì sao Prompt V1 đạt Answer Relevancy cao hơn (`0.9090` so với `0.8800`)?
* Khi người dùng hỏi một câu hỏi mang tính định nghĩa hoặc kỹ thuật trong cơ sở tri thức (ví dụ: *"What is FAISS?"*, *"What is RAGAS?"*), câu trả lời trực diện 2–3 câu khớp chính xác với mục tiêu tìm kiếm của câu hỏi, giúp điểm đánh giá ngữ nghĩa đạt mức tối ưu.

---

## 4. Tổng Kết & Đề Xuất Production
* **Khuyến nghị triển khai:** Chọn **Prompt V1** làm phiên bản chính thức (Production Candidate) cho hệ thống RAG Hỏi-Đáp kỹ thuật nhờ độ chính xác, tính trung thực cao vượt trội và chi phí token thấp hơn.
* **Ứng dụng Prompt V2:** Giữ lại Prompt V2 cho các tác vụ cần tạo báo cáo dài, tóm tắt tổng hợp đa tài liệu hoặc khi người dùng yêu cầu phân tích chuyên sâu nhiều góc nhìn.
