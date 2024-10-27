Для настройки Spring Cloud Contract в существующем проекте Spring Boot, нужно выполнить следующие шаги:

1. Добавить зависимости в pom.xml или build.gradle:

Для Maven:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <scope>test</scope>
</dependency>
```

Для Gradle:

```groovy
testImplementation 'org.springframework.cloud:spring-cloud-starter-contract-verifier'
```

2. Добавить плагин Spring Cloud Contract:

Для Maven:

```xml
<plugin>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-maven-plugin</artifactId>
    <version>${spring-cloud-contract.version}</version>
    <extensions>true</extensions>
    <configuration>
        <baseClassForTests>com.example.BaseTestClass</baseClassForTests>
    </configuration>
</plugin>
```

Для Gradle:

```groovy
plugins {
    id 'org.springframework.cloud.contract' version '3.1.3'
}
```

3. Создать базовый тестовый класс:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public abstract class BaseTestClass {
    
    @Autowired
    private WebApplicationContext context;
    
    @BeforeEach
    public void setup() {
        RestAssuredMockMvc.webAppContextSetup(context);
    }
}
```

4. Создать контракты в директории src/test/resources/contracts:

```groovy
Contract.make {
    request {
        method 'GET'
        url '/api/users/1'
    }
    response {
        status 200
        body([
            id: 1,
            name: 'John Doe'
        ])
        headers {
            contentType('application/json')
        }
    }
}
```

5. Запустить генерацию тестов:

Для Maven: `mvn clean install`
Для Gradle: `./gradlew clean build`

6. Реализовать функциональность, описанную в контрактах.

7. Для использования стабов в тестах потребителя, добавьте зависимость:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
    <scope>test</scope>
</dependency>
```

И используйте аннотацию @AutoConfigureStubRunner в тестах:

```java
@SpringBootTest
@AutoConfigureStubRunner(
    ids = "com.example:your-service:+:stubs:8080",
    stubsMode = StubRunnerProperties.StubsMode.LOCAL
)
public class ConsumerTest {
    // Тесты
}
```

Это базовая настройка Spring Cloud Contract для существующего проекта Spring Boot. Дальнейшая конфигурация может потребоваться в зависимости от специфики вашего проекта.

Citations:
[1] https://habr.com/ru/articles/764402/
[2] https://spring.io/projects/spring-cloud-contract/
[3] https://www.balynsky.com/ac-backup-project-config-server/
[4] https://docs.spring.vmware.com/spring-cloud-contract/docs/4.0.6/reference/html/project-features.html
[5] https://www.j-labs.pl/blog-technologiczny/spring-cloud-contract/
[6] https://ru.stackoverflow.com/questions/981822/spring-cloud-contract-producer-%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B8%D1%80%D1%83%D0%B5%D1%82-%D1%82%D0%B5%D1%81%D1%82-%D0%BF%D0%BE-%D0%BA%D0%BE%D0%BD%D1%82%D1%80%D0%B0%D0%BA%D1%82%D1%83-%D0%B8-%D0%B2-%D0%BA%D0%BE%D0%BD%D1%86%D0%B5-%D1%81%D1%82%D0%B0%D0%B2%D0%B8%D1%82-nul
[7] https://blog.mimacom.com/spring-cloud-contract/
[8] https://habr.com/ru/articles/712628/