# CORS

Cross-Origin Resource Sharing и как реализовать его в API Gateway, используя библиотеку Spring Cloud Gateway (часть экосистемы Spring Boot).

---

## Что такое CORS

CORS (Cross-Origin Resource Sharing) — это механизм безопасности, реализованный в браузерах, который позволяет контролировать доступ веб-приложений к ресурсам с другого домена. По умолчанию, браузеры ограничивают веб-страницы в доступе к ресурсам, находящимся на другом домене, порте или протоколе. CORS предоставляет способ разрешить легальный cross-origin доступ через настройку соответствующих HTTP-заголовков.

### Основные понятия

- Origin (Происхождение): Комбинация протокола, домена и порта (например, <https://example.com:8080>).
- Preflight-запрос: HTTP-запрос методом OPTIONS, отправляемый браузером для проверки разрешений перед выполнением основного запроса.
- Simple-запросы: Запросы, которые соответствуют определенным критериям и не требуют preflight-запроса.
- Credentials (Учетные данные): Куки, HTTP-авторизация и другие данные, передаваемые вместе с запросом.

## Почему нужен API Gateway

API Gateway выступает как единая точка входа для всех клиентских запросов к микросервисной архитектуре. Он:

- Управляет маршрутизацией запросов к соответствующим микросервисам.
- Обеспечивает безопасность и аутентификацию.
- Реализует кросс-функциональные задачи, такие как логирование, мониторинг, ограничение частоты запросов и, конечно, настройку CORS.

Реализация CORS на уровне API Gateway позволяет централизованно управлять политиками доступа, что упрощает администрирование и повышает безопасность.

## Реализация CORS в Spring Cloud Gateway

Spring Cloud Gateway предоставляет гибкие способы конфигурации CORS как глобально, так и для отдельных маршрутов.

### 1. Глобальная конфигурация CORS

Глобальная настройка CORS применяется ко всем маршрутам в API Gateway. Это удобно, если все микросервисы имеют одинаковые требования к CORS.

#### Шаги

1. Добавьте зависимости в ваш pom.xml или build.gradle:

    <!-- Для Maven -->
    ```bash
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    ```

    <!-- Для Gradle -->
    ```bash
    dependencies {
        implementation 'org.springframework.cloud:spring-cloud-starter-gateway'
    }
    ```

2. Настройте CORS в application.yml:

    ```yml
    spring:
      cloud:
        gateway:
          globalcors:
            corsConfigurations:
              '[/**]':
                allowedOrigins: "<http://localhost:3000>, <https://yourdomain.com>"
                allowedMethods:
                  - GET
                  - POST
                  - PUT
                  - DELETE
                  - OPTIONS
                allowedHeaders: "*"
                allowCredentials: true
                maxAge: 3600
    ```

    {% note info "Пояснения" %}

    - allowedOrigins: Список доменов, которым разрешен доступ. Можно использовать "*" для всех доменов, но это не рекомендуется при включенных allowCredentials.
    - allowedMethods: Разрешенные HTTP-методы.
    - allowedHeaders: Разрешенные заголовки.
    - allowCredentials: Разрешить передачу учетных данных (куки, авторизационные заголовки).
    - maxAge: Время в секундах, в течение которого браузер будет использовать кэшированные preflight-запросы.

    {% endnote %}

### 2. Конфигурация CORS для отдельных маршрутов

Иногда требуется применять разные CORS-политики для разных маршрутов. В этом случае настройка производится на уровне конкретных маршрутов.

#### Шаги

1. Настройте CORS в application.yml для отдельных маршрутов:

    ```yml
    spring:
      cloud:
        gateway:
          routes:
            - id: service1
              uri: <http://localhost:8081>
              predicates:
                - Path=/service1/**
              filters:
                - RemoveRequestHeader=Cookie
                - AddResponseHeader=Access-Control-Allow-Origin, <http://localhost:3000>
                - AddResponseHeader=Access-Control-Allow-Methods, GET,POST,PUT,DELETE,OPTIONS
                - AddResponseHeader=Access-Control-Allow-Headers, *
                - AddResponseHeader=Access-Control-Allow-Credentials, true
            - id: service2
              uri: <http://localhost:8082>
              predicates:
                - Path=/service2/**
              filters:
                - RemoveRequestHeader=Cookie
                - AddResponseHeader=Access-Control-Allow-Origin, <https://anotherdomain.com>
                - AddResponseHeader=Access-Control-Allow-Methods, GET,POST
                - AddResponseHeader=Access-Control-Allow-Headers, Content-Type
                - AddResponseHeader=Access-Control-Allow-Credentials, false
    ```

    {% note info %}

    Примечание: Использование фильтров AddResponseHeader вручную для CORS может быть менее гибким и требовать дополнительной логики для обработки preflight-запросов. Лучше использовать встроенные возможности Spring Cloud Gateway для CORS.

    {% endnote %}

2. Лучший подход — использовать глобальную конфигурацию с роутами, если это возможно.

## Пример реализации

Рассмотрим пример настройки глобального CORS в Spring Cloud Gateway.

### Шаг 1: Создание проекта Spring Boot

Создайте новый проект Spring Boot с зависимостью Spring Cloud Gateway. Можно использовать Spring Initializr (<https://start.spring.io/>) для генерации проекта.

### Шаг 2: Добавление зависимостей

В pom.xml добавьте зависимость для Spring Cloud Gateway и укажите версию Spring Cloud:

```bash
<dependencies>
    <!-- Spring Cloud Gateway -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <!-- Другие зависимости -->
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR12</version> <!-- Используйте актуальную версию -->
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### Шаг 3: Настройка CORS в application.yml

```yml
spring:
  cloud:
    gateway:
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "<http://localhost:3000>, <https://yourdomain.com>"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
              - OPTIONS
            allowedHeaders: "*"
            allowCredentials: true
            maxAge: 3600
      routes:
        - id: service1
          uri: <http://localhost:8081>
          predicates:
            - Path=/service1/**
        - id: service2
          uri: <http://localhost:8082>
          predicates:
            - Path=/service2/**
```

Этот конфиг применит CORS к всем маршрутам, разрешая запросы из <http://localhost:3000> и <https://yourdomain.com>.

### Шаг 4: Запуск и проверка

Запустите ваш API Gateway и убедитесь, что настройки CORS применяются корректно. Попробуйте сделать запрос с клиента (например, из фронтенда на <http://localhost:3000>) к вашему API Gateway и проверьте наличие необходимых заголовков в ответе.

## Дополнительные рекомендации

1. Используйте точные значения доменов: Избегайте использования "*" для allowedOrigins, особенно если включены allowCredentials, чтобы избежать потенциальных уязвимостей.

2. Ограничьте методы и заголовки: Разрешайте только необходимые методы и заголовки для ваших API. Это улучшит безопасность.

3. Интеграция с Spring Security: Если вы используете Spring Security, убедитесь, что конфигурация CORS интегрирована с его настройками. Spring Security может переопределять настройки CORS, поэтому важно правильно настроить обе части.

    Пример конфигурации Spring Security с CORS:

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.security.config.web.server.ServerHttpSecurity;
    import org.springframework.security.web.server.SecurityWebFilterChain;

    @Configuration
    public class SecurityConfig {

        @Bean
        public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
            http
                .csrf().disable()
                .cors().and()
                .authorizeExchange()
                .anyExchange().permitAll();
            return http.build();
        }
    }
    ```

4. Логирование и мониторинг: Включите логирование CORS-запросов для упрощения отладки и мониторинга.

5. Предотвращение CSRF: Хотя CORS не защищает от CSRF (Cross-Site Request Forgery), убедитесь, что ваши приложения также защищены от таких атак.

## Тестирование CORS конфигурации

1. Использование браузера:

    - Создайте простой клиент (например, с помощью fetch API) и попробуйте сделать запрос к вашему API Gateway.
    - Откройте Developer Tools в браузере и проверьте раздел Network на наличие заголовков CORS в ответах.

    ```js
    fetch('<http://your-api-gateway.com/service1/data>', {
        method: 'GET',
        credentials: 'include',
    })
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));
    ```

2. Использование cURL:

    - Preflight-запрос:

        ```bash
        curl -X OPTIONS http://localhost:8080/service1/data \
          -H "Access-Control-Request-Method: POST" \
          -H "Origin: http://localhost:3000"
        ```

        Ожидаемый ответ:

        Заголовки Access-Control-Allow-Origin, Access-Control-Allow-Methods и другие должны присутствовать в ответе.

    - Основной запрос:

        ```bash
            curl -X GET http://localhost:8080/service1/data \
          -H "Origin: http://localhost:3000"
        ```

3. Использование инструментов:

    - Postman: Хотя Postman не выполняет политику CORS, он может помочь проверить, правильно ли настроены заголовки ответа.
    - Online CORS Testers: Существуют онлайн-инструменты для проверки CORS, такие как Test CORS (<https://www.test-cors.org/>).

## Заключение

Настройка CORS в API Gateway на базе Spring Cloud Gateway обеспечивает централизованный контроль доступа к вашим микросервисам из различных источников. Это повышает безопасность приложения и упрощает управление политиками доступа. Важно тщательно продумать, какие источники и методы вы разрешаете, чтобы обеспечить баланс между удобством использования и безопасностью.

## Ссылки

- Spring Cloud Gateway Documentation (<https://spring.io/projects/spring-cloud-gateway>)
- Spring Framework Documentation - CORS (<https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#webmvc-cors>)
- MDN Web Docs - CORS (<https://developer.mozilla.org/ru/docs/Web/HTTP/CORS>)
- Spring Security CORS Documentation (<https://docs.spring.io/spring-security/reference/servlet/integration/cors.html>)

