## 📌 Giới thiệu (Introduction)
Dự án này tập trung vào bài toán **Phân tích Cảm xúc theo Khía cạnh (Aspect-Based Sentiment Analysis - ABSA)** dành cho các đánh giá sản phẩm làm đẹp trên các nền tảng thương mại điện tử tại Việt Nam. 

Khác với phân loại cảm xúc mức độ câu (sentence-level sentiment analysis) thông thường, dự án này đi sâu vào việc bóc tách từng khía cạnh cụ thể của sản phẩm (ví dụ: *Màu sắc, Kết cấu, Giá cả, Giao hàng, v.v.*) và phân loại thái độ của người dùng (Tích cực, Tiêu cực, Trung tính) đối với từng khía cạnh đó.

Việc giải quyết bài toán này đóng vai trò quan trọng trong quy trình khai phá dữ liệu văn bản, giúp tự động hóa quá trình thu thập phản hồi, từ đó tối ưu hóa sản phẩm và dịch vụ một cách chính xác. Dự án ứng dụng các mô hình ngôn ngữ lớn (Large Language Models) tiên tiến như **PhoBERT** để xử lý đặc thù của tiếng Việt, đồng thời đối chiếu hiệu năng giữa hai kiến trúc học máy: **Single-Task Learning (STL)** và **Multi-Task Learning (MTL)**.

---

## 🗂️ Tập Dữ liệu (Dataset)
Dữ liệu bao gồm các đánh giá về sản phẩm mỹ phẩm (chủ yếu là son môi) bằng tiếng Việt, được gán nhãn chi tiết.
- **Các khía cạnh (Aspects):** `stayingpower` (Độ bền màu), `texture` (Kết cấu), `smell` (Mùi hương), `price` (Giá cả), `colour` (Màu sắc), `shipping` (Giao hàng), `packing` (Đóng gói). 
  *(Lưu ý: Khía cạnh `others` đã được loại bỏ trong quá trình tiền xử lý do tính chất phân phối nhãn hầu như chỉ tập trung ở mức 'neutral', không mang lại giá trị phân biệt đặc trưng cho mô hình).*
- **Nhãn cảm xúc (Sentiment Labels):** Positive (Tích cực), Negative (Tiêu cực), Neutral (Trung tính).
- **Phân chia dữ liệu:** Dữ liệu được chia tiêu chuẩn thành `Train` (đã được tăng cường/augment lên 122,911 mẫu để tăng hiệu suất), `Test` (1,623 mẫu) và `Validation` (1,623 mẫu).
- **Định dạng nhãn Aspect:** Dữ liệu được cấu trúc theo chuẩn IOB (Inside, Outside, Beginning) dành cho bài toán nhận dạng thực thể (NER).

---

## ⚙️ Quy trình Thực hiện (Workflow)

### 1. Tiền xử lý Dữ liệu (Data Preprocessing)
- **Xử lý định dạng IOB:** Ánh xạ các từ trong câu đánh giá với nhãn tương ứng để chuẩn bị cho bài toán Token Classification.
- **Tối ưu hóa đặc trưng:** Loại bỏ nhãn `others` (đóng vai trò như một đặc trưng nhiễu, làm giảm độ tập trung của mô hình), giúp hệ thống tinh gọn và phân loại các khía cạnh chính xác hơn.
- **Data Augmentation:** Tăng cường dữ liệu tập huấn luyện (từ ~12.9k lên ~122.9k mẫu) nhằm giải quyết vấn đề mất cân bằng phân phối và cải thiện độ bao phủ ngữ cảnh của mô hình.

### 2. Xây dựng Kiến trúc Mô hình (Model Architecture)
Dự án triển khai và so sánh hai cách tiếp cận dựa trên **PhoBERT**:

#### Phương pháp 1: Single-Task Learning (STL) - Pipeline Độc lập
- **Mô hình 1 (Aspect Detection):** Đóng vai trò là bài toán Token Classification (NER). Fine-tune PhoBERT với classification layer để dự đoán chuỗi nhãn IOB.
- **Mô hình 2 (Sentiment Classification):** Đóng vai trò là bài toán Sequence Classification. Input được tạo ra bằng cách ghép khía cạnh đã trích xuất với câu gốc (VD: `[CLS] màu sắc [SEP] Son lên màu đẹp nhưng mau trôi. [SEP]`) để cung cấp ngữ cảnh dự đoán cảm xúc.

#### Phương pháp 2: Multi-Task Learning (MTL) - Học Đa Nhiệm
- Xây dựng một kiến trúc duy nhất với **Shared Encoder** (thân PhoBERT chung) để trích xuất biểu diễn ngữ nghĩa.
- Tích hợp 2 **Heads** riêng biệt trên cùng một shared encoder:
  - *Aspect Detection Head:* Lớp Linear cho bài toán Token Classification.
  - *Sentiment Classification Head:* Lớp Linear cho bài toán Sequence Classification.
- **Tối ưu hóa:** Mô hình được huấn luyện đồng thời thông qua hàm Loss tổng hợp (tổng có trọng số giữa loss của Aspect và Sentiment). Cách tiếp cận này giúp mô hình chia sẻ tri thức, tổng quát hóa tốt hơn và khắc phục điểm yếu lỗi lan truyền (error propagation) của phương pháp STL.

### 3. Đánh giá Mô hình (Evaluation Metrics)
Hệ thống được đánh giá toàn diện thông qua các độ đo:
- **Aspect Detection:** Sử dụng `F1-score` (Precision, Recall) dựa trên Exact Match (đánh giá đúng khi khớp cả phạm vi từ/span và loại/type).
- **Sentiment Classification:** Sử dụng `Macro F1-score` và `Weighted F1-score` để phản ánh chính xác hiệu suất trên các tập dữ liệu không cân bằng.
- **End-to-End Evaluation:** Đây là độ đo khắt khe và thực tế nhất — một dự đoán chỉ được coi là chính xác hoàn toàn khi hệ thống nhận diện đúng **cả** khía cạnh (aspect) lẫn cảm xúc (sentiment) tương ứng trong câu đánh giá.
README.md
Đang hiển thị README.md.
