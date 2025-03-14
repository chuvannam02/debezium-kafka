# Markdown

Tệp tin register-mysql.json
Tệp này cấu hình một Debezium MySQL Connector để theo dõi thay đổi trong cơ sở dữ liệu MySQL và gửi chúng đến Kafka. 🚀

docker-compose -f docker-compose-mysql.yml --env-file .env up -d
1️⃣ docker-compose

- Đây là công cụ để chạy các dịch vụ trong Docker theo cấu hình trong tập tin docker-compose.yml hoặc tập tin được chỉ định.

2️⃣ -f docker-compose-mysql.yml
-f chỉ định một tập tin docker-compose.yml khác thay vì dùng mặc định (docker-compose.yml).

- Ở đây, docker-compose-mysql.yml được sử dụng để định nghĩa các container.

3️⃣ --env-file .env

- Chỉ định một tập tin .env để nạp biến môi trường.
- Biến môi trường trong .env có thể chứa thông tin như:
  DEBEZIUM_VERSION=2.7
  MYSQL_ROOT_PASSWORD=debezium
- Các biến này có thể được sử dụng trong docker-compose-mysql.yml như:
  image: quay.io/debezium/example-mysql:${DEBEZIUM_VERSION}
  → Sẽ thay ${DEBEZIUM_VERSION} bằng giá trị thực trong .env.

4️⃣ up -d
up: Chạy tất cả các container được định nghĩa trong docker-compose-mysql.yml.
-d (detached mode): Chạy container ở chế độ nền (background), không chiếm terminal.


## Các câu lệnh Docker sử dụng

Kiểm tra log của một service cụ thể theo thời gian thực

docker logs -f demo_kafka


## Tệp tin register-mysql.json

[]![Debezium MySQL Connector](https://debezium.io/documentation/reference/1.6/connectors/mysql.html)

```json
{
  "name": "mysql-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.server.id": "184054",
    "database.server.name": "dbserver1",
    "database.whitelist": "inventory",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "schema-changes.inventory"
  }
}
```

[//]: # Hướng dẫn sử dụng

## Hướng dẫn sử dụng

Sau khi chạy bằng lệnh docker-compose -f docker-compose-mysql.yml --env-file .env up -d thành công. Hệ thống sẽ khởi chạy những thành phần dưới đây:

* 🦓 **Zookeeper** chạy và lắng nghe ở các cổng `2181`, `2888` và `3888`. Đây là thành phần bắt buộc giúp Kafka hoạt động ổn định trong môi trường phân tán.
* 🐘 **Kafka** chạy ở cổng `9092` và sẽ được khởi tạo ngay khi Zookeeper hoàn tất. Kafka là message broker trung tâm để tiếp nhận và phân phối dữ liệu.
* 🧠 **Kafka-ui** sử dụng thư viện [provectuslabs](https://github.com/provectus/kafka-ui) cung cấp giao diện trực quan để thao tác và làm việc với Kafka. Tất nhiên Kafka phải sẵn sàng thì component Kafka-ui mới có thể khởi tạo thành công.
* 🐬 **MySQL** chạy ở cổng `3307` (nội bộ là `3306`) với image từ Debezium, dùng để phục vụ mục đích demo Change Data Capture (CDC) từ cơ sở dữ liệu tới Kafka.
* 🐘 **PostgreSQL** chạy ở cổng `5432` với database mặc định là `inventory`, thuận tiện cho việc thử nghiệm song song giữa các hệ quản trị cơ sở dữ liệu.
* 🧊 **Redis** chạy ở cổng `6379`, đóng vai trò là một hệ thống cache, thích hợp cho việc lưu trạng thái tạm thời hoặc hỗ trợ các chức năng realtime.
* 🔌 **Kafka Connect** chạy ở cổng `8083`, là cầu nối quan trọng giữa các nguồn dữ liệu như MySQL và Kafka. Thành phần này sử dụng Debezium connector để lắng nghe các thay đổi dữ liệu (CDC) và đẩy vào Kafka.

> 📌 Lưu ý: Tất cả các service đều được cấu hình `restart: always` nhằm đảm bảo tự động khởi động lại nếu container gặp lỗi hoặc hệ thống reboot.


### 🔗 Thành phần chính


| Thành phần        | Cổng mặc định    | Ghi chú                                                                                      |
| ------------------- | -------------------- | --------------------------------------------------------------------------------------------- |
| 🦓**Zookeeper**     | `2181`,`2888`,`3888` | Dịch vụ điều phối cho Kafka                                                              |
| 🐘**Kafka**         | `9092`               | Được khởi tạo sau khi Zookeeper sẵn sàng                                               |
| 🧠**Kafka UI**      | `9089`→`8080`       | Giao diện quản lý Kafka do[Provectus Labs](https://github.com/provectus/kafka-ui)cung cấp |
| 🐬**MySQL**         | `3307`→`3306`       | Container dùng image Debezium để hỗ trợ CDC                                              |
| 🐘**PostgreSQL**    | `5432`               | Hệ quản trị CSDL quan hệ khác nếu bạn dùng JSONB                                      |
| 🧊**Redis**         | `6379`               | Hệ thống cache sử dụng Redis 7                                                            |
| 🧩**Kafka Connect** | `8083`               | Thành phần trung gian của Debezium để kết nối và stream data từ DB đến Kafka       |

> 🔁 **Lưu ý:** thứ tự khởi tạo đã được xử lý bằng `depends_on`, đảm bảo đúng chuỗi:
> `Zookeeper → Kafka → Kafka UI / Connect` và `MySQL → Connect`.
>
