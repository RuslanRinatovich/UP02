# Занятие 3. Формы, валидация, архитектура и CRUD

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 4](Lesson4.md)

## План
1. [Введение: что было и что будет](#введение)
2. [GET vs POST: почему форма — это POST](#get-vs-post)
3. [DTO для формы товара](#dto-для-формы-товара)
4. [Bean Validation: правила валидации](#bean-validation)
5. [Слои приложения: зачем нужна архитектура](#слои-приложения)
6. [Целевая структура пакетов](#целевая-структура-пакетов)
7. [Шаг 1. Создаём пакеты](#шаг-1-создаём-пакеты)
8. [Шаг 2. Переносим Entity](#шаг-2-переносим-entity)
9. [Шаг 3. Переносим Repository](#шаг-3-переносим-repository)
10. [Шаг 4. Переносим DTO](#шаг-4-переносим-dto)
11. [Шаг 5. Создаём Service](#шаг-5-создаём-service)
12. [Шаг 6. Пакет util — утилиты и хелперы](#шаг-6-пакет-util)
13. [Шаг 7. Переносим Controller](#шаг-7-переносим-controller)
14. [Шаг 8. Форма добавления товара](#шаг-8-форма-добавления-товара)
15. [Шаг 9. Загрузка изображения](#шаг-9-загрузка-изображения)
16. [Шаг 10. Редактирование товара](#шаг-10-редактирование-товара)
17. [Шаг 11. Удаление товара](#шаг-11-удаление-товара)
18. [Шаг 12. Проверка и финальный чек-лист](#шаг-12-проверка)
19. [Задание](#задание)

---

## Введение

На прошлом занятии мы:
- Создали Entity для `Category` и `Product`.
- Научились доставать данные через `Repository`.
- Вывели товары в JSON и HTML.

**Проблема:** мы только **читаем** данные, а код лежит в одном пакете. Интернет-магазин — это не только просмотр, но и **добавление**, **редактирование**, **удаление**, а ещё код должен быть **организованным**.

**Сегодня (большое занятие из двух частей):**
1. **Часть 1 — архитектура.** Разложим проект по слоям и пакетам.
2. **Часть 2 — CRUD.** Научимся создавать формы, валидировать данные, добавлять/редактировать/удалять товары через веб-интерфейс.

> 💡 Это занятие длиннее обычного, потому что мы делаем сразу два больших дела. Но каждое из них — необходимо для нормальной разработки. Если вы быстро справитесь с одной частью — переходите к следующей, задания в конце хватит всем.

---

## GET vs POST

> 💡 **GET vs POST**

| Метод | Когда используется | Что происходит |
|-------|-------------------|----------------|
| **GET** | Просмотр страниц, поиск | Параметры передаются в URL (`/products?category=1`) |
| **POST** | Создание, изменение данных | Данные передаются в **теле запроса**, не видны в URL |

**Почему нельзя использовать GET для добавления товара?**
1. **Идемпотентность:** GET можно вызывать сколько угодно раз — ничего не изменится. POST меняет состояние.
2. **Безопасность:** данные формы не видны в истории браузера и логах сервера.
3. **Размер:** GET ограничен длиной URL (~2000 символов), POST — нет.

**Как это выглядит в коде:**
```java
@GetMapping("/products/new")   // показать форму
public String showForm(Model model) { ... }

@PostMapping("/products/new")  // обработать отправку
public String saveProduct(@Valid ProductForm form, BindingResult result) { ... }
```

---

## DTO для формы товара

> 💡 **DTO (Data Transfer Object)** — это простой Java-класс, который используется для передачи данных между слоями. В нашем случае DTO — это то, что приходит из HTML-формы.

**Почему не использовать Entity напрямую?**
1. Entity привязан к таблице БД. Если форма не заполняет все поля — будут ошибки.
2. В форме могут быть поля, которых нет в Entity (например, `photoFile` для загрузки).
3. DTO позволяет добавить специфичные для формы поля и валидацию.

Создайте файл `src/main/java/com/example/shop/dto/ProductForm.java`:

```java
package com.example.shop.dto;

import jakarta.validation.constraints.*;
import lombok.Data;
import org.springframework.web.multipart.MultipartFile;

import java.math.BigDecimal;

@Data
public class ProductForm {

    @NotBlank(message = "Артикул обязателен")
    @Size(min = 3, max = 10, message = "Артикул от 3 до 10 символов")
    @Pattern(regexp = "^[A-Z0-9]+$", message = "Только заглавные буквы и цифры")
    private String id;

    @NotBlank(message = "Название обязательно")
    @Size(min = 2, max = 100, message = "Название от 2 до 100 символов")
    private String title;

    @NotNull(message = "Цена обязательна")
    @DecimalMin(value = "0.01", message = "Цена должна быть больше 0")
    private BigDecimal cost;

    @NotNull(message = "Количество обязательно")
    @Min(value = 0, message = "Количество не может быть отрицательным")
    private Integer quantityInStock;

    @Size(max = 500, message = "Описание до 500 символов")
    private String description;

    @NotNull(message = "Категория обязательна")
    private Integer categoryId;

    private MultipartFile photoFile;
}
```

---

## Bean Validation

> 💡 **Bean Validation** — это стандарт Java (JSR 380), который позволяет описывать правила валидации прямо в классах через аннотации. Spring Boot автоматически подключает его, если добавить зависимость `spring-boot-starter-validation`.

| Аннотация | Что проверяет |
|-----------|--------------|
| `@NotBlank` | Строка не пустая и не состоит из пробелов |
| `@NotNull` | Значение не `null` |
| `@Size(min, max)` | Длина строки в диапазоне |
| `@Pattern(regexp)` | Соответствие регулярному выражению |
| `@Min(value)` | Число ≥ указанного |
| `@DecimalMin(value)` | Число ≥ указанного (для `BigDecimal`) |
| `@Email` | Корректный email |
| `@Positive` | Число > 0 |

**Добавьте зависимость в `pom.xml`** (если её нет):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

> ⚠️ В Spring Boot 3.x `validation` не входит в `web` автоматически. Нужно добавить явно.

**`message`** — это текст ошибки, который увидит пользователь. Если не указать, Spring покажет сообщение по умолчанию на английском.

---

## Слои приложения

До этого момента мы писали код «как удобно»: все классы лежали в одном пакете `com.example.shop`. Для учебных примеров это работало, но:

1. **Непонятно, где что искать.** Через 20 классов в одной папке вы уже путаетесь.
2. **Нет разделения ответственности.** Контроллер сам лезет в репозиторий, сам преобразует DTO в Entity, сам валидирует.
3. **Сложно тестировать.** Нельзя протестировать логику отдельно от веб-слоя.
4. **Нельзя переиспользовать.** Логика «создать товар» нужна и в веб-контроллере, и в REST API, и в фоновой задаче.

> 💡 **Классическая архитектура Spring-приложения (слоистая)**

```
┌─────────────────────────────────────────┐
│  Controller (веб-слой)                  │  ← принимает HTTP, возвращает HTML/JSON
├─────────────────────────────────────────┤
│  Service (бизнес-логика)                │  ← правила, транзакции, преобразования
├─────────────────────────────────────────┤
│  Repository (доступ к данным)           │  ← SQL-запросы (генерируются Spring)
├─────────────────────────────────────────┤
│  Entity (модель БД)                     │  ← таблицы
└─────────────────────────────────────────┘
       ↑                ↑
       │                │
   DTO (передача)   util (хелперы)
```

**Правила:**
- **Controller** не знает про Entity — он работает с DTO.
- **Controller** не лезет в Repository напрямую — только через Service.
- **Service** знает про Repository и Entity, но не знает про HTTP.
- **Repository** знает только про Entity.
- **Entity** не знает ни про кого.
- **util** — статические утилиты и хелперы, которые может использовать **любой** слой.

---

## Целевая структура пакетов

```
com.example.shop/
├── ShopApplication.java
│
├── entity/                      ← сущности БД
│   ├── Category.java
│   └── Product.java
│
├── repository/                  ← интерфейсы Spring Data JPA
│   ├── CategoryRepository.java
│   └── ProductRepository.java
│
├── dto/                         ← DTO для форм и API
│   ├── ProductForm.java
│   └── CategoryForm.java
│
├── service/                     ← бизнес-логика
│   ├── ProductService.java
│   └── CategoryService.java
│
├── controller/                  ← веб-слой
│   ├── ProductController.java
│   ├── CategoryController.java
│   └── admin/
│       └── ProductAdminController.java
│
├── util/                        ← утилиты и хелперы
│   ├── FileUtils.java
│   ├── PriceUtils.java
│   └── ValidationUtils.java
│
└── config/                      ← конфигурация (пока пусто)
```

> 💡 **Почему пакеты по слоям, а не по фичам?** Для учебного проекта слоистая структура понятнее. В больших проектах часто используют **feature-based** (`product/`, `order/`, `user/`), где внутри каждой фичи свои `controller`, `service`, `repository`. Мы выберем слоистую — она проще для восприятия.

---

## Шаг 1. Создаём пакеты

В IntelliJ:
1. Правой кнопкой на `com.example.shop` → **New → Package**.
2. Создайте по очереди: `entity`, `repository`, `dto`, `service`, `controller`, `controller.admin`, `util`, `config`.

> 💡 В IntelliJ пакеты отображаются как дерево. Пакет `controller.admin` — это вложенный пакет, он будет внутри `controller`.

---

## Шаг 2. Переносим Entity

1. Выделите `Category.java` → **F6** (Refactor → Move) → выберите `com.example.shop.entity`.
2. То же для `Product.java`.

**Не забудьте добавить package в начало каждого файла.**

**Пример `entity/Category.java`:**

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "category")
@Data
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "title", nullable = false, length = 200)
    private String title;
}
```

**Пример `entity/Product.java`:**

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

import java.math.BigDecimal;

@Entity
@Table(name = "product")
@Data
public class Product {

    @Id
    @Column(name = "id", length = 10)
    private String id;

    @Column(name = "title", nullable = false, length = 100)
    private String title;

    @Column(name = "cost", nullable = false)
    private BigDecimal cost;

    @Column(name = "max_discount_amount")
    private Integer maxDiscountAmount;

    @Column(name = "discount_amount")
    private Integer discountAmount;

    @Column(name = "quantity_in_stock", nullable = false)
    private Integer quantityInStock;

    @Column(name = "description", columnDefinition = "text")
    private String description;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private Category category;

    @Column(name = "photo")
    private byte[] photo;
}
```

> ⚠️ **Тип `BigDecimal` для `cost`:** в БД `cost` — это `numeric`. В Java для денег используется `BigDecimal`, а не `double`, чтобы избежать ошибок округления.

---

## Шаг 3. Переносим Repository

1. Выделите `CategoryRepository.java` → **F6** → `com.example.shop.repository`.
2. То же для `ProductRepository.java`.

```java
package com.example.shop.repository;

import com.example.shop.entity.Category;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CategoryRepository extends JpaRepository<Category, Integer> {
}
```

```java
package com.example.shop.repository;

import com.example.shop.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, String> {
}
```

---

## Шаг 4. Переносим DTO

1. Выделите `ProductForm.java` → **F6** → `com.example.shop.dto`.

Код DTO уже приведён выше — убедитесь, что package — `com.example.shop.dto`.

---

## Шаг 5. Создаём Service

> 💡 **Service — это сердце приложения.** Здесь живёт бизнес-логика: «что значит создать товар», «как проверить, что артикул уникален», «какие поля обязательны». Controller только принимает запрос и передаёт его в Service.

Создайте файл `src/main/java/com/example/shop/service/ProductService.java`:

```java
package com.example.shop.service;

import com.example.shop.dto.ProductForm;
import com.example.shop.entity.Category;
import com.example.shop.entity.Product;
import com.example.shop.repository.CategoryRepository;
import com.example.shop.repository.ProductRepository;
import com.example.shop.util.FileUtils;
import com.example.shop.util.ValidationUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private CategoryRepository categoryRepository;

    public List<Product> findAll() {
        return productRepository.findAll();
    }

    public Product findById(String id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Товар не найден: " + id));
    }

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

        // Используем утилиту для чтения файла
        product.setPhoto(FileUtils.readBytes(form.getPhotoFile()));

        return productRepository.save(product);
    }

    @Transactional
    public Product updateFromForm(String id, ProductForm form) {
        Product product = findById(id);

        product.setTitle(ValidationUtils.normalize(form.getTitle()));
        product.setCost(form.getCost());
        product.setQuantityInStock(form.getQuantityInStock());
        product.setDescription(ValidationUtils.normalize(form.getDescription()));

        Category category = categoryRepository.findById(form.getCategoryId())
                .orElseThrow(() -> new IllegalArgumentException("Категория не найдена"));
        product.setCategory(category);

        // Фото обновляем только если загружено новое
        byte[] newPhoto = FileUtils.readBytes(form.getPhotoFile());
        if (newPhoto != null) {
            product.setPhoto(newPhoto);
        }

        return productRepository.save(product);
    }

    @Transactional
    public void deleteById(String id) {
        productRepository.deleteById(id);
    }
}
```

Создайте `service/CategoryService.java`:

```java
package com.example.shop.service;

import com.example.shop.entity.Category;
import com.example.shop.repository.CategoryRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class CategoryService {

    @Autowired
    private CategoryRepository categoryRepository;

    public List<Category> findAll() {
        return categoryRepository.findAll();
    }

    public Category findById(Integer id) {
        return categoryRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Категория не найдена: " + id));
    }

    public Category save(Category category) {
        return categoryRepository.save(category);
    }
}
```

| Аннотация | Что делает |
|-----------|-----------|
| `@Service` | Помечает класс как сервис |
| `@Transactional` | Все действия внутри метода — одна транзакция |
| `@Autowired` | Внедрение зависимостей |

---

## Шаг 6. Пакет util

> 💡 **util (utility)** — пакет для **утилитных классов**: статические методы, которые выполняют одну конкретную задачу и не имеют состояния. Их можно вызывать из любого слоя: из Service, из Controller, из другого util.

**Правила для util-классов:**
1. Класс — `final`, конструктор — `private` (нельзя создать экземпляр).
2. Все методы — `static`.
3. У каждого метода — одна ответственность.
4. Не зависят от Spring-контекста (не `@Service`, не `@Component`).
5. Легко тестируются (чистые функции).

### 6.1. `util/FileUtils.java`

```java
package com.example.shop.util;

import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

public final class FileUtils {

    private FileUtils() {
    }

    /**
     * Читает содержимое загруженного файла в массив байтов.
     * Возвращает null, если файл не был загружен.
     */
    public static byte[] readBytes(MultipartFile file) {
        if (file == null || file.isEmpty()) {
            return null;
        }
        try {
            return file.getBytes();
        } catch (IOException e) {
            throw new RuntimeException("Не удалось прочитать файл: " + file.getOriginalFilename(), e);
        }
    }

    /**
     * Проверяет, что файл является изображением.
     */
    public static boolean isImage(MultipartFile file) {
        if (file == null || file.isEmpty()) {
            return false;
        }
        String contentType = file.getContentType();
        return contentType != null && contentType.startsWith("image/");
    }
}
```

### 6.2. `util/PriceUtils.java`

```java
package com.example.shop.util;

import java.math.BigDecimal;
import java.math.RoundingMode;

public final class PriceUtils {

    private PriceUtils() {
    }

    /**
     * Вычисляет итоговую цену товара с учётом скидки.
     */
    public static BigDecimal applyDiscount(BigDecimal basePrice, Integer discountAmount) {
        if (basePrice == null) {
            return BigDecimal.ZERO;
        }
        if (discountAmount == null || discountAmount <= 0) {
            return basePrice.setScale(2, RoundingMode.HALF_UP);
        }
        BigDecimal factor = BigDecimal.ONE.subtract(
                BigDecimal.valueOf(discountAmount).divide(BigDecimal.valueOf(100))
        );
        return basePrice.multiply(factor).setScale(2, RoundingMode.HALF_UP);
    }

    /**
     * Форматирует цену в вид "1 234,56 ₽".
     */
    public static String format(BigDecimal price) {
        if (price == null) {
            return "—";
        }
        return String.format("%,.2f ₽", price);
    }
}
```

### 6.3. `util/ValidationUtils.java`

```java
package com.example.shop.util;

import java.util.regex.Pattern;

public final class ValidationUtils {

    private static final Pattern ARTICLE_PATTERN = Pattern.compile("^[A-Z0-9]{3,10}$");
    private static final Pattern EMAIL_PATTERN =
            Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$");

    private ValidationUtils() {
    }

    public static boolean isValidArticle(String article) {
        return article != null && ARTICLE_PATTERN.matcher(article).matches();
    }

    public static boolean isValidEmail(String email) {
        return email != null && EMAIL_PATTERN.matcher(email).matches();
    }

    /**
     * Убирает пробелы в начале/конце, а также двойные пробелы внутри.
     */
    public static String normalize(String input) {
        if (input == null) {
            return null;
        }
        return input.trim().replaceAll("\\s+", " ");
    }
}
```

### 6.4. Что **не** должно быть в util

| ❌ Плохо | ✅ Хорошо |
|---------|----------|
| Класс с `@Service` | Класс `final`, все методы `static` |
| Метод, который лезет в БД | Чистая функция |
| Большой класс на 500 строк | Небольшие классы по 50–100 строк |

---

## Шаг 7. Переносим Controller

1. `CategoryController.java`, `ProductController.java` → **F6** → `com.example.shop.controller`.
2. Создайте новый `ProductAdminController.java` в `com.example.shop.controller.admin`.

`controller/ProductController.java`:

```java
package com.example.shop.controller;

import com.example.shop.entity.Product;
import com.example.shop.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping("/products-jpa")
    public List<Product> products() {
        return productService.findAll();
    }
}
```

---

## Шаг 8. Форма добавления товара

> 💡 **`@RestController` vs `@Controller`**
>
> | Аннотация | Что возвращает метод | Как обрабатывается |
> |-----------|---------------------|-------------------|
> | `@Controller` | Имя HTML-шаблона или `ModelAndView` | Spring ищет файл в `templates/` и рендерит его через Thymeleaf |
> | `@RestController` | Данные (объект, список, строку) | Spring сериализует их в JSON и отдаёт как HTTP-ответ |

Создайте файл `controller/admin/ProductAdminController.java`:

```java
package com.example.shop.controller.admin;

import com.example.shop.dto.ProductForm;
import com.example.shop.service.CategoryService;
import com.example.shop.service.ProductService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/admin/products")
public class ProductAdminController {

    @Autowired
    private ProductService productService;

    @Autowired
    private CategoryService categoryService;

    // Показать форму добавления
    @GetMapping("/new")
    public String showForm(Model model) {
        model.addAttribute("productForm", new ProductForm());
        model.addAttribute("categories", categoryService.findAll());
        return "admin/product-form";
    }

    // Обработать отправку формы
    @PostMapping("/new")
    public String saveProduct(@Valid @ModelAttribute("productForm") ProductForm form,
                              BindingResult result,
                              Model model) {
        // Если есть ошибки валидации — вернуть форму с ошибками
        if (result.hasErrors()) {
            model.addAttribute("categories", categoryService.findAll());
            return "admin/product-form";
        }

        productService.createFromForm(form);
        return "redirect:/products-jpa-page";
    }
}
```

| Код | Что делает |
|-----|-----------|
| `@RequestMapping("/admin/products")` | Базовый URL для всех методов контроллера |
| `@GetMapping("/new")` | Показать форму |
| `@PostMapping("/new")` | Обработать отправку |
| `@Valid` | Запускает валидацию Bean Validation |
| `@ModelAttribute("productForm")` | Связывает данные из формы с объектом |
| `BindingResult` | Содержит результаты валидации |
| `result.hasErrors()` | Проверяет, есть ли ошибки |
| `redirect:/products-jpa-page` | Перенаправление (PRG-паттерн) |

> 💡 **PRG (Post-Redirect-Get):** после POST-запроса делаем redirect на GET-страницу. Это предотвращает повторную отправку формы при обновлении страницы (F5).

> ⚠️ **Важно:** `BindingResult` должен идти **сразу после** валидируемого объекта.

### 8.1. Thymeleaf-форма

Создайте файл `src/main/resources/templates/admin/product-form.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Добавить товар</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; max-width: 600px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, select, textarea { width: 100%; padding: 8px; box-sizing: border-box; }
        .error { color: red; font-size: 14px; }
        .is-invalid { border-color: red; }
        button { background: #28a745; color: white; padding: 10px 20px; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Добавить товар</h1>

    <form th:action="@{/admin/products/new}" th:object="${productForm}" method="post" enctype="multipart/form-data">
        <div class="form-group">
            <label>Артикул *</label>
            <input type="text" th:field="*{id}" th:classappend="${#fields.hasErrors('id')} ? 'is-invalid'"/>
            <span class="error" th:if="${#fields.hasErrors('id')}" th:errors="*{id}"></span>
        </div>

        <div class="form-group">
            <label>Название *</label>
            <input type="text" th:field="*{title}" th:classappend="${#fields.hasErrors('title')} ? 'is-invalid'"/>
            <span class="error" th:if="${#fields.hasErrors('title')}" th:errors="*{title}"></span>
        </div>

        <div class="form-group">
            <label>Цена *</label>
            <input type="number" step="0.01" th:field="*{cost}" th:classappend="${#fields.hasErrors('cost')} ? 'is-invalid'"/>
            <span class="error" th:if="${#fields.hasErrors('cost')}" th:errors="*{cost}"></span>
        </div>

        <div class="form-group">
            <label>Количество на складе *</label>
            <input type="number" th:field="*{quantityInStock}" th:classappend="${#fields.hasErrors('quantityInStock')} ? 'is-invalid'"/>
            <span class="error" th:if="${#fields.hasErrors('quantityInStock')}" th:errors="*{quantityInStock}"></span>
        </div>

        <div class="form-group">
            <label>Категория *</label>
            <select th:field="*{categoryId}" th:classappend="${#fields.hasErrors('categoryId')} ? 'is-invalid'">
                <option value="">-- Выберите категорию --</option>
                <option th:each="cat : ${categories}" 
                        th:value="${cat.id}" 
                        th:text="${cat.title}"></option>
            </select>
            <span class="error" th:if="${#fields.hasErrors('categoryId')}" th:errors="*{categoryId}"></span>
        </div>

        <div class="form-group">
            <label>Описание</label>
            <textarea th:field="*{description}" rows="3"></textarea>
        </div>

        <div class="form-group">
            <label>Фото товара</label>
            <input type="file" name="photoFile" accept="image/*"/>
        </div>

        <button type="submit">Сохранить</button>
        <a th:href="@{/products-jpa-page}" style="margin-left: 10px;">Отмена</a>
    </form>
</body>
</html>
```

> 💡 **Thymeleaf (коротко)**
>
> Thymeleaf — это серверный **шаблонизатор**: он берёт обычный HTML-файл и динамически подставляет в него данные из Java-контроллера. Шаблон можно открыть прямо в браузере (без сервера) — браузер проигнорирует `th:*` атрибуты.
>
> | Атрибут | Что делает |
> |---------|-----------|
> | `th:action` | URL для отправки формы |
> | `th:object` | Объект, в который связываются поля |
> | `th:field` | Связывает поле с свойством объекта |
> | `th:each` | Цикл по коллекции |
> | `th:if` | Показать блок, если условие истинно |
> | `th:errors` | Вывести сообщение об ошибке |
> | `#fields.hasErrors('field')` | Проверить, есть ли ошибка у поля |

---

## Шаг 9. Загрузка изображения

> 💡 В таблице `product` есть колонка `photo` типа `bytea` (бинарные данные). Мы уже добавили `MultipartFile photoFile` в DTO и `enctype="multipart/form-data"` в форму.

### 9.1. Настройте `application.properties`

```properties
spring.servlet.multipart.max-file-size=5MB
spring.servlet.multipart.max-request-size=5MB
```

### 9.2. Проверьте чтение файла

В `ProductService.createFromForm` уже есть строка:

```java
product.setPhoto(FileUtils.readBytes(form.getPhotoFile()));
```

`FileUtils.readBytes` вернёт `null`, если файл не загружен — это нормально, колонка `photo` nullable.

---

## Шаг 10. Редактирование товара

### 10.1. Добавьте методы в `ProductAdminController`

```java
// Показать форму редактирования
@GetMapping("/edit/{id}")
public String showEditForm(@PathVariable String id, Model model) {
    Product product = productService.findById(id);

    ProductForm form = new ProductForm();
    form.setId(product.getId());
    form.setTitle(product.getTitle());
    form.setCost(product.getCost());
    form.setQuantityInStock(product.getQuantityInStock());
    form.setDescription(product.getDescription());
    form.setCategoryId(product.getCategory().getId());

    model.addAttribute("productForm", form);
    model.addAttribute("categories", categoryService.findAll());
    model.addAttribute("editMode", true);
    return "admin/product-form";
}

// Обработать редактирование
@PostMapping("/edit/{id}")
public String updateProduct(@PathVariable String id,
                            @Valid @ModelAttribute("productForm") ProductForm form,
                            BindingResult result,
                            Model model) {
    if (result.hasErrors()) {
        model.addAttribute("categories", categoryService.findAll());
        model.addAttribute("editMode", true);
        return "admin/product-form";
    }

    productService.updateFromForm(id, form);
    return "redirect:/products-jpa-page";
}
```

> 💡 **`@PathVariable`** — извлекает значение из URL. В `/edit/{id}` фигурные скобки означают переменную, которая передаётся в параметр метода.

### 10.2. Обновите шаблон `product-form.html`

Добавьте **скрытое поле** и **измените action формы** в зависимости от режима:

```html
<form th:action="${editMode != null && editMode} 
                    ? @{/admin/products/edit/{id}(id=*{id})} 
                    : @{/admin/products/new}" 
      th:object="${productForm}" 
      method="post" 
      enctype="multipart/form-data">

    <!-- Артикул: только для чтения при редактировании -->
    <div class="form-group">
        <label>Артикул *</label>
        <input type="text" th:field="*{id}" 
               th:readonly="${editMode != null && editMode}"
               th:classappend="${#fields.hasErrors('id')} ? 'is-invalid'"/>
        <span class="error" th:if="${#fields.hasErrors('id')}" th:errors="*{id}"></span>
    </div>

    <!-- ... остальные поля без изменений ... -->
</form>
```

> ⚠️ При редактировании артикул (id) менять нельзя — это первичный ключ. Поэтому поле `readonly`.

### 10.3. Добавьте кнопку «Редактировать» в список товаров

В `templates/products-jpa.html`:

```html
<td>
    <a th:href="@{/admin/products/edit/{id}(id=${p.id})}">Редактировать</a>
</td>
```

---

## Шаг 11. Удаление товара

### 11.1. Метод в контроллере

```java
@PostMapping("/delete/{id}")
public String deleteProduct(@PathVariable String id) {
    productService.deleteById(id);
    return "redirect:/products-jpa-page";
}
```

> 💡 **Почему POST, а не GET?** Удаление меняет состояние. GET-запрос может быть случайно вызван браузером (например, префетчем), и товар удалится. Только POST.

### 11.2. Кнопка удаления в списке

В `templates/products-jpa.html`:

```html
<td>
    <a th:href="@{/admin/products/edit/{id}(id=${p.id})}">Редактировать</a>
    <form th:action="@{/admin/products/delete/{id}(id=${p.id})}" method="post" style="display:inline">
        <button type="submit" onclick="return confirm('Удалить товар?')">Удалить</button>
    </form>
</td>
```

> 💡 **`onclick="return confirm(...)"`** — встроенное подтверждение браузера. Если пользователь нажмёт «Отмена», форма не отправится.

---

## Шаг 12. Проверка

### 12.1. Убедитесь, что всё компилируется

**Build → Rebuild Project** (`Ctrl+Shift+F9`). Ошибок быть не должно.

### 12.2. Запустите приложение

| URL | Что должно быть |
|-----|----------------|
| `http://localhost:8080/products-jpa` | JSON с товарами |
| `http://localhost:8080/categories` | JSON с категориями |
| `http://localhost:8080/products-jpa-page` | HTML-таблица с товарами |
| `http://localhost:8080/admin/products/new` | Форма добавления |

### 12.3. Проверьте сценарии

1. **Добавление:** откройте `/admin/products/new`, заполните, сохраните. Проверьте в списке.
2. **Валидация:** попробуйте ввести неверные данные (например, `abc` в артикул) — увидите ошибки.
3. **Редактирование:** нажмите «Редактировать» рядом с товаром, измените цену, сохраните.
4. **Удаление:** нажмите «Удалить», подтвердите — товар исчезнет.

### 12.4. Финальный чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | Все классы разложены по пакетам (`entity`, `repository`, `dto`, `service`, `controller`, `util`) | |
| 2 | `ProductService` используется в контроллерах | |
| 3 | `FileUtils`, `PriceUtils`, `ValidationUtils` созданы | |
| 4 | Форма добавления работает | |
| 5 | Валидация показывает ошибки | |
| 6 | Загрузка фото работает | |
| 7 | Редактирование работает | |
| 8 | Удаление работает | |
| 9 | После POST идёт redirect (PRG) | |

---

## Итоговая структура

```
com.example.shop/
├── ShopApplication.java
├── config/
├── controller/
│   ├── CategoryController.java
│   ├── ProductController.java
│   └── admin/
│       └── ProductAdminController.java
├── dto/
│   ├── CategoryForm.java
│   └── ProductForm.java
├── entity/
│   ├── Category.java
│   └── Product.java
├── repository/
│   ├── CategoryRepository.java
│   └── ProductRepository.java
├── service/
│   ├── CategoryService.java
│   └── ProductService.java
└── util/
    ├── FileUtils.java
    ├── PriceUtils.java
    └── ValidationUtils.java
```

### Поток данных

```
Браузер → Controller → Service → Repository → БД
                    ↓         ↓
                  DTO       util
```

---

## Задание

1. **Создайте `ManufacturerService` и `ManufacturerController`**:
   - Service по образцу `CategoryService`.
   - `GET /manufacturers` — JSON.
   - `GET /manufacturers-page` — HTML.

2. **Создайте `ManufacturerAdminController`**:
   - `GET /admin/manufacturers/new` — форма.
   - `POST /admin/manufacturers/new` — сохранение.
   - DTO `ManufacturerForm` с `@NotBlank`, `@Size(min=2, max=200)`.

3. **Создайте `util/StringUtils.java`**:
   - `truncate(String, int)` — обрезает строку до N символов с `…`.
   - `capitalize(String)` — первая буква заглавная.
   - `isEmpty(String)` — null или пустая строка.
   Используйте `truncate` в `ProductService` для обрезки `description` до 100 символов при сохранении.

4. **Добавьте редактирование категорий**:
   - `GET /admin/categories/edit/{id}` — форма.
   - `POST /admin/categories/edit/{id}` — сохранение.
   - `POST /admin/categories/delete/{id}` — удаление.

5. **Рефакторинг**: уберите все прямые вызовы `Repository` из контроллеров. Только через Service.

6. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 4](Lesson4.md) |

---

## 💡 Что дальше (анонс урока 4)

На следующем занятии:
- Добавим **Spring Security**.
- Создадим `User` entity и `UserRepository`.
- Настроим **BCrypt** для хеширования паролей.
- Ограничим доступ к `/admin/**` только для ролей `ADMIN` и `MANAGER`.
- Сделаем **страницу входа** и **кнопку выхода**.
- В `util` появится `SecurityUtils` — для получения текущего пользователя.
