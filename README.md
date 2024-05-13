# Development

- [Development](#development)
  - [Set Up](#set-up)
    - [Prerequisites](#prerequisites)
    - [How To Run](#how-to-run)
      - [Keycloak](#keycloak)
      - [Clean Keycloak Restart](#clean-keycloak-restart)
      - [cart](#cart)
      - [checkout](#checkout)
      - [pim](#pim)
      - [shop](#shop)
  - [인증 방식](#인증-방식)
    - [오리지날 with pkce](#오리지날-with-pkce)
    - [오리지날 without pkce](#오리지날-without-pkce)
    - [(x) 기존 shop clientId 를 사용](#x-기존-shop-clientid-를-사용)

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

#### Keycloak

```bash
docker-compose up keycloak
```

#### Clean Keycloak Restart

Keycloak 를 깔끔하게 재시작하기 위해서는 다음과 같이 하면 된다.

keycloak 를 실행시킨 상태에서, `docker ps | grep key` 로 keycloak 의 id 확인 후 다음 명령을 실행시켜 주면 된다.

```bash
docker rm -f {container_id}
```

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

## 인증 방식

### 오리지날 with pkce

checkout web app 이 아래 설정으로 인증을 시도하여, 다시 로그인이 필요함. 시나리오는 아래와 같다.

1. keycloak:8081 에서 로그인
2. cart 에 물건 추가하고, checkout 페이지로 이동
3. keycloak 로그인 화면에서 다시 로그인
4. checkout 항목 확인

```json
{
    "realm": "bookshop",
    "auth-server-url": "http://localhost:8080/",
    "resource": "checkout",
    "public-client": true
}
```

그리고 `checkout/src/index.jsx`에서 다음과 같이 작성함

```javascript
const kc = new Keycloak('/keycloak.json');

// ... 중략 ...

kc.init({
    onLoad: 'login-required',
    useNonce: true,
    pkceMethod: 'S256',
    enableLogging: true,
    acrValues: document.cookie.split('; ').find((e) => e.startsWith('acr_values='))?.split('=')[1],
})
    .then((authenticated) => {
        if (!authenticated) {
            console.log("user is not authenticated..!");
        } else {
            createRoot(document.getElementById("app")).render(<Checkout kc={kc}/>)
        }
    })
    .catch(console.error);
```

### 오리지날 without pkce

코드를 다음과 같이 수정하여 checkout 실행

```javascript
const kc = new Keycloak({
    url: 'http://keycloak:8080/',
    realm: 'bookshop',
    clientId: 'checkout'
});

// ... 중략 ...

kc.init({
    onLoad: 'login-required',
    // useNonce: true,
    // // pkceMethod: 'S256',
    // enableLogging: true,
    // acrValues: document.cookie.split('; ').find((e) => e.startsWith('acr_values='))?.split('=')[1],
})
    .then((authenticated) => {
        if (!authenticated) {
            console.log("user is not authenticated..!");
        } else {
            console.log("user is authenticated..!");
            createRoot(document.getElementById("app")).render(<Checkout kc={kc}/>)
        }
    })
    .catch(console.error);
```

Keycloak Admin Console 에서 checkout client 에 다음을 추가해 주어야 한다.

- Access settings
  - Valid redirect URIs : `http://keycloak:3000/*`

시나리오는 다음과 같이 변경된다.

1. keycloak:8081 에서 로그인
2. cart 에 물건 추가하고, checkout 페이지로 이동
3. ~~keycloak 로그인 화면에서 다시 로그인~~ 로그인이 자동으로 수행됨
4. checkout 항목 확인

### (x) 기존 shop clientId 를 사용

코드를 다음과 같이 수정하여 checkout 실행

```javascript
const kc = new Keycloak({
    url: 'http://keycloak:8080/',
    realm: 'bookshop',
    clientId: 'shop'
});

// ... 중략 ...

kc.init({
    onLoad: 'check-sso',
    // useNonce: true,
    // // pkceMethod: 'S256',
    // enableLogging: true,
    // acrValues: document.cookie.split('; ').find((e) => e.startsWith('acr_values='))?.split('=')[1],
})
    .then((authenticated) => {
        if (!authenticated) {
            console.log("user is not authenticated..!");
        } else {
            console.log("user is authenticated..!");
            createRoot(document.getElementById("app")).render(<Checkout kc={kc}/>)
        }
    })
    .catch(console.error);
```

Keycloak Admin Console 에서 shop client 에 다음을 추가해 주어야 한다.

- Access settings
  - Valid redirect URIs : `http://keycloak:3000/*`

다음과 같은 문제로 안된다.

```bash
keycloak-1  | 2024-05-13 07:56:25,489 WARN  [org.keycloak.events] (executor-thread-17) type="CODE_TO_TOKEN_ERROR", realmId="bookshop", clientId="shop", userId="null", ipAddress="172.29.80.1", error="invalid_client_credentials", grant_type="authorization_code"
```

http://keycloak:8080/realms/bookshop/protocol/openid-connect/token 를 호출할 때 401 에러가 발생한다.

```json
{
    "error": "unauthorized_client",
    "error_description": "Invalid client or Invalid client credentials"
}
```

이상하다, 이건 그냥 기존 토큰을 사용하면 되는거 아닌가?

안되는 것 같다.
