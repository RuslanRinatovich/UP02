# Занятие 8. Логирование, профили, обработка ошибок и первые тесты

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 9](Lesson9.md)

## План
1. [Введение: что было и куда идём](#введение)
2. [Зачем логировать](#зачем-логировать)
3. [SLF4J + Logback: логирование в Spring Boot](#slf4j)
4. [Уровни логирования и настройка](#уровни-логирования)
5. [Логирование в Service и Controller](#логирование-в-коде)
6. [Spring Profiles: dev и prod](#spring-profiles)
7. [Файлы application-dev.properties и application-prod.properties](#файлы-профилей)
8. [Обработка ошибок в MVC](#обработка-ошибок-mvc)
9. [Страницы 404 и 500](#страницы-404-и-500)
10. [Что такое тесты и зачем они нужны](#что-такое-тесты)
11. [JUnit 5 + Mockito: первые тесты](#junit-mockito)
12. [Проверка и чек-лист](#проверка)
13. [Задание](#задание)

---

## Введение

За семь занятий мы построили полноценное веб-приложение:
- Каталог, корзина, оформление заказов.
- Админ-панель с управлением заказами.
- REST API для внешних клиентов.

**Проблема:** когда что-то падает в проде, мы не знаем почему. Нет логов. Нет разделения конфигураций для dev/prod. Пользователь видит «Whitelabel Error Page» вместо нормальной 404. И мы не можем проверить, что наш код работает — приходится вручную кликать по формам.

**Сегодня:**
1. Логирование — чтобы видеть, что происходит.
2. Профили — чтобы dev и prod не мешали друг другу.
3. Красивые страницы ошибок.
4. Первые тесты — чтобы не бояться рефакторить.

---

## Зачем логировать

> 💡 **Логи** — это текстовые записи о том, что происходит в приложении. Аналог чёрного ящика.

**Зачем они нужны:**
- **Отладка:** почему заказ не создался? В логах видно: «Пользователь X, товар Y, ошибка Z».
- **Мониторинг:** сколько пользователей пришло за день, сколько заказов оформлено.
- **Безопасность:** попытки входа с неверным паролем.
- **Аудит:** кто и когда что делал.

**Антипаттерн:** `System.out.println("Тут всё ок")`. В проде это не работает — вывод идёт в консоль, нет уровней, нет формата, нельзя отключить.

---

## SLF4J + Logback

> 💡 В Spring Boot из коробки идёт **SLF4J** (интерфейс) + **Logback** (реализация). Это стандарт в Java-мире.

**Как использовать:**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(ProductService.class);
```

Или через Lombok:
```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class ProductService { ... }
```

> 💡 **Lombok `@Slf4j`** автоматически создаёт поле `log`. Короче и удобнее — используем этот вариант.

---

## Уровни логирования

| Уровень | Когда использовать |
|---------|-------------------|
| **TRACE** | Очень детальная отладка (значения каждой переменной в цикле) |
| **DEBUG** | Отладка: «вошли в метод, получили такой-то параметр» |
| **INFO** | Важные события: «заказ создан», «пользователь зарегистрирован» |
| **WARN** | Что-то подозрительное, но не критичное: «попытка входа с неверным паролем» |
| **ERROR** | Ошибка: «не удалось сохранить заказ» |

> 💡 **Правило:** в проде обычно INFO и выше. DEBUG включают только когда что-то расследуют.

**В `application.properties`:**

```properties
# Глобальный уровень
logging.level.root=INFO

# Наш пакет — подробнее
logging.level.com.example.shop=DEBUG

# Spring Security — потише
logging.level.org.springframework.security=WARN

# Hibernate SQL — выключить в проде
logging.level.org.hibernate.SQL=DEBUG
```

---

## Логирование в коде

### 5.1. В ProductService

```java
package com.example.shop.service;

import lombok.extern.slf4j.Slf4j;
// ...

@Slf4j
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // ...

    @Transactional
    public Product createFromForm(ProductForm form) {
        log.info("Создание товара: id={}, title={}", form.getId(), form.getTitle());

        try {
            Product product = new Product();
            // ... заполнение ...

            Product saved = productRepository.save(product);
            log.info("Товар успешно создан: id={}", saved.getId());
            return saved;

        } catch (Exception e) {
            log.error("Ошибка создания товара: id={}", form.getId(), e);
            throw e;
        }
    }

    @Transactional
    public void deleteById(String id) {
        log.warn("Удаление товара: id={}", id);
        productRepository.deleteById(id);
        log.info("Товар удалён: id={}", id);
    }
}
```

### 5.2. В SecurityConfig или отдельном listener

```java
@Slf4j
@Component
public class AuthenticationEventListener {

    @EventListener
    public void onSuccess(AuthenticationSuccessEvent event) {
        log.info("Успешный вход: user={}", event.getAuthentication().getName());
    }

    @EventListener
    public void onFailure(AbstractAuthenticationFailureEvent event) {
        log.warn("Неудачная попытка входа: user={}", event.getAuthentication().getName());
    }
}
```

### 5.3. Что логировать — а что нет

| ✅ Логировать | ❌ Не логировать |
|--------------|------------------|
| Создание / удаление сущностей | Пароли |
| Изменение статусов | Токены |
| Неудачные попытки входа | Номера банковских карт |
| Ошибки в Service | Полные стектрейсы на каждый чих |
| Долгие операции (> 1 сек) | Всё подряд на уровне DEBUG |

**Формат сообщений:** используйте `{}` вместо конкатенации:
```java
// ❌ Плохо — строка строится всегда
log.debug("Товар: " + product);

// ✅ Хорошо — строится только если уровень включён
log.debug("Товар: {}", product);
```

---

## Spring Profiles

> 💡 **Профиль** — это способ иметь разные конфигурации для разных окружений: `dev`, `test`, `prod`.

**Зачем:**
- В dev — H2 или локальный Postgres, DEBUG-логи, `ddl-auto=update`.
- В prod — боевой Postgres, INFO-логи, `ddl-auto=validate`.

**Как активировать профиль:**
- Через `application.properties`: `spring.profiles.active=dev`.
- Через переменную окружения: `SPRING_PROFILES_ACTIVE=prod`.
- Через аргумент запуска: `--spring.profiles.active=prod`.

---

## Файлы профилей

### 7.1. Основной `application.properties`

```properties
spring.application.name=shop
spring.profiles.active=dev
```

> 💡 Указание `spring.profiles.active` в основном файле — плохая практика. Лучше через переменную окружения. Но для учебного проекта допустимо.

### 7.2. `application-dev.properties`

```properties
spring.datasource.url=jdbc:postgresql://192.168.2.202:5432/demoN
spring.datasource.username=userN
spring.datasource.password=ВАШ_ПАРОЛЬ
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

logging.level.com.example.shop=DEBUG
logging.level.org.hibernate.SQL=DEBUG
```

### 7.3. `application-prod.properties`

```properties
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USER}
spring.datasource.password=${DATABASE_PASSWORD}
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

logging.level.root=INFO
logging.level.com.example.shop=INFO

server.port=8080
```

> 💡 **`${DATABASE_URL}`** — Spring подставит значение из переменной окружения. Пароли в проде **никогда** не хранятся в файлах.

### 7.4. Как запускать с профилем

**В IntelliJ:** Run → Edit Configurations → Active profiles: `prod`.

**В командной строке:**
```bash
java -jar shop.jar --spring.profiles.active=prod
```

**Через переменную окружения:**
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar shop.jar
```

---

## Обработка ошибок в MVC

Мы уже сделали `ApiExceptionHandler` для REST (урок 7). Теперь сделаем то же для MVC — HTML-страниц.

### 8.1. MvcExceptionHandler

`src/main/java/com/example/shop/controller/MvcExceptionHandler.java`:

```java
package com.example.shop.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;

@Slf4j
@ControllerAdvice(basePackages = "com.example.shop.controller")
public class MvcExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String handleNotFound(IllegalArgumentException e, Model model) {
        log.warn("Не найдено: {}", e.getMessage());
        model.addAttribute("message", e.getMessage());
        return "error/404";
    }

    @ExceptionHandler(SecurityException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public String handleForbidden(SecurityException e, Model model) {
        log.warn("Доступ запрещён: {}", e.getMessage());
        model.addAttribute("message", e.getMessage());
        return "error/403";
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String handleAll(Exception e, Model model) {
        log.error("Внутренняя ошибка", e);
        model.addAttribute("message", "Что-то пошло не так. Мы уже разбираемся.");
        return "error/500";
    }
}
```

> 💡 **`@ControllerAdvice`** — глобальный перехватчик исключений для всех `@Controller`. Если в любом контроллере бросить `IllegalArgumentException` — сработает этот метод.

> ⚠️ **`ApiExceptionHandler` в пакете `api` и `MvcExceptionHandler` в пакете `controller`** — они не пересекаются. REST-запросы идут в `api`, MVC — в `controller`. Такое разделение корректно.

---

## Страницы 404 и 500

### 9.1. `templates/error/404.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Не найдено', content=~{::content})}">
<body>
<div th:fragment="content">
    <div style="text-align: center; padding: 60px 20px;">
        <h1 style="font-size: 72px; color: #0066cc;">404</h1>
        <h2>Страница не найдена</h2>
        <p th:text="${message} ?: 'Такой страницы не существует.'"></p>
        <a th:href="@{/}" style="margin-top: 20px; display: inline-block;">
            На главную
        </a>
    </div>
</div>
</body>
</html>
```

### 9.2. `templates/error/403.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Доступ запрещён', content=~{::content})}">
<body>
<div th:fragment="content">
    <div style="text-align: center; padding: 60px 20px;">
        <h1 style="font-size: 72px; color: #d9534f;">403</h1>
        <h2>Доступ запрещён</h2>
        <p th:text="${message} ?: 'У вас нет прав на этот раздел.'"></p>
        <a th:href="@{/}" style="margin-top: 20px; display: inline-block;">
            На главную
        </a>
    </div>
</div>
</body>
</html>
```

### 9.3. `templates/error/500.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Ошибка сервера', content=~{::content})}">
<body>
<div th:fragment="content">
    <div style="text-align: center; padding: 60px 20px;">
        <h1 style="font-size: 72px; color: #d9534f;">500</h1>
        <h2>Внутренняя ошибка</h2>
        <p th:text="${message} ?: 'Что-то пошло не так.'"></p>
        <a th:href="@{/}" style="margin-top: 20px; display: inline-block;">
            На главную
        </a>
    </div>
</div>
</body>
</html>
```

> 💡 **`th:text="${message} ?: 'дефолт'"`** — оператор Элвиса. Если `message` пустой — подставляется строка после `?:`.

> 💡 **Spring Boot сам подхватывает** шаблоны из `templates/error/` по коду ошибки. Но у нас есть `@ControllerAdvice` — он перехватывает первым.

---

## Что такое тесты

> 💡 **Тест** — это код, который проверяет, что другой код работает правильно.

**Зачем писать тесты:**
1. **Уверенность при рефакторинге.** Изменил код — запустил тесты — они не сломались.
2. **Документация.** Тесты показывают, как использовать код.
3. **Быстрая проверка.** Не нужно вручную кликать по формам — тест делает это за секунды.
4. **Защита от регрессий.** Кто-то случайно сломал — тесты это поймают.

**Виды тестов:**

| Вид | Что проверяет | Скорость |
|-----|--------------|----------|
| **Unit** (модульные) | Один класс/метод в изоляции | Миллисекунды |
| **Integration** | Взаимодействие нескольких классов + БД | Секунды |
| **End-to-end** | Весь сценарий через HTTP | Десятки секунд |

Сегодня — **unit-тесты**. Интеграционные и E2E — на уроке 10.

---

## JUnit 5 + Mockito

> 💡 **JUnit 5** — фреймворк для тестов. **Mockito** — для «заглушек» (mock-объектов), чтобы тестировать в изоляции.

**Spring Boot уже включает их** в `spring-boot-starter-test`. Дополнительно ставить не нужно.

### 11.1. Первый тест: PriceUtils

`src/test/java/com/example/shop/util/PriceUtilsTest.java`:

```java
package com.example.shop.util;

import org.junit.jupiter.api.Test;
import java.math.BigDecimal;

import static org.junit.jupiter.api.Assertions.assertEquals;

class PriceUtilsTest {

    @Test
    void applyDiscount_shouldReturnSamePrice_whenDiscountIsNull() {
        BigDecimal result = PriceUtils.applyDiscount(new BigDecimal("100.00"), null);
        assertEquals(new BigDecimal("100.00"), result);
    }

    @Test
    void applyDiscount_shouldReturnDiscountedPrice() {
        BigDecimal result = PriceUtils.applyDiscount(new BigDecimal("100.00"), 10);
        assertEquals(new BigDecimal("90.00"), result);
    }

    @Test
    void applyDiscount_shouldReturnZero_whenPriceIsNull() {
        BigDecimal result = PriceUtils.applyDiscount(null, 10);
        assertEquals(BigDecimal.ZERO, result);
    }

    @Test
    void format_shouldReturnDash_whenPriceIsNull() {
        assertEquals("—", PriceUtils.format(null));
    }
}
```

**Разбор:**

| Код | Что делает |
|-----|-----------|
| `@Test` | Помечает метод как тест |
| `assertEquals(expected, actual)` | Проверяет, что значения равны |
| `_should..._when...` | Соглашение об именах: что делает + при каком условии |

**Запустить:** правой кнопкой на классе → **Run 'PriceUtilsTest'**.

### 11.2. Тест с Mockito: ProductService

`src/test/java/com/example/shop/service/ProductServiceTest.java`:

```java
package com.example.shop.service;

import com.example.shop.dto.ProductForm;
import com.example.shop.entity.Category;
import com.example.shop.entity.Product;
import com.example.shop.repository.CategoryRepository;
import com.example.shop.repository.ProductRepository;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class ProductServiceTest {

    @Mock
    private ProductRepository productRepository;

    @Mock
    private CategoryRepository categoryRepository;

    @InjectMocks
    private ProductService productService;

    @Test
    void createFromForm_shouldSaveProduct_whenCategoryExists() {
        // Arrange — готовим данные
        ProductForm form = new ProductForm();
        form.setId("TEST01");
        form.setTitle("  Ручка  ");
        form.setCost(new BigDecimal("50"));
        form.setQuantityInStock(10);
        form.setCategoryId(1);

        Category category = new Category();
        category.setId(1);
        category.setTitle("Канцтовары");

        when(categoryRepository.findById(1)).thenReturn(Optional.of(category));
        when(productRepository.save(any(Product.class)))
                .thenAnswer(invocation -> invocation.getArgument(0));

        // Act — вызываем метод
        Product result = productService.createFromForm(form);

        // Assert — проверяем
        assertNotNull(result);
        assertEquals("TEST01", result.getId());
        assertEquals("Ручка", result.getTitle()); // проверяем, что normalize убрал пробелы
        assertEquals(category, result.getCategory());

        // Verify — убеждаемся, что репозиторий был вызван
        verify(productRepository, times(1)).save(any(Product.class));
    }

    @Test
    void createFromForm_shouldThrow_whenCategoryNotFound() {
        ProductForm form = new ProductForm();
        form.setId("TEST02");
        form.setTitle("Ручка");
        form.setCost(new BigDecimal("50"));
        form.setQuantityInStock(10);
        form.setCategoryId(999);

        when(categoryRepository.findById(999)).thenReturn(Optional.empty());

        assertThrows(IllegalArgumentException.class,
                () -> productService.createFromForm(form));
    }
}
```

**Разбор:**

| Код | Что делает |
|-----|-----------|
| `@ExtendWith(MockitoExtension.class)` | Включает Mockito в JUnit 5 |
| `@Mock` | Создаёт mock-объект репозитория |
| `@InjectMocks` | Создаёт `ProductService` и внедряет в него моки |
| `when(...).thenReturn(...)` | Настраивает mock: «если вызовут X — верни Y» |
| `verify(...)` | Проверяет, что метод mock-объекта был вызван N раз |
| `assertThrows(...)` | Проверяет, что метод бросает исключение |

> 💡 **Паттерн AAA:** Arrange (подготовка) → Act (действие) → Assert (проверка). Используйте его всегда — код тестов становится читаемым.

> ⚠️ **Почему mock, а не реальный `ProductRepository`?** Unit-тест должен быть быстрым и не зависеть от БД. Мы подменяем репозиторий заглушкой — тест длится миллисекунды.

### 11.3. Тест для Thymeleaf-контроллера (интеграционный, но упрощённый)

`src/test/java/com/example/shop/controller/ProductControllerTest.java`:

```java
package com.example.shop.controller;

import com.example.shop.entity.Product;
import com.example.shop.service.ProductService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import java.math.BigDecimal;
import java.util.List;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProductService productService;

    @MockBean
    private com.example.shop.service.CategoryService categoryService;

    @Test
    void apiProducts_shouldReturnJson() throws Exception {
        Product product = new Product();
        product.setId("N592T4");
        product.setTitle("Стикеры");
        product.setCost(new BigDecimal("34"));
        product.setQuantityInStock(17);

        when(productService.findAll()).thenReturn(List.of(product));

        mockMvc.perform(get("/products-jpa"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[0].id").value("N592T4"))
                .andExpect(jsonPath("$[0].title").value("Стикеры"));
    }
}
```

| Код | Что делает |
|-----|-----------|
| `@WebMvcTest(ProductController.class)` | Поднимает только MVC-часть, без БД |
| `@MockBean` | Заменяет бин в контексте на mock |
| `MockMvc` | Имитирует HTTP-запросы без запуска сервера |
| `jsonPath(...)` | Проверяет JSON-ответ по пути |

> ⚠️ `@WebMvcTest` не тянет за собой БД и Spring Security по умолчанию. Если тест падает на безопасности — добавьте `@AutoConfigureMockMvc(addFilters = false)`.

---

## Проверка

### 12.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | Логи выводятся в консоль при запуске | |
| 2 | В `ProductService` есть `log.info` при создании товара | |
| 3 | В логах видно SQL-запросы (`show-sql=true`) | |
| 4 | Создан `application-dev.properties` | |
| 5 | Создан `application-prod.properties` | |
| 6 | При запуске в консоли виден профиль (`The following 1 profile is active: "dev"`) | |
| 7 | Открытие `/unknown-page` — красивая 404 | |
| 8 | Открытие `/admin` под клиентом — красивая 403 | |
| 9 | Тест `PriceUtilsTest` проходит | |
| 10 | Тест `ProductServiceTest` проходит | |
| 11 | Тест `ProductControllerTest` проходит | |

### 12.2. Запуск всех тестов

Правой кнопкой на папке `src/test/java` → **Run 'All Tests'**. Или через Maven:

```bash
./mvnw test
```

Все тесты должны быть **зелёными**.

---

## Задание

1. **Настройте логирование**:
   - Добавьте `@Slf4j` в `OrderService`, `CartService`, `CategoryService`.
   - Логируйте создание/удаление сущностей на уровне `INFO`.
   - Логируйте попытки входа на уровне `INFO` / `WARN`.

2. **Создайте `logback-spring.xml`** в `src/main/resources`:
   - Логи пишутся не только в консоль, но и в файл `logs/shop.log`.
   - Формат: дата, уровень, класс, сообщение.
   - Размер файла — не больше 10 МБ (ротация).
   - Пример: [Baeldung: Logback](https://www.baeldung.com/spring-boot-logging).

3. **Создайте `util/LogUtils.java`**:
   - `logException(Logger, String, Exception)` — логирует исключение с контекстом.
   - Используйте в `MvcExceptionHandler` и `ApiExceptionHandler`.

4. **Настройте профиль `test`**:
   - `application-test.properties` с H2 in-memory.
   - В тестах используйте `@ActiveProfiles("test")`.

5. **Напишите тесты**:
   - `ValidationUtilsTest` — проверьте `isValidArticle`, `normalize`, `isValidEmail`.
   - `CartServiceTest` — с mock `HttpSession` (используйте `MockHttpSession`).
   - `CategoryServiceTest` — проверьте `findById` с несуществующим id.

6. **Создайте страницу `/error`** (универсальная):
   - Spring Boot автоматически использует `templates/error.html`, если `@ControllerAdvice` не сработал.
   - Показывает код ошибки и сообщение.

7. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 9](Lesson9.md) |

-

## 📚 Полезные ссылки

- 🇬🇧 [Spring Boot Logging](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)
- 🇬🇧 [Baeldung: Logback](https://www.baeldung.com/spring-boot-logging)
- 🇬🇧 [Baeldung: Spring Profiles](https://www.baeldung.com/spring-profiles)
- 🇬🇧 [Baeldung: JUnit 5](https://www.baeldung.com/junit-5)
- 🇬🇧 [Baeldung: Mockito](https://www.baeldung.com/mockito-series)
- 🇬🇧 [Baeldung: @WebMvcTest](https://www.baeldung.com/spring-boot-testing)
