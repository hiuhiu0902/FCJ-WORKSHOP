---
title: "AWS Cloud Mastery Series #1"
date: "2025-11-15T08:30:00+07:00"
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---


# Báo cáo tổng hợp: “AI/ML/GenAI on AWS Workshop”

### Tổng quan sự kiện
**Thời gian:** 15/11/2025
**Địa điểm:** Văn phòng AWS Việt Nam (Tòa nhà Bitexco)
**Quy mô:** Hơn 200 người tham dự, bao gồm sinh viên và thành viên AWS Cloud Club.

### Mục tiêu sự kiện
- Cung cấp cái nhìn tổng quan về bối cảnh AI/ML tại Việt Nam.
- Đi sâu vào **Amazon SageMaker** cho quy trình Machine Learning toàn diện (End-to-end).
- Khám phá năng lực **Generative AI** với **Amazon Bedrock**.
- Học các chiến lược thực tế để xây dựng AI Agents và triển khai RAG (Retrieval-Augmented Generation).

### Diễn giả
- **Kha Van**
- **Kiet Lam**
- **Aiden Dinh**

### Điểm nhấn chính

#### 1. Tổng quan Dịch vụ AWS AI/ML (ML Truyền thống)
- **Amazon SageMaker:** Được giới thiệu là nền tảng toàn diện cho vòng đời ML.
- **Quy trình:** Bao gồm chuẩn bị/gán nhãn dữ liệu, huấn luyện mô hình (training), tinh chỉnh (tuning) và triển khai (deployment).
- **MLOps:** Nhấn mạnh tầm quan trọng của MLOps tích hợp để vận hành mượt mà.
- **Live Demo:** Trải nghiệm trực tiếp giao diện SageMaker Studio.

#### 2. Generative AI với Amazon Bedrock
- **Mô hình nền tảng (Foundation Models):** So sánh và hướng dẫn chọn lựa giữa các mô hình hàng đầu như **Claude, Llama, Titan**.
- **Prompt Engineering:** Các kỹ thuật nâng cao như Chain-of-Thought (tư duy chuỗi) và Few-shot learning để cải thiện kết quả đầu ra.
- **RAG (Retrieval-Augmented Generation):** Phân tích kiến trúc kết nối AI với cơ sở dữ liệu riêng (Vector Database) để giảm thiểu ảo giác (hallucination).
- **Bedrock Agents:** Xây dựng quy trình làm việc đa bước, nơi AI có thể thực thi tác vụ (gọi API) thay vì chỉ chat.
- **Guardrails:** Thiết lập bộ lọc an toàn để chặn nội dung độc hại.

#### 3. Từ PoC đến Production
- Buổi chia sẻ nhấn mạnh hành trình từ bản thử nghiệm (Proof-of-Concept) đến giải pháp thực tế.
- Tập trung vào chiến lược sử dụng **Amazon Bedrock AgentCore** để xây dựng giải pháp có khả năng mở rộng.

### Bài học rút ra

#### Sức mạnh của sự lựa chọn
- Với Amazon Bedrock, chúng ta không bị khóa vào một mô hình duy nhất. Ta có thể chọn công cụ phù hợp nhất (ví dụ: Claude cho tư duy logic, Titan cho việc tạo vector embeddings).

#### RAG là yếu tố then chốt
- Để xây dựng ứng dụng doanh nghiệp đáng tin cậy, không thể chỉ dựa vào dữ liệu huấn luyện của mô hình. RAG là chìa khóa để cung cấp ngữ cảnh chính xác và cập nhật.

#### Bảo mật trong AI
- **Guardrails** là bắt buộc. Việc cấu hình bộ lọc nội dung là bước không thể thiếu trước khi phát hành bất kỳ ứng dụng GenAI nào ra công chúng.

### Áp dụng vào công việc

- **Thử nghiệm Bedrock:** Tôi dự định xây dựng một chatbot đơn giản dùng Amazon Bedrock để test các kỹ thuật prompt khác nhau.
- **Triển khai RAG:** Tôi sẽ thử tích hợp một bộ dữ liệu nhỏ (ví dụ: tài liệu sản phẩm) để xem RAG cải thiện câu trả lời như thế nào so với prompt thông thường.
- **Khám phá Agents:** Tìm hiểu cách dùng Bedrock Agents để tự động hóa các tác vụ backend trong dự án Bán Thẻ Game (ví dụ: tra cứu trạng thái đơn hàng bằng ngôn ngữ tự nhiên).

### Trải nghiệm sự kiện

Không khí tại văn phòng AWS Việt Nam rất sôi nổi với hơn **200 người tham dự**.

**Điều đọng lại:**
- **Nội dung toàn diện:** Sự chuyển tiếp từ ML truyền thống (SageMaker) sang GenAI hiện đại (Bedrock) giúp tôi có cái nhìn trọn vẹn về hệ sinh thái AWS.
- **Demo cuốn hút:** Phần Live Demo xây dựng chatbot GenAI giúp các khái niệm trừu tượng trở nên cụ thể, dễ hiểu.
- **Tinh thần cộng đồng:** Mini-game Kahoot cuối giờ là cách tuyệt vời để kiểm tra kiến thức và gắn kết mọi người.
- **Sự dẫn dắt:** Lời cảm ơn đặc biệt đến mentor **Anh Nguyễn Gia Hưng** đã hiện thực hóa chương trình "First Cloud AI Journey" này.

> **Kết luận:** Workshop này đã thu hẹp khoảng cách giữa lý thuyết và thực hành, giúp tôi tự tin hơn để bắt đầu tích hợp các tính năng AI vào dự án của mình.