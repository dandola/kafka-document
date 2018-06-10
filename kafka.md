# Tìm hiểu về kafka
## 1. khái niệm cơ bản về kafka
- Kafka được hiểu là một hệ thống truyền thông điệp phân tán, hoặc có thể hiểu là một hệ thống logging phân tán, lưu trữ các trạng thái của hệ thống, nhằm phòng tránh mất thông tin
- Các thông điệp sẽ được thực hiện theo cơ chết public/subscribe, tương tự như message queue.
- Thông điệp trong kafka được replicated nhằm phòng tránh mất dữ liệu.
- Kafka có khả năng mở rộng hệ thống rất tốt.
- Hệ thống kafka được chạy trên một cụm bao gồm một hoặc nhiều server.
- kafka lưu trữ các thông điệp tới các thể loại, hay còn gọi là Topics.
- Mỗi một thông điệp bao gồm: 1 key, 1 value và một timestamp.
- Hệ thống kafka gồm có 4 APIs chính:
    
    + Producer API: cho phép ứng dụng publish một thông điệp tới một hoặc nhiều topics, hay nói cách khác nhiệm vụ của Producer là với thông điệp này sẽ được đưa vào topics nào.
    + Consumer API: cho phép ứng dụng subscribe vào một hoặc nhiều topics và xử lý các thông điệp thuộc topics.
    + Streams API: Được hiểu như một bộ xử lý luồng dữ liệu, với dữ liệu đầu vào từ một hoặc nhiều topics và đưa ra output tới một hoặc nhiều topics.
    + Connector API: Được sử dụng để kết nối topics tới các hệ thống như database.

## 2. Topics và logs
- Topics hay còn gọi là chủ để, hoặc thể loại do người dùng định nghĩa được sử dụng như một message queue, các thông điệp mới do một hoặc nhiều producer đẩy vào. Các thông điệp giống nhau hoặc tương tự nhau sẽ thuộc cùng 1 topic.
- Mỗi một Topics sẽ được lưu thành một partitioned log như hình bên dưới:

    <img src="partition_log.png" />
- Trong mỗi Topic sẽ có nhiều partition, mỗi partition sẽ được sắp xếp thứ tự các thông điệp, với một thông điệp mới đẩy vào partition thì sẽ luôn được đẩy vào cuối partition. Mỗi thông điệp trong partition sẽ được định danh một id theo thứ tự, hay còn gọi là offset, và định danh này là duy nhất trong mỗi partition. Thứ tự giữa các thông điệp trong một partition là bất biến, tức là không thay đổi được.
- Trong hệ thống, kafka lưu tất cả các message đã được publish, cho dù nó đã được sử dụng hoặc chưa được sử dụng. thời gian lưu message sẽ được tuỳ chỉnh thông qua log retension.
- Vì consumer chỉ lưu thông tin ở dạng metadata( có thể là offset hoặc vị trí của message trên partition), các giá trị offset có thể được điều khiển bởi consumer: thông thường, consumer sẽ đọc offset theo thứ tự tăng dần, nhưng nó có thể reset vị trí offset để xử lý lại một số message, ví dụ như: đưa các message mới đẩy lên đầu, hoặc bỏ qua một số message được sử dụng gần đây,...
- ví trong log có nhiều partition, nên hệ thống cho phép log có khả năng mở rộng tốt, tức là có khả năng lưu trữ dữ liệu một cách tuỳ ý.
- Mỗi partition phải có không gian lưu trữ phù hợp trên mỗi server chứa nó.
- Các partitions trong log có thể hoạt động song song, tức là tại một thời điểm có thể có nhiều consumer đọc trên nhiều partition nằm trong cùng 1 log.


## 3. Phân tán trên kafka
- Các partition thuộc 1 log được phần tán trên các server của cụm. Mỗi partition sẽ được sao lưu đến một số server khác nhằm tăng khả năng chịu lỗi.
- Với mỗi partition sẽ có một server làm "leader", server này sẽ lưu bản chính, và có nhiều server làm "followers". Leader sẽ đọc và xử lý request kiên quan đến partition, còn Followers chỉ liên quan đến sao lưu dữ liệu từ leader. Nếu leader bị lỗi, một trong số followers sẽ được bầu lên làm Leader mới.

## 4. Producers
- Producers publish thông điệp tới topics mà chúng lựa chọn, tức là nó sẽ chọn thông điệp nào sẽ được gán cho partition nào trong topic.

## 5. Consumers
- Các consumers sẽ được gắn với một consumer group, mỗi thông điệp được publish một topic sẽ được phân phối đến một consumer instance trong mỗi consumer group nó đăng ký.
- nếu các consumer instances nằm trong cùng 1 group, thì các bản ghi sẽ được phân phối đều trên các consumer instances -- load balance. Từ đó ta được hệ thống queue
- Nếu các consumer instance thuộc các group khác nhau, thì mỗi bản ghi sẽ được broadcast tới các consumers. Như vậy ta được hệ thống pub/sub
- Trong một nhóm, các consumer sẽ chiếm một số partition tương tự nhau tại bất kỳ thời điểm, đảm bảo cân bằng tải. Vấn đề này được xử lý bởi giao thức kafka. Nếu 1 consumer instancce mới tham giao vào nhóm, nó sẽ chiếm lấy một số partition của các consumer khác trong nhóm, nếu nó bị lỗi, các partition được phân tán các consumer còn lại. Mục đích của giao thức là đảm bảo cân bằng tải giữa các partition cũng như tăng khả năng chống lỗi.

## 6. Guarantees
- kafka đảm bảo thứ tự các thông điệp trong 1 partition đúng với thứ tự khi được gửi thông điệp từ một producer. Ví dụ message1 và message được gửi cùng một producer, đâu tiên là gửi message1, sau đó gửi message2, thì khi đó message1 sẽ được đẩy vào log sớm hơn có offset nhỏ hơn message2.
- Một consumer sẽ **nhìn thấy** thứ tự các thông điệp mà chúng được lưu trong log.
- Đối với một topic với N bản sao, hệ thống sẽ chịu được tối đa N-1 server bị lỗi(hoặc chết) mà không bị mất bất kỳ dữ liệu nào nằm trong log.

## 7. kafka giống như một hệ thống truyền thông điệp
- Truyeèn thông điệp theo kiểu truyền thống có hai models: **queuing** và **publish-subscribe**. Đối với hàng đợi, các consumers có thể đọc từ một server và mỗi record chỉ đi tới một trong số chúng. còn publish-subscrib, record được broadcast tới tất cả consumers.
- Điểm mạnh của queue là nó cho phép bạn chia quá trình xử lý dữ liệu tới nhiều consumer, cho phép bạn mở rộng quá trình xử lý, tuy nhiên queues không phải là multi-subsciber.
- Publish-Subscribe cho phép bạn broadcast dữ liệu tới nhiều processes. tuy nhiên ko có cách nào để mở rộng quá trình xử lý bởi vì mọi messages đều đi tới mọi subscriber.
- Kafka có thể giải quyết được 2 vấn đề trên bằng cách sử dụng consumer group. Giống như hàng đợi, consumer group phân chia quá trình xử lý cho nhiều consumers nằm trong group. Giống như Publish-Subscribe, kafka cho phép bạn broadcast data tới nhiều consumer group mà nó đăng ký. Như vậy mọi topic đều có cả 2 điểm mạnh trên. nó có khả năng mở rộng quá trình xử lý và cũng là multi-subscriber.
- Đối với hàng đợi, các bản ghi được sắp xếp theo thứ tự trên server, khi nhiều consumers gửi yêu cầu từ  hàng đợi, server sẽ truyền bản ghi theo thứ tự trong hàng đợi cho các consumer. Tuy nhiên các bản ghi sẽ được phân phối một cách không đồng bộ tới các consumers, tức là thứ tự gửi các bản ghi sẽ khác so với thứ tự nhận các bản ghi trên consumers, điều này bị ảnh hưởng bởi việc thực hiện song song. Do đó các Message systems đưa ra một khái niệm là  "exclusive consumer" (người dùng độc quyền), tức là chỉ cho một consumer xử lý và sử dụng một hàng đợi, và dĩ nhiên với cách làm việc này thì không có xử lý song song.
- Kafka có thể xử lý được vấn đề này, để có thể thực hiện xong xong mà vẫn đảm bảo được thứ tự các bản ghi, các pảrtitions trong topic sẽ được gửi tới các consumer trong consumer group, do đó mỗi partition được tiêu thụ bởi một consumer trong group, điều này đảm bảo một consumer chỉ đọc partition và tiêu thụ data theo thứ tự nằm trên partition đó. mặc dù có nhiều partitions trong topic những vẫn đảm bảo được cân bằng tải trên nhiều consumer nằm trong group. chú ý là số lượng consumer nằm trong group không nhiều hơn partitions.

## 8.kafka giống như một hệ thống lưu trữ
- 





## 10. Cài đặt nhanh
- Bước 1:
    + Download bản 1.1.0 sau đó giải nén tệp:

        ``> tar -xzf kafka_2.11-1.1.0.tgz``

        ``> cd kafka_2.11-1.1.0``
- Bước 2: start the server
    + Vì kafka sử dụng Zookeeper, nên cần chạy Zookeeper server đầu tiên.

        ``> bin/zookeeper-server-start.sh config/zookeeper.properties``

    + sau đó run kafka server:

        ``> bin/kafka-server-start.sh config/server.properties``

- Bước 3: Create a topic
    + Để tạo 1 topic có thên là "test" với 1 partition và 1 bản sao sử dụng câu lệnh:

        ``> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partition 1 --topic test``

    + Để xem danh sách các topics sử dụng câu lệnh:

        ``> bin/kafka-topics.sh --list --zookeeper localhost:2181``

- Bước 4: Send some messages

    + Để gửi thông điệp từ producer tới kafka cluster, thực hiện câu lệnh:

    
        ``> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test``   
        This is a message   
        This is another message
    
    + mặc định là mỗi một dòng là một thông điệp riêng (This is a message là một thông điệp, this is another message là  một thông điệp khác).
- Bước 5 : Start a consumer
    + Để thực chạy consumer, thực hiện câu lệnh:  
        ``> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning  ``   
        This is a message   
        This is a another message   
    + Nếu bạn chạy câu lệnh này trên terminal khác, thì khi bạn nhập message trong producer, thì message này sẽ được hiện ra ở consumer terminal.

- Bước 6: setting up a multi-broker cluster
    + Để tạo nhiều broker trong cụm kafka, đầu tiên cần tạo ra 1 file config cho từng con broker.
        
        ``> cp config/server.properties config/server-1.properties``    
        ``> cp config/server.properties config/server-1-properties``    
    + Thông tin của 2 broker hiển thị như sau:

    broker 1:

        config/server-1.properties: 
            broker.id=1
            listeners=PLAINTEXT://:9093
            log.dir=/tmp/kafka-logs-1
 
    broker 2:

        config/server-2.properties:
            broker.id=2
            listeners=PLAINTEXT://:9094
            log.dir=/tmp/kafka-logs-2

    + broker.id có giá trị duy nhất và cũng là định danh của broker trong cụm.

