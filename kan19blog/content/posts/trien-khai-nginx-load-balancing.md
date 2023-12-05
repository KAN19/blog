---
title: "Nghịch ngợm triển khai load balancing với Nginx"
date: 2023-12-05T22:53:44+07:00
---

Hôm nay mình sẽ thử triển khai load balancing sử dụng Nginx nhé. Các kiến thức cơ bản về Load balancing, trông nó ra sao, hoạt động thế nào, mình đã có một bài viết ở đây: ***[Load Balancing là gì? Có ăn được hong?](https://theengineerintern.dev/posts/basic-load-balancing/)***

## 1. Load balancing là gì

Load balancing là việc phân phối lưu lượng truy cập đến một nhóm các backend servers.

Module thực hiện việc load balancing gọi là load balancer. Load balancer sẽ đứng trước server và routing các request của client đến các server có khả năng thực hiện các request đó, đảm bảo không có server nào nhận quá nhiều request.

## 2. Hello Nginx!

Nginx đơn giản là một phần mềm mã nguồn mở hỗ trợ chúng ta trong việc xây dựng web server. Nginx ngày càng phổ biến nhờ vào tính ổn định, hiệu suất và khả năng tối ưu hoá resource. 

> *Nginx ban đầu được tạo ra để giải quyết vấn đề C10k, một thuật ngử được đặt ra vào năm 1999 để mô tả khó khăn mà các web server tại thời điểm đó khi phải xử lý số lượng lớn (10K) kết nối đồng thời*
> 

Ngoài việc làm web server, Nginx còn có thể làm nhiều “trò” khác như: reverse proxy, TLS/SSL termination, caching,… và load balancing mà chúng ta sẽ thực hiện trong bài viết này.


![statistic-nginx.png](/nginx-load-balancing/statistic-nginx.png)

Giới thiệu thế đủ rồi. Giờ mình tay dô làm nhé

## 3. Sơ đồ triển khai

![trien-khai.png](/nginx-load-balancing/trien-khai.png)

Mình sẽ sử dụng docker để triển khai nginx trên máy local. Bên cạnh đó, mình tạo ra thêm 3 instance của demo service, mỗi instance này sẽ có port khác nhau

## 4. Triển khai step by step

### Config app serivce

Đầu tiên chúng ta sẽ tạo ứng dụng Spring boot làm vai trò application service cho sơ đồ trên. 

![spring-initializr.png](/nginx-load-balancing/spring-initializr.png)

Chúng ta chỉ cần dùng Spring Web Dependencies là đủ rồi. 

Tiếp đến, chúng ta tạo một controller và implement một GetMapping method: 

```java
package com.example.demoservice.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {
   @Value("${server.port:unknown}")
    String portId;

    @GetMapping("/hello")
    public String hello() {
        return String.format("Hello from instance %s", portId);
    }
}
```

Vậy thui là đủ rồi. Chúng ta tiếp tục đến với việc config nginx. 

### Config Nginx

Đầu tiên chúng ta tạo 1 file config cho nginx. 

```
#File name: nginx.conf

upstream demo-service {
    server host.docker.internal:4001;
    server host.docker.internal:4002;
    server host.docker.internal:4003;
}
server {
  server_name default localhost;
  listen    80;

  location / {
    proxy_pass  http://demo-service;
  }
}
```

Ở đây trong `demo-service` , mình thể dùng “[localhost](http://localhost):xxx” được, vì mình triển khai nginx trên docker container, còn các service kia thì chạy trực tiếp trên máy local. Bản thân container muốn giao tiếp với host của nó thì phải dùng host.docker.internal. Trước đây mình có từng đau đầu mấy buổi liền về vấn đề này :((((( Kỷ niệm khó quên lúc mới dô công ty luôn. 
Khi config upstream như trên, nginx sẽ sử dụng thuật toán balancing default là Round Robin. 

Dưới đây là một vài config khác để cài đặt các thuật toán load balancing: 

- Weighted Round Robin

```
upstream demo-service {
    server host.docker.internal:4001 weight=1;
    server host.docker.internal:4002 weight=2;
    server host.docker.internal:4003 weight=3;
}
```

- Least connection

```
upstream demo-service {
		least_conn;
    server host.docker.internal:4001;
    server host.docker.internal:4002;
    server host.docker.internal:4003;
}
```

- IP Hash

```
upstream demo-service {
		ip_hash;
    server host.docker.internal:4001;
    server host.docker.internal:4002;
    server host.docker.internal:4003;
}
```

- Least time (Nginx plus only)

```
upstream demo-service {
		least_time header;
    server host.docker.internal:4001;
    server host.docker.internal:4002;
    server host.docker.internal:4003;
}
```

- Random

```
upstream demo-service {
		random two least_conn
    server host.docker.internal:4001;
    server host.docker.internal:4002;
    server host.docker.internal:4003;
}
```

Tiếp đến mình sẽ sử dụng 1 docker compose file. Lưu ý là file docker-compose này phải để cùng thư mục với file `nginx.conf`  nhé

```docker
version: '3.7'

# Define services
services:
  proxy:
    container_name: proxy-server
    image: nginx
    volumes:
      - ./:/etc/nginx/conf.d
    ports:
      - "8080:80"
```

Mọi thứ đã gần xong. Giờ cùng khởi động nó lên thôi. 

### Run app thui

Đầu tiên ta di chuyển đến thư mục demo-service và chạy lệnh dưới đây để start services lên.

```
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Dserver.port=4001"
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Dserver.port=4002"
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Dserver.port=4003"
```

Sau đó mình di chuyển đến thư mục chứa docker compose file rồi chạy `docker-compose up`

### Test app

Mình sử dụng 1 app js nhỏ để request. Thiệt ra chúng ta có thể đơn giản hơn bằng việc sử dụng curl chạy trực tiếp trên command. Nhưng không hiểu sao khi mình thêm loop vào cái curl đó thì nó xịt keo tịt đứng cứng ngắc lun

```js
// clientAPI.js
const http = require('http');

const getMessage = () => {

    const request = http.get('http://localhost:8080/hello', (response) => {
        let data = '';
        response.on('data', (chunk) => {
            data += chunk;
        });
        response.on('end', () => {
            console.log(data);
        });
    });
    request.on('error', (error) => {
        console.log('An error', error);
    });

    request.end();
}

setInterval(getMessage, 1000);
```

Sau đó gọi node clientAPI.js và thu được kết quả ở dưới. (Sử dụng Round robin)

![result-round-robin.png](/nginx-load-balancing/result-round-robin.png)

Còn đây là kết quả khi sử dụng weighted rounded robin 

![result-weighted-round-robin.png](/nginx-load-balancing/result-weighted-round-robin.png)

Đối với các thuật toán khác, mời các bạn chạy thử và xem kết quả nha!!