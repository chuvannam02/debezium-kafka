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

