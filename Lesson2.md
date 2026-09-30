# Занятие 2. Spring Data JPA, Entity, Repository и CRUD

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 3](Lesson3.md)

## План
1. [Введение: что мы делали и куда идём](#введение)
2. [Что такое JPA и Spring Data JPA](#что-такое-jpa-и-spring-data-jpa)
3. [Создание первой Entity: Category](#создание-первой-entity-category)
4. [Создание Repository](#создание-repository)
5. [Первый контроллер с JPA](#первый-контроллер-с-jpa)
6. [Entity для Product: связи ManyToOne](#entity-для-product-связи-manytoone)
7. [Thymeleaf-шаблон со связями](#thymeleaf-шаблон-со-связями)
8. [Задание](#задание)

---

## Введение

На прошлом занятии мы:
- Создали БД `demoN` из SQL-скрипта.
- Запустили Spring Boot приложение.
- Научились доставать данные через `JdbcTemplate` и выводить их в JSON и HTML.

**Проблема `JdbcTemplate`:** мы пишем SQL-запросы руками в виде строк. Если изменится название колонки или таблицы, приложение упадёт. Нет проверки типов на этапе компиляции. Много boilerplate-кода.

**Решение:** **Spring Data JPA**. Мы создаём Java-классы, которые «знают», как связаны с таблицами БД, и Spring сам генерирует SQL.

---

## Что такое JPA и Spring Data JPA

> 💡 **JPA (Java Persistence API)** — это стандарт Java для работы с базами данных через объекты. Вместо того чтобы писать `SELECT * FROM product`, вы пишете `productRepository.findAll()`. JPA — это «мостик» между Java-объектами и таблицами БД.

> 💡 **Spring Data JPA** — это надстройка над JPA, которая убирает почти весь boilerplate-код. Вы создаёте интерфейс `ProductRepository extends JpaRepository<Product, String>` и получаете десятки готовых методов (`save`, `findAll`, `findById`, `deleteById`) бесплатно .

**Как это работает:**
1. Вы создаёте класс `Product` с аннотацией `@Entity` — Spring понимает, что этот класс = таблица `product`.
2. Вы создаёте интерфейс `ProductRepository extends JpaRepository<Product, String>` — Spring **автоматически** генерирует реализацию всех методов.
3. В контроллере вы вызываете `productRepository.findAll()` — Spring выполняет `SELECT * FROM product` и возвращает `List<Product>`.

---

## Создание первой Entity: Category

Начнём с простой таблицы — `category`.

### 3.1. Создание класса

Создайте файл `src/main/java/com/example/shop/Category.java`:

```java
package com.example.shop;

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

### 3.2. Разбор аннотаций

| Аннотация | Что делает |
|-----------|-----------|
| `@Entity` | Говорит Spring: «этот класс = таблица в БД»  |
| `@Table(name = "category")` | Указывает точное имя таблицы. Если не указать — Spring возьмёт имя класса в нижнем регистре |
| `@Id` | Помечает поле как первичный ключ |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | Говорит, что `id` генерируется БД (автоинкремент). Для PostgreSQL с `serial4` это правильный вариант |
| `@Column(name = "title", nullable = false, length = 200)` | Настраивает колонку: имя, обязательность, длину |
| `@Data` | **Lombok**: автоматически создаёт getters, setters, `toString`, `equals`, `hashCode`  |

> ⚠️ **Важно:** тип `id` — `Integer`, а не `int`, потому что `serial4` в PostgreSQL — это 4-байтовое целое, которое может быть `NULL` до сохранения. Использование wrapper-класса обязательно .

---

## Создание Repository

Создайте файл `src/main/java/com/example/shop/CategoryRepository.java`:

```java
package com.example.shop;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CategoryRepository extends JpaRepository<Category, Integer> {
}
```

### 4.1. Что мы получили бесплатно

`JpaRepository<Category, Integer>` — это дженерик, где:
- `Category` — тип сущности
- `Integer` — тип первичного ключа

Spring автоматически создаёт реализацию с методами :

| Метод | Что делает |
|-------|-----------|
| `findAll()` | `SELECT * FROM category` |
| `findById(id)` | `SELECT * FROM category WHERE id = ?` |
| `save(category)` | `INSERT INTO category ...` или `UPDATE category ...` |
| `deleteById(id)` | `DELETE FROM category WHERE id = ?` |
| `count()` | `SELECT COUNT(*) FROM category` |

> 💡 **Почему интерфейс, а не класс?** Spring Data JPA использует **dynamic proxy**: во время запуска приложения Spring создаёт реализацию этого интерфейса «на лету». Вам не нужно писать ни строчки кода — всё уже написано за вас .

---

## Первый контроллер с JPA

Теперь заменим `JdbcTemplate` на `CategoryRepository`.

Создайте файл `src/main/java/com/example/shop/CategoryController.java`:

```java
package com.example.shop;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class CategoryController {

    @Autowired
    private CategoryRepository categoryRepository;

    @GetMapping("/categories")
    public List<Category> categories() {
        return categoryRepository.findAll();
    }
}
```

Перезапустите приложение и откройте `http://localhost:8080/categories`.

**Ожидаемый результат:** JSON-массив с категориями.

```json
[
  {"id":1,"title":"Школьные принадлежности"},
  {"id":2,"title":"Для офиса"},
  {"id":3,"title":"Бумага офисная"},
  {"id":4,"title":"Тетради школьные"}
]
```

> ✅ Сравните с прошлым занятием: вместо `jdbcTemplate.queryForList("SELECT ...")` мы просто вызвали `categoryRepository.findAll()`. Spring сгенерировал SQL сам.

---

## Entity для Product: связи ManyToOne

Таблица `product` сложнее: у неё есть внешние ключи на `category`, `manufacturer`, `supplier`, `unittype`.

### 6.1. Создание Entity

Создайте файл `src/main/java/com/example/shop/Product.java`:

```java
package com.example.shop;

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

    // Для упрощения пока не добавляем manufacturer, supplier, unittype
}
```

### 6.2. Разбор связи `@ManyToOne`

| Аннотация | Что делает |
|-----------|-----------|
| `@ManyToOne` | Говорит: «много товаров могут относиться к одной категории»  |
| `@JoinColumn(name = "category_id")` | Указывает, что связь идёт через колонку `category_id` в таблице `product` |
| `FetchType.LAZY` | Категория загружается **только когда к ней обращаются**. Это эффективнее, чем `EAGER` (загружать сразу) |

**Как это работает:**
- В БД у `product` есть колонка `category_id` (INT).
- В Java у `Product` есть поле `Category category`.
- Spring знает: чтобы загрузить `category`, нужно сделать `SELECT * FROM category WHERE id = product.category_id`.

> ⚠️ **Тип `BigDecimal` для `cost`:** в БД `cost` — это `numeric`. В Java для денег используется `BigDecimal`, а не `double`, чтобы избежать ошибок округления .

### 6.3. Repository для Product

Создайте `ProductRepository.java`:

```java
package com.example.shop;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, String> {
}
```

> ⚠️ Второй тип в дженерике — `String`, потому что `id` у `Product` — строка (артикул `N592T4`).

### 6.4. Контроллер для Product

Создайте `ProductJpaController.java`:

```java
package com.example.shop;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class ProductJpaController {

    @Autowired
    private ProductRepository productRepository;

    @GetMapping("/products-jpa")
    public List<Product> products() {
        return productRepository.findAll();
    }
}
```

Откройте `http://localhost:8080/products-jpa`.

**Ожидаемый результат:** JSON с товарами. Обратите внимание: `category` будет вложенным объектом.

```json
[
  {
    "id":"N592T4",
    "title":"Стикеры",
    "cost":34,
    "quantityInStock":17,
    "category":{"id":2,"title":"Для офиса"}
  }
]
```

> 💡 **Проблема:** `category` загружается для каждого товара отдельным запросом (N+1 проблема). Позже мы решим её через `JOIN FETCH`. Пока это нормально для обучения.

---

## Thymeleaf-шаблон со связями

Теперь выведем товары с категориями в HTML.

Создайте файл `src/main/resources/templates/products-jpa.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Товары (JPA)</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
        th { background: #f4f4f4; }
    </style>
</head>
<body>
    <h1>Список товаров (через JPA)</h1>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Название</th>
                <th>Цена</th>
                <th>Остаток</th>
                <th>Категория</th>
            </tr>
        </thead>
        <tbody>
            <tr th:each="p : ${products}">
                <td th:text="${p.id}"></td>
                <td th:text="${p.title}"></td>
                <td th:text="${p.cost}"></td>
                <td th:text="${p.quantityInStock}"></td>
                <td th:text="${p.category != null ? p.category.title : '—'}"></td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

> 💡 **`p.category != null ? p.category.title : '—'`** — это ** Elvis-оператор** в Thymeleaf. Если `category` не null, показываем `title`, иначе тире. Так мы защищаемся от `NullPointerException`.

Добавьте метод в контроллер:

```java
@Controller
public class ProductViewController {

    @Autowired
    private ProductRepository productRepository;

    @GetMapping("/products-jpa-page")
    public String productsJpaPage(Model model) {
        model.addAttribute("products", productRepository.findAll());
        return "products-jpa";
    }
}
```

Откройте `http://localhost:8080/products-jpa-page`.

> ⚠️ **Важно:** для работы `p.category.title` в Thymeleaf сессия Hibernate должна быть открыта. Если вы видите ошибку `LazyInitializationException`, добавьте в `application.properties`:
> ```properties
> spring.jpa.open-in-view=true
> ```
> Это откроет сессию на время рендеринга шаблона.

---

## Задание

1. **Создайте Entity для `Manufacturer`** (производитель):
   - Поля: `id` (Integer), `title` (String).
   - Таблица: `manufacturer`.
   - Создайте `ManufacturerRepository extends JpaRepository<Manufacturer, Integer>`.
   - Создайте endpoint `GET /manufacturers`, возвращающий JSON.
   - Создайте HTML-страницу `/manufacturers-page` с таблицей производителей.

2. **Добавьте связь в `Product`:**
   - Добавьте поле `@ManyToOne private Manufacturer manufacturer;` с `@JoinColumn(name = "manufacturer_id")`.
   - Выведите название производителя в таблице на `/products-jpa-page`.

3. **Создайте Entity для `UnitType`** (единица измерения):
   - Поля: `id` (Integer), `title` (String).
   - Таблица: `unittype`.
   - Добавьте связь в `Product`.

4. **Загрузите результат на Gogs** в репозиторий `Lesson1` (обновите его).

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 3](Lesson3.md) |

---

## 💡 Что дальше (анонс урока 3)

На следующем занятии:
- Добавим **Spring Security** (аутентификация и роли).
- Сделаем **форму добавления товара** (`@PostMapping`).
- Добавим **фильтрацию** по категории.
- Решим **N+1 проблему** через `@Query` и `JOIN FETCH`.
