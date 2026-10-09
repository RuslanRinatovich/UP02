# Занятие 9. Интеграционные тесты, работа с файлами и оптимизация запросов

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 10](Lesson10.md)

## План
1. [Введение: что было и куда идём](#введение)
2. [Unit-тесты vs интеграционные тесты](#unit-vs-integration)
3. [@DataJpaTest: тесты Repository](#datajpatest)
4. [@SpringBootTest: тесты Service с реальной БД](#springboottest)
5. [TestContainers: настоящий PostgreSQL в Docker](#testcontainers)
6. [Покрытие тестами: как измерить](#покрытие-тестами)
7. [Проблема N+1 запросов](#проблема-n1)
8. [JOIN FETCH и @EntityGraph](#join-fetch)
9. [Хранение файлов: bytea vs файловая система](#хранение-файлов)
10. [FileStorageService](#filestorageservice)
11. [Миграция фото из БД в файлы](#миграция-фото)
12. [Кэширование справочников](#кэширование)
13. [Проверка и чек-лист](#проверка)
14. [Задание](#задание)

---

## Введение

За восемь занятий мы сделали:
- Работающее приложение со всеми функциями магазина.
- Unit-тесты для утилит и сервисов с mock-объектами.
- Профили `dev`/`prod`, логи, обработку ошибок.

**Проблемы, которые остались:**
1. **Unit-тесты не проверяют главного** — что приложение работает целиком. Мы мокаем репозиторий, а в реальности SQL-запрос может упасть.
2. **N+1 проблема.** При выводе 100 товаров Hibernate делает 1 запрос на товары + 100 запросов на категории = 101 запрос. Это медленно.
3. **Фото товаров лежат в `bytea`.** БД распухает до гигабайтов, бэкапы становятся огромными, а отдача фото нагружает сервер.
4. **Справочники (категории, статусы, пункты выдачи) не меняются,** но мы их читаем из БД при каждом запросе.

**Сегодня:**
- Научимся писать **интеграционные тесты** — они поднимают реальную БД и проверяют всё вместе.
- Разберём **TestContainers** — настоящий PostgreSQL в Docker для тестов.
- Решим **N+1 проблему** через `JOIN FETCH`.
- Перенесём **фото в файловую систему**.
- Добавим **кэширование** справочников.

> 💡 Это занятие — про качество. Функционал у нас уже есть, теперь делаем его **быстрым, надёжным и проверяемым**.

---

## Unit-тесты vs интеграционные тесты

> 💡 **Unit-тест** проверяет один класс в изоляции. Все зависимости (репозитории, сервисы) подменяются mock-объектами.
>
> 💡 **Интеграционный тест** проверяет, что несколько классов **работают вместе** — например, Service + Repository + реальная БД.

**Сравнение:**

| Критерий | Unit | Integration |
|----------|------|-------------|
| Скорость | Миллисекунды | Секунды |
| Что проверяет | Логику одного метода | Взаимодействие слоёв |
| Зависимости | Mock-объекты | Реальная БД / контейнер |
| Что может сломаться | Логика кода | SQL, связи, транзакции |
| Когда использовать | Бизнес-логика | Проверка запросов, связей JPA |

**Что мы тестируем сегодня:**

| Слой | Инструмент | Что проверяем |
|------|-----------|---------------|
| Repository | `@DataJpaTest` | SQL-запросы, связи, каскады |
| Service | `@SpringBootTest` | Полный цикл: DTO → Entity → БД → Entity |
| Controller | `@WebMvcTest` + `MockMvc` | HTTP, JSON, статусы (уже делали в уроке 8) |

> 💡 **Правило:** больше всего тестов — unit, меньше — integration, ещё меньше — E2E. Это «пирамида тестирования».

---

## @DataJpaTest

> 💡 **`@DataJpaTest`** — поднимает только JPA-часть: `Entity`, `Repository`, `DataSource`. Контроллеры и сервисы не загружаются.

**По умолчанию** используется встроенная H2 (in-memory). Это быстро, но не всегда совпадает с реальным PostgreSQL.

Создайте `src/test/java/com/example/shop/repository/ProductRepositoryTest.java`:

```java
package com.example.shop.repository;

import com.example.shop.entity.Category;
import com.example.shop.entity.Product;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.test.context.ActiveProfiles;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@ActiveProfiles("test")
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private CategoryRepository categoryRepository;

    @Test
    void findByCategoryId_shouldReturnOnlyMatchingProducts() {
        // Arrange
        Category office = new Category();
        office.setTitle("Для офиса");
        office = categoryRepository.save(office);

        Category school = new Category();
        school.setTitle("Школьные");
        school = categoryRepository.save(school);

        Product p1 = buildProduct("P001", "Ручка", office);
        Product p2 = buildProduct("P002", "Карандаш", office);
        Product p3 = buildProduct("P003", "Тетрадь", school);

        productRepository.saveAll(List.of(p1, p2, p3));

        // Act
        Page<Product> result = productRepository.findByCategoryId(office.getId(),
                PageRequest.of(0, 10));

        // Assert
        assertEquals(2, result.getTotalElements());
        assertTrue(result.getContent().stream()
                .allMatch(p -> p.getCategory().getId().equals(office.getId())));
    }

    @Test
    void findByTitleContainingIgnoreCase_shouldBeCaseInsensitive() {
        Category cat = new Category();
        cat.setTitle("Канцтовары");
        cat = categoryRepository.save(cat);

        productRepository.save(buildProduct("P001", "Ручка шариковая", cat));
        productRepository.save(buildProduct("P002", "РУЧКА гелевая", cat));
        productRepository.save(buildProduct("P003", "Карандаш", cat));

        List<Product> result = productRepository.findByTitleContainingIgnoreCase("ручка");

        assertEquals(2, result.size());
    }

    @Test
    void deleteProduct_shouldNotDeleteCategory() {
        Category cat = new Category();
        cat.setTitle("Канцтовары");
        cat = categoryRepository.save(cat);

        Product product = buildProduct("P001", "Ручка", cat);
        productRepository.save(product);

        productRepository.deleteById("P001");

        Optional<Category> found = categoryRepository.findById(cat.getId());
        assertTrue(found.isPresent(), "Категория не должна удаляться вместе с товаром");
    }

    private Product buildProduct(String id, String title, Category category) {
        Product product = new Product();
        product.setId(id);
        product.setTitle(title);
        product.setCost(new BigDecimal("50"));
        product.setQuantityInStock(10);
        product.setCategory(category);
        return product;
    }
}
```

### 3.1. Разбор

| Аннотация | Что делает |
|-----------|-----------|
| `@DataJpaTest` | Загружает только JPA-часть Spring-контекста |
| `@ActiveProfiles("test")` | Активирует профиль `test` — там H2 |
| `@Autowired` | Внедряет репозитории |

> 💡 **Каждый тест — в транзакции.** `@DataJpaTest` автоматически откатывает изменения после теста. БД не «загрязняется».

> 💡 **Почему `@ActiveProfiles("test")`?** По умолчанию `@DataJpaTest` использует H2, но нам нужен **свой** профиль — чтобы задать конкретные настройки (диалект, схема).

### 3.2. Настройте профиль `test`

`src/test/resources/application-test.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;MODE=PostgreSQL
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
spring.jpa.show-sql=true
```

Добавьте зависимость в `pom.xml` (scope `test`):

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

> ⚠️ **H2 в режиме PostgreSQL:** `MODE=PostgreSQL` заставляет H2 понимать большинство конструкций PostgreSQL (например, `serial4` → `int auto_increment`). Но не всё: `bytea`, `ON DELETE CASCADE` могут отличаться. Поэтому для серьёзных проектов — TestContainers.

---

## @SpringBootTest

> 💡 **`@SpringBootTest`** поднимает **всё приложение**: контроллеры, сервисы, репозитории, безопасность. Используйте, когда надо проверить сценарий целиком.

`src/test/java/com/example/shop/service/ProductServiceIntegrationTest.java`:

```java
package com.example.shop.service;

import com.example.shop.dto.ProductForm;
import com.example.shop.entity.Category;
import com.example.shop.entity.Product;
import com.example.shop.repository.CategoryRepository;
import com.example.shop.repository.ProductRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@ActiveProfiles("test")
@Transactional
class ProductServiceIntegrationTest {

    @Autowired
    private ProductService productService;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private CategoryRepository categoryRepository;

    private Category category;

    @BeforeEach
    void setUp() {
        productRepository.deleteAll();
        categoryRepository.deleteAll();

        category = new Category();
        category.setTitle("Канцтовары");
        category = categoryRepository.save(category);
    }

    @Test
    void createFromForm_shouldPersistProductWithCategory() {
        ProductForm form = new ProductForm();
        form.setId("TEST01");
        form.setTitle("  Ручка шариковая  ");
        form.setCost(new BigDecimal("99.99"));
        form.setQuantityInStock(25);
        form.setDescription("Синяя, 0.7 мм");
        form.setCategoryId(category.getId());

        Product saved = productService.createFromForm(form);

        assertNotNull(saved);
        assertEquals("TEST01", saved.getId());
        assertEquals("Ручка шариковая", saved.getTitle(), "Пробелы должны быть убраны");
        assertEquals(category.getId(), saved.getCategory().getId());

        // Проверяем, что запись действительно в БД
        List<Product> all = productRepository.findAll();
        assertEquals(1, all.size());
    }

    @Test
    void updateFromForm_shouldUpdateExistingProduct() {
        // Создаём
        ProductForm createForm = new ProductForm();
        createForm.setId("TEST02");
        createForm.setTitle("Ручка");
        createForm.setCost(new BigDecimal("50"));
        createForm.setQuantityInStock(10);
        createForm.setCategoryId(category.getId());
        productService.createFromForm(createForm);

        // Обновляем
        ProductForm updateForm = new ProductForm();
        updateForm.setId("TEST02");
        updateForm.setTitle("Ручка премиум");
        updateForm.setCost(new BigDecimal("150"));
        updateForm.setQuantityInStock(5);
        updateForm.setCategoryId(category.getId());

        Product updated = productService.updateFromForm("TEST02", updateForm);

        assertEquals("Ручка премиум", updated.getTitle());
        assertEquals(new BigDecimal("150"), updated.getCost());
        assertEquals(5, updated.getQuantityInStock());
    }
}
```

| Аннотация | Что делает |
|-----------|-----------|
| `@SpringBootTest` | Загружает полный Spring-контекст |
| `@ActiveProfiles("test")` | Использует H2 |
| `@Transactional` | Каждый тест — в транзакции, откат после |

> ⚠️ **`@Transactional` на тесте** — важно: без него данные из одного теста «протекут» в другой.

> 💡 **`@BeforeEach setUp()`** — вызывается **перед каждым** тестом. Используйте для очистки и подготовки данных.

---

## TestContainers

> 💡 **TestContainers** — библиотека, которая поднимает **реальный Docker-контейнер** с PostgreSQL во время тестов. Это самый близкий к продакшену способ тестирования.

**Зачем:**
- H2 не полностью совпадает с PostgreSQL.
- SQL, работающий в H2, может упасть в PostgreSQL (например, `bytea`, `serial4`, `ON DELETE SET NULL`).
- TestContainers даёт **настоящую** БД с настоящей версией.

### 5.1. Зависимости

В `pom.xml`:

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
```

### 5.2. Базовый абстрактный класс

`src/test/java/com/example/shop/AbstractIntegrationTest.java`:

```java
package com.example.shop;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@SpringBootTest
@Testcontainers
public abstract class AbstractIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.jpa.hibernate.ddl-auto", () -> "create-drop");
    }
}
```

### 5.3. Использование

`src/test/java/com/example/shop/service/OrderServiceContainerTest.java`:

```java
package com.example.shop.service;

import com.example.shop.AbstractIntegrationTest;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;

import static org.junit.jupiter.api.Assertions.assertNotNull;

class OrderServiceContainerTest extends AbstractIntegrationTest {

    @Autowired
    private OrderService orderService;

    @Test
    void containerStartsAndServiceWorks() {
        assertNotNull(orderService);
    }
}
```

### 5.4. Разбор

| Аннотация | Что делает |
|-----------|-----------|
| `@Testcontainers` | Активирует управление контейнерами |
| `@Container` | Помечает поле как контейнер (стартует/останавливается автоматически) |
| `@DynamicPropertySource` | Подставляет URL/логин/пароль контейнера в Spring-контекст |

> ⚠️ **Нужен установленный Docker.** Если Docker не запущен — тесты упадут.
>
> 💡 **Первый запуск медленный** (скачивает образ `postgres:16-alpine` ~ 80 МБ). Последующие — быстрые.

---

## Покрытие тестами

> 💡 **Покрытие (coverage)** — процент кода, который выполняется во время тестов. Хороший уровень — **70–80%**.

Добавьте плагин в `pom.xml`:

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Запустите:

```bash
./mvnw test
```

Отчёт появится в `target/site/jacoco/index.html`. Откройте в браузере — увидите, какие классы покрыты, а какие нет.

> 💡 **Что не нужно покрывать:**
> - Геттеры/сеттеры (Lombok их генерирует).
> - Тривиальные DTO.
> - Конфигурационные классы.
>
> **Что обязательно:**
> - Service (бизнес-логика).
> - Утилиты.
> - Сложные запросы в Repository.

---

## Проблема N+1

Откройте `application-dev.properties` и включите вывод SQL:

```properties
spring.jpa.show-sql=true
```

Теперь откройте `/products-jpa` (или любой endpoint, возвращающий список товаров). В консоли увидите:

```
select * from product limit 10;
select * from category where id = 1;
select * from category where id = 2;
select * from category where id = 3;
...
select * from category where id = 10;
```

**11 запросов** вместо одного. Это N+1 проблема: 1 запрос за списком + N запросов за связями.

> 💡 **Почему так:** `@ManyToOne(fetch = LAZY)` — категория загружается только когда её коснулись. А Thymeleaf обращается к `p.category.title` для каждого товара → 10 отдельных запросов.

---

## JOIN FETCH и @EntityGraph

### 8.1. JOIN FETCH через @Query

В `ProductRepository` добавьте:

```java
@Query("SELECT p FROM Product p JOIN FETCH p.category")
List<Product> findAllWithCategory();
```

Используйте в контроллере вместо `findAll()`:

```java
@GetMapping("/products-jpa")
public List<Product> products() {
    return productRepository.findAllWithCategory();
}
```

Теперь в логах один запрос:

```
select p.*, c.* from product p inner join category c on p.category_id = c.id;
```

### 8.2. @EntityGraph

Альтернатива — `@EntityGraph`:

```java
@EntityGraph(attributePaths = {"category"})
List<Product> findAll();
```

> 💡 **Разница:**
> - `JOIN FETCH` — явно пишем JPQL, полный контроль.
> - `@EntityGraph` — декларативно, Spring сам строит запрос.
>
> **Оба варианта решают N+1.** Выбирайте по вкусу.

### 8.3. Для Page с JOIN FETCH

> ⚠️ **Page + JOIN FETCH работает через `countQuery`.** Иначе Hibernate делает лишние запросы:

```java
@Query(value = "SELECT p FROM Product p JOIN FETCH p.category",
       countQuery = "SELECT COUNT(p) FROM Product p")
Page<Product> findAllWithCategory(Pageable pageable);
```

### 8.4. Проверка

Откройте `/products-jpa` — в логах должно быть **один** SQL-запрос вместо 11.

---

## Хранение файлов

Сейчас фото лежат в `bytea` внутри БД. Это плохо:

| Проблема | Последствие |
|----------|-------------|
| БД распухает | Бэкапы по гигабайту, долгие `pg_dump` |
| Каждый SELECT тащит фото | Медленные запросы даже без картинок |
| Фото нельзя отдать через CDN | Плохо масштабируется |

**Правильный подход:** файлы — на диск (или S3/MinIO), в БД — только **путь** к файлу.

---

## FileStorageService

### 10.1. Изменения в Entity

Добавьте в `Product` новое поле:

```java
@Column(name = "photo_path", length = 255)
private String photoPath;
```

И удалите `byte[] photo`. В DDL:

```sql
ALTER TABLE product ADD COLUMN photo_path varchar(255);
-- (позже, когда убедитесь, что всё работает)
-- ALTER TABLE product DROP COLUMN photo;
```

### 10.2. `service/FileStorageService.java`

```java
package com.example.shop.service;

import com.example.shop.util.FileUtils;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.UUID;

@Slf4j
@Service
public class FileStorageService {

    private final Path root;

    public FileStorageService(@Value("${app.upload-dir:uploads}") String uploadDir) {
        this.root = Paths.get(uploadDir).toAbsolutePath().normalize();
        try {
            Files.createDirectories(root);
        } catch (IOException e) {
            throw new IllegalStateException("Не удалось создать папку для загрузок: " + root, e);
        }
        log.info("Папка для загрузок: {}", root);
    }

    /**
     * Сохранить файл. Возвращает относительный путь (например, "products/abc-123.jpg").
     * Возвращает null, если файл пустой.
     */
    public String store(MultipartFile file, String subfolder) {
        if (file == null || file.isEmpty()) {
            return null;
        }

        if (!FileUtils.isImage(file)) {
            throw new IllegalArgumentException("Файл должен быть изображением");
        }

        String extension = FileUtils.getExtension(file.getOriginalFilename());
        String filename = UUID.randomUUID() + (extension.isEmpty() ? "" : "." + extension);

        Path folder = root.resolve(subfolder);
        Path target = folder.resolve(filename);

        try {
            Files.createDirectories(folder);
            Files.copy(file.getInputStream(), target, StandardCopyOption.REPLACE_EXISTING);
            log.info("Файл сохранён: {}", target);
            return subfolder + "/" + filename;
        } catch (IOException e) {
            throw new RuntimeException("Ошибка сохранения файла", e);
        }
    }

    /**
     * Удалить файл по относительному пути.
     */
    public void delete(String relativePath) {
        if (relativePath == null) {
            return;
        }
        try {
            Files.deleteIfExists(root.resolve(relativePath));
        } catch (IOException e) {
            log.warn("Не удалось удалить файл: {}", relativePath, e);
        }
    }

    /**
     * Прочитать файл по относительному пути.
     */
    public byte[] read(String relativePath) {
        if (relativePath == null) {
            return null;
        }
        try {
            Path file = root.resolve(relativePath);
            return Files.exists(file) ? Files.readAllBytes(file) : null;
        } catch (IOException e) {
            log.warn("Не удалось прочитать файл: {}", relativePath, e);
            return null;
        }
    }
}
```

### 10.3. Настройка `application-dev.properties`

```properties
app.upload-dir=uploads
```

Папка `uploads/` создастся при первом запуске рядом с приложением.

> 💡 **`@Value("${app.upload-dir:uploads}")`** — внедряет значение из properties. Если не задано — используется `uploads` по умолчанию.

### 10.4. Обновляем ProductService

```java
@Autowired
private FileStorageService fileStorageService;

@Transactional
public Product createFromForm(ProductForm form) {
    Product product = new Product();
    product.setId(form.getId());
    product.setTitle(ValidationUtils.normalize(form.getTitle()));
    product.setCost(form.getCost());
    product.setQuantityInStock(form.getQuantityInStock());
    product.setDescription(ValidationUtils.normalize(form.getDescription()));

    Category category = categoryRepository.findById(form.getCategoryId())
            .orElseThrow(() -> new IllegalArgumentException("Категория не найдена"));
    product.setCategory(category);

    // Сохраняем фото в файл, в БД пишем путь
    String path = fileStorageService.store(form.getPhotoFile(), "products");
    product.setPhotoPath(path);

    return productRepository.save(product);
}

@Transactional
public Product updateFromForm(String id, ProductForm form) {
    Product product = findById(id);
    // ...
    if (form.getPhotoFile() != null && !form.getPhotoFile().isEmpty()) {
        // Удалить старое фото
        fileStorageService.delete(product.getPhotoPath());
        // Сохранить новое
        product.setPhotoPath(fileStorageService.store(form.getPhotoFile(), "products"));
    }
    return productRepository.save(product);
}

@Transactional
public void deleteById(String id) {
    Product product = findById(id);
    fileStorageService.delete(product.getPhotoPath());
    productRepository.deleteById(id);
}
```

### 10.5. Обновляем ProductController

```java
@Autowired
private FileStorageService fileStorageService;

@GetMapping("/product/{id}/photo")
public ResponseEntity<byte[]> productPhoto(@PathVariable String id) {
    Product product = productService.findById(id);
    byte[] photo = fileStorageService.read(product.getPhotoPath());

    if (photo == null) {
        return ResponseEntity.notFound().build();
    }

    MediaType mediaType = detectImageType(photo);
    return ResponseEntity.ok()
            .contentType(mediaType)
            .header(HttpHeaders.CACHE_CONTROL, "max-age=86400")
            .body(photo);
}
```

### 10.6. `util/FileUtils.java` — обновляем

```java
public static String getExtension(String filename) {
    if (filename == null || !filename.contains(".")) {
        return "";
    }
    return filename.substring(filename.lastIndexOf('.') + 1).toLowerCase();
}
```

> 💡 **`UUID.randomUUID()`** — генерирует уникальный идентификатор. Так мы избегаем коллизий: если два пользователя загрузят `photo.jpg`, они не перезапишут друг друга.

### 10.7. `.gitignore`

Добавьте:

```
uploads/
```

> ⚠️ **Никогда не коммитьте папку `uploads/`.** Фото пользователей — не часть исходного кода.

---

## Миграция фото из БД в файлы

Одноразовый скрипт для существующих товаров. Создайте `util/PhotoMigrationRunner.java`:

```java
package com.example.shop.util;

import com.example.shop.entity.Product;
import com.example.shop.repository.ProductRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.UUID;

@Slf4j
@Component
public class PhotoMigrationRunner implements CommandLineRunner {

    @Autowired
    private ProductRepository productRepository;

    @Override
    public void run(String... args) throws Exception {
        Path root = Paths.get("uploads/products");
        Files.createDirectories(root);

        for (Product product : productRepository.findAll()) {
            if (product.getPhotoPath() == null && product.getPhoto() != null
                    && product.getPhoto().length > 0) {

                String filename = UUID.randomUUID() + ".jpg";
                Path target = root.resolve(filename);
                Files.write(target, product.getPhoto());

                product.setPhotoPath("products/" + filename);
                productRepository.save(product);

                log.info("Миграция фото: {} → {}", product.getId(), target);
            }
        }
    }
}
```

> ⚠️ **Одноразовый запуск.** После успешной миграции — удалите класс. Иначе он будет запускаться при каждом старте.

> 💡 **`CommandLineRunner`** — интерфейс, метод `run()` которого вызывается один раз при старте Spring Boot.

---

## Кэширование справочников

Категории, статусы и пункты выдачи меняются редко. Кэшируем их в памяти.

### 12.1. Включаем кэш

В `ShopApplication.java`:

```java
@SpringBootApplication
@EnableCaching
public class ShopApplication { ... }
```

В `application-dev.properties`:

```properties
spring.cache.type=simple
```

> 💡 **`simple`** — простой in-memory кэш (ConcurrentHashMap). В проде используют Redis или Caffeine.

### 12.2. Аннотации в CategoryService

```java
@Cacheable("categories")
public List<Category> findAll() {
    log.debug("Загрузка категорий из БД");
    return categoryRepository.findAll();
}

@CacheEvict(value = "categories", allEntries = true)
public Category save(Category category) {
    return categoryRepository.save(category);
}
```

| Аннотация | Что делает |
|-----------|-----------|
| `@Cacheable("categories")` | Первый вызов — идёт в БД. Последующие — из кэша |
| `@CacheEvict(...)` | Очищает кэш при изменении данных |

> 💡 **Не кэшируйте товары!** Их много, они меняются, и кэш быстро устаревает. Кэшируйте **справочники**.

### 12.3. Проверка

Откройте `/catalog` дважды. В логах:

```
Первый раз: Загрузка категорий из БД
Второй раз: (ничего)
```

---

## Проверка

### 13.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | `ProductRepositoryTest` проходит (`@DataJpaTest`) | |
| 2 | `ProductServiceIntegrationTest` проходит (`@SpringBootTest`) | |
| 3 | TestContainers поднимает контейнер при запуске | |
| 4 | Отчёт Jacoco показывает покрытие | |
| 5 | N+1 проблема решена (`JOIN FETCH`) | |
| 6 | `FileStorageService` сохраняет фото в `uploads/products/` | |
| 7 | Фото товаров открываются через `/product/{id}/photo` | |
| 8 | Папка `uploads/` в `.gitignore` | |
| 9 | При старте запускается `PhotoMigrationRunner` (один раз) | |
| 10 | Категории кэшируются (`@Cacheable`) | |
| 11 | При сохранении категории кэш очищается | |

### 13.2. Проверка N+1

1. Откройте `/products-jpa`.
2. Смотрите логи SQL.
3. Должен быть **один** запрос с `inner join`.

### 13.3. Проверка файлов

1. Создайте товар с фото.
2. Проверьте, что файл появился в `uploads/products/`.
3. Проверьте, что в БД `photo_path` заполнен, а `photo` пуст.
4. Откройте `/product/{id}/photo` — фото отображается.

### 13.4. Запуск всех тестов

```bash
./mvnw test
```

Отчёт в `target/site/jacoco/index.html`.

---

## Задание

1. **Напишите тесты для `CategoryRepository`**:
   - `@DataJpaTest`.
   - Проверьте `findByTitle`, `save`, `deleteById`.

2. **Напишите интеграционный тест для `OrderService`**:
   - `@SpringBootTest`.
   - Создайте заказ, проверьте, что он сохранён с позициями.
   - Проверьте, что `getCode` сгенерирован.

3. **Решите N+1 для `OrderProduct`**:
   - При выводе состава заказа не должно быть запроса на каждый `order_product.product`.
   - Используйте `JOIN FETCH p.product`.

4. **Создайте `FileStorageServiceTest`**:
   - Используйте `@TempDir` из JUnit 5.
   - Проверьте: сохранение, чтение, удаление файла.
   - Проверьте, что `store(null)` возвращает `null`.

5. **Расширьте `FileStorageService`**:
   - Метод `validateSize(MultipartFile, long maxBytes)` — проверка размера.
   - Метод `deleteAll(String subfolder)` — очистка папки.
   - Используйте `validateSize` в `ProductService` перед сохранением.

6. **Кэшируйте `Status` и `PickupPoint`**:
   - `@Cacheable("statuses")` в `StatusService`.
   - `@Cacheable("pickupPoints")` в `PickupPointService`.
   - Убедитесь, что кэш очищается при сохранении.

7. **Настройте Jacoco**:
   - Добейтесь покрытия **не менее 60%**.
   - Покажите отчёт преподавателю.

8. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 10](Lesson10.md) |

---

## 💡 Что дальше (анонс урока 10)

На следующем занятии:
- **Отчёты и экспорт** — CSV, Excel (Apache POI).
- **Графики** на дашборде (Chart.js).
- **Фильтры по датам** в админ-заказах.
- **Email-уведомления** при смене статуса заказа.
- **Пагинация** во всех админских таблицах.

---

## 📚 Полезные ссылки

- 🇬🇧 [Spring Boot: Testing](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
- 🇬🇧 [Baeldung: @DataJpaTest](https://www.baeldung.com/spring-boot-testing)
- 🇬🇧 [Baeldung: TestContainers](https://www.baeldung.com/spring-boot-testcontainers-integration-test)
- 🇬🇧 [Baeldung: JPA N+1 problem](https://www.baeldung.com/hibernate-join-fetch)
- 🇬🇧 [Baeldung: Spring Cache](https://www.baeldung.com/spring-cache-tutorial)
- 🇬🇧 [Baeldung: File Upload](https://www.baeldung.com/spring-file-upload)
- 🇷🇺 [Хабр: Проблема N+1 в JPA](https://habr.com/ru/articles/) *(поиск: «jpa n+1 join fetch»)*
- 🇷🇺 [Хабр: TestContainers на практике](https://habr.com/ru/articles/) *(поиск: «testcontainers spring boot»)*
