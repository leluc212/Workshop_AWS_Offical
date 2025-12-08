---
title: "Blog 3"
date: "2025-09-08"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---



# Blog Cơ sở dữ liệu AWS  
## Bắt đầu với Apache OFBiz và Amazon Aurora DSQL  
**Tác giả:** James Morle | **Ngày:** 26 tháng 3 năm 2025 | **Chuyên mục:** Advanced (300), Amazon Aurora, Best Practices, DSQL, Technical How-to  

---

Khi thiết kế Amazon Aurora DSQL, chúng tôi đã dành rất nhiều thời gian để tìm ra sự cân bằng phù hợp giữa đổi mới và cải tiến so với tính dễ sử dụng. Chúng tôi không muốn làm thay đổi những gì quen thuộc đối với người dùng PostgreSQL hiện tại trừ khi có lý do thực sự thuyết phục. Hai lĩnh vực thuộc nhóm này là hành vi kiểm soát đồng thời lạc quan (Optimistic Concurrency Control – OCC) và mô hình xác thực (authentication model), nơi mà hành vi của hệ thống có thể khiến các nhà phát triển ứng dụng quen thuộc với PostgreSQL cảm thấy khác lạ.

Trong bài viết này, chúng tôi sẽ trình bày một ví dụ thực tế về cách chuyển đổi một ứng dụng hiện có vốn hoạt động trên cơ sở dữ liệu PostgreSQL sang hoạt động với cơ sở dữ liệu Aurora DSQL. Thêm vào đó việc điều chỉnh để thích ứng với các khía cạnh đã đề cập, chúng tôi cũng sẽ xử lý một số vấn đề không tương thích về kiểu dữ liệu và tìm cách khắc phục một vài giới hạn hiện tại còn tồn tại trong Aurora DSQL.

---

## Ứng dụng

Để làm cho ví dụ này mang tính đại diện nhất có thể, chúng tôi đã chọn một ứng dụng mã nguồn mở có tính đa dạng chức năng cao và độ phức tạp đáng kể, phản ánh đúng các ứng dụng thực tế. Ứng dụng đó là Apache OFBiz, và mã nguồn có thể được tải xuống từ kho lưu trữ GitHub. Phiên bản được sử dụng trong bằng chứng khái niệm này là 18.12.16.

Ứng dụng OFBiz là một gói ERP và CRM đầy đủ tính năng, và là một ví dụ điển hình cho mục tiêu của chúng tôi ở đây:
- Lược đồ phức tạp: 837 bảng, 4161 chỉ mục
- Công cụ thực thể trừu tượng hóa (Java Persistence, Java Transactions, custom ORM)
- Đã hỗ trợ PostgreSQL
- Dữ liệu demo và chức năng nạp dữ liệu tích hợp sẵn
- Phần lớn dựa trên mô hình quan hệ, phù hợp với trọng tâm tương thích ban đầu của Aurora DSQL

Mục đích của bài viết này là kết hợp phần mô tả quá trình thử nghiệm nội bộ với một số ví dụ mã cụ thể về cách tiếp cận xác thực Aurora DSQL trong ứng dụng connection pool-based Java. Mặc dù chúng tôi cung cấp nhiều ngữ cảnh nhất có thể, nhưng không nhằm mục đích cung cấp một tập lệnh và mã đầy đủ để tái tạo toàn bộ quá trình này.

---

## Aurora DSQL

Aurora DSQL là một cơ sở dữ liệu tương thích với PostgreSQL, nhưng có một vài khác biệt về mặt ngữ nghĩa nhằm cải thiện khả năng mở rộng và bảo mật của dịch vụ so với PostgreSQL cộng đồng. Một số khía cạnh trong đó, đặc biệt là mức độ tương thích chức năng rộng với các tính năng PostgreSQL, sẽ phát triển nhanh chóng theo thời gian, nhưng một số ít yếu tố là cốt lõi của hệ thống. Đây là trọng tâm của các thay đổi được thực hiện trong ứng dụng OFBiz và được trình bày trong bài viết này. Những khía cạnh đó bao gồm:

- Mô hình xác thực Aurora DSQL – Cách kết nối và duy trì kết nối với cơ sở dữ liệu.
- Giới hạn kích thước giao dịch – Aurora DSQL không hỗ trợ các giao dịch có kích thước không giới hạn.
- Mức độ cô lập (Isolation level) – Aurora DSQL chỉ hỗ trợ snapshot isolation, tương đương với repeatable read trong PostgreSQL.
- Kiểm soát đồng thời lạc quan (Optimistic concurrency control) – Aurora DSQL không khóa tài nguyên trong giai đoạn xây dựng giao dịch; nó đảm bảo tính nhất quán khi commit. Điều này có nghĩa là một số commit có thể thất bại và phải được thực hiện lại từ đầu.
- Giới hạn kiểu dữ liệu trong khóa – Hiện tại, có một số giới hạn về kiểu dữ liệu (và kích thước) được hỗ trợ trong khóa chính và khóa phụ. Các giới hạn này sẽ giảm dần theo thời gian khi Aurora DSQL bổ sung thêm các kiểu dữ liệu mới. Các biện pháp khắc phục được trình bày trong bài viết này phản ánh các giới hạn tồn tại trong giai đoạn Preview của Aurora DSQL.

---

## Tạo cấu hình nguồn dữ liệu cho Aurora DSQL

Điều đầu tiên chúng tôi cần làm là định nghĩa một cấu hình nguồn dữ liệu (data source) mới cho cơ sở dữ liệu Aurora DSQL. Chúng tôi đã thêm một nguồn dữ liệu mới có tên là `localdsql` trong tệp `framework/entity/config/entityengine.xml` và chọn nó làm nguồn dữ liệu đang hoạt động (active data source). Các tham số của nguồn dữ liệu được sao chép từ nguồn dữ liệu hiện có `localpostgres`, với các thiết lập được gạch dưới thay đổi như sau:

```xml
<group-map group-name="org.apache.ofbiz" datasource-name="localdsql"/>
...
</datasource>
<datasource name="localdsql"
    helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO"
    schema-name="ofbiz"
    field-type-name="postgres"
    check-on-start="true"
    add-missing-on-start="true"
    use-fk-initially-deferred="false"
    alias-view-columns="false"
    join-style="ansi"
    use-binary-type-for-blob="true"
    use-order-by-nulls="true"
    result-fetch-size="50">
    <read-data reader-name="tenant"/>
    <read-data reader-name="seed"/>
    <read-data reader-name="seed-initial"/>
    <group-map group-name="org.apache.ofbiz" datasource-name="localdsql"/>
...
</datasource>
<datasource name="localdsql"
    helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO"
    schema-name="ofbiz"
    field-type-name="postgres"
    check-on-start="true"
    add-missing-on-start="true"
    use-fk-initially-deferred="false"
    alias-view-columns="false"
    join-style="ansi"
    use-binary-type-for-blob="true"
    use-order-by-nulls="true"
    result-fetch-size="50">
    <read-data reader-name="tenant"/>
    <read-data reader-name="seed"/>
    <read-data reader-name="seed-initial"/>
</datasource>
```

Trong ví dụ cấu hình ở phần trước, chúng tôi đã ẩn phần endpoint của jdbc-uri— nếu bạn tự thực hiện trên cụm (cluster) của mình, đừng quên cập nhật giá trị <Aurora DSQL_URL> để phản ánh đúng endpoint của cụm Aurora DSQL mà bạn đang dùng.  
Chúng tôi đặt giá trị jdbc-password thành hằng số đặc biệt IAM_AUTH. OFBiz mặc định giả định rằng chúng ta sẽ sử dụng xác thực bằng mật khẩu thông thường để kết nối cơ sở dữ liệu, nhưng Aurora DSQL yêu cầu cơ chế xác thực dựa trên AWS Identity and Access Management (IAM), nhằm cung cấp một môi trường tích hợp và an toàn hơn. Chúng tôi sẽ thêm hỗ trợ cho cơ chế này trong phần code ở bước tiếp theo.  
Chúng tôi cũng thay đổi mức cô lập (isolation level) thành RepeatableRead (đây là hành vi tương đương trong PostgreSQL với mức snapshot isolation của Aurora DSQL) và tăng kích thước nhóm tối thiểu (kết nối trong Aurora DSQL rẻ để duy trì, nhưng kết nối mã hóa thì tương đối chậm để khởi tạo). Aurora DSQL chỉ hỗ trợ kết nối mã hóa, vì vậy chúng tôi đã thêm tham số này vào chuỗi JDBC URI. Chúng tôi cũng triển khai phiên bản mới nhất của driver PG JDBC (42.7.4 tại thời điểm viết) để đảm bảo đây là phiên bản được chứng nhận tương thích với Aurora DSQL.

### Triển khai logic kết nối cho Aurora DSQL
Có ba yếu tố cần xem xét khi xây dựng logic kết nối cho Aurora DSQL:  
• Mật khẩu thực tế là token xác thực động, được tạo theo yêu cầu khi cơ chế xác thực IAM được xác minh.  
• Token xác thực có thời hạn — chúng chỉ hợp lệ trong khoảng thời gian được chỉ định khi tạo ra. Thời hạn này có thể lên đến một tuần, phù hợp khi dùng cho các công cụ phát triển, nhưng đối với ứng dụng thực tế, chúng tôi khuyến nghị thời hạn ngắn hơn nhiều để giảm nguy cơ truy cập trái phép. Chúng tôi chọn thời hạn 3 giờ trong trường hợp này.  
• Kết nối cũng có thời hạn, hiện tại mỗi giờ. Sự tương tác giữa token xác thực và thời hạn kết nối có thể gây nhầm lẫn với người dùng mới của Aurora DSQL, nên cần nhớ token xác thực chỉ được áp dụng khi tạo một kết nối (token phải vẫn còn hiệu lực tại thời điểm tạo kết nối) và thời hạn kết nối chỉ ảnh hưởng đến các kết nối hiện có (không phụ thuộc vào thời hạn của token sau khi kết nối được thiết lập).

Để hỗ trợ cơ chế này, chúng tôi chỉnh sửa tệp  
`framework/entity/src/main/java/org/apache/ofbiz/entity/config/model/EntityConfig.java` để thêm luồng xác thực riêng cho Aurora DSQL, bổ sung logic sau:

```java
public static String getJdbcPassword(InlineJdbc inlineJdbcElement) throws GenericEntityConfException {
    String jdbcPassword = inlineJdbcElement.getJdbcPassword();
+   if (!jdbcPassword.isEmpty() && ! jdbcPassword.equals("IAM_AUTH")) {
        return jdbcPassword;
    }
+   if (jdbcPassword.equals("IAM_AUTH")) {
+        return DSQLAuth.generateToken(inlineJdbcElement.getJdbcUri().split("/")[2], 
+            Region.US_EAST_1, 
```

Sau đó, chúng tôi chỉnh sửa tệp
framework/entity/src/main/java/org/apache/ofbiz/entity/connection/DBCPConnectionFactory.java để sử dụng ConnectionFactory dành riêng cho Aurora DSQL khi data source là localdsql. Chúng tôi cũng đặt thời gian tồn tại tối đa của kết nối (maximum connection lifetime) trong nhóm ngắn hơn một chút so với thời gian kết nối tối đa được hỗ trợ trong Aurora DSQL.

```java
      // create the connection factory
+        org.apache.commons.dbcp2.ConnectionFactory cf = null;
+        if (cacheKey.equals("localdsql")) {
+           cf = new DSQLConnectionFactory(jdbcDriver, jdbcUri, cfProps);
+        } else {
+           cf = new DriverConnectionFactory(jdbcDriver, jdbcUri, cfProps);
+        }

         // wrap it with a LocalXAConnectionFactory
         XAConnectionFactory xacf = new LocalXAConnectionFactory(txMgr, cf);

         // create the pool object factory
         PoolableConnectionFactory factory = new PoolableManagedConnectionFactory(xacf, null);
+        ((PoolableManagedConnectionFactory)factory).setMaxConnLifetimeMillis((long)55*60*1000);
         factory.setValidationQuery(jdbcElement.getPoolJdbcTestStmt());
```

Cuối cùng, chúng tôi thêm phần logic cốt lõi để quản lý các kết nối, bao gồm việc thêm Aurora DSQL SessionId vào thuộc tính của kết nối nhằm hỗ trợ việc chẩn đoán tốt hơn. Khi cần liên hệ với AWS Support để xử lý các vấn đề liên quan đến Aurora DSQL, việc cung cấp Session ID của kết nối cụ thể gặp sự cố sẽ rất hữu ích.
Chúng tôi đã triển khai các lớp cụ thể (concrete classes) sau đây:

* framework/entity/src/main/java/org/apache/ofbiz/entity/connection/Aurora DSQLAuth.java
* framework/entity/src/main/java/org/apache/ofbiz/entity/connection/Aurora DSQLConnectionFactory.java
* framework/entity/src/main/java/org/apache/ofbiz/entity/connection/Aurora DSQLConnection.java

### Chỉnh sửa lược đồ 
Chúng tôi thực hiện các thay đổi tối thiểu đối với lược đồ cơ sở dữ liệu để tuân thủ các giới hạn hiện đang áp dụng cho Aurora DSQL. Phần lớn lược đồ ban đầu đã tương thích với Aurora DSQL, nhưng chúng tôi có thể thực hiện nhiều điều chỉnh hơn nếu không cần giữ lại dữ liệu demo hiện có (vấn đề này sẽ được nói rõ hơn ở các phần sau).
Các giới hạn cần khắc phục đều liên quan đến kiểu dữ liệu được hỗ trợ trong khóa (key) của Aurora DSQL:
- Kiểu NUMERIC hiện chưa được hỗ trợ trong khóa:
  -	Chúng tôi chỉnh sửa applications/datamodel/entitydef/product-entitymodel.xml, đổi kiểu dữ liệu của minimumOrderQuantity sang floating-point (float) thay vì fixed-point (NUMERIC(18,6)). Nói chung, hai kiểu này không hoàn toàn tương đương về mặt logic, nhưng dữ liệu demo lại dùng kiểu dữ liệu thập phân. Một thay đổi tốt hơn ở đây sẽ là thay đổi dữ liệu demo, nhưng trong trường hợp này, logic vẫn hoạt động đúng (vì chỉ dùng phép so sánh “lớn hơn”).
  - Chúng tôi chỉnh sửa framework/entity/fieldtype/fieldtypepostgres.xml để thay đổi định nghĩa của <field-type-def type="numeric" sql-type="NUMERIC(20,0)" java-type="Long"/>  thành  <field-type-def type="numeric" sql-type="bigint" java-type="Long"/>. Việc dùng bigint ở đây thực tế phù hợp hơn so với NUMERIC(20,0). 
- Kiểu VARCHAR được hỗ trợ trong keys, nhưng kích thước tối đa nhỏ hơn so với VARCHAR thông thường:
  - Chúng tôi chỉnh sửa framework/entity/fieldtype/fieldtypepostgres.xml, để đổi định nghĩa của <field-type-def type="email" sql-type="VARCHAR(320)" java-type="String"/> thành <field-type-def type="email" sql-type="VARCHAR(220)" java-type="String"/>.

Sau khi thực hiện các thay đổi này, lược đồ (schema) đã có thể được tạo thành công trong quá trình tải dữ liệu (xem chi tiết ở phần tiếp theo). Chúng tôi sử dụng tùy chọn -l drop-constraints trong tiện ích OFBiz loader để đảm bảo rằng không có bảng nào được tạo bằng cách sử dụng ràng buộc khóa ngoại (foreign key constraints), những foreign key constraints này hiện tại chưa được hỗ trợ trong Aurora DSQL.

Như đã đề cập trước đó, chúng tôi có thể tối ưu hóa lược đồ sâu hơn. Một thay đổi đáng chú ý mà chúng tôi không cần thực hiện là chuyển kiểu dữ liệu của các trường ID sang UUID, điều vốn phổ biến trong ứng dụng sử dụng ORM. Bởi vì chúng tôi đang sử dụng dữ liệu demo, chúng tôi sẽ cần phải sửa đổi tất cả các khóa (và tham chiếu) trong dữ liệu demo để áp dụng các kiểu UUID.  Thay vào đó, chúng tôi giữ nguyên kiểu khóa dựa trên VARCHAR và tải dữ liệu demo mà không cần thay đổi.

### Tải dữ liệu demo

OFBiz data loader (xem OFBiz Technical Setup Guide) ban đầu thực hiện một số lượng lớn thao tác chèn trước mỗi lần commit, khiến một số giao dịch vượt quá giới hạn kích thước transaction trong Aurora DSQL trong một số trường hợp. Để khắc phục vấn đề này, chúng tôi chỉnh sửa phương thức storeAll() trong file
`framework/entity/src/main/java/org/apache/ofbiz/entity/GenericDelegator.java` để chia quá trình tải dữ liệu hàng loạt thành các lô có thể tải dữ liệu 1.000 dòng mỗi lần.

```java
              numberChanged += this.store(toStore);
                 }
             }
+            // commit every 1000 rows to conform to DSQL transaction limits
+            if (numberChanged%1000 == 0) {
+                if (Debug.verboseOn()) Debug.logVerbose("Committing at 1000 row threshold. beganTransaction=" + beganTransaction, module);
+                TransactionUtil.commit();
+                beganTransaction = TransactionUtil.begin();
+            }
         }
         TransactionUtil.commit(beganTransaction);
         return numberChanged;
```
**Lưu ý:** Giới hạn kích thước transaction trong Aurora DSQL thực tế lớn hơn 1.000 dòng, nhưng chúng tôi chọn giá trị nhỏ hơn này để dễ dàng quan sát hành vi batching (xử lý theo lô) sau khi thay đổi logic.
Chúng tôi cũng sửa một lỗi khác khá thú vị trong quá trình dò tìm. Một phần của logic tải dữ liệu gọi đến hàm findKey() trong file
`framework/entity/src/main/java/org/apache/ofbiz/entity/util/EntityCrypto.java` để kiểm tra sự tồn tại của khóa mã hóa (crypto key) trong cơ sở dữ liệu. Nếu khóa này không tồn tại ở vị trí mong đợi, quá trình tải dữ liệu sẽ dừng lại và báo lỗi. Khi thử nghiệm với Aurora DSQL, điều kiện bất biến này luôn bị kích hoạt. Sau khi tìm hiểu kỹ, chúng tôi phát hiện có một lỗi logic tổng quát, dẫn đến sự khác biệt trong hành vi giữa hai mức cô lập giao dịch read committed và repeatable read. Phương thức findKey() được gọi mà không bắt đầu một transaction mới, nên nó chỉ nhìn thấy dữ liệu tại thời điểm snapshot của transaction hiện tại. Vì snapshot đó xảy ra trước khi lệnh INSERT tương ứng được commit, nên findKey() không thể thấy dòng dữ liệu vừa được chèn. Trong mức cô lập read committed, lời gọi findKey() hoạt động bình thường vì session có thể thấy dữ liệu đã commit kể từ khi transaction bắt đầu.
Để khắc phục, chúng tôi bắt đầu một transaction mới mỗi khi hàm findKey() được gọi:

```java
GenericValue keyValue = null;
     try {
+        try {
+            keyValue = TransactionUtil.doNewTransaction(() -> {
+                return EntityQuery.use(delegator).from("EntityKeyStore").where("keyName", hashedKeyName).queryOne();
+            }, "checking encrypted key", 0, true);
+        } catch (GenericEntityException e) {
+            throw new EntityCryptoException(e);
+        }
     } catch (GenericEntityException e) {
         throw new EntityCryptoException(e);
     }
```

Chúng tôi đã tải dữ liệu khởi tạo (seed) và dữ liệu minh họa (demo) theo hướng dẫn trong tài liệu thiết lập kỹ thuật của OFBiz (OFBiz Technical Setup Guide).
Xử lý OCC aborts trong transaction command pattern
Ứng dụng OFBiz gửi các giao dịch (transactions) đến cơ sở dữ liệu bằng mẫu lệnh (command pattern) — sử dụng giao diện Callable — giúp việc xử lý các giao dịch của Aurora DSQL trở nên đơn giản hơn. Các ứng dụng khác (ngoài OFBiz) nếu cũng đóng gói toàn bộ giao dịch theo cách này thì việc quản lý cũng tương tự, vì ranh giới của từng giao dịch đều được xác định rõ ràng. Tất cả những gì cần thực hiện ở đây, ít nhất là cho mục đích minh họa, là thêm một vòng lặp retry quanh phần thực thi giao dịch. Vòng lặp này được đặt trong tệp:
`framework/entity/src/main/java/org/apache/ofbiz/entity/transaction/TransactionUtil.java` trong phương thức InTransaction.call().

```java
protected InTransaction(Callable<V> callable, String ifErrorMessage, int timeout, boolean printException) {
        this.callable = callable;
        this.ifErrorMessage = ifErrorMessage;
        this.timeout = timeout;
        this.printException = printException;
    }

    public V call() throws GenericEntityException {
+       boolean transactionComplete = false;
+       while (!transactionComplete) {
            boolean tx = TransactionUtil.begin(timeout);
            Throwable transactionAbortCause = null;
            try {
                try {
                    return callable.call();
                } catch (Throwable t) {
                    while (t.getCause() != null) {
                        t = t.getCause();
                    }
                    throw t;
                }
            } catch (Error | RuntimeException e) {
                transactionAbortCause = e;
                throw e;
            } catch (Throwable t) {
                transactionAbortCause = t;
+               if (t instanceof SQLException && (((SQLException) t).getSQLState().startsWith("OC")
+                       || ((SQLException) t).getSQLState().equals("40001"))) {
+                   if (Debug.verboseOn()) {
+                       Debug.logVerbose("Transaction abort, reason: " + t, module);
+                       Debug.logVerbose("Retrying on OCC conflict", module);
+                   }
+                   // Rollback to recover transaction state and loop again
+                   TransactionUtil.rollback(tx, ifErrorMessage, transactionAbortCause);
+                   continue;
+               } else {
                    throw new GenericEntityException(t);
+               }
            } finally {
                if (transactionAbortCause == null) {
                    TransactionUtil.commit(tx);
+                   transactionComplete = true;
                } else {
                    if (printException) {
                        Debug.logError(transactionAbortCause, module);
                    }
                    TransactionUtil.rollback(tx, ifErrorMessage, transactionAbortCause);
+                   transactionComplete = true;
+               }
            }
        }
+       throw new GenericEntityException("Unexpected state in transaction handler");
    }
}
```
**Lưu ý:** Đối với các ứng dụng sản xuất, chúng tôi cũng khuyên bạn nên bổ sung thêm logic cho exponential backoff và jitter, điều này giúp hệ thống hoạt động ổn định hơn trong điều kiện xung đột cao.
Các ứng dụng hiện có không mô hình hóa giao dịch rõ ràng theo cách này có thể cần được phân tích sâu hơn để xác định cách xử lý OCC aborts một cách chính xác. Chúng tôi dự định sẽ viết một bài đăng trong tương lai về chủ đề này, sau khi có thêm dữ liệu từ trải nghiệm thực tế của khách hàng để xác định các mô hình phổ biến nhất.

### Kết luận

Trong bài viết này, chúng tôi đã hướng dẫn bạn qua các thay đổi khác nhau được thực hiện đối với một ứng dụng phức tạp hiện có. Chúng tôi đã thực hiện một số sửa đổi nhỏ đối với schema để sử dụng các kiểu dữ liệu được hỗ trợ, tạo logic mới để tương tác đúng với cơ chế xác thực của Aurora DSQL, triển khai việc gom commit khi tải dữ liệu, và làm cho bộ xử lý giao dịch hoạt động đúng với ngữ nghĩa OCC.
Mặc dù bản proof of concept này tập trung vào tính khả thi kỹ thuật hơn là mức độ sẵn sàng cho môi trường production, nó đã chứng minh rằng Aurora DSQL có thể hỗ trợ các ứng dụng cơ sở dữ liệu phức tạp như OFBiz với nỗ lực di chuyển hợp lý. Thử nghiệm này xác nhận rằng ngay cả những ứng dụng có schema phức tạp vẫn có thể được điều chỉnh để hoạt động với Aurora DSQL, bao gồm các tính năng bảo mật và khả năng mở rộng được cải thiện, thông qua các điều chỉnh có mục tiêu đối với xác thực, xử lý giao dịch và thiết kế schema.
Những khía cạnh này cũng là các trọng tâm chính khi bạn lên kế hoạch migrations của riêng mình sang Aurora DSQL:
1.	Sử dụng các kiểu dữ liệu được hỗ trợ trong schema, bao gồm cả keys
2.	Triển khai logic xác thực dựa trên IAM cần thiết cho Aurora DSQL
3.	Đảm bảo kích thước giao dịch nằm trong giới hạn của Aurora DSQL
4.	Triển khai các giao dịch idempotent và có thể retry được
Nếu bạn đang làm việc với các ứng dụng hiện có sử dụng Aurora DSQL, chúng tôi mời bạn chia sẻ kinh nghiệm và quan sát của mình trong phần bình luận!

**Về tác giả**

James Morle là Principal Engineer tại Amazon Web Services và đã tham gia sâu vào quá trình thiết kế và triển khai Aurora DSQL ngay từ những ngày đầu tiên. Ông đã có kinh nghiệm thiết kế và xây dựng các cơ sở dữ liệu quy mô rất lớn từ đầu những năm 1990.

