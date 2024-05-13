# Development

- [Development](#development)
  - [Set Up](#set-up)
    - [Prerequisites](#prerequisites)
    - [How To Run](#how-to-run)
      - [cart](#cart)
      - [checkout](#checkout)
      - [pim](#pim)
      - [shop](#shop)

## Set Up

### Prerequisites

- Java 17

```bash
sudo apt install openjdk-17-jdk`
```

- maven

```bash
sudo apt-get -y install maven
```

- docker

### How To Run

소스 root 에서 `maven install` 로 app 빌드

```bash
mvn install
```

docker build 로 이미지 빌드

```bash
docker build -f ./docker/Dockerfile -t susukkekki/bookshop-all:240406-1633 .
```

그리고 docker-compose 명령어로 실행

```bash
docker-compose up
```

- Keycloak 관리자 콘솔 : http://keycloak:8080
  - admin / admin 으로 로그인
- Bookshop 홈 : http://keycloak:8081
  - Bookshop 사용자는 `Demo Bookshop` 에 사용자를 추가해 주어야 함.
- hosts 파일에 keycloak 도메인 IP 설정 필요

#### cart

```bash
java -jar cart/target/cart-0.0.5-SNAPSHOT.jar
```

#### checkout

```bash
cd checkout
```

```bash
yarn start --host
```

#### pim

```bash
java -jar pim/target/pim-0.0.5-SNAPSHOT-runner.jar
```

#### shop

```bash
export AUTH_SERVER_URL=http://keycloak:8080/realms/bookshop
export SHOP_CLIENT_ID=shop
export SHOP_CLIENT_SECRET=08d171bf-ba03-44db-8279-f678d68021d7
```

```bash
java -jar shop/target/shop-0.0.5-SNAPSHOT-runner.jar
```
