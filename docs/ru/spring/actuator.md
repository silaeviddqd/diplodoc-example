# Actuator

Чтобы обеспечить доступность Actuator «отовсюду» в микросервисном проекте, необходимо правильно настроить доступ к эндпоинтам Actuator, обеспечить безопасность и интеграцию с другими компонентами архитектуры, такими как API Gateway. Ниже приведены шаги и рекомендации для достижения этой цели.

## 1. Настройка Actuator в микросервисах

### a. Добавление зависимости Actuator

Убедитесь, что в каждом микросервисе добавлена зависимость Spring Actuator. В pom.xml для Maven:

```bash
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Для Gradle:

```bash
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

#### b. Конфигурация application.properties или application.yml

Настройте, какие эндпоинты необходимо открыть и какие протоколы используются.

Пример для application.yml:

```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"  # Открывает все эндпоинты Actuator
  endpoint:
    health:
      show-details: always  # Показывать подробности статуса здоровья
```

Предупреждение: Открытие всех эндпоинтов (include: "*") может представлять риск безопасности. Рекомендуется открывать только необходимые эндпоинты, например:

```yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

### 2. Интеграция с API Gateway

Для обеспечения централизованного доступа к Actuator эндпоинтам через API Gateway выполните следующие шаги:

#### a. Настройка маршрутизации в API Gateway

Добавьте маршруты для доступа к Actuator эндпоинтам каждого микросервиса.

Пример для Spring Cloud Gateway (application.yml):

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: service-a-actuator
          uri: <http://service-a:8080>
          predicates:
            - Path=/service-a/actuator/**
          filters:
            - RewritePath=/service-a/actuator/(?<segment>.*), /actuator/${segment}

        - id: service-b-actuator
          uri: http://service-b:8080
          predicates:
            - Path=/service-b/actuator/**
          filters:
            - RewritePath=/service-b/actuator/(?<segment>.*), /actuator/${segment}
```

В этом примере API Gateway перенаправляет запросы вида /service-a/actuator/* к соответствующим эндпоинтам микросервиса service-a.

#### b. Безопасность маршрутов Actuator через API Gateway

Чтобы защитить доступ к Actuator эндпоинтам, рекомендуется аутентификация и авторизация на уровне API Gateway.

Пример с использованием Basic Authentication:

```yml
spring:
  security:
    user:
      name: actuatorUser
      password: securePassword
  cloud:
    gateway:
      routes:
        - id: service-a-actuator
          uri: <http://service-a:8080>
          predicates:
            - Path=/service-a/actuator/**
          filters:
            - RewritePath=/service-a/actuator/(?<segment>.*), /actuator/${segment}
            - AddRequestHeader=Authorization, Basic YWN0dWF0b3JVc2VyOnNlY3VyZVBhc3N3b3Jk
```

Примечание: В реальных проектах рекомендуется использовать более безопасные методы аутентификации, такие как OAuth2 или JWT.

### 3. Обеспечение безопасности Actuator эндпоинтов

Так как Actuator предоставляет доступ к внутреннему состоянию приложения, крайне важно обеспечить их безопасность.

#### a. Ограничение доступа через Spring Security

Настройте доступ к Actuator эндпоинтам только для авторизованных пользователей.

Пример конфигурации безопасности (SecurityConfig.java):

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/actuator/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .httpBasic();  // Или другой метод аутентификации
    }
}
```

#### b. Использование HTTPS

Убедитесь, что все доступы к Actuator эндпоинтам происходят по защищённому протоколу HTTPS, чтобы предотвратить перехват данных.

### 4. Централизованный мониторинг и управление

Для удобства мониторинга и управления всеми микросервисами можно использовать инструменты, такие как Spring Boot Admin.

#### a. Добавление Spring Boot Admin

Добавьте зависимость Spring Boot Admin в свой проект мониторинга:

```bash
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.5.6</version>
</dependency>
```

#### b. Конфигурация микросервисов для регистрации

В каждом микросервисе добавьте зависимость клиента Spring Boot Admin и настройте регистрацию:

```bash
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.5.6</version>
</dependency>
```

Пример конфигурации application.yml:

```yml
spring:
  application:
    name: service-a
  boot:
    admin:
      client:
        url: <http://admin-server:8080>
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 5. Дополнительные рекомендации

#### a. Использование Service Mesh

Интеграция с Service Mesh, такой как Istio или Linkerd, может облегчить управление трафиком, безопасность и мониторинг между микросервисами.

#### b. Логирование и Трассировка

Используйте распределённые системы логирования и трассировки, такие как ELK Stack (Elasticsearch, Logstash, Kibana) или Prometheus и Grafana, для мониторинга состояния микросервисов через Actuator.

#### c. Ограничение Скорости и Защита от DDoS

Рассмотрите возможность добавления ограничений на количество запросов к Actuator эндпоинтам, чтобы предотвратить возможные атаки.

### Заключение

Обеспечение доступности Actuator эндпоинтов «отовсюду» в микросервисной архитектуре требует сбалансированного подхода между доступностью и безопасностью. Важно тщательно настроить маршрутизацию через API Gateway, обеспечить надёжную аутентификацию и авторизацию, а также использовать инструменты для централизованного мониторинга и управления. Следуя приведённым рекомендациям, вы сможете эффективно и безопасно сделать Actuator доступным для всех необходимых компонентов вашего микросервисного проекта.
