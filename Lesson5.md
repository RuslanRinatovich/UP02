# Занятие 5. Thymeleaf Fragments, главная страница, каталог и карточка товара

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 6](Lesson6.md)

## План
1. [Введение: что было и куда идём](#введение)
2. [Проблема дублирования HTML](#проблема-дублирования-html)
3. [Thymeleaf Fragments: общие блоки](#thymeleaf-fragments)
4. [Шаг 1. Создаём layout.html](#шаг-1-создаём-layouthtml)
5. [Шаг 2. Переводим страницы на layout](#шаг-2-переводим-страницы-на-layout)
6. [Главная страница](#главная-страница)
7. [Каталог с фильтрацией по категориям](#каталог-с-фильтрацией)
8. [Карточка товара](#карточка-товара)
9. [Отдача фото товара](#отдача-фото-товара)
10. [Пагинация каталога](#пагинация-каталога)
11. [Проверка и чек-лист](#проверка)
12. [Задание](#задание)

---

## Введение

За четыре занятия мы сделали:
- CRUD для товаров и категорий.
- Слоистую архитектуру с `entity`, `repository`, `dto`, `service`, `controller`, `util`.
- Spring Security с ролями и логином.

**Проблема:** каждая HTML-страница содержит свою копию `<head>`, `<style>`, навигацию, кнопку выхода. Стоит добавить новый пункт меню — и надо править 10 файлов.

**Сегодня:** вынесем общие части в **layout** (Thymeleaf Fragments), сделаем главную страницу, каталог с фильтрами по категориям, карточку товара с фото и пагинацию.

---

## Проблема дублирования HTML

Посмотрите на `products-jpa.html` и `admin/product-form.html` — обе страницы имеют:
- одинаковый `<head>` со стилями,
- одинаковый `body { font-family... }`,
- (теперь ещё) одинаковую навигацию и кнопку «Выйти».

**Плохой подход:**
```html
<!-- products-jpa.html -->
<!DOCTYPE html>
<html>... <style>...</style> ... <div class="menu">...</div> ... </html>

<!-- admin/product-form.html -->
<!DOCTYPE html>
<html>... <style>...</style> ... <div class="menu">...</div> ... </html>
```

**Хороший подход (Thymeleaf Fragments):**
```html
<!-- products-jpa.html -->
<html th:replace="~{layout :: page(content=~{::content})}">
    <div th:fragment="content">...</div>
</html>
```

---

## Thymeleaf Fragments

> 💡 **Fragment** — это переиспользуемый кусок HTML, который можно вставлять в другие шаблоны. Аналог «include» в других шаблонизаторах, но с более гибкими возможностями.

**Три способа использования:**

| Способ | Что делает |
|--------|-----------|
| `th:insert` | Вставляет фрагмент **внутрь** тега |
| `th:replace` | **Заменяет** тег на фрагмент |
| `th:include` | Вставляет **содержимое** фрагмента (устаревшее) |

**Синтаксис:**
- `~{fragments/header :: header}` — фрагмент `header` из файла `fragments/header.html`.
- `~{::content}` — фрагмент `content` из текущего файла.

**Передача параметров:**
```html
<div th:replace="~{layout :: page(title='Товары', content=~{::content})}">
    <div th:fragment="content">...</div>
</div>
```

---

## Шаг 1. Создаём layout.html

Создайте `src/main/resources/templates/layout.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" th:fragment="page(title, content)">
<head>
    <meta charset="UTF-8">
    <title th:text="${title}">Магазин канцтоваров</title>
    <link rel="stylesheet" th:href="@{/css/style.css}">
</head>
<body>

<header class="site-header">
    <div class="logo">
        <a th:href="@{/}">🛒 Магазин канцтоваров</a>
    </div>

    <nav class="main-nav">
        <a th:href="@{/}">Главная</a>
        <a th:href="@{/catalog}">Каталог</a>

        <span th:if="${#authorization.expression('hasAnyRole(''Админист'', ''Менеджер'')')}">
            <a th:href="@{/admin/products}">Управление товарами</a>
            <a th:href="@{/admin/products/new}">Добавить товар</a>
        </span>
    </nav>

    <div class="user-box">
        <span th:if="${#authorization.expression('isAuthenticated()')}">
            <b th:text="${#authentication.name}">guest</b>
            <form th:action="@{/logout}" method="post" style="display:inline">
                <button type="submit">Выйти</button>
            </form>
        </span>
        <a th:if="${!#authorization.expression('isAuthenticated()')}" th:href="@{/login}">Войти</a>
    </div>
</header>

<main class="container">
    <div th:replace="${content}">Содержимое страницы</div>
</main>

<footer class="site-footer">
    © 2026 Магазин канцтоваров
</footer>

</body>
</html>
```

Создайте `src/main/resources/static/css/style.css`:

```css
* { box-sizing: border-box; }

body {
    font-family: 'Segoe UI', Arial, sans-serif;
    margin: 0;
    background: #f4f6f8;
    color: #222;
}

a { color: #0066cc; text-decoration: none; }
a:hover { text-decoration: underline; }

.site-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 30px;
    background: #1f2d3d;
    color: white;
}
.site-header a { color: white; }

.main-nav a { margin-right: 15px; }

.user-box form { margin-left: 10px; }
.user-box button {
    background: #dc3545;
    color: white;
    border: none;
    padding: 5px 12px;
    border-radius: 4px;
    cursor: pointer;
}

.container {
    max-width: 1100px;
    margin: 30px auto;
    padding: 0 20px;
}

.site-footer {
    text-align: center;
    padding: 20px;
    color: #888;
    font-size: 14px;
}

/* Таблицы */
table { border-collapse: collapse; width: 100%; background: white; }
th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
th { background: #f0f0f0; }

/* Карточки товаров */
.product-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
    gap: 20px;
}
.product-card {
    background: white;
    border-radius: 8px;
    padding: 15px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.06);
    display: flex;
    flex-direction: column;
}
.product-card img {
    width: 100%;
    height: 180px;
    object-fit: contain;
    margin-bottom: 10px;
}
.product-card .price { font-weight: bold; margin-top: auto; }

/* Категории */
.category-list a {
    display: inline-block;
    margin-right: 8px;
    padding: 6px 12px;
    background: white;
    border-radius: 16px;
    border: 1px solid #ccc;
}
.category-list a.active { background: #0066cc; color: white; border-color: #0066cc; }

/* Пагинация */
.pagination { margin-top: 20px; text-align: center; }
.pagination a, .pagination span {
    display: inline-block;
    margin: 0 4px;
    padding: 6px 12px;
    border-radius: 4px;
    background: white;
    border: 1px solid #ddd;
}
.pagination .current { background: #0066cc; color: white; }
```

---

## Шаг 2. Переводим страницы на layout

### 2.1. Обновляем `products-jpa.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Товары', content=~{::content})}">
<body>
<div th:fragment="content">
    <h1>Список товаров</h1>

    <table>
        <thead>
        <tr>
            <th>ID</th>
            <th>Название</th>
            <th>Цена</th>
            <th>Остаток</th>
            <th>Категория</th>
            <th th:if="${#authorization.expression('hasRole(''Админист'')')}">Действия</th>
        </tr>
        </thead>
        <tbody>
        <tr th:each="p : ${products}">
            <td th:text="${p.id}"></td>
            <td>
                <a th:href="@{/product/{id}(id=${p.id})}" th:text="${p.title}"></a>
            </td>
            <td th:text="${p.cost}"></td>
            <td th:text="${p.quantityInStock}"></td>
            <td th:text="${p.category != null ? p.category.title : '—'}"></td>
            <td th:if="${#authorization.expression('hasRole(''Админист'')')}">
                <a th:href="@{/admin/products/edit/{id}(id=${p.id})}">Редактировать</a>
                <form th:action="@{/admin/products/delete/{id}(id=${p.id})}" method="post" style="display:inline">
                    <button type="submit" onclick="return confirm('Удалить товар?')">Удалить</button>
                </form>
            </td>
        </tr>
        </tbody>
    </table>
</div>
</body>
</html>
```

### 2.2. Обновляем `admin/product-form.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Добавить товар', content=~{::content})}">
<body>
<div th:fragment="content">
    <h1 th:text="${editMode != null && editMode ? 'Редактировать товар' : 'Добавить товар'}"></h1>

    <form th:action="${editMode != null && editMode} 
                       ? @{/admin/products/edit/{id}(id=*{id})} 
                       : @{/admin/products/new}"
          th:object="${productForm}" method="post" enctype="multipart/form-data">

        <div class="form-group">
            <label>Артикул *</label>
            <input type="text" th:field="*{id}"
                   th:readonly="${editMode != null && editMode}"/>
            <span class="error" th:if="${#fields.hasErrors('id')}" th:errors="*{id}"></span>
        </div>

        <!-- остальные поля без изменений -->
        <!-- ... -->

        <button type="submit">Сохранить</button>
        <a th:href="@{/admin/products}">Отмена</a>
    </form>
</div>
</body>
</html>
```

---

## Главная страница

Главная будет показывать:
- Приветствие.
- Список категорий.
- 8 популярных товаров (с фото).

### 6.1. Контроллер

`src/main/java/com/example/shop/controller/HomeController.java`:

```java
package com.example.shop.controller;

import com.example.shop.service.CategoryService;
import com.example.shop.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @Autowired
    private ProductService productService;

    @Autowired
    private CategoryService categoryService;

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("categories", categoryService.findAll());
        model.addAttribute("products", productService.findPopular(8));
        return "home";
    }
}
```

### 6.2. Метод в ProductService

Добавьте в `ProductService`:

```java
public List<Product> findPopular(int limit) {
    return productRepository.findAll(PageRequest.of(0, limit)).getContent();
}
```

> 💡 **`PageRequest.of(page, size)`** — Spring Data JPA умеет возвращать данные постранично. `PageRequest.of(0, 8)` означает: «первые 8 записей, страница 0».

Импорт:
```java
import org.springframework.data.domain.PageRequest;
```

### 6.3. Шаблон `home.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Главная', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Добро пожаловать в магазин канцтоваров!</h1>

    <h2>Категории</h2>
    <div class="category-list">
        <a th:each="c : ${categories}"
           th:href="@{/catalog(categoryId=${c.id})}"
           th:text="${c.title}">Категория</a>
    </div>

    <h2 style="margin-top: 30px;">Популярные товары</h2>
    <div class="product-grid">
        <div class="product-card" th:each="p : ${products}">
            <img th:src="@{/product/{id}/photo(id=${p.id})}" alt="Фото"/>
            <a th:href="@{/product/{id}(id=${p.id})}" th:text="${p.title}"></a>
            <span class="price" th:text="${p.cost} + ' ₽'"></span>
        </div>
    </div>

</div>
</body>
</html>
```

---

## Каталог с фильтрацией

Каталог — отдельная страница со списком всех товаров, фильтром по категории и пагинацией.

### 7.1. Метод в ProductService

```java
public Page<Product> findPage(Integer categoryId, int page, int size) {
    PageRequest pageRequest = PageRequest.of(page, size, Sort.by("title").ascending());
    if (categoryId == null) {
        return productRepository.findAll(pageRequest);
    }
    return productRepository.findByCategoryId(categoryId, pageRequest);
}
```

Импорты:
```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;
```

### 7.2. Метод в ProductRepository

```java
Page<Product> findByCategoryId(Integer categoryId, Pageable pageable);
```

> 💡 **Spring Data JPA сам сгенерирует SQL** по имени метода. `findByCategoryId` → `SELECT * FROM product WHERE category_id = ?`.

### 7.3. Контроллер каталога

Добавьте в `ProductController`:

```java
@GetMapping("/catalog")
public String catalog(@RequestParam(required = false) Integer categoryId,
                      @RequestParam(defaultValue = "0") int page,
                      Model model) {
    model.addAttribute("categories", categoryService.findAll());
    model.addAttribute("products", productService.findPage(categoryId, page, 12));
    model.addAttribute("selectedCategoryId", categoryId);
    model.addAttribute("currentPage", page);
    return "catalog";
}
```

> ⚠️ Не забудьте внедрить `CategoryService` в `ProductController`.

### 7.4. Шаблон `catalog.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Каталог', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Каталог</h1>

    <div class="category-list">
        <a th:href="@{/catalog}" 
           th:classappend="${selectedCategoryId == null} ? 'active'">Все</a>
        <a th:each="c : ${categories}"
           th:href="@{/catalog(categoryId=${c.id})}"
           th:classappend="${selectedCategoryId != null && selectedCategoryId == c.id} ? 'active'"
           th:text="${c.title}"></a>
    </div>

    <div class="product-grid" style="margin-top: 20px;">
        <div class="product-card" th:each="p : ${products.content}">
            <img th:src="@{/product/{id}/photo(id=${p.id})}" alt="Фото"/>
            <a th:href="@{/product/{id}(id=${p.id})}" th:text="${p.title}"></a>
            <span class="price" th:text="${p.cost} + ' ₽'"></span>
        </div>
    </div>

    <div class="pagination" th:if="${products.totalPages > 1}">
        <a th:if="${currentPage > 0}"
           th:href="@{/catalog(categoryId=${selectedCategoryId}, page=${currentPage - 1})}">← Назад</a>

        <span th:each="i : ${#numbers.sequence(0, products.totalPages - 1)}"
              th:classappend="${i == currentPage} ? 'current'">
            <a th:href="@{/catalog(categoryId=${selectedCategoryId}, page=${i})}"
               th:text="${i + 1}"></a>
        </span>

        <a th:if="${currentPage < products.totalPages - 1}"
           th:href="@{/catalog(categoryId=${selectedCategoryId}, page=${currentPage + 1})}">Вперёд →</a>
    </div>

</div>
</body>
</html>
```

> 💡 **`${products.content}`** — Page содержит список записей в поле `content`.
> **`${#numbers.sequence(0, N)}`** — генерирует последовательность чисел для пагинации.

---

## Карточка товара

### 8.1. Контроллер

```java
@GetMapping("/product/{id}")
public String productDetails(@PathVariable String id, Model model) {
    Product product = productService.findById(id);
    model.addAttribute("product", product);
    return "product-details";
}
```

### 8.2. Шаблон `product-details.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Товар', content=~{::content})}">
<body>
<div th:fragment="content">

    <div style="display: flex; gap: 30px; background: white; padding: 20px; border-radius: 8px;">

        <div style="flex: 1; max-width: 400px;">
            <img th:src="@{/product/{id}/photo(id=${product.id})}"
                 alt="Фото"
                 style="width: 100%; border-radius: 8px;"/>
        </div>

        <div style="flex: 2;">
            <h1 th:text="${product.title}">Название</h1>
            <p th:if="${product.description != null}"
               th:text="${product.description}">Описание</p>

            <p>
                <b>Категория:</b>
                <span th:text="${product.category != null ? product.category.title : '—'}"></span>
            </p>

            <p>
                <b>В наличии:</b>
                <span th:text="${product.quantityInStock}"></span> шт.
            </p>

            <p style="font-size: 24px; color: #0066cc;">
                <b th:text="${product.cost} + ' ₽'"></b>
            </p>

            <a th:href="@{/catalog}">← Назад в каталог</a>
        </div>

    </div>

</div>
</body>
</html>
```

---

## Отдача фото товара

В таблице `product` фотография хранится как `bytea`. Нужно сделать endpoint, который отдаёт её как изображение.

Добавьте в `ProductController`:

```java
@GetMapping("/product/{id}/photo")
public ResponseEntity<byte[]> productPhoto(@PathVariable String id) {
    Product product = productService.findById(id);
    byte[] photo = product.getPhoto();

    if (photo == null || photo.length == 0) {
        // Заглушка: 1x1 прозрачный PNG
        byte[] placeholder = new byte[]{
                (byte) 0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A
                // (сокращённый массив — для простоты лучше сгенерировать картинку-заглушку)
        };
        return ResponseEntity.ok()
                .contentType(MediaType.IMAGE_PNG)
                .body(placeholder);
    }

    // Определить тип по первым байтам (JPEG или PNG)
    MediaType mediaType = detectImageType(photo);

    return ResponseEntity.ok()
            .contentType(mediaType)
            .header(HttpHeaders.CACHE_CONTROL, "max-age=86400")
            .body(photo);
}

private MediaType detectImageType(byte[] data) {
    if (data.length >= 3
            && (data[0] & 0xFF) == 0xFF
            && (data[1] & 0xFF) == 0xD8
            && (data[2] & 0xFF) == 0xFF) {
        return MediaType.IMAGE_JPEG;
    }
    return MediaType.IMAGE_PNG;
}
```

Импорты:
```java
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpHeaders;
```

> 💡 **`ResponseEntity`** — это обёртка, которая позволяет явно указать HTTP-статус, заголовки и тело. Обычные `@RestController`-методы возвращают просто объект, но для бинарных данных нужен контроль над `Content-Type`.

> ⚠️ **Заглушка в коде выше некорректна** — я оставил её как пример. Лучше положить файл `placeholder.png` в `src/main/resources/static/images/` и возвращать его как `ClassPathResource`.

**Правильный вариант заглушки:**

```java
@Autowired
private ResourceLoader resourceLoader;

@GetMapping("/product/{id}/photo")
public ResponseEntity<byte[]> productPhoto(@PathVariable String id) throws IOException {
    Product product = productService.findById(id);
    byte[] photo = product.getPhoto();

    if (photo == null || photo.length == 0) {
        Resource placeholder = resourceLoader.getResource("classpath:static/images/placeholder.png");
        return ResponseEntity.ok()
                .contentType(MediaType.IMAGE_PNG)
                .body(placeholder.getInputStream().readAllBytes());
    }

    MediaType mediaType = detectImageType(photo);
    return ResponseEntity.ok()
            .contentType(mediaType)
            .header(HttpHeaders.CACHE_CONTROL, "max-age=86400")
            .body(photo);
}
```

---

## Пагинация каталога

Мы уже добавили пагинацию в шаблоне. Что важно помнить:

| Параметр | Значение |
|----------|----------|
| Номер страницы | Начинается с **0**, а не с 1 |
| Размер страницы | По умолчанию 12 товаров |
| Сортировка | По названию (`Sort.by("title")`) |
| URL при переходе | `/catalog?categoryId=2&page=1` |

> 💡 **`Page<Product>`** содержит:
> - `content` — сами записи.
> - `totalPages` — общее количество страниц.
> - `totalElements` — общее количество записей.
> - `number` — текущая страница.
> - `hasNext()`, `hasPrevious()` — есть ли следующая/предыдущая.

---

## Проверка

### 11.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | `layout.html` создан | |
| 2 | Все страницы переведены на `th:replace` | |
| 3 | CSS лежит в `static/css/style.css` | |
| 4 | Главная страница `/` показывает категории и популярные товары | |
| 5 | Каталог `/catalog` показывает все товары | |
| 6 | Фильтр по категории работает | |
| 7 | Пагинация работает | |
| 8 | Карточка товара `/product/{id}` открывается | |
| 9 | Фото товара отображается | |
| 10 | Навигация единая для всех страниц | |

### 11.2. Сценарии

1. Откройте `/` — должны быть категории и товары.
2. Кликните на категорию — переходите в каталог с фильтром.
3. Кликните на товар — открывается карточка.
4. Проверьте пагинацию: `/catalog?page=1`.
5. Убедитесь, что меню и кнопка «Выйти» одинаковы на всех страницах.

---

## Задание

1. **Создайте фрагмент `fragments/product-card.html`**:
   - Отдельный фрагмент для карточки товара.
   - Используйте его на главной и в каталоге через `th:replace`.

2. **Добавьте сортировку в каталог**:
   - Кнопки «По названию», «По цене (возр.)», «По цене (убыв.)».
   - Реализуйте через `@RequestParam(required = false) String sort` и `Sort.by(...)`.

3. **Добавьте поиск по названию**:
   - Поле ввода на странице каталога.
   - Метод в `ProductRepository`: `Page<Product> findByTitleContainingIgnoreCase(String title, Pageable pageable)`.
   - Комбинируйте с фильтром по категории.

4. **Создайте фрагмент `fragments/pagination.html`**:
   - Переиспользуйте пагинацию на всех страницах, где есть списки.

5. **Сделайте страницу `/about`** с описанием магазина:
   - Через layout.
   - Доступна всем.

6. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 6](Lesson6.md) |

---

## 💡 Что дальше (анонс урока 6)

На следующем занятии:
- Добавим **корзину** (сессионную или в БД).
- Реализуем **оформление заказа** (`order`).
- Используем таблицы `order`, `order_product`, `pickup_point`, `status`.
- Генерация **кода выдачи** через `util/OrderUtils`.
- Страница «Мои заказы» для клиента.
