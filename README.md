# Distributed Transactions in Microservices Using 2-Phase Commit (2PC)

The services in this project are designed with microservice architecture for performing distributed transaction using 2-Phase Commit (2PC).

Each microservice exposes REST API interfaces that can be accessed through OpenAPI endpoint (/swagger-ui.html)

## Tech Stack

1. SpringBoot
2. Spring Data JPA
3. MySQL Database
4. RabbitMQ

## Pre-Requisites

1. RabbitMQ
   - Start RabbitMQ in Docker with command `docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management` [Learn more](https://hub.docker.com/_/rabbitmq)
   - RabbitMQ 管理控制台UI: http://localhost:15672  guest/guest
2. MySQL Database
   - Start MySQL in Docker with command `docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag` [Learn more](https://hub.docker.com/_/mysql)
   - init data:

   ```sql
   create database distibuted_txn;

   create table accounts (id bigint not null primary key auto_increment, balance integer, customer_id bigint) engine =INNODB;
   create table orders (id bigint not null primary key auto_increment, customer_id bigint, product_id bigint, quantity integer) engine =INNODB ;
   create table products (id bigint not null primary key auto_increment, name varchar(255), price integer, quantity integer) engine =INNODB;

   insert into accounts values (1, 9999, 1);
   insert into products values (89, 'a test product', 3.2, 1000);
   ```

## Usage

1. Start `discovery-server`. Default port is 8761.
2. Start all microservices: `transaction-server`, `account-service`, `order-service`, `product-service`
3. Add some test data to `account-service` and `product-service`
4. Send order creation request to `order-service` for testing the flow.
5. 发起一个创建订单请求：`curl 'http://localhost:8081/orders' \ -H 'Accept: */*' \ -H 'Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7' \ -H 'Cache-Control: no-cache' \ -H 'Connection: keep-alive' \ -H 'Cookie: Idea-d73977e5=c55edb0c-bdc9-4d6f-a391-1ee92fd8b993; grafana_session=b2f237e09dc9674aa3e50d501d0a4f7a; grafana_session_expiry=1701593979; pma_lang=en; pmaUser-1=%7B%22iv%22%3A%22KXbaGSUUISqVWJhbPAEiiw%3D%3D%22%2C%22mac%22%3A%223083c06f225db464a4976e5543c2c60d960c400b%22%2C%22payload%22%3A%22YAegT9i4Bc02Kp7eEF%5C%2Fg1w%3D%3D%22%7D; m=59b9:true' \ -H 'Pragma: no-cache' \ -H 'Sec-Fetch-Dest: empty' \ -H 'Sec-Fetch-Mode: cors' \ -H 'Sec-Fetch-Site: none' \ -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36' \ -H '___internal-request-id: a40f8884-18b0-48f5-ad52-033c6ab42014' \ -H 'content-type: application/json' \ -H 'sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"' \ -H 'sec-ch-ua-mobile: ?0' \ -H 'sec-ch-ua-platform: "macOS"' \ --data-raw $'{\n  "productId":89,\n  "quantity": 1,\n  "customerId": 1\n}' \ --compressed`

## Architecture

In this microservice-based architecture design, `discovery-server` plays an important role for registering and retrieving the service instances from a centralize location.

The `transaction-server` is responsible for maintaining transaction status for multiple services for a given transactionId.

There are three applications: `order-service`, `account-service` and `product-service`.

The application `order-service` is communicating with `account-service` and `product-service`. All these applications are using MySQL database as a backend store.

![Architecutre](./resources/distributed-txn-architecture.png)

## Distributed Transaction Flow

![Application Flow](./resources/distributed-txn-flow.png)
