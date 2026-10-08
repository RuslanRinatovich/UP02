# Занятие 7. Админ-панель, управление заказами и REST API

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 8](Lesson8.md)

## План
1. [Введение: что было и куда идём](#введение)
2. [Что такое админ-панель и зачем она нужна](#что-такое-админ-панель)
3. [Дашборд `/admin` со статистикой](#дашборд)
4. [Управление заказами](#управление-заказами)
5. [Фильтры заказов по статусу и дате](#фильтры-заказов)
6. [Смена статуса заказа](#смена-статуса)
7. [Что такое REST API](#что-такое-rest-api)
8. [Первый REST-контроллер для товаров](#rest-товары)
9. [DTO для API](#dto-для-api)
10. [Обработка ошибок в REST](#обработка-ошибок)
11. [REST для заказов](#rest-заказы)
12. [Проверка и чек-лист](#проверка)
13. [Задание](#задание)

---

## Введение

За шесть занятий мы сделали:
- Каталог, карточку товара, пагинацию.
- Корзину в сессии.
- Оформление заказа с сохранением в БД.
- Страницу «Мои заказы» для клиента.

**Проблема:** менеджер и администратор не могут видеть все заказы, менять их статусы, смотреть статистику. А ещё наш сайт не отдаёт данные во внешние системы (мобильное приложение, SPA).

**Сегодня:**
1. Сделаем **админ-панель** с дашбордом и управлением заказами.
2. Добавим **REST API** — «второй вход» в приложение, уже для программ, а не людей.

---

## Что такое админ-панель

> 💡 **Админ-панель** — раздел сайта для сотрудников магазина. Обычно в ней:
> - **Дашборд** — сводная статистика (сколько товаров, заказов, пользователей).
> - **CRUD** по товарам и категориям (у нас уже есть).
> - **Заказы** — все заказы с фильтрами и сменой статуса.
> - **Пользователи** — список клиентов и менеджеров.
> - **Отчёты** — продажи за период и т.п.

Разграничение доступа:
- `/admin/**` — только `Админист` и `Менеджер`.
- `/admin/users/**`, `/admin/settings` — только `Админист`.

Мы уже настроили безопасность на уроке 4, теперь наполним админку.

---

## Дашборд

### 3.1. DashboardController

`src/main/java/com/example/shop/controller/admin/DashboardController.java`:

```java
package com.example.shop.controller.admin;

import com.example.shop.repository.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DashboardController {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private CategoryRepository categoryRepository;

    @GetMapping("/admin")
    public String dashboard(Model model) {
        model.addAttribute("productCount", productRepository.count());
        model.addAttribute("orderCount", orderRepository.count());
        model.addAttribute("userCount", userRepository.count());
        model.addAttribute("categoryCount", categoryRepository.count());
        model.addAttribute("latestOrders", orderRepository.findTop5ByOrderByCreateDateDesc());
        return "admin/dashboard";
    }
}
```

### 3.2. Метод в OrderRepository

```java
List<Order> findTop5ByOrderByCreateDateDesc();
```

> 💡 Spring Data JPA читает имя метода как «найди Top 5, отсортируй по `createDate` по убыванию». SQL: `SELECT * FROM "order" ORDER BY create_date DESC LIMIT 5`.

### 3.3. Шаблон `admin/dashboard.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Админ-панель', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Админ-панель</h1>

    <div style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; margin-bottom: 30px;">
        <div style="background: white; padding: 20px; border-radius: 8px; text-align: center;">
            <div style="font-size: 36px; font-weight: bold;" th:text="${productCount}"></div>
            <div>Товаров</div>
        </div>
        <div style="background: white; padding: 20px; border-radius: 8px; text-align: center;">
            <div style="font-size: 36px; font-weight: bold;" th:text="${orderCount}"></div>
            <div>Заказов</div>
        </div>
        <div style="background: white; padding: 20px; border-radius: 8px; text-align: center;">
            <div style="font-size: 36px; font-weight: bold;" th:text="${userCount}"></div>
            <div>Пользователей</div>
        </div>
        <div style="background: white; padding: 20px; border-radius: 8px; text-align: center;">
            <div style="font-size: 36px; font-weight: bold;" th:text="${categoryCount}"></div>
            <div>Категорий</div>
        </div>
    </div>

    <h2>Последние 5 заказов</h2>
    <table>
        <thead>
        <tr>
            <th>№</th>
            <th>Дата</th>
            <th>Пользователь</th>
            <th>Статус</th>
            <th>Сумма</th>
            <th></th>
        </tr>
        </thead>
        <tbody>
        <tr th:each="o : ${latestOrders}">
            <td th:text="${o.id}"></td>
            <td th:text="${o.createDate}"></td>
            <td th:text="${o.user != null ? o.user.username : '—'}"></td>
            <td th:text="${o.status.title}"></td>
            <td></td>
            <td><a th:href="@{/admin/orders/{id}(id=${o.id})}">Открыть</a></td>
        </tr>
        </tbody>
    </table>

    <div style="margin-top: 30px;">
        <a th:href="@{/admin/products}" style="margin-right: 15px;">Товары</a>
        <a th:href="@{/admin/categories}" style="margin-right: 15px;">Категории</a>
        <a th:href="@{/admin/orders}">Заказы</a>
    </div>

</div>
</body>
</html>
```

> 💡 В `layout.html` добавьте ссылку на дашборд в навигацию:
> ```html
> <a th:if="${#authorization.expression('hasAnyRole(''Админист'', ''Менеджер'')')}"
>    th:href="@{/admin}">Админ-панель</a>
> ```

---

## Управление заказами

### 4.1. OrderAdminController

`src/main/java/com/example/shop/controller/admin/OrderAdminController.java`:

```java
package com.example.shop.controller.admin;

import com.example.shop.entity.Order;
import com.example.shop.entity.Status;
import com.example.shop.repository.OrderRepository;
import com.example.shop.repository.StatusRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/admin/orders")
public class OrderAdminController {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private StatusRepository statusRepository;

    @GetMapping
    public String list(@RequestParam(required = false) Integer statusId,
                       Model model) {
        model.addAttribute("statuses", statusRepository.findAll());
        model.addAttribute("selectedStatusId", statusId);

        if (statusId != null) {
            model.addAttribute("orders",
                    orderRepository.findByStatusIdOrderByCreateDateDesc(statusId));
        } else {
            model.addAttribute("orders",
                    orderRepository.findAllByOrderByCreateDateDesc());
        }
        return "admin/orders";
    }

    @GetMapping("/{id}")
    public String details(@PathVariable Integer id, Model model) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Заказ не найден"));
        model.addAttribute("order", order);
        model.addAttribute("statuses", statusRepository.findAll());
        return "admin/order-details";
    }

    @PostMapping("/{id}/status")
    public String changeStatus(@PathVariable Integer id,
                               @RequestParam Integer statusId) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Заказ не найден"));
        Status newStatus = statusRepository.findById(statusId)
                .orElseThrow(() -> new IllegalArgumentException("Статус не найден"));
        order.setStatus(newStatus);
        orderRepository.save(order);
        return "redirect:/admin/orders/" + id;
    }
}
```

### 4.2. Методы в OrderRepository

```java
List<Order> findAllByOrderByCreateDateDesc();

List<Order> findByStatusIdOrderByCreateDateDesc(Integer statusId);
```

### 4.3. Шаблон `admin/orders.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Заказы', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Все заказы</h1>

    <div style="margin-bottom: 20px;">
        <a th:href="@{/admin/orders}"
           th:classappend="${selectedStatusId == null} ? 'active'"
           style="padding: 6px 12px; border-radius: 16px; border: 1px solid #ccc; margin-right: 8px;">
            Все
        </a>
        <a th:each="s : ${statuses}"
           th:href="@{/admin/orders(statusId=${s.id})}"
           th:classappend="${selectedStatusId != null && selectedStatusId == s.id} ? 'active'"
           th:text="${s.title}"
           style="padding: 6px 12px; border-radius: 16px; border: 1px solid #ccc; margin-right: 8px;"></a>
    </div>

    <table>
        <thead>
        <tr>
            <th>№</th>
            <th>Дата</th>
            <th>Пользователь</th>
            <th>Пункт выдачи</th>
            <th>Статус</th>
            <th>Код выдачи</th>
            <th></th>
        </tr>
        </thead>
        <tbody>
        <tr th:each="o : ${orders}">
            <td th:text="${o.id}"></td>
            <td th:text="${o.createDate}"></td>
            <td th:text="${o.user != null ? o.user.username : '—'}"></td>
            <td th:text="${o.pickupPoint.address}"></td>
            <td th:text="${o.status.title}"></td>
            <td th:text="${o.getCode}"></td>
            <td><a th:href="@{/admin/orders/{id}(id=${o.id})}">Открыть</a></td>
        </tr>
        </tbody>
    </table>

</div>
</body>
</html>
```

### 4.4. Шаблон `admin/order-details.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Заказ', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Заказ №<span th:text="${order.id}"></span></h1>

    <div style="background: white; padding: 20px; border-radius: 8px;">
        <p><b>Пользователь:</b> <span th:text="${order.user != null ? order.user.username : '—'}"></span></p>
        <p><b>Дата создания:</b> <span th:text="${order.createDate}"></span></p>
        <p><b>Дата выдачи:</b> <span th:text="${order.deliveryDate}"></span></p>
        <p><b>Пункт выдачи:</b> <span th:text="${order.pickupPoint.address}"></span></p>
        <p><b>Код выдачи:</b> <span th:text="${order.getCode}"></span></p>

        <p>
            <b>Статус:</b>
            <span th:text="${order.status.title}"></span>
        </p>

        <form th:action="@{/admin/orders/{id}/status(id=${order.id})}" method="post"
              style="margin-top: 10px;">
            <label>Изменить статус:</label>
            <select name="statusId" required>
                <option th:each="s : ${statuses}"
                        th:value="${s.id}"
                        th:text="${s.title}"
                        th:selected="${s.id == order.status.id}"></option>
            </select>
            <button type="submit">Сохранить</button>
        </form>

        <h2 style="margin-top: 30px;">Состав заказа</h2>
        <table>
            <thead>
            <tr>
                <th>Товар</th>
                <th>Цена</th>
                <th>Количество</th>
                <th>Итого</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${order.items}">
                <td th:text="${item.product.title}"></td>
                <td th:text="${item.product.cost} + ' ₽'"></td>
                <td th:text="${item.count}"></td>
                <td th:text="${item.product.cost * item.count} + ' ₽'"></td>
            </tr>
            </tbody>
        </table>
    </div>

</div>
</body>
</html>
```

---

## Фильтры заказов

Мы уже добавили фильтр по статусу. Добавим ещё фильтр по дате.

### 5.1. Метод в OrderRepository

```java
List<Order> findByCreateDateBetweenOrderByCreateDateDesc(LocalDate from, LocalDate to);
```

### 5.2. Расширьте контроллер

```java
@GetMapping
public String list(@RequestParam(required = false) Integer statusId,
                   @RequestParam(required = false) 
                   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate from,
                   @RequestParam(required = false) 
                   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate to,
                   Model model) {
    // ... логика комбинирования фильтров
}
```

> 💡 **`@DateTimeFormat`** — говорит Spring, как парсить дату из строки. `ISO.DATE` — формат `2026-10-08`.

> 💡 В реальном проекте комбинировать фильтры лучше через **`Specification`** или **QueryDSL**. Мы используем простые методы — для учебного достаточно.

---

## Смена статуса заказа

Мы уже реализовали `POST /admin/orders/{id}/status`. Обратите внимание на **PRG-паттерн**: после POST идёт redirect на GET-страницу — это предотвращает повторную отправку при F5.

> 💡 **Проверка прав** — по умолчанию `/admin/**` защищён ролями `Админист` и `Менеджер`. Проверку, что менеджер может менять статус, а не удалять товары, можно реализовать через `@PreAuthorize`:
> ```java
> @PreAuthorize("hasAnyRole('Админист', 'Менеджер')")
> @PostMapping("/{id}/status")
> public String changeStatus(...) { ... }
> ```
> Для работы нужна аннотация `@EnableMethodSecurity` в `SecurityConfig`.

---

## Что такое REST API

> 💡 **REST API** — это способ дать доступ к данным приложения по HTTP в машиночитаемом формате (обычно JSON). В отличие от HTML-страниц, REST возвращает **данные**, а не разметку.

**Зачем это нужно:**
- Мобильное приложение (Android/iOS) — получает JSON.
- SPA (React, Vue, Angular) — рендерит JSON на клиенте.
- Интеграция с другими системами (1С, CRM).
- Публичный API для партнёров.

**Отличия от MVC-контроллеров:**

| Что | MVC | REST |
|-----|-----|------|
| Аннотация | `@Controller` | `@RestController` |
| Возвращает | HTML-страницу | JSON |
| URL | `/products-page` | `/api/products` |
| Аутентификация | Сессия + cookie | Токен (JWT) |

**Ключевые принципы REST:**
1. **Ресурсы, а не действия.** `/api/products` — ресурс «товары». Уже не «получить список товаров», а сам ресурс.
2. **HTTP-методы = CRUD.**
   - `GET /api/products` — список.
   - `GET /api/products/{id}` — один.
   - `POST /api/products` — создать.
   - `PUT /api/products/{id}` — обновить.
   - `DELETE /api/products/{id}` — удалить.
3. **Статусы HTTP.**
   - `200 OK` — успех.
   - `201 Created` — создано.
   - `400 Bad Request` — ошибка в запросе.
   - `401 Unauthorized` — не аутентифицирован.
   - `403 Forbidden` — нет прав.
   - `404 Not Found` — не найдено.
   - `500 Internal Server Error` — ошибка сервера.

---

## Первый REST-контроллер для товаров

Мы уже делали `/products-jpa` как JSON. Но это не REST, а «случайный JSON». Сделаем полноценный.

### 8.1. ProductApiController

`src/main/java/com/example/shop/controller/api/ProductApiController.java`:

```java
package com.example.shop.controller.api;

import com.example.shop.dto.ProductDto;
import com.example.shop.entity.Product;
import com.example.shop.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/products")
public class ProductApiController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public List<ProductDto> list() {
        return productService.findAll().stream()
                .map(this::toDto)
                .collect(Collectors.toList());
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductDto> getById(@PathVariable String id) {
        try {
            Product product = productService.findById(id);
            return ResponseEntity.ok(toDto(product));
        } catch (IllegalArgumentException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @GetMapping("/search")
    public List<ProductDto> search(@RequestParam String q) {
        return productService.searchByTitle(q).stream()
                .map(this::toDto)
                .collect(Collectors.toList());
    }

    private ProductDto toDto(Product product) {
        ProductDto dto = new ProductDto();
        dto.setId(product.getId());
        dto.setTitle(product.getTitle());
        dto.setCost(product.getCost());
        dto.setQuantityInStock(product.getQuantityInStock());
        dto.setDescription(product.getDescription());
        if (product.getCategory() != null) {
            dto.setCategoryId(product.getCategory().getId());
            dto.setCategoryTitle(product.getCategory().getTitle());
        }
        return dto;
    }
}
```

> 💡 **`ResponseEntity<T>`** — обёртка для явного указания статуса. `ResponseEntity.ok(dto)` = 200 + тело. `ResponseEntity.notFound().build()` = 404 без тела.

### 8.2. DTO для API

`src/main/java/com/example/shop/dto/ProductDto.java`:

```java
package com.example.shop.dto;

import lombok.Data;

import java.math.BigDecimal;

@Data
public class ProductDto {

    private String id;
    private String title;
    private BigDecimal cost;
    private Integer quantityInStock;
    private String description;
    private Integer categoryId;
    private String categoryTitle;
}
```

> 💡 **Почему DTO, а не Entity?** Entity содержит **двунаправленные связи** (Product ↔ Category). Если сериализовать Entity напрямую, Jackson попытается сериализовать `category.products`, потом `product.category`, потом `category.products` — бесконечная рекурсия. DTO разрывает связи.

### 8.3. Метод поиска в ProductService и ProductRepository

```java
// ProductService
public List<Product> searchByTitle(String query) {
    return productRepository.findByTitleContainingIgnoreCase(query);
}

// ProductRepository
List<Product> findByTitleContainingIgnoreCase(String title);
```

### 8.4. Проверка

Откройте в браузере или Postman:

| URL | Что увидите |
|-----|-------------|
| `/api/products` | JSON-массив всех товаров |
| `/api/products/N592T4` | JSON одного товара |
| `/api/products/UNKNOWN` | 404 Not Found |
| `/api/products/search?q=ручка` | JSON товаров со словом «ручка» |

---

## Обработка ошибок в REST

> 💡 В REST плохая практика — возвращать `null` или кидать исключение. Правильно — вернуть **HTTP-статус и понятное сообщение**.

### 10.1. GlobalExceptionHandler

`src/main/java/com/example/shop/controller/api/ApiExceptionHandler.java`:

```java
package com.example.shop.controller.api;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.Map;

@RestControllerAdvice(basePackages = "com.example.shop.controller.api")
public class ApiExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleNotFound(IllegalArgumentException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(Map.of("error", e.getMessage()));
    }

    @ExceptionHandler(SecurityException.class)
    public ResponseEntity<Map<String, String>> handleForbidden(SecurityException e) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
                .body(Map.of("error", e.getMessage()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleAll(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Map.of("error", "Внутренняя ошибка сервера"));
    }
}
```

> 💡 **`@RestControllerAdvice`** — глобальный обработчик для всех `@RestController` из указанного пакета. Вместо 500 с простыней стектрейса клиент получит JSON с понятным сообщением.

**Пример ответа:**
```json
{"error": "Товар не найден: N592T4"}
```

---

## REST для заказов

### 11.1. OrderApiController

```java
package com.example.shop.controller.api;

import com.example.shop.entity.Order;
import com.example.shop.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/orders")
public class OrderApiController {

    @Autowired
    private OrderService orderService;

    @GetMapping("/my")
    public List<Map<String, Object>> myOrders() {
        return orderService.findMyOrders().stream()
                .map(this::toMap)
                .collect(Collectors.toList());
    }

    private Map<String, Object> toMap(Order o) {
        return Map.of(
                "id", o.getId(),
                "status", o.getStatus().getTitle(),
                "createDate", o.getCreateDate().toString(),
                "deliveryDate", o.getDeliveryDate().toString(),
                "pickupPoint", o.getPickupPoint().getAddress(),
                "getCode", o.getGetCode()
        );
    }
}
```

> 💡 **`Map.of(...)`** — быстрый способ собрать неизменяемый словарь. В реальных проектах лучше сделать DTO `OrderDto` с явными полями.

### 11.2. Проверка

Войдите как `maia`, откройте `/api/orders/my`. Получите JSON своих заказов.

> ⚠️ **Проблема:** REST API обычно не использует сессии — мобильные приложения не хранят cookie. В реальности нужен **JWT**. Но для учебного проекта мы оставим сессии — они тоже работают.

---

## Проверка

### 12.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | `/admin` открывается для `Админист`/`Менеджер` | |
| 2 | Дашборд показывает статистику | |
| 3 | `/admin/orders` — все заказы | |
| 4 | Фильтр по статусу работает | |
| 5 | `/admin/orders/{id}` — детали заказа | |
| 6 | Смена статуса работает | |
| 7 | `/api/products` отдаёт JSON | |
| 8 | `/api/products/{id}` — один товар | |
| 9 | `/api/products/UNKNOWN` — 404 с `{"error": "..."}` | |
| 10 | `/api/products/search?q=...` — поиск | |
| 11 | `/api/orders/my` — заказы пользователя | |

### 12.2. Сценарий

1. Войдите как `damir` (Админист).
2. Откройте `/admin` — увидите статистику.
3. Откройте `/admin/orders`, отфильтруйте по статусу.
4. Откройте один заказ, смените статус на «Завершен».
5. Откройте `/api/products` — JSON.
6. Откройте `/api/products/UNKNOWN` — 404.
7. Войдите как `maia`, откройте `/api/orders/my`.

---

## Задание

1. **Создайте `CategoryApiController`**:
   - `GET /api/categories` — список категорий.
   - `GET /api/categories/{id}` — одна категория.
   - DTO `CategoryDto`.

2. **Создайте `PickupPointApiController`**:
   - `GET /api/pickup-points` — список всех пунктов выдачи.

3. **Добавьте пагинацию в REST для продуктов**:
   - `/api/products?page=0&size=20`.
   - Возвращайте не `List<ProductDto>`, а объект с полями `content`, `totalPages`, `totalElements`.

4. **Создайте `util/ApiUtils.java`**:
   - `error(String message)` — возвращает `Map<String, String>` с ошибкой.
   - `success(String message)` — возвращает `Map<String, String>` с успехом.
   - Используйте в контроллерах API.

5. **Сделайте тестовый REST-клиент**:
   - Используйте [Postman](https://www.postman.com/downloads/) или [curl](https://curl.se/).
   - Попробуйте выполнить `GET /api/products`, `GET /api/products/N592T4`, `GET /api/products/UNKNOWN`.
   - Сохраните результаты (можно сделать скриншоты и положить в репозиторий).

6. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 8](Lesson8.md) |
