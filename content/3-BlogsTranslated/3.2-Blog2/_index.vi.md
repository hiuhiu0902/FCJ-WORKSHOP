---
title: "Karrot xây dựng Feature Platform trên AWS, Phần 2: Nhập tính năng (Feature ingestion)"
date: "2025-08-14T10:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Karrot đã xây dựng nền tảng tính năng (feature platform) trên AWS như thế nào, Phần 2: Nhập tính năng (Feature ingestion)

> Tác giả: Hyeonho Kim, Jinhyeong Seo, Minjae Kwon, Hyuk Lee, Jinhyun Park, Jungwoo Song, và Nak-Kwon Choi | Ngày: 14 THÁNG 8, 2025 | Trong: [Advanced (300)](https://aws.amazon.com/blogs/architecture/category/learning-levels/advanced-300/), [Amazon Managed Streaming for Apache Kafka (Amazon MSK)](https://aws.amazon.com/msk/), [AWS Batch](https://aws.amazon.com/batch/), [AWS Fargate](https://aws.amazon.com/fargate/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/) | [Permalink](https://aws.amazon.com/blogs/architecture/how-karrot-built-a-feature-platform-on-aws-part-2-feature-ingestion/) | [Comments](https://aws.amazon.com/blogs/architecture/how-karrot-built-a-feature-platform-on-aws-part-2-feature-ingestion/#comments) |
>
> *Bài viết này đồng tác giả với Hyeonho Kim, Jinhyeong Seo và Minjae Kwon từ Karrot.*

Trong [Phần 1]({{< ref "/3-blogstranslated/3.1-blog1" >}}) của loạt bài này, chúng tôi đã thảo luận về cách [Karrot](https://about.daangn.com/en/) phát triển một feature platform mới, bao gồm ba thành phần chính: feature serving, một stream ingestion pipeline, và một batch ingestion pipeline. Chúng tôi đã thảo luận về các yêu cầu, kiến trúc giải pháp và feature serving bằng cách sử dụng multi-level cache. Trong bài viết này, chúng tôi sẽ chia sẻ về stream và batch ingestion pipelines cũng như cách chúng nhập dữ liệu vào một online store từ nhiều nguồn sự kiện khác nhau.

---

## Tổng quan về giải pháp

Sơ đồ tổng quan về kiến trúc đã được giới thiệu trong [Phần 1]({{< ref "/3-blogstranslated/3.1-blog1" >}}).
<img src="/images/bl2-1.png" alt="" width="50%">

## Stream ingestion (Nhập dữ liệu luồng)

Stream ingestion là quá trình thu thập dữ liệu từ các nguồn sự kiện khác nhau theo thời gian thực, chuyển đổi nó thành các features, và lưu trữ chúng. Nó bao gồm hai thành phần chính:

* **Message broker** – [Amazon Managed Streaming for Apache Kafka](https://aws.amazon.com/msk/) (Amazon MSK) được sử dụng để lưu trữ các sự kiện được xuất bản từ nhiều dịch vụ nền tảng khác nhau và các sự kiện change data capture (CDC) từ [Amazon Aurora](https://aws.amazon.com/rds/aurora/).
* **Consumer** – Đây là các pods nằm trên [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS) có nhiệm vụ xử lý các sự kiện theo feature group specifications được định nghĩa trong feature platform và tải chúng vào database và remote cache.

Các Consumers xử lý không chỉ các sự kiện nguồn mà còn cả các sự kiện được [re-published](https://docs.confluent.io/platform/current/streams/faq.html#streams-faq-republish-output-topic). Khi tải features, chúng được thực hiện bằng cách xem xét các chiến lược khác nhau, chẳng hạn như [write-through](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/write-through.html) và write-around, và được tải chi tiết bằng cách xem xét cardinality, kích thước dữ liệu và các mẫu truy cập.

Hầu hết các features được tạo dựa trên hai loại sự kiện: các sự kiện xảy ra do hành động người dùng theo thời gian thực, và các sự kiện không đồng bộ xảy ra do thay đổi trạng thái trong dữ liệu người dùng và bài viết. Các sự kiện và features này có mối quan hệ M:N, nghĩa là một sự kiện có thể là nguồn của nhiều features, và một feature có thể được tạo dựa trên nhiều sự kiện.

> *Sơ đồ kiến trúc của stream ingestion pipeline.*
<img src="/images/bl2-2.png" alt="" width="50%">

Để xử lý hiệu quả các mối quan hệ M:N, cần có một cấu trúc để nhận các sự kiện và phân phối chúng đến nhiều logic xử lý feature. Hai thành phần cốt lõi đã được thiết kế cho mục đích này:

* **Dispatcher** – Nhận các sự kiện từ nhiều consumer groups và truyền chúng đến logic xử lý feature có liên quan
* **Aggregator** – Xử lý các sự kiện nhận được từ Dispatcher thành các features thực tế

Stream processing pipeline này cho phép tạo và lưu trữ features theo thời gian thực.

### Tối ưu hóa Message broker: Fast at-least-once delivery

Feature platform xử lý tới 25.000 sự kiện mỗi giây, bao gồm cả các sự kiện user behavior log, với tốc độ cao. Tuy nhiên, khi lưu lượng worker tăng đột biến, lỗi xử lý sự kiện hoặc lỗi cơ sở hạ tầng đôi khi gây ra mất sự kiện. Để giải quyết vấn đề này, [automatic commit mode](https://cwiki.apache.org/confluence/display/KAFKA/KIP-41+-+KafkaConsumer+Max+Poll+Records) hiện có đã được thay đổi thành manual commit trong Amazon MSK. Điều này cho phép các sự kiện chỉ được commit khi chúng đã được xử lý chắc chắn, và các sự kiện bị lỗi được gửi đến một retry topic riêng biệt và xử lý hậu kỳ thông qua một dedicated worker.

Tuy nhiên, việc xử lý khối lượng lớn sự kiện một cách đồng bộ với manual commit làm cho tốc độ xử lý chậm hơn khoảng 10 lần và làm tăng độ trễ. Mặc dù tài nguyên consumer group có sẵn, việc đơn thuần tăng số lượng partitions trong Amazon MSK không phải là một giải pháp do các quyền phân partition dành riêng cho từng nhóm. Platform đã thiết kế xử lý song song bên trong các single partitions và triển khai một custom consumer hỗ trợ chức năng retry. Cốt lõi của việc triển khai là đọc số lượng messages bằng fetch size từ partition tại một thời điểm và xử lý chúng bằng cách tạo worker threads song song cho mỗi message. Khi quá trình xử lý hoàn tất, các offsets của các messages thành công sẽ được sắp xếp và một manual commit được thực hiện cho offset lớn nhất, và các messages bị lỗi được republished đến retry topic. Điều này cho phép xử lý song song ngay cả trong một single partition, và concurrency có thể được kiểm soát tự động. Kết quả là, tốc độ xử lý sự kiện nhanh hơn so với phương pháp automatic commit hiện có, và nó được xử lý ổn định mà không bị trễ ngay cả khi số lượng sự kiện tăng lên.

---

## Stream processing (Xử lý luồng)

Stream ingestion pipeline chỉ thực hiện logic [extract, transform, and load](https://aws.amazon.com/what-is/etl/) (ETL) và xác thực đơn giản. Đã có nhiều yêu cầu về complex stream processing trong feature platform, và một dịch vụ riêng biệt đã được tạo ra để đáp ứng chúng. Feature platform đã không giải quyết các yêu cầu này vì những lý do sau:

* Mục đích của stream ingestion trong feature platform là thu thập và lưu trữ features theo thời gian thực, trong khi mục đích chính của stream processing là xử lý dữ liệu.
* Không phải tất cả các features đều yêu cầu xử lý phức tạp. Chúng tôi quyết định rằng không phù hợp để làm phức tạp toàn bộ quá trình thu thập luồng cho một số features.
* Dữ liệu kết quả của stream processing có thể được sử dụng bên ngoài feature platform, và có các yêu cầu cần xem xét điều này. Do đó, việc tạo một dịch vụ riêng biệt phù hợp hơn với tình hình của Karrot.
* Ngoài ra, một số dữ liệu nguồn không tồn tại trong AWS, điều này có thể dẫn đến chi phí bổ sung đáng kể nếu mọi thứ được xử lý trong feature platform.

Mặc dù là một dịch vụ riêng biệt với feature platform, sau đây là phần giới thiệu ngắn gọn về cách feature platform sử dụng dữ liệu thông qua stream processing:

* **Các trường hợp content embedding đa dạng** – Chúng tôi thực hiện stream processing bằng cách sử dụng các mô hình, và sử dụng các nội dung khác nhau (bài viết, hình ảnh, v.v.) làm giá trị đầu vào cho các mô hình đã được huấn luyện trước để tạo ra các embeddings. Các embeddings này được lưu trữ trong feature platform và được sử dụng làm features trong quá trình đề xuất để cải thiện chất lượng đề xuất.
* **Các trường hợp Rich feature generation** – Một số dữ liệu đã được xử lý được xử lý thêm bằng cách sử dụng large language models (LLMs) để sử dụng làm features. Một ví dụ là dự đoán một sản phẩm cũ cụ thể thuộc danh mục nào và sử dụng giá trị dự đoán này làm feature.

---

## Batch ingestion (Nhập dữ liệu theo lô)

Batch ingestion chịu trách nhiệm xử lý và lưu trữ một lượng lớn dữ liệu thành features theo lô. Điều này được chia thành một cron job chạy định kỳ và một backfill job tải một lượng lớn dữ liệu một lần.

Vì mục đích này, [AWS Batch](https://aws.amazon.com/batch/) dựa trên [AWS Fargate](https://aws.amazon.com/fargate/) được sử dụng. AWS Batch jobs chạy trên Fargate được cung cấp độc lập với các môi trường khác, cho phép xử lý quy mô lớn an toàn. Ví dụ, ngay cả khi hơn 1.000 servers hoặc 10.000 vCPUs được sử dụng để backfilling một lượng lớn dữ liệu, chúng vẫn được vận hành tách biệt với các dịch vụ khác và có thể được vận hành hiệu quả với usage-based billing method (phương thức thanh toán dựa trên mức sử dụng).

Khi thêm features mới, việc tải batch dữ liệu quá khứ hoặc tải định kỳ một lượng lớn dữ liệu là một trong những chức năng cốt lõi của feature platform. Các yêu cầu chính được xem xét trong thiết kế là:

* Nó phải có khả năng xử lý một lượng lớn dữ liệu.
* Nó phải có khả năng bắt đầu vào thời gian người dùng mong muốn và hoàn thành công việc trong một khoảng thời gian thích hợp.
* Nó phải có chi phí vận hành thấp. Tốt nhất là nó nên là một managed service, và sẽ tốt hơn nếu có ít công việc bổ sung hoặc kiến thức chuyên môn cụ thể để vận hành. Ngoài ra, nó nên tái sử dụng mã dịch vụ hiện có càng nhiều càng tốt.
* Các hoạt động phức tạp cho features hoặc cấu hình của Directed Acyclic Graphs (DAGs) không nhất thiết phải có.

Có một số tùy chọn để lựa chọn, chẳng hạn như [Apache Airflow](https://airflow.apache.org/), nhưng AWS Batch đã được chọn để tránh over-engineering khi xem xét chi phí vận hành theo các yêu cầu hiện tại.

> *Sơ đồ kiến trúc của batch ingestion pipeline.*
> <img src="/images/bl2-3.png" alt="" width="50%">

Các thành phần chính là:

* **Scheduler** – Nó trích xuất các mục tiêu cần thực hiện batch jobs theo các specifications như `FeatureGroupSpec` và `IngestionSpec` do người dùng viết trên feature platform, và đăng ký các job specifications tương ứng vào một AWS Batch job (submit job).
* **AWS Batch** – Các jobs được submit bởi Scheduler được thực thi bằng cách sử dụng job queue và computing environment đã được cấu hình trước. Trong trường hợp AWS Batch, bạn có thể cấu hình môi trường Fargate riêng biệt với các dịch vụ production khác, để ngay cả khi bạn cung cấp tài nguyên quy mô lớn và thực hiện các tác vụ, bạn vẫn có thể thực hiện các tác vụ một cách ổn định mà không ảnh hưởng đến các dịch vụ production khác.

### Cải tiến trong tương lai cho Batch ingestion

Cấu hình hiện tại hoạt động tốt và đáng tin cậy, nhưng có một số lĩnh vực cần cải tiến:

* **No DAG support** – Feature platform ban đầu thực hiện các tác vụ tương đối đơn giản, chẳng hạn như parsing batch data sources, chuyển đổi chúng sang feature schema, và lưu trữ chúng. Tuy nhiên, khi platform trở nên tiên tiến hơn, các hoạt động phức tạp hơn trở nên cần thiết, và do đó, việc hỗ trợ các cấu hình DAG có thể xử lý features bằng cách thực hiện tuần tự các dependent jobs khác nhau trở nên cần thiết.
* **Manual configuration for parallel processing** – Hiện tại, khi xử lý dữ liệu quy mô lớn một cách song song, worker phải ước tính thủ công số lượng jobs sẽ được xử lý song song và cung cấp nó trong specification, và Scheduler thực hiện submit job song song dựa trên điều này. Phương pháp này hoàn toàn dựa trên kinh nghiệm, và để hệ thống trở nên tiên tiến hơn, hệ thống phải có khả năng tự động trừu tượng hóa và tối ưu hóa mức độ parallel processing thích hợp.
* **Limited AWS Batch monitoring usability** – Giám sát AWS Batch có một số hạn chế, chẳng hạn như jobs không chuyển từ trạng thái Runnable sang Running, thiếu hệ thống thông báo thích hợp cho các trường hợp như vậy, và không thể kiểm tra trực tiếp các failed jobs thông qua các URL parameters khi nhận cảnh báo. Các khía cạnh này nên được cải thiện từ góc độ tiện lợi trong vận hành.

---

## Kết quả

Tính đến tháng 2 năm 2025, Karrot đã giải quyết các vấn đề chính được đề cập trong giai đoạn đầu phát triển feature platform:

* **Decoupling recommendation logic from flea market server** – Hệ thống đề xuất hiện sử dụng feature platform trên hơn 10 không gian và dịch vụ đề xuất khác nhau.
* **Securing scalability of features used in recommendation logic** – Với hơn 1.000 high-quality và rich features thu được từ nhiều dịch vụ khác nhau như flea market, advertisements, local jobs, và real estate, chúng tôi đang đóng góp vào sự tiến bộ của logic đề xuất và giúp tất cả các kỹ sư Karrot dễ dàng explore và thêm features.
* **Maintaining the reliability of feature data sources** – Thông qua feature platform, chúng tôi đang cung cấp dữ liệu đáng tin cậy bằng cách sử dụng consistent schema và ingestion pipeline.

Các kỹ sư Karrot không ngừng cải thiện trải nghiệm người dùng bằng cách nâng cao các đề xuất thông qua các high-quality features nhờ vào feature platform. Điều này đã góp phần tăng click-through rates lên 30% và conversion rates lên 70% so với trước đây bằng cách đề xuất các bài viết mà người dùng có thể quan tâm.

Điều này có thể thực hiện được là nhờ các dịch vụ AWS được sử dụng trong feature platform đã hỗ trợ vững chắc. **[Amazon DynamoDB](https://aws.amazon.com/dynamodb/)** có scalability đáng kinh ngạc trong mọi khía cạnh read, write, và storage, vì vậy có thể xử lý khối lượng công việc thay đổi động mà không phát sinh chi phí vận hành riêng biệt. **[Amazon ElastiCache](https://aws.amazon.com/elasticache/)** đã thể hiện sự ổn định dịch vụ đáng tin cậy cao, vì vậy chúng tôi có thể sử dụng nó một cách tự tin. Ngoài ra, việc scale up, scale down, scale in, và scale out rất dễ dàng và ổn định, vì vậy có thể giảm bớt gánh nặng vận hành. Nó cũng tích hợp liền mạch với hệ sinh thái của Redis OSS, vì vậy chúng tôi có thể sử dụng các hệ sinh thái open source như [Redis Exporter](https://github.com/oliver006/redis_exporter). Amazon MSK cũng hỗ trợ hoạt động đáng tin cậy và tích hợp liền mạch với hệ sinh thái Apache Kafka, giúp việc phát triển và vận hành feature platform dễ dàng.

Hơn nữa, làm việc với AWS cho phép vận hành hiệu quả về chi phí dựa trên sự hỗ trợ và chuyên môn đa dạng của họ. Gần đây, chúng tôi đã gặp vấn đề over-provisioning với ElastiCache cluster của mình. Việc Right-sizing ElastiCache cluster với sự giúp đỡ của nhiều chuyên gia khác nhau (bao gồm cả Solutions Architects) đã giúp tối ưu hóa chi phí ElastiCache gần 40%. Các nguồn nhân lực kỹ thuật từ AWS này đã vô cùng quý giá trong việc vận hành feature platform bằng cách sử dụng các sản phẩm AWS.

---

## Kết luận

Trong loạt bài này, chúng tôi đã thảo luận về cách Karrot xây dựng một feature platform trên AWS. Chúng tôi tin rằng bằng cách kết hợp các dịch vụ AWS và kinh nghiệm của chúng tôi, bạn có thể phát triển và vận hành một feature store mà không gặp khó khăn bằng cách điều chỉnh nó cho phù hợp với yêuV.cầu của công ty bạn. Hãy thử nghiệm việc triển khai này và cho chúng tôi biết suy nghĩ của bạn trong phần bình luận.