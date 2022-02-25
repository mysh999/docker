```bash
# cat docker-compose.yml 
version: '2'
services:
  rabbitmq:
    container_name: rabbitmq
    image: rabbitmq:3.9.11-management
    restart: always
    hostname: rabbit-robog
    environment:
      - TZ=Asia/Shanghai
      - RABBITMQ_NODENAME=rabbit-robog-01
    volumes:
      - ./rabbitmq_data:/var/lib/rabbitmq
    ports:
      - 15672:15672
      - 5672:5672
    network_mode: host
```

