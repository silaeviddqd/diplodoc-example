# Micrometer

Это современная библиотека для измерения и мониторинга метрик в приложениях на Java. Она предоставляет унифицированный API для сбора метрик и их отправки в различные системы мониторинга, такие как Prometheus, Grafana, Datadog, New Relic и другие. Micrometer особенно хорошо интегрируется с Spring Boot, что делает его популярным выбором для разработки микросервисов и облачных приложений.

## Основные Возможности Micrometer

1. Унифицированный API для метрик:
   - Предоставляет стандартизированные способы сбора различных типов метрик (счетчики, таймеры, гистограммы и т.д.).

2. Поддержка множества мониторинговых систем:
   - Легко интегрируется с различными бекендами мониторинга через регистры (например, Prometheus, Datadog, Graphite, InfluxDB и другие).

3. Легкая интеграция с Spring Boot:
   - Автоматическая конфигурация и поддержка Spring Actuator для экспонирования метрик.

4. Многообразие типов метрик:
   - Счетчики (Counters)
   - Таймеры (Timers)
   - Гистограммы (Distribution Summaries)
   - Дистрибутивные графики (Distribution Statistics)

5. Настраиваемость и расширяемость:
   - Возможность создания собственных метрик и расширения функционала через плагины.

## Установка и Настройка Micrometer в Spring Boot Приложении

### 1. Добавление Зависимостей

Для начала необходимо добавить зависимости Micrometer и нужного модуля мониторинга в ваш проект. Рассмотрим пример с Prometheus.

Если вы используете Maven, добавьте следующие зависимости в ваш pom.xml:

```bash
<dependencies>
    <!-- Micrometer Core -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-core</artifactId>
    </dependency>

    <!-- Micrometer Prometheus Registry -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>

    <!-- Spring Boot Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

Для Gradle используйте следующую конфигурацию:

```bash
dependencies {
    implementation 'io.micrometer:micrometer-core'
    implementation 'io.micrometer:micrometer-registry-prometheus'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

### 2. Конфигурация Приложения

В файле application.properties или application.yml необходимо настроить экспонирование метрик и подключение к выбранному регистру.

Пример для application.yml с Prometheus:

```bash
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

Это позволит активировать эндпоинт /actuator/prometheus, который будет предоставлять метрики в формате, совместимом с Prometheus.

### 3. Использование Метрик в Коде

Micrometer предоставляет удобные способы для добавления метрик в ваше приложение.

Пример использования счетчика (Counter):

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final Counter userCount;

    public UserService(MeterRegistry registry) {
        this.userCount = Counter.builder("users.registered")
                                 .description("Количество зарегистрированных пользователей")
                                 .register(registry);
    }

    public void registerUser(User user) {
        // Логика регистрации пользователя
        userCount.increment();
    }
}
```

Пример использования таймера (Timer):

```java
import io.micrometer.core.annotation.Timed;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    @Timed(value = "orders.processing.time", description = "Время обработки заказа")
    public void processOrder(Order order) {
        // Логика обработки заказа
    }
}
```

## Интеграция с Различными Системами Мониторинга

Micrometer поддерживает множество систем мониторинга. Рассмотрим несколько примеров:

### 1. Prometheus

Prometheus является одним из самых популярных решений для мониторинга и алертинга.

Настройка Prometheus:

После настройки вашего приложения с Micrometer и эндпоинтом /actuator/prometheus, необходимо настроить ваш prometheus.yml следующим образом:

```yml
scrape_configs:

- job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
  - targets: ['localhost:8080']
```

### 2. Grafana

Grafana может использовать Prometheus в качестве источника данных для визуализации метрик.

Настройка Grafana:

1. Добавьте Prometheus как Data Source в Grafana.
2. Создайте дашборды, используя доступные метрики.

### 3. Datadog

Если вы используете Datadog, вы можете добавить зависимости Micrometer для Datadog Registry.

Добавление Зависимости:

```bash
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-datadog</artifactId>
</dependency>

Конфигурация:

management:
  metrics:
    export:
      datadog:
        api-key: YOUR_DATADOG_API_KEY
        enabled: true
```

## Продвинутые Возможности Micrometer

### 1. Создание Кастомных Метрик

Вы можете создавать собственные метрики для более точного мониторинга специфичных аспектов вашего приложения.

Пример создания гистограммы:

```java
import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class InventoryService {

    private final DistributionSummary inventorySummary;

    public InventoryService(MeterRegistry registry) {
        this.inventorySummary = DistributionSummary.builder("inventory.items")
                                                  .description("Количество элементов в инвентаре")
                                                  .baseUnit("items")
                                                  .register(registry);
    }

    public void addItem(Item item) {
        // Логика добавления элемента
        inventorySummary.record(item.getQuantity());
    }
}
```

### 2. Сбор Метрик из Spring Actuator

Spring Boot Actuator автоматически собирает множество стандартных метрик, таких как показатели JVM, HTTP запросов, состояния кэшей и т.д.

Пример доступа к метрикам:

Откройте эндпоинт /actuator/metrics для просмотра доступных метрик и /actuator/metrics/{metric.name} для получения деталей конкретной метрики.

### 3. Метки (Tags) для Метрик

Метки позволяют классифицировать метрики по различным параметрам, что упрощает их фильтрацию и анализ.

Пример с метками:

```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Service;

@Service
public class PaymentService {

    private final Timer paymentTimer;

    public PaymentService(MeterRegistry registry) {
        this.paymentTimer = Timer.builder("payments.process.time")
                                 .tag("paymentType", "credit_card")
                                 .description("Время обработки платежей")
                                 .register(registry);
    }

    public void processPayment(Payment payment) {
        paymentTimer.record(() -> {
            // Логика обработки платежа
        });
    }
}
```

## Преимущества Использования Micrometer

1. Универсальность:
   - Позволяет использовать различные системы мониторинга без изменения кода приложения.

2. Гибкость:
   - Легко добавлять и настраивать новые метрики согласно требованиям проекта.

3. Интеграция:
   - Глубокая интеграция с Spring Boot и Spring Cloud, что упрощает настройку и использование.

4. Поддержка Сообщества:
   - Активное развитие и поддержка сообщества, регулярные обновления и улучшения.

5. Производительность:
   - Низкое влияние на производительность приложения благодаря эффективному сбору и экспонированию метрик.

## Заключение

Micrometer — мощный инструмент для мониторинга и сбора метрик в Java-приложениях. Его гибкость, универсальность и легкая интеграция с популярными фреймворками и системами мониторинга делают его отличным выбором для современных приложений. Используя Micrometer, вы сможете получать глубокое понимание работы вашего приложения, выявлять узкие места и обеспечивать высокое качество обслуживания пользователей.

Если у вас возникнут дополнительные вопросы или потребуется помощь в настройке Micrometer для конкретных сценариев, не стесняйтесь обращаться!
