---
title: "Karrot xây dựng Feature Platform trên AWS, Phần 1: Động lực và Feature Serving"
date: "2025-08-14T10:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
# Karrot đã xây dựng nền tảng tính năng (feature platform) trên AWS như thế nào, Phần 1: Động lực và phục vụ tính năng (feature serving)

> Tác giả: Hyeonho Kim, Jinhyeong Seo, Minjae Kwon, Hyuk Lee, Jinhyun Park, Jungwoo Song, và Nak-KWon Choi | Ngày: 14 THÁNG 8, 2025 | Trong: [Advanced (300)](https://aws.amazon.com/blogs/architecture/category/learning-levels/advanced-300/), [Amazon DynamoDB](https://aws.amazon.com/dynamodb/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/), [Amazon ElastiCache](https://aws.amazon.com/elasticache/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/)
>
> *Bài viết này đồng tác giả với Hyeonho Kim, Jinhyeong Seo và Minjae Kwon từ Karrot.*
<img src="/images/bl1.png" alt="" width="70%">

**[Karrot](https://about.daangn.com/en/)** là cộng đồng địa phương hàng đầu của Hàn Quốc, một dịch vụ tập trung vào tất cả các kết nối có thể trong khu phố. Vượt ra ngoài các khu chợ trời đơn thuần, Karrot củng cố mối liên hệ giữa hàng xóm, cửa hàng địa phương và các tổ chức công cộng, đồng thời tạo ra một khu phố ấm áp và năng động như giá trị cốt lõi của mình.

Karrot sử dụng một hệ thống đề xuất để cung cấp cho người dùng các kết nối phù hợp với sở thích và khu phố của họ, đồng thời mang lại trải nghiệm cá nhân hóa. Nội dung được cá nhân hóa liên tục được cập nhật bằng cách phân tích các mẫu hoạt động của người dùng mà không cần phải thiết lập một danh mục sở thích đặc biệt. Cốt lõi của feed là cung cấp nội dung mới và thú vị, và Karrot không ngừng nỗ lực để cải thiện sự hài lòng của người dùng cho mục đích này. Karrot tích cực sử dụng hệ thống đề xuất để cung cấp nội dung được cá nhân hóa và đề xuất.

Trong hệ thống này, **nền tảng tính năng (feature platform)** đóng vai trò then chốt cùng với **mô hình đề xuất học máy (ML recommendation model)**. Nền tảng tính năng hoạt động như một kho dữ liệu lưu trữ và phục vụ dữ liệu cần thiết cho mô hình đề xuất ML, chẳng hạn như lịch sử hành vi của người dùng và thông tin bài viết.

Loạt bài hai phần này bắt đầu bằng việc trình bày động lực, yêu cầu và kiến trúc giải pháp của Karrot, tập trung vào việc phục vụ tính năng (feature serving). [Phần 2]({{< ref "/3-blogstranslated/3.2-blog2" >}}) sẽ đề cập đến quy trình thu thập tính năng theo thời gian thực và nhập dữ liệu theo lô (batch ingestion) vào online store, cùng với các cách tiếp cận kỹ thuật để vận hành ổn định.

---

## Bối cảnh về nền tảng tính năng tại Karrot

Karrot nhận ra sự cần thiết của nền tảng tính năng vào đầu năm 2021, khoảng 2 năm sau khi triển khai hệ thống đề xuất trong ứng dụng của họ. Vào thời điểm đó, Karrot đang đạt được sự tăng trưởng đáng kể (hơn 30% click-through rates) và mức độ hài lòng của người dùng cao hơn nhờ việc sử dụng tích cực hệ thống đề xuất. Khi tác động của hệ thống đề xuất tiếp tục tăng lên, nhóm ML phải đối mặt với thách thức nâng cao hệ thống.

Trong các hệ thống dựa trên ML, nhiều loại dữ liệu đầu vào chất lượng cao (như lượt nhấp chuột, hành động chuyển đổi, v.v.) được coi là một yếu tố then chốt. Những dữ liệu đầu vào này thường được gọi là **features**. Tại Karrot, dữ liệu bao gồm nhật ký hành vi người dùng, nhật ký hành động và các giá trị trạng thái được gọi chung là *user features*, còn các nhật ký liên quan đến bài đăng/tin rao vặt được gọi là *article features*.

Để cải thiện độ chính xác của các đề xuất cá nhân hóa, cần có nhiều loại features khác nhau. Một hệ thống có thể quản lý hiệu quả các features này và nhanh chóng cung cấp chúng cho các mô hình đề xuất ML là điều cần thiết. Ở đây, *serving* có nghĩa là quá trình cung cấp dữ liệu theo thời gian thực cần thiết khi hệ thống đề xuất gợi ý nội dung cá nhân hóa cho người dùng. Tuy nhiên, cách tiếp cận quản lý feature trong hệ thống đề xuất hiện có có một số hạn chế, với các vấn đề chính sau:

* **Sự phụ thuộc vào máy chủ chợ trời (flea market server)** – Vì hệ thống đề xuất ban đầu tồn tại dưới dạng một thư viện nội bộ trên flea market server, nên phải thay đổi mã nguồn của ứng dụng web bất cứ khi nào logic đề xuất được sửa đổi hoặc một feature được thêm vào. Điều này làm giảm tính linh hoạt của việc triển khai và gây khó khăn cho việc tối ưu hóa tài nguyên.
* **Khả năng mở rộng hạn chế của logic đề xuất và features** – Hệ thống đề xuất ban đầu phụ thuộc trực tiếp vào database chợ trời và chỉ xem xét các bài viết chợ trời. Điều này làm cho việc mở rộng sang các loại bài viết mới như cộng đồng địa phương, việc làm địa phương và quảng cáo, vốn được quản lý bởi các nguồn dữ liệu khác, là không thể. Ngoài ra, mã liên quan đến feature được hardcoded, gây khó khăn cho việc explore, thêm hoặc sửa đổi features.
* **Thiếu độ tin cậy của nguồn dữ liệu feature** – Mặc dù các features được truy xuất từ nhiều repository khác nhau như [Amazon Simple Storage Service](https://aws.amazon.com/s3/) (Amazon S3), [Amazon ElastiCache](https://aws.amazon.com/elasticache/), và [Amazon Aurora](https://aws.amazon.com/rds/aurora/), nhưng độ tin cậy của chất lượng dữ liệu thấp do thiếu một consistent schema và collection pipeline. Đây là một hạn chế lớn trong việc đảm bảo các features mới nhất và tính nhất quán.

> *Sơ đồ minh họa cấu trúc backend của hệ thống đề xuất ban đầu.*
<img src="/images/bl1-1.png" alt="" width="55%">

Để giải quyết những vấn đề này, chúng tôi cần một hệ thống trung tâm mới có thể hỗ trợ hiệu quả việc quản lý feature, real-time ingestion, và serving, và vì vậy chúng tôi đã bắt đầu dự án feature platform.

---

## Các yêu cầu của nền tảng tính năng (Feature platform)

Các yêu cầu chức năng sau đây đã được tổ chức bằng cách tách feature platform thành một dịch vụ độc lập:

* Ghi lại và nhanh chóng phục vụ N hành động gần đây nhất được thực hiện bởi người dùng. Cho phép parameterization cả giá trị N và khoảng thời gian lookup.
* Hỗ trợ các features dành riêng cho người dùng, chẳng hạn như từ khóa thông báo, ngoài các action features.
* Xử lý các features từ các loại bài viết khác nhau, ngoài các bài viết chợ trời.
* Xử lý các loại dữ liệu tùy ý (arbitrary data types) cho tất cả các features, bao gồm primitive types, lists, sets, và maps.
* Cung cấp các bản cập nhật theo thời gian thực cho cả action features và user characteristic features.
* Cung cấp tính linh hoạt trong danh sách features, số lượng, và khoảng thời gian lookup cho mỗi yêu cầu.

Để triển khai các yêu cầu chức năng này, một platform mới là cần thiết. Platform này cần ba khả năng cốt lõi: real-time ingestion của các loại feature khác nhau, storage với consistent schema, và phản hồi nhanh chóng đối với các yêu cầu truy vấn đa dạng. Mặc dù ban đầu những yêu cầu này có vẻ mơ hồ, việc thiết kế một cấu trúc tổng quát đã cho phép cấu hình hiệu quả các data ingestion pipelines, storage methods, và serving schemas, dẫn đến các mục tiêu phát triển rõ ràng hơn.

Ngoài các yêu cầu chức năng, các yêu cầu kỹ thuật bao gồm:

* **Serving traffic:** 1.500 hoặc nhiều hơn requests per second (RPS)
* **Ingestion traffic:** 400 hoặc nhiều hơn writes per second (WPS)
* **Top N values:** 30–50
* **Single feature size:** Lên đến 8 KB
* **Total number of features:** Hơn 3 tỷ hoặc nhiều hơn

Vào thời điểm đó, sự đa dạng và số lượng features đang được sử dụng bị hạn chế, và các mô hình đề xuất rất đơn giản, dẫn đến các yêu cầu kỹ thuật khiêm tốn. Tuy nhiên, xem xét tốc độ tăng trưởng nhanh chóng, một sự gia tăng đáng kể trong các yêu cầu hệ thống đã được dự đoán. Dựa trên dự đoán này, các mục tiêu cao hơn đã được đặt ra ngoài các yêu cầu ban đầu. Tính đến tháng 2 năm 2025, serving và ingestion traffic đã tăng khoảng 90 lần so với các yêu cầu ban đầu, và tổng số features đã tăng lên hàng trăm lần. Khả năng xử lý sự tăng trưởng nhanh chóng này có được nhờ kiến trúc có khả năng mở rộng cao của feature platform, mà chúng tôi thảo luận trong các phần sau.

---

## Tổng quan về giải pháp

> *Sơ đồ minh họa kiến trúc của feature platform.*
> 
> <img src="/images/bl1-2.png" alt="" width="55%">

Feature platform bao gồm ba thành phần chính: feature serving, một stream ingestion pipeline, và một batch ingestion pipeline.

Phần 1 của loạt bài này sẽ đề cập đến **feature serving**. Feature serving là chức năng cốt lõi của việc nhận các yêu cầu từ client và cung cấp các features cần thiết. Karrot đã thiết kế hệ thống này với bốn thành phần chính:

* **Server** – Một server nhận và xử lý các yêu cầu feature serving, và là một pod nằm trên [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS).
* **Remote cache** – Một lớp remote cache được chia sẻ bởi các servers, và sử dụng ElastiCache.
* **Database** – Một lớp lưu trữ liên tục (persistence layer) lưu trữ các features, và sử dụng [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).
* **On-demand feature server** – Một server phục vụ các features không thể lưu trữ trong remote cache và database do các vấn đề tuân thủ (compliance issues), hoặc cần tính toán theo thời gian thực mỗi lần.

Từ góc độ data store, feature serving nên phục vụ các high-cardinality features với độ trễ thấp ở quy mô lớn. Karrot đã giới thiệu multi-level cache và phân loại các chiến lược serving theo đặc điểm của các features:

* **Local cache (tier 1 cache)** – Một in-memory store nằm bên trong server, phù hợp cho các trường hợp kích thước dữ liệu nhỏ và được truy cập thường xuyên hoặc yêu cầu thời gian phản hồi nhanh.
* **Remote cache (tier 2 cache)** – Phù hợp cho các trường hợp kích thước dữ liệu vừa và được truy cập thường xuyên.
* **Database (tier 3 cache)** – Phù hợp cho các trường hợp kích thước dữ liệu lớn và không được truy cập thường xuyên hoặc ít nhạy cảm với thời gian phản hồi.

---

## Thiết kế Schema

Feature platform lưu trữ nhiều features cùng nhau bằng cách sử dụng khái niệm **feature groups**, chẳng hạn như column families. Tất cả các feature groups được định nghĩa thông qua feature group schema, được gọi là **feature group specifications**, và mỗi feature group specification định nghĩa tên của feature group, các features cần thiết, v.v.

Dựa trên khái niệm này, key design được định nghĩa như sau:
Partition key: <feature_group_name>#<feature_group_id> Sort key: <feature_group_timestamp> hoặc một chuỗi đại diện cho null

Để minh họa cách thức hoạt động này trong thực tế, hãy khám phá một ví dụ về một feature group đại diện cho các bài viết chợ trời đã nhấp gần đây (recently clicked flea market articles) của người dùng 1234. Xem xét kịch bản sau:

* **Feature group name:** recent_user_clicked_fleaMarketArticles
* **User ID:** 1234
* **Click timestamp:** 987654321
* **Features trong feature group:**
    * Clicked article ID: a
    * User session ID: 1111

Trong ví dụ này, các keys và feature group được tạo như sau:

Partition key: recent_user_clicked_fleaMarketArticles#1234 Sort Key: 987654321 Value: {"0": "a", "1": "1111"}

Các features được định nghĩa trong feature group specification duy trì một thứ tự cố định, sử dụng thứ tự này như một enum khi lưu feature group.

---

## Luồng đọc/ghi của Feature serving

Feature platform sử dụng multi-level cache và database cho feature serving, như được hiển thị trong sơ đồ sau.

> *Sơ đồ minh họa luồng đọc/ghi của feature serving.*
> <img src="/images/bl1-3.png" alt="" width="55%">

Để minh họa quá trình này, hãy xem xét cách hệ thống truy xuất feature groups 1, 2, và 3 từ các bài viết chợ trời. **Luồng đọc (read flow)** (đường liền nét trong sơ đồ trước) thể hiện việc tối ưu hóa truy cập dữ liệu bằng cách sử dụng chiến lược multi-level cache:

1.  Khi một query request đến, trước tiên hãy kiểm tra **local cache**.
2.  Dữ liệu không có trong local cache được tìm kiếm trong **ElastiCache**.
3.  Dữ liệu không có trong ElastiCache được tìm kiếm trong **DynamoDB**.
4.  Các feature groups được tìm thấy ở mỗi giai đoạn được thu thập và trả về làm phản hồi cuối cùng.

**Luồng ghi (write flow)** (đường chấm chấm trong sơ đồ trước) bao gồm các bước sau:

1.  Các feature groups bị lỗi bộ đệm (cache misses) được lưu trữ ở mỗi cấp độ cache.
2.  Dữ liệu không tìm thấy trong local cache nhưng tìm thấy trong remote cache hoặc database được lưu trữ trong upper-level cache.
3.  Dữ liệu tìm thấy trong ElastiCache được lưu trữ trong local cache.
4.  Dữ liệu tìm thấy trong DynamoDB được lưu trữ trong cả ElastiCache và local cache.
5.  Các hoạt động ghi cache được thực hiện không đồng bộ (asynchronously) trong nền.

Cách tiếp cận này trình bày một chiến lược để duy trì tính nhất quán của dữ liệu và cải thiện thời gian truy cập trong tương lai trong cấu trúc multi-level cache. Trong một tình huống lý tưởng, serving hoạt động tốt mà không có bất kỳ vấn đề nào chỉ với luồng trước đó. Tuy nhiên, thực tế không phải như vậy. Các vấn đề gặp phải bao gồm lỗi cache, tính nhất quán và vấn đề penetration:

* **Cache miss problem** – Lỗi cache thường xuyên làm chậm thời gian phản hồi và đặt gánh nặng lên next level cache hoặc database. Karrot sử dụng kỹ thuật [Probabilistic Early Expirations](https://cseweb.ucsd.edu//~varghese/papers/caching-in-cache-ton.pdf) (PEE) để chủ động làm mới dữ liệu có khả năng được truy xuất lại trong tương lai, nhờ đó duy trì độ trễ thấp và giảm thiểu cache stampede.
* **Cache consistency problem** – Nếu Time-To-Live (TTL) của cache được đặt không chính xác, nó có thể ảnh hưởng đến chất lượng đề xuất hoặc giảm hiệu quả hệ thống. Karrot đặt soft và hard TTL riêng biệt, và đôi khi sử dụng [chiến lược write-through caching](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/write-through.html) cùng nhau để đồng bộ hóa cache và database nhằm giảm bớt các vấn đề về tính nhất quán. Ngoài ra, [jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/) được thêm vào để phân tán thời gian xóa TTL nhằm giảm bớt [cache stampede](https://en.wikipedia.org/wiki/Cache_stampede) của các feature groups được ghi vào những thời điểm tương tự.
* **Cache penetration problem** – Các truy vấn liên tục cho các feature groups không tồn tại có thể dẫn đến các truy vấn DynamoDB, làm tăng chi phí và thời gian phản hồi. Platform giải quyết vấn đề này thông qua [negative caching](https://aws.amazon.com/blogs/database/caching-empty-results-with-amazon-elasticache-for-redis/), lưu trữ thông tin về các feature groups không tồn tại để giảm các truy vấn database không cần thiết. Ngoài ra, hệ thống giám sát tỷ lệ missing feature groups trong DynamoDB, negative cache hit rates, và các vấn đề tiềm ẩn về tính nhất quán.

---

## Cải tiến trong tương lai cho Feature serving

Karrot đang xem xét các cải tiến trong tương lai sau đây cho giải pháp feature serving của họ:

* **Large data caching** – Gần đây, nhu cầu lưu trữ các features dữ liệu lớn đang tăng lên. Điều này là do khi Karrot phát triển, số lượng features cũng tăng lên. Ngoài ra, khi nhuH.cầu về embeddings tăng lên cùng với sự phát triển nhanh chóng của large language models (LLMs), kích thước dữ liệu cần lưu trữ đã tăng lên. Theo đó, chúng tôi đang xem xét serving hiệu quả hơn bằng cách sử dụng embedded database.
* **Sử dụng bộ nhớ cache hiệu quả:** Ngay cả khi giá trị TTL hiệu quả được đặt ban đầu, hiệu quả có xu hướng giảm khi mẫu sử dụng của người dùng thay đổi và mô hình được thay đổi. Ngoài ra, khi nhiều feature groups được định nghĩa hơn, việc giám sát trở nên khó khăn hơn. Cần phải đơn giản để tìm ra giá trị TTL tối ưu cho cache dựa trên dữ liệu. Chúng tôi đang xem xét một phương pháp để sử dụng bộ nhớ hiệu quả trong khi vẫn duy trì chất lượng đề xuất cao thông qua cache hit rate và phòng ngừa mất feature group. Liệu chúng ta có nên cache một feature group chỉ được truy xuất một lần? Còn một feature group được truy xuất hai lần thì sao? Feature platform hiện tại cố gắng caching ngay cả khi lỗi cache chỉ xảy ra một lần. Chúng tôi tin rằng tất cả các feature groups bị lỗi cache đều đáng để cache. Điều này đương nhiên làm tăng sự kém hiệu quả của caching. Cần có một chính sách nâng cao để xác định và cache các feature groups đáng để cache dựa trên dữ liệu khác nhau. Điều này sẽ làm tăng hiệu quả sử dụng cache.
* **Multi-level cache optimization:** Hiện tại, feature platform có cấu trúc multi-level cache, và độ phức tạp sẽ tăng lên nếu một embedded database được thêm vào trong tương lai. Do đó, cần phải tìm và đặt các cài đặt tối ưu bằng cách xem xét các cấp độ cache khác nhau. Trong tương lai, chúng tôi sẽ cố gắng tối đa hóa hiệu quả bằng cách xem xét các cấp độ cài đặt cache khác nhau.

---

## Kết luận

Trong bài viết này, chúng tôi đã xem xét cách Karrot xây dựng feature platform của họ, tập trung vào khả năng feature serving. Tính đến tháng 2 năm 2025, platform xử lý đáng tin cậy hơn 100.000 RPS với độ trễ P99 dưới 30 mili giây, cung cấp các dịch vụ đề xuất ổn định thông qua một kiến trúc có khả năng mở rộng (scalable architecture) quản lý hiệu quả sự gia tăng lưu lượng truy cập.

[Phần 2]({{< ref "/3-blogstranslated/3.2-blog2" >}}) sẽ khám phá cách các features được tạo bằng cách sử dụng consistent feature schemas và ingestion pipelines thông qua feature platform.