# Resilience4j

это популярная библиотека для обеспечения устойчивости приложений на Java, предоставляющая различные шаблоны устойчивости, такие как Circuit Breaker, Retry, Rate Limiter, Bulkhead и Time Limiter. В контексте Spring, Resilience4j интегрируется особенно хорошо, предоставляя удобные аннотации и возможности конфигурации для обеспечения устойчивости приложений.

## Что такое Rate Limiter?

Rate Limiter (ограничитель скорости) — это механизм, который контролирует количество запросов, обрабатываемых системой в заданный промежуток времени. Он помогает защитить сервисы от перегрузок, ограничивая скорость входящих запросов и обеспечивая стабильную работу приложения даже под высоким нагрузкой.

## Resilience4j Rate Limiter

Resilience4j предоставляет реализацию Rate Limiter, которая легко интегрируется со Spring Boot приложениями. Основные преимущества использования Resilience4j Rate Limiter включают:

- Легковесность
- Простота конфигурации
- Гибкость настроек
- Поддержка реактивных стримов (Reactor)

## Установка зависимости

Для начала работы с Resilience4j Rate Limiter в Spring Boot приложении, необходимо добавить соответствующие зависимости в ваш проект. Если вы используете Maven, добавьте следующие зависимости в ваш pom.xml:

```bash
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <!-- Resilience4j Spring Boot Starter -->
    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot2</artifactId>
        <version>1.7.1</version>
    </dependency>

    <!-- Аспектно-ориентированное программирование (AOP) для аннотаций -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>
```

Для Gradle используется следующая конфигурация:

```bash
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'io.github.resilience4j:resilience4j-spring-boot2:1.7.1'
    implementation 'org.springframework.boot:spring-boot-starter-aop'
}
```

## Конфигурация Rate Limiter

Resilience4j Rate Limiter можно настроить через файл application.yml или application.properties. Рассмотрим пример конфигурации в application.yml:

```bash
resilience4j.ratelimiter:
  instances:
    myRateLimiter:
      limitForPeriod: 10           # Максимальное количество запросов за период
      limitRefreshPeriod: 500ms    # Период обновления лимита
      timeoutDuration: 2s          # Максимальное время ожидания получения разрешения
```

### Пояснение параметров

- limitForPeriod: Максимальное количество разрешений, доступных в каждом периоде.
- limitRefreshPeriod: Интервал времени, через который лимит обновляется.
- timeoutDuration: Время ожидания для получения разрешения. Если разрешение не получено за это время, запрос будет отклонен.

## Применение Rate Limiter с помощью аннотаций

Resilience4j предоставляет удобные аннотации для интеграции Rate Limiter в ваши сервисы. Рассмотрим пример:

```java
import io.github.resilience4j.ratelimiter.annotation.RateLimiter;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    @RateLimiter(name = "myRateLimiter", fallbackMethod = "rateLimiterFallback")
    public String performLimitedOperation() {
        // Логика вашего метода
        return "Operation performed successfully";
    }

    public String rateLimiterFallback(Throwable t) {
        return "Rate limit exceeded. Please try again later.";
    }
}
```

### Пояснение

- @RateLimiter(name = "myRateLimiter"): Применяет Rate Limiter с именем myRateLimiter, который мы настроили ранее.
- fallbackMethod: Указывает метод, который будет вызван, если лимит превышен или происходит другая ошибка. Этот метод должен иметь ту же сигнатуру, что и оригинальный метод, с добавлением параметра Throwable.

## Программная настройка Rate Limiter

Помимо аннотаций, вы можете программно настроить Rate Limiter. Это полезно, если вам нужна более динамическая конфигурация.

```java
import io.github.resilience4j.ratelimiter.RateLimiter;
import io.github.resilience4j.ratelimiter.RateLimiterConfig;
import io.github.resilience4j.ratelimiter.RateLimiterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;

@Configuration
public class RateLimiterConfig {

    @Bean
    public RateLimiter myRateLimiter() {
        RateLimiterConfig config = RateLimiterConfig.custom()
                .limitForPeriod(10)
                .limitRefreshPeriod(Duration.ofMillis(500))
                .timeoutDuration(Duration.ofSeconds(2))
                .build();

        RateLimiterRegistry registry = RateLimiterRegistry.of(config);
        return registry.rateLimiter("myRateLimiter");
    }
}
```

### Использование программного Rate Limiter

```java
import io.github.resilience4j.ratelimiter.RateLimiter;
import io.github.resilience4j.ratelimiter.RequestNotPermitted;
import io.github.resilience4j.decorators.Decorators;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.function.Supplier;

@Service
public class MyService {

    private final RateLimiter rateLimiter;

    @Autowired
    public MyService(RateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }

    public String performLimitedOperation() {
        Supplier<String> restrictedSupplier = Decorators.ofSupplier(() -> "Operation performed successfully")
                .withRateLimiter(rateLimiter)
                .withFallback(throwable -> "Rate limit exceeded. Please try again later.")
                .decorate();

        try {
            return restrictedSupplier.get();
        } catch (RequestNotPermitted e) {
            return "Rate limit exceeded. Please try again later.";
        }
    }
}
```

## Интеграция с Spring WebFlux (Реактивное программирование)

Resilience4j также поддерживает реактивные стримы, что позволяет использовать Rate Limiter в приложениях на Spring WebFlux.

```java
import io.github.resilience4j.ratelimiter.ReactorRateLimiter;
import io.github.resilience4j.ratelimiter.RateLimiterConfig;
import io.github.resilience4j.ratelimiter.RateLimiterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

@Configuration
public class RateLimiterConfig {

    @Bean
    public RateLimiterRegistry rateLimiterRegistry() {
        RateLimiterConfig config = RateLimiterConfig.custom()
                .limitForPeriod(10)
                .limitRefreshPeriod(Duration.ofMillis(500))
                .timeoutDuration(Duration.ofSeconds(2))
                .build();
        return RateLimiterRegistry.of(config);
    }

    @Bean
    public ReactorRateLimiter reactorRateLimiter(RateLimiterRegistry registry) {
        return new ReactorRateLimiter(registry.rateLimiter("reactorRateLimiter"));
    }
}
```

Использование в контроллере:

```java
import io.github.resilience4j.ratelimiter.ReactorRateLimiter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class MyController {

    private final ReactorRateLimiter rateLimiter;

    @Autowired
    public MyController(ReactorRateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }

    @GetMapping("/limited")
    public Mono<String> limitedEndpoint() {
        return rateLimiter
                .acquirePermissionMono()
                .flatMap(permission -> {
                    if (permission) {
                        return Mono.just("Operation performed successfully");
                    } else {
                        return Mono.just("Rate limit exceeded. Please try again later.");
                    }
                });
    }
}
```

## Логирование и мониторинг

Resilience4j Rate Limiter предоставляет встроенные метрики, которые можно использовать для мониторинга состояния Rate Limiter. Интеграция с такими системами, как Micrometer, позволяет собирать и анализировать метрики.

Пример конфигурации с Micrometer:

```bash
resilience4j.ratelimiter:
  instances:
    myRateLimiter:
      limitForPeriod: 10
      limitRefreshPeriod: 500ms
      timeoutDuration: 2s

management:
  metrics:
    export:
      prometheus:
        enabled: true
```

После настройки вы можете собирать метрики, такие как resilience4j_ratelimiter_calls, resilience4j_ratelimiter_waited, и использовать их для мониторинга и алертинга.

## Заключение

Использование Resilience4j Rate Limiter в Spring приложениях позволяет эффективно управлять нагрузкой, предотвращать перегрузки и обеспечивать стабильную работу сервиса даже в условиях высокого трафика. Благодаря удобной интеграции, гибкой конфигурации и поддержке различных стилей программирования (императивного и реактивного), Resilience4j является отличным выбором для реализации механизмов ограничений скорости и других шаблонов устойчивости.

Если у вас возникнут дополнительные вопросы или потребуется помощь с конкретными аспектами использования Rate Limiter, не стесняйтесь обращаться!
