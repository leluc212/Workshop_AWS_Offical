---
title: "Blog 2"
date: "2025-09-08"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---



**AWS DevOps & Developer Productivity Blog**  
# Khám phá AWS Console: Chẩn đoán lỗi với Amazon Q Developer 
**Tác giả:** Marco Frattallone, Matthias Seeger, và Surabhi Tandon — Ngày 10 tháng 1 năm 2025  
**Danh mục:** Amazon Q, Amazon Q Developer, AWS Management Console, DevOps, Intermediate (200)  

---  

## Giới thiệu  
Các Developers, IT Operators, và trong một số trường hợp là Site Reliability Engineers (SREs), chịu trách nhiệm triển khai và vận hành hạ tầng cùng ứng dụng, đồng thời xử lý và khắc phục sự cố một cách hiệu quả và kịp thời. Quản lý sự cố hiệu quả đòi hỏi chẩn đoán nhanh, phân tích nguyên nhân gốc rễ, và thực hiện hành động khắc phục. Việc xác định nguyên nhân gốc có thể rất phức tạp trong các hệ thống hiện đại, vốn bao gồm nhiều tài nguyên được triển khai phân tán trên nhiều môi trường khác nhau. Amazon Q Developer, một trợ lý được hỗ trợ bởi AI tạo sinh, có thể đơn giản hóa quy trình này bằng cách chẩn đoán các lỗi xuất hiện trong AWS Management Console.  

Amazon Q Developer giúp bạn tiết kiệm thời gian quan trọng khi xử lý sự cố production, bằng cách hỗ trợ chẩn đoán lỗi liên quan đến môi trường AWS của bạn. Những lỗi này có thể bắt nguồn từ các sai cấu hình giữa nhiều tài nguyên khác nhau, thường yêu cầu bạn chuyển qua lại giữa nhiều console của các dịch vụ AWS để tìm nguyên nhân gốc. Amazon Q Developer sử dụng generative AI để tự động hóa quá trình chẩn đoán lỗi phát sinh trong giao diện AWS Console, giúp giảm Mean Time To Repair (MTTR) và hạn chế tối đa tác động của sự cố lên hoạt động kinh doanh.  

Bài viết này sẽ giải thích cách Amazon Q Developer hỗ trợ chẩn đoán lỗi trong AWS Console khi làm việc với các dịch vụ AWS được hỗ trợ. Đồng thời mô tả cách tính năng này hoạt động nhằm hướng dẫn bạn trong quá trình khắc phục sự cố. Chúng tôi cũng sẽ phân tích phía sau hậu trường để cho thấy các quy trình vận hành giúp kích hoạt và hỗ trợ tính năng này.  

---  

## Chẩn đoán với Amazon Q  
Tính năng Diagnose with Amazon Q giúp chẩn đoán hầu hết các lỗi phổ biến xảy ra trong AWS Management Console đối với các dịch vụ AWS hiện đang được hỗ trợ bởi chức năng này. Tính năng này được kích hoạt khi người dùng có quyền phù hợp nhấn vào nút Diagnose with Amazon Q nằm cạnh thông báo lỗi. Amazon Q sẽ cung cấp giải thích bằng ngôn ngữ tự nhiên, trong đó phân tích nguyên nhân gốc rễ của lỗi. Sau đó, khi người dùng nhấn Help me resolve, Amazon Q sẽ hiển thị danh sách các bước hướng dẫn được sắp xếp theo thứ tự, giúp bạn khắc phục tình trạng lỗi. Sau khi hoàn thành, bạn có thể gửi phản hồi về việc giải pháp mà Amazon Q đưa ra có hữu ích hay không.  

Ví dụ minh họa cho thấy cách Amazon Q Developer có thể giúp chẩn đoán lỗi khi khởi tạo instance trong Amazon EC2 và cung cấp hướng dẫn từng bước để xử lý lỗi đó.  

### Chẩn đoán với Amazon Q – Quyền IAM liên quan đến lỗi khởi tạo EC2 instance  

---  

## Phía sau hậu trường: Cách Amazon Q tạo chẩn đoán  
Để minh họa các khái niệm, chúng tôi sẽ trình bày phần giải thích trong bối cảnh hai ví dụ thực tế.  

**Ví dụ 1:** Giả sử bạn cố gắng xóa một Amazon S3 bucket mà bucket đó không rỗng. Điều này dẫn đến thông báo lỗi:  
_This bucket is not empty. Buckets must be empty before they can be deleted. To delete all objects in the bucket, use the empty bucket configuration._  

**Ví dụ 2:** Giả sử bạn cố gắng liệt kê các objects trong một S3 bucket cụ thể, nhưng thiếu AWS Identity and Access Management (IAM) permissions để thực hiện. Điều này dẫn đến thông báo lỗi:  
_Insufficient permissions to list objects. After you or your AWS administrator has updated your permissions to allow the s3:ListBucket action, refresh the page. Learn more about Identity and access management in Amazon S3._  

Khi bạn nhấp vào nút Diagnose with Amazon Q bên cạnh thông báo lỗi trong AWS Management Console, Amazon Q sẽ tạo ra một phân tích (Analysis) diễn giải nguyên nhân gốc rễ của lỗi bằng ngôn ngữ tự nhiên. Bước này được hỗ trợ bởi mô hình ngôn ngữ lớn (LLMs). Thông tin ngữ cảnh được cung cấp cho LLM bao gồm thông báo lỗi hiển thị trong console, URL của hành động kích hoạt, và IAM role của người dùng đã đăng nhập vào AWS Console. Dịch vụ luôn hoạt động trong phạm vi quyền được cấp cho role của bạn khi bạn thao tác trong AWS Console, và các đặc quyền không bao giờ bị leo thang vượt quá những gì đã được gán cho bạn.  

Khi bạn nhấp vào nút Help me resolve sau khi đã xem xét phân tích, Amazon Q truy xuất thêm thông tin về trạng thái của các resources trong AWS Account nơi lỗi xảy ra. Ở giai đoạn này, hệ thống chủ động quyết định thông tin nào còn thiếu và phát sinh các yêu cầu thẩm vấn tới các dịch vụ nội bộ để đáp ứng nhu cầu thông tin. Việc thẩm vấn không cần thiết cho các lỗi đơn giản, như ví dụ 1 ở trên, nhưng trở nên thiết yếu để giải quyết các lỗi phức tạp hơn, khi thông tin từ ngữ cảnh không đủ.  

Dựa trên ngữ cảnh, phân tích lỗi, quyền của người dùng, và kết quả thẩm vấn tài khoản, Amazon Q tạo ra các hướng dẫn Resolution theo từng bước. Bước này được hỗ trợ bởi LLMs.  

Sau khi triển khai và xác thực các bước do Amazon Q cung cấp để giải quyết lỗi trong console, bạn có tùy chọn cung cấp phản hồi về trải nghiệm của mình.  

---  

## Sơ đồ mô tả tương tác giữa Người dùng, AWS Console và Amazon Q Developer  

---  

## Thông tin ngữ cảnh  
Thông tin ngữ cảnh giúp các mô hình ngôn ngữ lớn (LLMs) tạo ra phản hồi chính xác và phù hợp hơn. Dữ liệu ngữ cảnh được tự động truyền vào Amazon Q từ giao diện AWS Console. Vì là cơ sở cho toàn bộ quá trình phân tích và ra quyết định, nên thông tin ngữ cảnh cần được cung cấp càng đầy đủ càng tốt. Ở mức tối thiểu, Amazon Q sẽ thu thập thông báo lỗi (error message), URL của hành động gây ra lỗi, và IAM role mà người dùng đăng nhập đang sử dụng.  

Hệ thống sẽ tự động trích xuất các định danh (identifiers) có liên quan từ thông tin ngữ cảnh này.  

Trong ví dụ 1 đang chạy của chúng tôi, URL có thể là:  
https://s3.console.aws.amazon.com/s3/bucket/my-bucket-123456/delete?region=us-west-2, từ đó Amazon Q trích xuất aws_region = "us-west-2" và s3_bucket_name = "my-bucket-123456".  

Ngoài phần ngữ cảnh tối thiểu này, Amazon Q có thể thu thập thông tin bổ sung từ bảng điều khiển, liên quan đến những gì người dùng nhìn thấy trên màn hình khi lỗi xảy ra, chẳng hạn như nội dung trong các ô văn bản hoặc thành phần giao diện (widgets) trong giao diện hiện tại. Amazon Q cũng có thể sử dụng ngữ cảnh cụ thể do dịch vụ nền tảng cung cấp.  

Trong trường hợp ví dụ 2 ở trên, tên bucket được trích xuất từ URL, hành động s3:ListBucket từ thông báo lỗi, và Amazon Q có thể lấy thêm thông tin từ IAM về các chính sách liên quan cũng như các câu lệnh cho phép hoặc từ chối.  

---  

## Thẩm vấn tài khoản người dùng đã đăng nhập  
Tính năng Diagnose with Amazon Q không chỉ thụ động nhận thông tin ngữ cảnh mà còn có khả năng chủ động truy vấn thêm thông tin. Amazon Q thực hiện các truy vấn chỉ đọc không thay đổi dữ liệu nhằm thu thập thêm ngữ cảnh về tài nguyên trong tài khoản AWS, trạng thái của các tài nguyên đó, và mối quan hệ giữa chúng với tài nguyên đang gặp lỗi. Ngữ cảnh về mối quan hệ này giúp mô hình LLM phân tích nguyên nhân gốc chính xác hơn khi chẩn đoán lỗi.  

Amazon Q thẩm vấn tài khoản người dùng đã đăng nhập bằng AWS Cloud Control API (CCAPI) để xác định các tài nguyên đang được triển khai trong tài khoản. Trong quá trình thiết lập ban đầu (onboarding) Amazon Q, chính sách được quản lý AmazonQFullAccess sẽ được gán cho vai trò IAM mà người dùng đang sử dụng. Chính sách này bao gồm cloudformation:ListResources và cloudformation:GetResource quyền IAM của CCAPI, cho phép truy cập vào các endpoint đọc (read) và liệt kê (list) của CCAPI. Nếu bạn không muốn gán chính sách được quản lý AmazonQFullAccess bạn có thể thêm cloudformation:ListResources và cloudformation:GetResource trực tiếp vào IAM Role.  

Trong ví dụ đơn giản ví dụ 1, nơi lỗi xảy ra do bucket S3 không trống, thông báo lỗi và URL trong bảng điều khiển đã chứa đầy đủ thông tin cần thiết để xử lý, nên không cần truy vấn thêm vào tài khoản AWS.  

Ngược lại, với lỗi quyền IAM trong ví dụ 2, việc hiểu các quyền của IAM role gắn với tài nguyên gặp lỗi là rất hữu ích. Amazon Q có thể lấy thông tin về chính sách ở cấp danh tính (identity-level) cho role và chính sách ở cấp tài nguyên cho tài nguyên bị ảnh hưởng. Dựa trên đó, Amazon Q sẽ phân tích nguyên nhân của lỗi bằng cách sử dụng các dịch vụ IAM nội bộ.  

Cụ thể, URL trong ví dụ 2 có thể là:  
https://s3.console.aws.amazon.com/s3/buckets/my-bucket-123456?region=us-west-2&bucketType=general&tab=objects  
từ đó Amazon Q trích xuất được khu vực và tên S3 bucket. Nó cũng có thể trích xuất hành động s3:ListBucket trực tiếp từ thông báo lỗi. Dựa trên các thông tin này, Amazon Q có thể lấy các chính sách bucket cho my-bucket-123456 các chính sách cấp danh tính của vai trò đó, sau đó quét để kiểm tra xem có hay không của hành động s3:ListBucket hoặc gọi các dịch vụ nội bộ của IAM để thu thập thêm thông tin về nguyên nhân dẫn đến việc bị từ chối truy cập.  

Amazon Q chỉ hoạt động trong phạm vi quyền hạn được cấp bởi vai trò IAM của người dùng đã đăng nhập, đảm bảo rằng các đặc quyền không bao giờ bị mở rộng vượt quá những gì vai trò đó được gán. Amazon Q gọi CCAPI thay mặt cho người dùng đã đăng nhập, sử dụng đúng các quyền mà vai trò IAM của người dùng cho phép. CCAPI sẽ kế thừa quyền của người dùng và có cùng mức truy cập để truy vấn tài nguyên trong tài khoản của người dùng.  

Trong ví dụ 2, nếu người dùng đăng nhập không có quyền truy cập vào chính sách của bucket my-bucket-123456, thì Amazon Q cũng sẽ không thể truy cập được. Tất cả các lệnh gọi API đều được ghi lại trong CloudTrail, bao gồm việc Amazon Q gọi CCAPI, và CCAPI tiếp tục gọi đến các dịch vụ đích (ví dụ: S3, IAM) tùy thuộc vào yêu cầu.  

---  

## Tạo hướng dẫn khắc phục theo từng bước  
Ở giai đoạn này, tất cả thông tin thu thập được được Amazon Q tổng hợp để tạo ra các hướng dẫn khắc phục hữu ích và khả thi theo từng bước. Để minh họa, dưới đây là các hướng dẫn mẫu cho các ví dụ đang xét. Khi các mô hình được cập nhật và cải tiến theo thời gian, các phản hồi có thể thay đổi.  

Đối với Ví dụ 1, các hướng dẫn mẫu có thể như sau:  
1. Truy cập S3 console, nhấp "Buckets" và chọn bucket my-bucket-123456.  
2. Nhấp vào tab "Empty".  
3. Nếu bucket chứa số lượng lớn objects, việc tạo một lifecycle rule để xóa tất cả objects trong bucket có thể là cách hiệu quả hơn để làm rỗng bucket.  
4. Nhập "permanently delete" vào ô nhập văn bản và xác nhận rằng tất cả objects sẽ bị xóa.  
5. Thử xóa lại S3 bucket my-bucket-123456.  

Đối với Ví dụ 2, bạn có thể thực hiện:  
1. Vào IAM console. Chỉnh sửa IAM policy đính kèm với role ReadOnly  
2. Cho phép hành động s3:ListBucket cho resource có ARN: arn:aws:s3:::my-bucket-123456  
3. Lưu IAM policy đã được cập nhật.  
4. Làm mới trang S3 console để liệt kê objects trong bucket my-bucket-123456.  

Lưu ý rằng các hướng dẫn chứa thông tin được suy ra từ ngữ cảnh, chẳng hạn như tên bucket my-bucket-123456, thay vì các giá trị giữ chỗ (placeholders). Các hướng dẫn được trả về bởi Diagnose with Amazon Q là đầy đủ và chi tiết, đủ để người dùng có thể thực hiện mà không cần thêm bất kỳ bước bổ sung nào.  

Trên thực tế, mặc dù dịch vụ sử dụng LLM để tổng hợp các hướng dẫn khắc phục, Amazon Q vẫn áp dụng xử lý hậu kỳ (post-processing) để chỉnh sửa các lỗi thường gặp. Ví dụ, trong Ví dụ 2 ở trên, mô hình LLM có thể trả về ARN ở dạng  
arn:aws:s3:<region>::<bucket_name> và hệ thống sẽ tự động chỉnh lại chính xác như đã hiển thị ở trên.  

Các hướng dẫn được trả về cho Ví dụ 2 giả định rằng lý do người dùng không thể liệt kê các object là do thiếu câu lệnh Allow trong các policy gắn với role ReadOnly. Tuy nhiên, các nguyên nhân gốc khác có thể là câu lệnh Deny trong một policy được gắn với S3 bucket hoặc với role ReadOnly. Diagnose with Amazon Q có thể sử dụng quá trình truy vấn tài khoản để xác định nguyên nhân gốc chính xác và đề xuất giải pháp phù hợp.  

Trong ví dụ trên, Amazon Q có thể lấy các policy gắn với role ReadOnly để kiểm tra xem quyền s3:ListBucket có thực sự bị thiếu hay không, hoặc lấy các policy gắn với bucket bucket-123456.  

---  

## Xác thực  
Một trong những mục tiêu của Diagnose with Amazon Q là duy trì tiêu chuẩn chất lượng cao, đảm bảo người dùng luôn nhận được các gợi ý hữu ích và có thể hành động được bất cứ khi nào gặp lỗi. Điều kiện tiên quyết quan trọng là phải có một hệ thống đánh giá mạnh mẽ và linh hoạt. Việc đánh giá các hệ thống dựa trên Generative AI là một thách thức do không gian đầu ra rất lớn (ngôn ngữ tự nhiên) và tính phi xác định trong phản hồi.  

Tóm lại, hệ thống xác thực của chúng tôi được xây dựng dựa trên một bộ dữ liệu lớn về các lỗi, trong đó mỗi bản ghi có một số lượng chú thích nhất định. Mỗi bản ghi chứa ngữ cảnh (thông báo lỗi theo mẫu và URL của Console; nghĩa là bucket-123456 được thay bằng {{s3_bucket_name}}, us-west-2 được thay bằng {{aws_region}}). Các chú thích bao gồm mô tả Infrastructure as Code (CloudFormation) về trạng thái tài khoản có lỗi và hành động kích hoạt lỗi, cũng như các phản hồi đúng được cung cấp bởi các chuyên gia chú thích.  

Những bản ghi này cho phép chúng tôi mô phỏng hành vi của các biến thể hệ thống mà không cần sự tương tác của con người, và thực hiện nhanh hơn thời gian thực nhiều lần (bằng cách song song hóa). Chúng tôi cũng đang phát triển các chỉ số đánh giá tự động để so sánh giữa chú thích chuẩn và phản hồi của hệ thống, dựa trên đó các bài đánh giá ngoại tuyến có thể được thực hiện một cách hoàn toàn tự động.  

Hệ thống xác thực này cho phép chúng tôi nhanh chóng kiểm thử các ý tưởng mới bằng cách so sánh với trạng thái hiện tại, đồng thời ngăn chặn việc thoái lui chất lượng. Mặc dù các chuyên gia con người vẫn cần thiết để tạo chú thích cho các bản ghi lỗi, chúng tôi liên tục đổi mới nhằm tăng tốc và đơn giản hóa các tác vụ này bằng cách phát triển các công cụ chú thích này được thiết kế để tránh việc nhập liệu bằng ngôn ngữ tự nhiên, tích hợp sẵn cơ chế kiểm tra hợp lệ, và yêu cầu chỉnh sửa đầu ra của hệ thống khi cần thiết hơn là cung cấp chú thích thực tế từ đầu.  

---  

## Kết luận  
Tính năng Diagnose with Amazon Q của Amazon Q Developer cho phép bạn xác định nguyên nhân của lỗi trong AWS Console mà không cần phải điều hướng qua nhiều bảng điều khiển dịch vụ khác nhau. Bằng cách cung cấp các hướng dẫn chi tiết, từng bước, được tùy chỉnh theo tài khoản AWS và ngữ cảnh lỗi cụ thể, Amazon Q Developer giúp bạn xử lý và khắc phục sự cố một cách hiệu quả.  

Điều này giúp tổ chức của bạn nâng cao hiệu quả vận hành, giảm thời gian ngừng hoạt động, cải thiện chất lượng dịch vụ, và giải phóng nguồn nhân lực quý giá để họ có thể tập trung vào các hoạt động mang lại giá trị cao hơn.  

Chúng tôi cũng cung cấp thông tin chi tiết về cách AI và machine learning hoạt động phía sau để kích hoạt tính năng này.  

---  

## Về các tác giả  

**Matthias Seeger**  
Matthias Seeger là Principal Applied Scientist tại AWS. Ông quan tâm đến học Bayes và ra quyết định dựa trên mô hình xác suất, lý thuyết và ứng dụng của mô hình Gaussian Process, dự báo xác suất, và gần đây nhất là các mô hình ngôn ngữ lớn (LLM) cùng những thách thức trong việc tạo và gán nhãn dữ liệu liên quan.  

**Marco Frattallone**  
Marco Frattallone là Senior Technical Account Manager tại AWS, tập trung vào hỗ trợ các đối tác. Ông làm việc chặt chẽ với họ để xây dựng, triển khai và tối ưu hóa giải pháp trên AWS, đồng thời đưa ra hướng dẫn và chia sẻ các phương pháp tốt nhất. Marco đam mê công nghệ và luôn giúp các đối tác dẫn đầu trong đổi mới sáng tạo. Ngoài công việc, ông yêu thích đạp xe, chèo thuyền buồm và khám phá các nền văn hóa mới.  

**Surabhi Tandon**  
Surabhi Tandon là Senior Technical Account Manager tại Amazon Web Services (AWS). Cô hỗ trợ khách hàng doanh nghiệp đạt được hiệu quả vận hành cao và giúp đỡ họ trong hành trình cloud trên AWS bằng cách cung cấp hướng dẫn kỹ thuật chiến lược. Surabhi có niềm đam mê với Generative AI, tự động hóa, và DevOps. Ngoài công việc, cô thích leo núi, đọc sách và dành thời gian bên gia đình và bạn bè.

