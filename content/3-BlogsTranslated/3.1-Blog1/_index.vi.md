---
title: "Blog 1"
date: "2025-09-08"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

**Migration & Modernization**  
# Tối ưu hóa chi phí điện toán đám mây: Cẩm nang Rehost Migration (Phần 4 – Di trú: Hiện thực hóa kế hoạch tiết kiệm chi phí) 
**Tác giả:** Dan Krueger, James Gaines, và Rohit Vaitheeswaran – 31/03/2025  
**Chuyên mục:** AWS Cloud Financial Management, AWS Compute Optimizer, AWS Cost Explorer, Best Practices, Billing & Account Management, Cloud Cost Optimization, Customer Enablement, Enterprise Strategy, Thought Leadership  

---  

Bài viết này là phần thứ tư trong chuỗi bốn phần hướng dẫn từng bước cách tối ưu chi phí trong suốt quá trình Rehost migration trên AWS, cụ thể:  
- Khám phá các thành phần chi phí và môi trường on-premises.  
- Xây dựng business case chính xác trong giai đoạn assess.  
- Hiểu và kiểm soát chi tiêu trên cloud trong giai đoạn mobilize.  
- Tối ưu chi phí để đạt được các khoản tiết kiệm tài chính đã hoạch định trong giai đoạn migrate.  

---  

**Hình 1:** Tổng quan hoạt động chi phí của quá trình Rehost Migration theo từng giai đoạn  

---  

## 1. Tối ưu chi phí trên AWS:  
Chúng tôi sử dụng các dịch vụ AWS và công cụ cụ thể trong suốt quá trình Rehost migration để tối ưu hóa chi phí.  
Các dịch vụ AWS này cung cấp đề xuất giúp tối ưu chi phí Amazon EC2 instances, chi phí lưu trữ (storage costs), và chi phí vận hành ứng dụng tổng thể (overall application run cost). Khoản tiết kiệm chi phí có thể tăng thêm vì quá trình migration và tối ưu hóa ban đầu được thực hiện dựa trên application performance metrics từ on-premises data center (bao gồm CPU, memory, disk I/O, và network). Việc tối ưu chi phí sau khi di trú (post-migration cost optimization) yêu cầu phân tích các số liệu sử dụng, vốn khác nhau tùy thuộc vào loại và kích cỡ của instance.  

---  

### 1.1. Trang chủ AWS Billing and Cost Management  
AWS Billing and Cost Management cung cấp các dịch vụ quan trọng để tối ưu chi phí trong suốt quá trình Rehost migration.  
Nếu bạn đang sử dụng tổ chức AWS (AWS Organizations) với quản lí tài khoản (management account) để quản trị tập trung môi trường AWS, hãy cấu hình tài khoản thành viên (member accounts) truy cập các dịch vụ này theo nhu cầu và chính sách truy cập của tổ chức. Các dịch vụ này được áp dụng cho nhiều nhóm làm việc khác, không chỉ riêng các cost management stakeholders được chỉ định. AWS Billing and Cost Management console cung cấp thông tin tóm tắt, cho phép account owner có thể đi sâu vào từng danh mục cụ thể (specific category).  

Landing page bao gồm các default widgets, cung cấp cái nhìn nhanh (quick view) về dự báo chi phí hiện tại (current forecast) so với các chi phí trong vài tháng gần đây.  
Info tab hiển thị bản tóm tắt chi tiết (detailed summary) và giải thích nội dung mà mỗi widget cung cấp cho account owner.  

Các tiện ích mặc định trong AWS Billing and Cost Management console bao gồm:  
- **Tóm tắt chi phí (Cost Summary)** – Hiển thị xu hướng chi tiêu hiện tại so với mức chi của tháng trước. Các chi phí hiển thị tại đây không bao gồm khoản tín dụng và chiết khấu nào.  
- **Giám sát chi phí (Cost Monitor)** – Hiển thị chi phí, ngân sách và các bất thường về chi phí được AWS phát hiện. Trạng thái bất thường về chi phí trong giám sát chi phí (Cost Monitor) được xác định dựa trên cấu hình trong giám sát chi phí (Cost Monitor), thuộc tab phát hiện chi phí bất thường (Cost Anomaly Detection).  
- **Phân tích chi phí (Cost Breakdown)** – Bảng phân tích chi phí trong 6 tháng gần nhất để giúp hiểu rõ xu hướng và yếu tố chi phí. Nhóm chi phí theo các chỉ số như dịch vụ AWS, tài khoản thành viên, vùng, thẻ phân bố chi phí, và danh mục chi phí.  
- **Hành động đề xuất (Recommended Actions)** – Cung cấp hướng dẫn cho chủ tài khoản AWS nhằm tuân thủ các thực tiễn quản lí tài chính đám mây (AWS cloud financial management best practices) và tối ưu chi phí dựa trên các đề xuất được đưa ra.  
- **Cơ hội tiết kiệm (Savings Opportunities)** – Đưa ra các đề xuất từ trung tâm tối ưu hóa chi phí (Cost Optimization Hub) trong nhiều danh mục khác nhau, chẳng hạn như điều chỉnh kích thước, các loại phiên bản khác nhau, và khuyến nghị xóa bỏ các tài nguyên không được sử dụng trong tài khoản.  

---  

### 1.2. AWS Cost Explorer  
Tiện ích phân tích chi phí (Cost Breakdown widget) có thể mở rộng sang AWS Cost Explorer, cho phép chủ tài khoản và nhóm quản lý chi phí trực quan hóa, phân tích và quản lý chi tiêu trên AWS.  
AWS Cost Explorer cung cấp các góc nhìn tổng quát và chi tiết về xu hướng chi tiêu. Lọc và dự báo để xác định các yếu tố chi phí và sự bất thường. Người dùng có thể tùy chỉnh báo cáo theo khoảng thời gian (giờ, ngày, tháng) và nhóm theo dịch vụ. Người dùng có thể lưu báo cáo trong thư viện để sử dụng lại trong tương lai.  

---  

### 1.3. Trung tâm tối ưu hóa chi phí AWS (AWS Cost Optimization Hub)  
Trung tâm tối ưu chi phí AWS (AWS Cost Optimization Hub) cung cấp cái nhìn tổng quan về các cơ hội tối ưu hóa chi phí cho các EC2 instances đã được migrate.  

**Hình 2:** Trung tâm Tối ưu Chi phí và Các Khuyến nghị  

Sau khi quá trình Rehost migration hoàn tất, hệ thống đưa ra khuyến nghị chuyển sang sử dụng Graviton instances để đạt được mức tiết kiệm chi phí bổ sung.  
Ngoài ra, dựa trên các chỉ số sử dụng CPU và bộ nhớ, trung tâm tối ưu chi phí (Cost Optimization Hub) cũng sẽ đề xuất điều chỉnh kích thước phù hợp cho các instance, cũng như xóa bỏ những tài nguyên đang không hoạt động.  

Khai thác trung tâm tối ưu chi phí (Cost Optimization Hub) giúp đội ngũ quản lý chi phí xác định, ưu tiên và triển khai các biện pháp tiết kiệm hiệu quả hơn, nâng cao quản lí tài chính đám mây, tối ưu hóa tài nguyên và mức tiết kiệm chi phí bổ sung ngoài phần tiết kiệm đã đạt được trong quá trình migration lên AWS.  

---  

### 1.4. Trình tối ưu hóa tính toán AWS (AWS Compute Optimizer)  
AWS Compute Optimizer giúp giảm chi phí EC2 instances thông qua các đề xuất kích thước phù hợp. Công cụ này cung cấp tổng quan tiết kiệm, nâng cao hiệu năng và đề xuất tối ưu hóa theo từng vùng cho các tài nguyên sau:  
- Amazon EC2  
- Amazon EC2 Auto Scaling Groups  
- Amazon EBS  
- AWS Lambda functions  
- Amazon ECS on AWS Fargate  
- Commercial software licenses  
- Amazon RDS DB instances and storage  

Khi bật Compute Optimizer, AWS sẽ đánh giá tài nguyên bằng cách xem xét thông số kĩ thuật và mô hình sử dụng được ghi lại bởi Amazon CloudWatch trong 14 ngày gần nhất, bao gồm các chỉ số như mức sử dụng CPU, truyền mạng, hoạt động đĩa, mức độ sử dụng instance hiện tại.  

---  

## 2. Kết luận  
Thiết lập quyền sở hữu chuyên trách và triển khai quy trình giám sát chi phí có hệ thống bằng cách sử dụng AWS Cost Explorer, AWS Budgets, và AWS Cost and Usage Reports.  
Các công cụ này, khi được kết hợp với AWS Cost Optimization Hub và AWS Compute Optimizer, cung cấp các khuyến nghị hành động cho EC2 instances, bộ lưu trữ, và tối ưu hóa ứng dụng. Việc phân tích thường xuyên các mô hình sử dụng tài nguyên và triển khai các khuyến nghị về điều chỉnh kích thước giúp duy trì hiệu quả chi phí lâu dài, vượt ra ngoài khoản tiết kiệm ban đầu trong quá trình migration, tạo nên nền tảng cho việc tối ưu chi phí liên tục.  

Trong bài viết này, chúng ta đã kết thúc hành trình tối ưu chi phí Rehost migration trong giai đoạn migrate bằng cách:  
- Xem xét các dịch vụ có sẵn trong 1.1 AWS Billing and Cost Management Home;  
- Sử dụng 1.2 AWS Cost Explorer để trực quan hóa và phân tích chi phí;  
- Và xem xét 1.3 AWS Cost Optimization Hub cùng 1.4 AWS Compute Optimizer để nhận các khuyến nghị chi tiết về tối ưu chi phí và cơ hội điều chỉnh quy mô.  

---  

## Chuỗi Blog  
Liên kết trực tiếp đến từng bài viết trong chuỗi như sau:  
- Phần 1: Assess – Explore Cost Components  
- Phần 2: Assess – Build a Business Case  
- Phần 3: Mobilize – Understand and Control Cloud Spend  
- Phần 4: Migrate – Realizing Planned Savings  

---  

## Tài nguyên bổ sung  
Liên hệ với chuyên gia migration của AWS để trao đổi về cách chúng tôi có thể hỗ trợ tổ chức của bạn.  
Đã sẵn sàng migrate và tối ưu hóa chi phí? Dưới đây là một số tài nguyên bổ sung:  
- Xem qua AWS Cloud Financial Management Guide để điều chỉnh các quy trình tài chính của bạn, giúp chúng sẵn sàng cho môi trường cloud.  
- Khám phá những nội dung mới nhất trong AWS Cloud Financial Management.  
- Tìm hiểu thêm về cách thực hiện migration và hiện đại hóa với AWS.  
- Hãy khám phá Amazon Q Developer — một trợ lý được hỗ trợ bởi AI (AI-powered assistant), giúp bạn chuyển đổi các workload .NET, mainframe, VMware, và Java, vượt ra ngoài mô hình Rehost truyền thống.  

---  

## Giới thiệu tác giả  
**Dan Krueger** – Senior Customer Solutions Manager tại Amazon Web Services, có nhiều kinh nghiệm phục vụ khách hàng thuộc Chính phủ Liên bang Hoa Kỳ. Tại AWS, ông đã dẫn dắt nhiều dự án cloud migration quy mô lớn, giúp các cơ quan nâng cao năng lực thực thi nhiệm vụ. Trước đó, khi còn làm Program Executive tại IBM, Dan tập trung vào hiện đại hóa nền tảng dữ liệu và triển khai các giải pháp công nghệ phức tạp cho khách hàng chính phủ.  

**James Gaines** – Senior Solutions Architect trong lĩnh vực Healthcare and Life Sciences tại AWS. Ông có nền tảng làm việc trong các môi trường có quy định nghiêm ngặt như Department of Defense và pharmaceutical industry. James sở hữu toàn bộ AWS Certifications và chuyên về cloud migrations, application modernization, cùng phân tích nâng cao để thúc đẩy đổi mới trong lĩnh vực y tế và khoa học đời sống.  

**Rohit Vaitheeswaran** – Senior Solutions Architect tại AWS, chuyên về Healthcare and Life Sciences. Ông có kinh nghiệm đa ngành, từng dẫn dắt nhiều dự án migration strategy và cloud optimization initiatives quy mô lớn. Trong suốt sự nghiệp, Rohit đã giúp các tổ chức thuộc Financial Services, Fintech, và Healthcare tối ưu hành trình lên cloud của họ trên AWS, đặc biệt tập trung vào migration strategy và cost optimization, giúp khách hàng tối đa hóa khoản đầu tư cloud và tuân thủ các yêu cầu đặc thù của ngành.
