# Занятие 11–12. Документирование, Docker, деплой и защита проекта

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Итоги](Lesson12.md)

## План
1. [Введение: финишная прямая](#введение)
2. [Что такое OpenAPI и Swagger](#что-такое-openapi)
3. [Springdoc-openapi: автодокументация](#springdoc-openapi)
4. [Аннотации для описания API](#аннотации-api)
5. [Группировка endpoints и безопасность в Swagger](#группировка-и-безопасность)
6. [Postman-коллекция](#postman-коллекция)
7. [Что такое Docker](#что-такое-docker)
8. [Dockerfile для Spring Boot](#dockerfile)
9. [Docker Compose: приложение + PostgreSQL](#docker-compose)
10. [CI в Gogs](#ci-gogs)
11. [Финальный README.md](#финальный-readme)
12. [Чек-лист проекта](#чек-лист-проекта)
13. [Подготовка к защите](#подготовка-к-защите)
14. [Типичные вопросы на защите](#типичные-вопросы)
15. [Финальное задание](#финальное-задание)

---

## Введение

Мы прошли **десять занятий** и построили полноценное веб-приложение:
- Каталог, карточка, корзина, оформление заказов, «Мои заказы».
- Админ-панель с дашбордом, CRUD, управлением заказами.
- REST API для внешних клиентов.
- Тесты, логи, файловое хранилище, кэширование, отчёты, экспорт.

**Осталось два больших дела:**
1. **Документирование и деплой** — чтобы проект могли запустить и использовать другие люди.
2. **Финализация и защита** — чтобы вы могли показать проект и объяснить код.

Это занятие объединяет **11 и 12** — потому что они логически связаны: сначала приводим проект в порядок (документация, Docker), потом готовимся к защите.

---

## Что такое OpenAPI и Swagger

> 💡 **OpenAPI** — это стандарт описания REST API в формате JSON или YAML. Он описывает:
> - Какие endpoints есть.
> - Какие параметры они принимают.
> - Какие ответы возвращают.
> - Какие схемы данных используются.

> 💡 **Swagger UI** — это веб-интерфейс, который читает OpenAPI-спецификацию и показывает её как интерактивную документацию. Можно нажимать «Try it out» и отправлять реальные запросы.

**Зачем это нужно:**
- **Разработчикам фронтенда** — не нужно читать Java-код, чтобы понять API.
- **Тестировщикам** — можно проверять API прямо в браузере.
- **Внешним партнёрам** — документация автоматически обновляется вместе с кодом.

---

## Springdoc-openapi

> 💡 **Springdoc-openapi** — библиотека, которая автоматически генерирует OpenAPI-спецификацию по вашему Spring Boot коду. Никаких ручных YAML — всё из аннотаций.

### 3.1. Зависимость в `pom.xml`

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

> 💡 **`-ui`** в конце артефакта означает, что подключается и Swagger UI. Если нужна только спецификация (JSON) — используйте `-api`.

### 3.2. Настройка `application.properties`

```properties
# Swagger UI
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
springdoc.swagger-ui.tagsSorter=alpha
springdoc.swagger-ui.tryItOutEnabled=true

# OpenAPI info
springdoc.api-docs.path=/v3/api-docs
```

| Параметр | Что делает |
|----------|-----------|
| `swagger-ui.path` | URL, по которому доступен UI |
| `operationsSorter=method` | Сортировка endpoints по HTTP-методу |
| `tagsSorter=alpha` | Сортировка тегов по алфавиту |
| `tryItOutEnabled=true` | Разрешить кнопку «Try it out» по умолчанию |

### 3.3. Проверка

Запустите приложение и откройте:
- `http://localhost:8080/swagger-ui.html` — Swagger UI.
- `http://localhost:8080/v3/api-docs` — OpenAPI-спецификация в JSON.

Swagger автоматически найдёт все `@RestController` и покажет их. Но без аннотаций описания будут минимальными.

---

## Аннотации для описания API

> 💡 Swagger читает **сваггер-аннотации** (`io.swagger.v3.oas.annotations`) и добавляет их в спецификацию.

### 4.1. Основные аннотации

| Аннотация | Где применяется | Что описывает |
|-----------|----------------|---------------|
| `@Tag` | Класс контроллера | Группа endpoints (например, «Products API») |
| `@Operation` | Метод | Summary, description endpoint |
| `@Parameter` | Параметр метода | Описание параметра (path, query, header) |
| `@ApiResponse` | Метод | Возможный HTTP-ответ с кодом и описанием |
| `@Schema` | DTO / Entity | Описание поля модели |

### 4.2. Пример: ProductApiController с аннотациями

```java
package com.example.shop.controller.api;

import com.example.shop.dto.ProductDto;
import com.example.shop.service.ProductService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Tag(name = "Products", description = "API для работы с товарами")
@RestController
@RequestMapping("/api/products")
public class ProductApiController {

    @Autowired
    private ProductService productService;

    @Operation(summary = "Получить все товары",
               description = "Возвращает полный список товаров без пагинации")
    @ApiResponse(responseCode = "200", description = "Успешно",
                 content = @Content(schema = @Schema(implementation = ProductDto.class)))
    @GetMapping
    public List<ProductDto> list() {
        // ...
    }

    @Operation(summary = "Получить товар по артикулу",
               description = "Возвращает один товар по его ID (артикулу)")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "Товар найден"),
            @ApiResponse(responseCode = "404", description = "Товар не найден")
    })
    @GetMapping("/{id}")
    public ResponseEntity<ProductDto> getById(
            @Parameter(description = "Артикул товара", example = "N592T4")
            @PathVariable String id) {
        // ...
    }

    @Operation(summary = "Поиск товаров по названию")
    @GetMapping("/search")
    public List<ProductDto> search(
            @Parameter(description = "Поисковая строка", example = "ручка")
            @RequestParam String q) {
        // ...
    }
}
```

> 💡 **`example`** в `@Parameter` — показывает пример значения в Swagger UI.

### 4.3. Аннотации в DTO

```java
package com.example.shop.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import java.math.BigDecimal;

@Data
@Schema(description = "Информация о товаре")
public class ProductDto {

    @Schema(description = "Артикул", example = "N592T4")
    private String id;

    @Schema(description = "Название товара", example = "Стикеры")
    private String title;

    @Schema(description = "Цена в рублях", example = "34.00")
    private BigDecimal cost;

    @Schema(description = "Остаток на складе", example = "17")
    private Integer quantityInStock;
}
```

---

## Группировка и безопасность

### 5.1. `@Tag` для группировки

```java
@Tag(name = "Orders", description = "API для работы с заказами")
@RestController
@RequestMapping("/api/orders")
public class OrderApiController { ... }
```

Если не указать `@Tag`, Swagger создаст группу по имени контроллера (`product-api-controller`), что выглядит некрасиво.

### 5.2. Basic Auth в Swagger UI

> 💡 Если API защищён Spring Security, Swagger UI не сможет отправлять запросы без аутентификации. Нужно добавить кнопку «Authorize».

В `SecurityConfig` добавьте:

```java
@Bean
public OpenAPI customOpenAPI() {
    return new OpenAPI()
            .info(new Info()
                    .title("Shop API")
                    .version("1.0")
                    .description("API интернет-магазина канцтоваров"))
            .components(new Components()
                    .addSecuritySchemes("basicAuth",
                            new SecurityScheme()
                                    .type(SecurityScheme.Type.HTTP)
                                    .scheme("basic")));
}
```

И разрешите Swagger без логина:

```java
.requestMatchers(
        "/swagger-ui.html",
        "/swagger-ui/**",
        "/v3/api-docs/**"
).permitAll()
```

> 💡 Swagger UI должен быть доступен **без логина**, иначе его не открыть. А для отправки запросов — кнопка «Authorize».

---

## Postman-коллекция

> 💡 **Postman** — инструмент для тестирования API. Позволяет сохранять запросы, объединять их в коллекции и делиться с командой.

**Как экспортировать коллекцию:**
1. Откройте Postman.
2. Создайте коллекцию «Shop API».
3. Добавьте запросы: `GET /api/products`, `GET /api/products/{id}`, `GET /api/orders/my`.
4. Экспортируйте: **Collection → … → Export → Collection v2.1**.
5. Положите файл `shop-api.postman_collection.json` в репозиторий.

> 💡 **Альтернатива:** можно импортировать OpenAPI-спецификацию прямо в Postman — он создаст коллекцию автоматически.

---

## Что такое Docker

> 💡 **Docker** — это инструмент для упаковки приложения и его зависимостей в **контейнер**. Контейнер — это изолированная среда, которая работает одинаково на любой машине.

**Зачем это нужно:**
- **«У меня работает»** больше не проблема. Контейнер работает везде.
- **Одна команда для запуска** — `docker-compose up`.
- **Изоляция** — приложение не зависит от версий Java, PostgreSQL на хосте.

**Ключевые понятия:**

| Термин | Что это |
|--------|---------|
| **Образ (image)** | Шаблон для создания контейнера |
| **Контейнер (container)** | Запущенный экземпляр образа |
| **Dockerfile** | Инструкция для сборки образа |
| **docker-compose** | Инструмент для запуска нескольких контейнеров |

---

## Dockerfile для Spring Boot

> 💡 **Многоэтапная сборка (multi-stage build)** — лучшая практика. Первый этап собирает приложение, второй — только запускает. Это уменьшает итоговый образ.

### 8.1. `Dockerfile` в корне проекта

```dockerfile
# syntax=docker/dockerfile:1

# Этап 1: Сборка
FROM eclipse-temurin:21-jdk-jammy AS build
WORKDIR /build

COPY --chmod=0755 mvnw mvnw
COPY .mvn/ .mvn/
COPY pom.xml pom.xml

RUN --mount=type=cache,target=/root/.m2 ./mvnw dependency:go-offline -DskipTests

COPY src src/
RUN --mount=type=cache,target=/root/.m2 ./mvnw package -DskipTests

# Этап 2: Запуск
FROM eclipse-temurin:21-jre-jammy AS final

ARG UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    appuser

WORKDIR /app

COPY --from=build /build/target/*.jar app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 8.2. `.dockerignore`

```
target/
.idea/
*.iml
.vscode/
.git/
.gitignore
README.md
uploads/
logs/
*.md
```

### 8.3. Сборка и запуск

```bash
docker build -t shop:latest .
docker run -p 8080:8080 shop:latest
```

> ⚠️ **Без БД приложение не запустится.** Нужен `docker-compose`.

---

## Docker Compose

> 💡 **Docker Compose** — инструмент для запуска нескольких контейнеров одной командой.

### 9.1. `docker-compose.yml`

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    container_name: shop-db
    environment:
      POSTGRES_DB: shop
      POSTGRES_USER: shop_user
      POSTGRES_PASSWORD: shop_password
    ports:
      - "5432:5432"
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./ScriptWithData.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U shop_user -d shop"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build: .
    container_name: shop-app
    depends_on:
      db:
        condition: service_healthy
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/shop
      SPRING_DATASOURCE_USERNAME: shop_user
      SPRING_DATASOURCE_PASSWORD: shop_password
    ports:
      - "8080:8080"

volumes:
  db-data:
```

### 9.2. `application-docker.properties`

```properties
spring.datasource.url=jdbc:postgresql://db:5432/shop
spring.datasource.username=shop_user
spring.datasource.password=shop_password
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

logging.level.root=INFO
logging.level.com.example.shop=INFO
```

> 💡 **`db` в URL** — это имя сервиса в `docker-compose.yml`. Docker создаёт внутреннюю DNS, где `db` резолвится в IP контейнера PostgreSQL.

### 9.3. Запуск

```bash
docker-compose up --build       # собрать и запустить
docker-compose up -d --build    # в фоне
docker-compose down             # остановить
docker-compose down -v          # остановить и удалить данные БД
```

### 9.4. Проверка

- Приложение: `http://localhost:8080`
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- PostgreSQL: `localhost:5432` (можно подключиться из DBeaver)

---

## CI в Gogs

> 💡 **CI (Continuous Integration)** — автоматическая сборка и тесты при каждом push.

> ⚠️ **Gogs не имеет встроенного CI.** В отличие от GitHub Actions, Gogs — это только Git-сервер. Для CI нужен отдельный инструмент: **Drone CI**, **Jenkins** или **Woodpecker**.

### 10.1. Вариант с Drone CI

Если Drone настроен на вашем сервере, создайте `.drone.yml`:

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: build-and-test
    image: eclipse-temurin:21-jdk
    commands:
      - ./mvnw clean verify
    environment:
      SPRING_PROFILES_ACTIVE: test
```

### 10.2. Вариант с Jenkins

`Jenkinsfile` в корне:

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.9'
        jdk 'JDK-21'
    }
    stages {
        stage('Build') {
            steps { sh './mvnw clean compile' }
        }
        stage('Test') {
            steps { sh './mvnw test' }
        }
        stage('Package') {
            steps { sh './mvnw package -DskipTests' }
        }
    }
    post {
        always { junit 'target/surefire-reports/*.xml' }
    }
}
```

### 10.3. Если CI нет

Просто используйте **локальную проверку** перед push:

```bash
./mvnw clean verify
```

> 💡 **Правило:** если `./mvnw clean verify` падает локально — не пушьте.

---

## Финальный README.md

Обновите `README.md`:

```markdown
# Shop — Интернет-магазин канцтоваров

Учебный проект по УП 02. Spring Boot + PostgreSQL + Thymeleaf.

## 🚀 Запуск

### Через Docker (рекомендуется)

```bash
docker-compose up --build
```

Приложение: http://localhost:8080
Swagger UI: http://localhost:8080/swagger-ui.html

### Локально

1. Установите PostgreSQL 16, создайте БД `shop`.
2. Выполните `ScriptWithData.sql`.
3. Настройте `application-dev.properties`.
4. Запустите `./mvnw spring-boot:run`.

## 🧪 Тесты

```bash
./mvnw test
```

## 🛠️ Стек

- Java 21
- Spring Boot 3.x
- Spring Data JPA
- Spring Security
- Thymeleaf
- PostgreSQL 16
- Docker

## 📚 Документация

- Swagger UI: `/swagger-ui.html`
- OpenAPI JSON: `/v3/api-docs`

## 👤 Автор

Сафиулин Руслан Ринатович
```

---

## Чек-лист проекта

Перед защитой пройдитесь по всему списку. Каждый пункт — обязателен.

### Функционал

| # | Что проверить | ✅ |
|---|--------------|---|
| 1 | Главная страница открывается, показывает категории и популярные товары | |
| 2 | Каталог работает с фильтром по категории и пагинацией | |
| 3 | Карточка товара открывается, фото отображается | |
| 4 | Поиск по названию работает | |
| 5 | Корзина: добавление, изменение количества, удаление | |
| 6 | Оформление заказа: выбор пункта выдачи, сохранение в БД | |
| 7 | Страница «Мои заказы» показывает заказы текущего пользователя | |
| 8 | Регистрация нового пользователя работает | |
| 9 | Вход/выход работает | |
| 10 | Роли: клиент не может зайти в `/admin/**` | |

### Админ-панель

| # | Что проверить | ✅ |
|---|--------------|---|
| 11 | Дашборд показывает статистику | |
| 12 | CRUD товаров работает | |
| 13 | CRUD категорий работает | |
| 14 | Управление заказами: список, фильтр по статусу и датам | |
| 15 | Смена статуса заказа работает | |
| 16 | Экспорт в CSV и Excel работает | |
| 17 | Страница отчётов с графиком открывается | |

### REST API

| # | Что проверить | ✅ |
|---|--------------|---|
| 18 | `/api/products` отдаёт JSON | |
| 19 | `/api/products/{id}` работает | |
| 20 | `/api/products/UNKNOWN` → 404 с `{"error": "..."}` | |
| 21 | `/api/orders/my` работает для залогиненного | |
| 22 | Swagger UI открывается | |
| 23 | Все endpoints задокументированы | |

### Качество кода

| # | Что проверить | ✅ |
|---|--------------|---|
| 24 | Все классы разложены по пакетам | |
| 25 | Нет дублирования логики | |
| 26 | Логирование настроено | |
| 27 | Профили `dev`, `prod`, `test`, `docker` работают | |
| 28 | Страницы 404, 403, 500 настроены | |
| 29 | Покрытие тестами ≥ 60% | |
| 30 | `./mvnw clean verify` проходит без ошибок | |

### Деплой

| # | Что проверить | ✅ |
|---|--------------|---|
| 31 | `Dockerfile` собирается | |
| 32 | `docker-compose up --build` работает | |
| 33 | Приложение доступно на `localhost:8080` | |
| 34 | `README.md` содержит инструкцию запуска | |
| 35 | `.gitignore` исключает `target/`, `.idea/`, `uploads/` | |

---

## Подготовка к защите

### 13.1. Что нужно сдать

| Артефакт | Формат | Где |
|----------|--------|-----|
| **Репозиторий** | Gogs | `Lesson1` |
| **README.md** | Markdown | Корень репозитория |
| **Презентация** | 5–7 слайдов | PDF или PPTX |
| **Скринкаст** | 2–3 минуты | MP4 |
| **Postman-коллекция** | JSON | Корень репозитория |
| **Словарь данных** | XLSX | Корень репозитория |

### 13.2. Структура презентации

**5–7 слайдов:**

| # | Слайд | Что показать |
|---|-------|--------------|
| 1 | Титульный | ФИО, группа, тема, год |
| 2 | Цель и задачи | Что делали, зачем |
| 3 | Стек технологий | Java 21, Spring Boot, PostgreSQL, Thymeleaf, Docker |
| 4 | Архитектура | Схема слоёв (`Controller → Service → Repository → Entity`) |
| 5 | Демонстрация | Скриншоты каталога, корзины, админки, Swagger |
| 6 | Сложности и решения | Что было сложно, как решили |
| 7 | Итоги | Что освоили, что дальше |

### 13.3. Скринкаст

**2–3 минуты.** Сценарий:
1. Открыть главную страницу (10 сек).
2. Перейти в каталог, применить фильтр (20 сек).
3. Открыть карточку товара (10 сек).
4. Добавить товар в корзину, оформить заказ (40 сек).
5. Войти как админ, показать дашборд (20 сек).
6. Открыть Swagger UI (15 сек).
7. Показать запуск через `docker-compose` (15 сек).

**Запись:** OBS Studio, встроенная запись экрана Windows / macOS.

### 13.4. Структура защиты

| Этап | Время | Что делает студент |
|------|-------|-------------------|
| Презентация | 5 мин | Рассказывает о проекте |
| Демонстрация | 3 мин | Показывает работающее приложение |
| Вопросы | 5 мин | Отвечает на вопросы преподавателя |
| Разбор кода | 2 мин | Объясняет фрагмент кода по запросу |

---

## Типичные вопросы на защите

### Про архитектуру

**Q: Почему вы используете DTO, а не передаёте Entity напрямую?**

A: Entity привязан к таблице БД, содержит двунаправленные связи. Если сериализовать Entity в JSON — Jackson зациклится на `category.products → product.category → ...`. DTO разрывает связи и позволяет контролировать, какие поля попадают в ответ.

**Q: Зачем нужен слой Service?**

A: Service — это бизнес-логика. Контроллер принимает HTTP, Repository работает с БД, а правила («нельзя создать товар с отрицательной ценой», «удалить фото при удалении товара») живут в Service. Это позволяет переиспользовать логику: один и тот же метод вызывается из MVC-контроллера, REST-контроллера и фоновой задачи.

**Q: Что такое `@Transactional` и зачем он нужен?**

A: `@Transactional` оборачивает метод в транзакцию. Если внутри метода что-то упадёт — все изменения в БД откатятся. Например, при создании заказа: если сохранение позиции упадёт, заказ тоже не сохранится. Без `@Transactional` БД может остаться в «полусостоянии».

### Про Spring

**Q: Чем `@RestController` отличается от `@Controller`?**

A: `@RestController` = `@Controller` + `@ResponseBody`. Каждый метод возвращает данные (JSON), а не имя шаблона. `@Controller` возвращает имя HTML-шаблона, который рендерит Thymeleaf.

**Q: Что такое `@Autowired`?**

A: Внедрение зависимости. Spring находит бин подходящего типа и подставляет его. Можно делать через конструктор (лучшая практика) или поле.

**Q: Как Spring Security понимает, что пользователь залогинен?**

A: Spring Security хранит `Authentication` в `SecurityContext`, который привязан к текущему потоку. При входе через форму `UsernamePasswordAuthenticationFilter` проверяет логин/пароль, создаёт `Authentication` и кладёт его в контекст. При следующих запросах `SecurityContextPersistenceFilter` восстанавливает контекст из сессии (по cookie `JSESSIONID`).

### Про БД

**Q: Почему `ddl-auto=validate`, а не `update`?**

A: `update` может «дописать» свои колонки и сломать структуру, созданную SQL-скриптом. `validate` только проверяет, что Entity совпадает с таблицами — это страховка от случайных изменений.

**Q: Что такое N+1 проблема и как её решить?**

A: При выводе 100 товаров Hibernate делает 1 запрос на товары + 100 запросов на категории = 101 запрос. Решается через `JOIN FETCH` в `@Query` или `@EntityGraph` — тогда Hibernate делает 1 запрос с `INNER JOIN`.

**Q: Зачем BCrypt?**

A: Пароли нельзя хранить в открытом виде. BCrypt — это медленная хеш-функция с «солью». Из хеша нельзя восстановить пароль, но можно проверить: если хеш от введённого пароля совпадает с сохранённым — пароль верный.

### Про Docker

**Q: Зачем Docker, если приложение и так работает?**

A: Docker решает проблему «у меня работает, а у тебя нет». Контейнер содержит всё: Java, зависимости, настройки. Другой разработчик запускает `docker-compose up` и получает то же самое, независимо от его ОС и версий.

**Q: Чем контейнер отличается от виртуальной машины?**

A: ВМ эмулирует полноценную ОС со своим ядром — это медленно и тяжело. Контейнер использует ядро хост-системы, изолирует только процессы и файловую систему — это быстро и легко.

---

## Финальное задание

### Что сдать

1. **Обновлённый репозиторий `Lesson1` в Gogs**:
   - Все исходники в правильной структуре пакетов.
   - `README.md` с инструкцией запуска.
   - `Dockerfile` и `docker-compose.yml`.
   - `ScriptWithData.sql`.
   - `DataDictionaryOfФамилияИмя.xlsx`.
   - `.gitignore` (без `target/`, `.idea/`, `uploads/`).
   - Postman-коллекция `shop-api.postman_collection.json`.

2. **Презентация** (5–7 слайдов) в PDF.

3. **Скринкаст** (2–3 минуты) в MP4.

4. **Запущенное приложение** на защите:
   - Через `docker-compose up --build`.
   - Или локально (`./mvnw spring-boot:run`).

### Критерии оценивания

| Критерий | Вес |
|----------|-----|
| Работающий функционал (каталог, корзина, заказы) | 25% |
| Архитектура и чистота кода | 20% |
| Безопасность (роли, BCrypt, защита админки) | 15% |
| REST API + Swagger | 10% |
| Тесты и покрытие ≥ 60% | 10% |
| Документация (README, словарь данных) | 10% |
| Docker и деплой | 5% |
| Защита (объяснение кода, ответы на вопросы) | 5% |

### Что нельзя делать

- ❌ Коммитить `application.properties` с реальными паролями.
- ❌ Коммитить папку `target/`.
- ❌ Коммитить `uploads/`.
- ❌ Использовать `System.out.println` вместо логирования.
- ❌ Хранить пароли в открытом виде.
- ❌ Открывать `/admin/**` без проверки ролей.

---

## Итоги практики

### Что вы освоили

За 12 занятий вы прошли путь от «подключиться к БД» до «запустить production-ready веб-приложение в Docker». Вот что теперь в вашем портфолио:

| Область | Навыки |
|---------|--------|
| **Backend** | Spring Boot, Spring Data JPA, Spring Security, REST API |
| **Frontend** | Thymeleaf, HTML, CSS, Bootstrap-подобные стили |
| **БД** | PostgreSQL, DBeaver, SQL, JPA, N+1, кэширование |
| **Архитектура** | Слоистая архитектура, DTO, Service Layer, Repository |
| **Тестирование** | JUnit 5, Mockito, `@DataJpaTest`, `@SpringBootTest`, TestContainers |
| **Инфраструктура** | Docker, Docker Compose, Git, Gogs, CI |
| **Документация** | Swagger/OpenAPI, README, словарь данных |

### Что дальше учить

Если хотите развиваться в Java-разработке:

| Направление | Что учить |
|-------------|-----------|
| **Микросервисы** | Spring Cloud, Eureka, Feign, Kafka |
| **Реактивность** | Project Reactor, WebFlux |
| **Базы** | MongoDB, Redis, Elasticsearch |
| **Docker/K8s** | Kubernetes, Helm, Istio |
| **CI/CD** | GitLab CI, Jenkins, GitHub Actions |
| **Тестирование** | Selenium, Cucumber, Gatling |

### Полезные ссылки

- 🇬🇧 [Spring Guides](https://spring.io/guides)
- 🇬🇧 [Baeldung — туториалы по Spring](https://www.baeldung.com/)
- 🇬🇧 [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- 🇬🇧 [Docker Docs](https://docs.docker.com/)
- 🇷🇺 [Хабр: Spring на русском](https://habr.com/ru/hubs/spring_framework/)
- 🇷🇺 [JavaRush: курс по Spring](https://javarush.com/quests/lectures/ru.javarush.java.spring.lecture.level01.lecture00)

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Итоги](Lesson12.md) |

---

<div align="center">
    <i>Поздравляю с завершением практики! 🎓</i><br/>
    <i>Теперь у вас есть полноценный проект в портфолио.</i>
</div>
