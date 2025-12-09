---
title: "Các bài blogs đã dịch"
date: "2025-09-08"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---



###  [Blog 1 - Tối ưu hóa chi phí điện toán đám mây: Cẩm nang Rehost Migration (Phần 4 – Di trú: Hiện thực hóa kế hoạch tiết kiệm chi phí)](3.1-Blog1/)
Bài blog này là phần 4 trong chuỗi hướng dẫn tối ưu chi phí khi di trú (Rehost Migration) lên AWS, tập trung vào giai đoạn migrate - hiện thực hóa kế hoạch tiết kiệm chi phí đã lập. Bài viết giới thiệu 4 công cụ chính của AWS để tối ưu chi phí sau khi migration: AWS Billing and Cost Management (giám sát và phân tích chi phí tổng thể), AWS Cost Explorer (trực quan hóa xu hướng chi tiêu), AWS Cost Optimization Hub (đưa ra khuyến nghị tối ưu như chuyển sang Graviton instances, điều chỉnh kích thước, xóa tài nguyên không dùng), và AWS Compute Optimizer (đề xuất rightsizing cho EC2, EBS, Lambda, RDS dựa trên metrics thực tế). Mục tiêu là giúp doanh nghiệp không chỉ đạt được khoản tiết kiệm ban đầu mà còn duy trì tối ưu chi phí liên tục thông qua phân tích thường xuyên và triển khai các khuyến nghị điều chỉnh kích thước phù hợp.
###  [Blog 2 - Khám phá AWS Console: Chẩn đoán lỗi với Amazon Q Developer ](3.2-Blog2/)
Bài blog giới thiệu tính năng "Diagnose with Amazon Q" của Amazon Q Developer - một trợ lý AI hỗ trợ chẩn đoán và khắc phục lỗi trực tiếp trong AWS Management Console. Khi gặp lỗi, người dùng chỉ cần nhấn nút "Diagnose with Amazon Q", hệ thống sẽ sử dụng LLM (Large Language Models) để phân tích nguyên nhân gốc rễ dựa trên thông tin ngữ cảnh (error message, URL, IAM role). Sau đó, tính năng "Help me resolve" sẽ cung cấp hướng dẫn khắc phục chi tiết từng bước. Amazon Q hoạt động bằng cách: thu thập thông tin ngữ cảnh từ console, chủ động truy vấn thêm dữ liệu qua AWS Cloud Control API (trong phạm vi quyền IAM của người dùng), và tổng hợp thành giải pháp cụ thể. Mục tiêu là giảm MTTR (Mean Time To Repair), tiết kiệm thời gian troubleshooting, và giúp developers/operators xử lý sự cố hiệu quả hơn mà không cần chuyển qua lại nhiều console khác nhau.
###  [Blog 3 - Bắt đầu với Apache OFBiz và Amazon Aurora DSQL](3.3-Blog3/)
Bài blog trình bày case study thực tế về việc migrate ứng dụng Apache OFBiz (ERP/CRM mã nguồn mở với 837 bảng, 4161 indexes) từ PostgreSQL sang Amazon Aurora DSQL. Những thay đổi chính bao gồm: (1) Triển khai xác thực IAM-based thay vì password authentication, với token có thời hạn và quản lý connection pooling; (2) Điều chỉnh schema - thay đổi kiểu dữ liệu NUMERIC sang BIGINT, giảm VARCHAR size trong keys do giới hạn hiện tại của Aurora DSQL; (3) Xử lý transaction limits - chia batch data loading thành các lô 1000 rows mỗi lần commit; (4) Xử lý OCC (Optimistic Concurrency Control) - thêm retry logic cho transactions có thể bị abort do conflicts, vì Aurora DSQL chỉ hỗ trợ snapshot isolation và không lock resources trong quá trình build transaction. Kết quả chứng minh Aurora DSQL có thể hỗ trợ ứng dụng phức tạp với nỗ lực migration hợp lý, tập trung vào 4 khía cạnh: sử dụng đúng data types, IAM authentication, transaction size limits, và idempotent retry-able transactions.
