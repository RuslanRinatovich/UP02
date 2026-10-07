# Занятие 6. Корзина, оформление заказа и «Мои заказы»

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 7](Lesson7.md)

## План
1. [Введение: что было и куда идём](#введение)
2. [Как хранить корзину: сессия vs БД](#как-хранить-корзину)
3. [Модель корзины в сессии](#модель-корзины)
4. [CartService](#cartservice)
5. [CartController: добавление, просмотр, удаление](#cartcontroller)
6. [Кнопка «В корзину» в каталоге и карточке товара](#кнопка-в-корзину)
7. [Страница корзины](#страница-корзины)
8. [Entity Order и OrderProduct](#entity-order-и-orderproduct)
9. [OrderRepository и OrderService](#orderrepository-и-orderservice)
10. [Оформление заказа](#оформление-заказа)
11. [Страница «Мои заказы»](#страница-мои-заказы)
12. [OrderUtils: генерация кода выдачи](#orderutils)
13. [Проверка и чек-лист](#проверка)
14. [Задание](#задание)

---

## Введение

За пять занятий мы сделали:
- CRUD для товаров.
- Слоистую архитектуру с пакетами.
- Spring Security с ролями.
- Каталог, карточку товара, пагинацию.

**Проблема:** покупатель может смотреть товары, но не может их **купить**. Нет корзины, нет оформления заказа.

**Сегодня:** сделаем корзину, оформление заказа, сохранение заказов в БД и страницу «Мои заказы».

В вашей БД уже есть таблицы:
- `order` — заказы.
- `order_product` — состав заказа.
- `pickup_point` — пункты выдачи.
- `status` — статусы заказов.

Мы их используем напрямую.

---

## Как хранить корзину

> 💡 **Корзина — временные данные.** Пользователь ещё ничего не купил, но набрал товары. Есть два подхода:

| Подход | Плюсы | Минусы |
|--------|-------|--------|
| **Сессия** (в памяти) | Просто, быстро, не нужна БД | Теряется при закрытии браузера, не работает при нескольких серверах |
| **БД** | Сохраняется между сессиями | Сложнее: нужна таблица `cart`, больше кода |

**Для учебного проекта:** сессия. Для реального магазина — БД или Redis.

Мы будем использовать `HttpSession` — встроенный механизм Spring для хранения данных между запросами одного пользователя.

> 💡 **Сессия** — это способ «запомнить» пользователя между HTTP-запросами. Когда пользователь впервые открывает сайт, сервер создаёт для него `sessionId` и отправляет в cookie. При следующих запросах браузер возвращает этот cookie — сервер узнаёт «своего» пользователя и достаёт его данные.

---

## Модель корзины

Создайте `src/main/java/com/example/shop/dto/CartItem.java`:

```java
package com.example.shop.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.math.BigDecimal;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class CartItem implements Serializable {

    private String productId;
    private String title;
    private BigDecimal price;
    private int quantity;
    private byte[] photo;

    /**
     * Итоговая стоимость позиции.
     */
    public BigDecimal getTotal() {
        return price.multiply(BigDecimal.valueOf(quantity));
    }
}
```

> 💡 **`Serializable`** — обязательно, потому что объект будет храниться в `HttpSession`. Некоторые серверы сериализуют сессию на диск.

> 💡 **`@NoArgsConstructor` и `@AllArgsConstructor`** — Lombok генерирует конструкторы. Первый — пустой (нужен для сериализации), второй — со всеми полями.

Создайте `src/main/java/com/example/shop/dto/Cart.java`:

```java
package com.example.shop.dto;

import lombok.Data;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Data
public class Cart implements Serializable {

    private List<CartItem> items = new ArrayList<>();

    public void addItem(CartItem newItem) {
        Optional<CartItem> existing = items.stream()
                .filter(i -> i.getProductId().equals(newItem.getProductId()))
                .findFirst();

        if (existing.isPresent()) {
            existing.get().setQuantity(existing.get().getQuantity() + newItem.getQuantity());
        } else {
            items.add(newItem);
        }
    }

    public void removeItem(String productId) {
        items.removeIf(i -> i.getProductId().equals(productId));
    }

    public void updateQuantity(String productId, int quantity) {
        if (quantity <= 0) {
            removeItem(productId);
            return;
        }
        items.stream()
                .filter(i -> i.getProductId().equals(productId))
                .findFirst()
                .ifPresent(i -> i.setQuantity(quantity));
    }

    public void clear() {
        items.clear();
    }

    public BigDecimal getTotal() {
        return items.stream()
                .map(CartItem::getTotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public int getItemCount() {
        return items.stream().mapToInt(CartItem::getQuantity).sum();
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }
}
```

### 3.1. Разбор логики

| Метод | Что делает |
|-------|-----------|
| `addItem` | Если товар уже в корзине — увеличивает количество. Иначе — добавляет новый |
| `removeItem` | Удаляет товар по `productId` |
| `updateQuantity` | Меняет количество. Если ≤ 0 — удаляет |
| `getTotal` | Сумма всех `CartItem.getTotal()` |
| `getItemCount` | Общее количество единиц товаров |
| `isEmpty` | Есть ли что-то в корзине |

> 💡 **Stream API** (`stream()`, `filter`, `map`, `reduce`) — это функциональный стиль Java. Если не знакомы — можно заменить на обычные `for`-циклы. Но в реальной разработке стримы — норма.

---

## CartService

> 💡 **CartService** — работает с корзиной в `HttpSession`. Ключ — `"cart"`, значение — объект `Cart`.

`src/main/java/com/example/shop/service/CartService.java`:

```java
package com.example.shop.service;

import com.example.shop.dto.Cart;
import com.example.shop.dto.CartItem;
import com.example.shop.entity.Product;
import jakarta.servlet.http.HttpSession;
import org.springframework.stereotype.Service;

@Service
public class CartService {

    private static final String CART_KEY = "cart";

    /**
     * Получить корзину из сессии. Если её нет — создать.
     */
    public Cart getCart(HttpSession session) {
        Cart cart = (Cart) session.getAttribute(CART_KEY);
        if (cart == null) {
            cart = new Cart();
            session.setAttribute(CART_KEY, cart);
        }
        return cart;
    }

    /**
     * Добавить товар в корзину.
     */
    public void addToCart(HttpSession session, Product product, int quantity) {
        Cart cart = getCart(session);
        CartItem item = new CartItem(
                product.getId(),
                product.getTitle(),
                product.getCost(),
                quantity,
                product.getPhoto()
        );
        cart.addItem(item);
    }

    public void removeFromCart(HttpSession session, String productId) {
        getCart(session).removeItem(productId);
    }

    public void updateQuantity(HttpSession session, String productId, int quantity) {
        getCart(session).updateQuantity(productId, quantity);
    }

    public void clearCart(HttpSession session) {
        getCart(session).clear();
    }
}
```

> ⚠️ **`@Service` без `@Transactional`:** CartService не работает с БД, поэтому транзакции не нужны.

---

## CartController

`src/main/java/com/example/shop/controller/CartController.java`:

```java
package com.example.shop.controller;

import com.example.shop.dto.Cart;
import com.example.shop.entity.Product;
import com.example.shop.service.CartService;
import com.example.shop.service.ProductService;
import jakarta.servlet.http.HttpSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/cart")
public class CartController {

    @Autowired
    private CartService cartService;

    @Autowired
    private ProductService productService;

    @GetMapping
    public String viewCart(HttpSession session, Model model) {
        Cart cart = cartService.getCart(session);
        model.addAttribute("cart", cart);
        return "cart";
    }

    @PostMapping("/add")
    public String addToCart(@RequestParam String productId,
                            @RequestParam(defaultValue = "1") int quantity,
                            HttpSession session) {
        Product product = productService.findById(productId);
        cartService.addToCart(session, product, quantity);
        return "redirect:/cart";
    }

    @PostMapping("/remove")
    public String removeFromCart(@RequestParam String productId, HttpSession session) {
        cartService.removeFromCart(session, productId);
        return "redirect:/cart";
    }

    @PostMapping("/update")
    public String updateQuantity(@RequestParam String productId,
                                 @RequestParam int quantity,
                                 HttpSession session) {
        cartService.updateQuantity(session, productId, quantity);
        return "redirect:/cart";
    }

    @PostMapping("/clear")
    public String clearCart(HttpSession session) {
        cartService.clearCart(session);
        return "redirect:/cart";
    }
}
```

> 💡 Все методы, изменяющие корзину, — **POST**. Это позволяет избежать случайных изменений при обновлении страницы (F5).

---

## Кнопка «В корзину»

### 6.1. В карточке товара `product-details.html`

Добавьте **после цены**:

```html
<form th:action="@{/cart/add}" method="post" style="margin-top: 20px;">
    <input type="hidden" name="productId" th:value="${product.id}"/>
    <label>Количество:</label>
    <input type="number" name="quantity" value="1" min="1"
           th:max="${product.quantityInStock}" style="width: 80px;"/>
    <button type="submit"
            th:disabled="${product.quantityInStock == 0}">
        <span th:if="${product.quantityInStock > 0}">В корзину</span>
        <span th:if="${product.quantityInStock == 0}">Нет в наличии</span>
    </button>
</form>
```

### 6.2. В карточках каталога и главной

В `catalog.html` и `home.html` добавьте в каждую карточку:

```html
<form th:action="@{/cart/add}" method="post" style="margin-top: 10px;">
    <input type="hidden" name="productId" th:value="${p.id}"/>
    <input type="hidden" name="quantity" value="1"/>
    <button type="submit"
            th:disabled="${p.quantityInStock == 0}">
        <span th:if="${p.quantityInStock > 0}">В корзину</span>
        <span th:if="${p.quantityInStock == 0}">Нет</span>
    </button>
</form>
```

### 6.3. Ссылка на корзину в шапке

Добавьте в `layout.html` в блок `.user-box`:

```html
<a th:href="@{/cart}">
    🛒 Корзина
    <span th:if="${session.cart != null && session.cart.itemCount > 0}"
          th:text="'(' + ${session.cart.itemCount} + ')'"></span>
</a>
```

> 💡 **`session.cart`** — доступ к объекту в сессии через `session.*` в Thymeleaf.

---

## Страница корзины

Создайте `src/main/resources/templates/cart.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Корзина', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Корзина</h1>

    <div th:if="${cart.isEmpty()}" style="padding: 30px; background: white;
                                             text-align: center; border-radius: 8px;">
        <p>Корзина пуста</p>
        <a th:href="@{/catalog}">Перейти в каталог</a>
    </div>

    <div th:if="${!cart.isEmpty()}">
        <table>
            <thead>
            <tr>
                <th>Фото</th>
                <th>Название</th>
                <th>Цена</th>
                <th>Количество</th>
                <th>Итого</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${cart.items}">
                <td>
                    <img th:src="@{/product/{id}/photo(id=${item.productId})}"
                         style="width: 50px; height: 50px; object-fit: contain;"/>
                </td>
                <td>
                    <a th:href="@{/product/{id}(id=${item.productId})}"
                       th:text="${item.title}"></a>
                </td>
                <td th:text="${item.price} + ' ₽'"></td>
                <td>
                    <form th:action="@{/cart/update}" method="post" style="display: flex; gap: 5px;">
                        <input type="hidden" name="productId" th:value="${item.productId}"/>
                        <input type="number" name="quantity" th:value="${item.quantity}"
                               min="1" style="width: 60px;"/>
                        <button type="submit">↻</button>
                    </form>
                </td>
                <td th:text="${item.total} + ' ₽'"></td>
                <td>
                    <form th:action="@{/cart/remove}" method="post">
                        <input type="hidden" name="productId" th:value="${item.productId}"/>
                        <button type="submit">Удалить</button>
                    </form>
                </td>
            </tr>
            </tbody>
        </table>

        <div style="margin-top: 20px; text-align: right; font-size: 20px;">
            <b>Итого: <span th:text="${cart.total} + ' ₽'"></span></b>
        </div>

        <div style="margin-top: 20px; display: flex; justify-content: space-between;">
            <form th:action="@{/cart/clear}" method="post">
                <button type="submit" onclick="return confirm('Очистить корзину?')">
                    Очистить
                </button>
            </form>
            <a th:href="@{/checkout}"
               style="background: #28a745; color: white; padding: 10px 20px; border-radius: 4px;">
                Оформить заказ
            </a>
        </div>
    </div>

</div>
</body>
</html>
```

---

## Entity Order и OrderProduct

Теперь сохраним заказ в БД. У нас есть таблицы `order` и `order_product`.

### 8.1. Entity Order

`src/main/java/com/example/shop/entity/Order.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "\"order\"")
@Data
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "status_id", nullable = false)
    private Status status;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "pickuppoint_id", nullable = false)
    private PickupPoint pickupPoint;

    @Column(name = "create_date", nullable = false)
    private LocalDate createDate;

    @Column(name = "delivery_date", nullable = false)
    private LocalDate deliveryDate;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "username")
    private User user;

    @Column(name = "get_code", nullable = false)
    private Integer getCode;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderProduct> items = new ArrayList<>();
}
```

> ⚠️ **`@Table(name = "\"order\"")`** — `order` в PostgreSQL зарезервированное слово, поэтому экранируем кавычками.

### 8.2. Entity OrderProduct

`src/main/java/com/example/shop/entity/OrderProduct.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "order_product")
@Data
public class OrderProduct {

    @EmbeddedId
    private OrderProductId id;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("productId")
    @JoinColumn(name = "product_id")
    private Product product;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("orderId")
    @JoinColumn(name = "order_id")
    private Order order;

    @Column(name = "count")
    private Integer count;
}
```

`src/main/java/com/example/shop/entity/OrderProductId.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.Column;
import jakarta.persistence.Embeddable;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Embeddable
@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderProductId implements Serializable {

    @Column(name = "product_id")
    private String productId;

    @Column(name = "order_id")
    private Integer orderId;
}
```

> 💡 **Составной ключ:** в таблице `order_product` первичный ключ — это пара `(product_id, order_id)`. В JPA это делается через `@EmbeddedId` + `@Embeddable`.

### 8.3. Entity Status и PickupPoint

`src/main/java/com/example/shop/entity/Status.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "status")
@Data
public class Status {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "title", nullable = false, length = 50)
    private String title;
}
```

`src/main/java/com/example/shop/entity/PickupPoint.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "pickup_point")
@Data
public class PickupPoint {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "address", nullable = false)
    private String address;
}
```

---

## OrderRepository и OrderService

### 9.1. Репозитории

`src/main/java/com/example/shop/repository/OrderRepository.java`:

```java
package com.example.shop.repository;

import com.example.shop.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface OrderRepository extends JpaRepository<Order, Integer> {

    List<Order> findByUserUsernameOrderByCreateDateDesc(String username);
}
```

`src/main/java/com/example/shop/repository/StatusRepository.java`:

```java
package com.example.shop.repository;

import com.example.shop.entity.Status;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface StatusRepository extends JpaRepository<Status, Integer> {

    Optional<Status> findByTitle(String title);
}
```

`src/main/java/com/example/shop/repository/PickupPointRepository.java`:

```java
package com.example.shop.repository;

import com.example.shop.entity.PickupPoint;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface PickupPointRepository extends JpaRepository<PickupPoint, Integer> {
}
```

### 9.2. OrderService

`src/main/java/com/example/shop/service/OrderService.java`:

```java
package com.example.shop.service;

import com.example.shop.dto.Cart;
import com.example.shop.dto.CartItem;
import com.example.shop.entity.*;
import com.example.shop.repository.*;
import com.example.shop.util.OrderUtils;
import com.example.shop.util.SecurityUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.util.List;

@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private StatusRepository statusRepository;

    @Autowired
    private PickupPointRepository pickupPointRepository;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private ProductRepository productRepository;

    @Transactional
    public Order createOrder(Cart cart, Integer pickupPointId) {
        String username = SecurityUtils.getCurrentUsername();
        if (username == null) {
            throw new SecurityException("Необходимо войти в систему");
        }

        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new IllegalArgumentException("Пользователь не найден"));

        PickupPoint pickupPoint = pickupPointRepository.findById(pickupPointId)
                .orElseThrow(() -> new IllegalArgumentException("Пункт выдачи не найден"));

        Status newStatus = statusRepository.findByTitle("Новый ")
                .orElseThrow(() -> new IllegalStateException("Статус 'Новый' не найден в БД"));

        Order order = new Order();
        order.setUser(user);
        order.setPickupPoint(pickupPoint);
        order.setStatus(newStatus);
        order.setCreateDate(LocalDate.now());
        order.setDeliveryDate(LocalDate.now().plusDays(7));
        order.setGetCode(OrderUtils.generatePickupCode());

        for (CartItem cartItem : cart.getItems()) {
            Product product = productRepository.findById(cartItem.getProductId())
                    .orElseThrow(() -> new IllegalArgumentException("Товар не найден"));

            OrderProduct op = new OrderProduct();
            op.setId(new OrderProductId(product.getId(), null)); // id заказа будет после save
            op.setProduct(product);
            op.setOrder(order);
            op.setCount(cartItem.getQuantity());

            order.getItems().add(op);
        }

        return orderRepository.save(order);
    }

    public List<Order> findMyOrders() {
        String username = SecurityUtils.getCurrentUsername();
        if (username == null) {
            return List.of();
        }
        return orderRepository.findByUserUsernameOrderByCreateDateDesc(username);
    }
}
```

> 💡 **`ON DELETE SET NULL`** в вашей БД у `order.username` — если пользователя удалить, заказ останется с `NULL` в `username`. `@ManyToOne` без `nullable=false` это допускает.

---

## Оформление заказа

### 10.1. CheckoutController

`src/main/java/com/example/shop/controller/CheckoutController.java`:

```java
package com.example.shop.controller;

import com.example.shop.dto.Cart;
import com.example.shop.entity.Order;
import com.example.shop.repository.PickupPointRepository;
import com.example.shop.service.CartService;
import com.example.shop.service.OrderService;
import jakarta.servlet.http.HttpSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/checkout")
public class CheckoutController {

    @Autowired
    private CartService cartService;

    @Autowired
    private OrderService orderService;

    @Autowired
    private PickupPointRepository pickupPointRepository;

    @GetMapping
    public String checkoutPage(HttpSession session, Model model) {
        Cart cart = cartService.getCart(session);
        if (cart.isEmpty()) {
            return "redirect:/cart";
        }
        model.addAttribute("cart", cart);
        model.addAttribute("pickupPoints", pickupPointRepository.findAll());
        return "checkout";
    }

    @PostMapping
    public String processCheckout(@RequestParam Integer pickupPointId,
                                  HttpSession session) {
        Cart cart = cartService.getCart(session);
        if (cart.isEmpty()) {
            return "redirect:/cart";
        }

        Order order = orderService.createOrder(cart, pickupPointId);

        // очищаем корзину после успешного заказа
        cartService.clearCart(session);

        return "redirect:/orders/" + order.getId() + "?success=true";
    }
}
```

### 10.2. Шаблон `checkout.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Оформление заказа', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Оформление заказа</h1>

    <div style="background: white; padding: 20px; border-radius: 8px;">
        <h2>Ваш заказ</h2>
        <table>
            <thead>
            <tr>
                <th>Название</th>
                <th>Цена</th>
                <th>Количество</th>
                <th>Итого</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${cart.items}">
                <td th:text="${item.title}"></td>
                <td th:text="${item.price} + ' ₽'"></td>
                <td th:text="${item.quantity}"></td>
                <td th:text="${item.total} + ' ₽'"></td>
            </tr>
            </tbody>
        </table>

        <p style="text-align: right; font-size: 20px; margin-top: 20px;">
            <b>Итого: <span th:text="${cart.total} + ' ₽'"></span></b>
        </p>

        <form th:action="@{/checkout}" method="post" style="margin-top: 20px;">
            <div class="form-group">
                <label>Пункт выдачи *</label>
                <select name="pickupPointId" required>
                    <option value="">-- Выберите пункт выдачи --</option>
                    <option th:each="pp : ${pickupPoints}"
                            th:value="${pp.id}"
                            th:text="${pp.address}"></option>
                </select>
            </div>

            <button type="submit" style="background: #28a745; color: white;
                                          padding: 12px 24px; border: none; border-radius: 4px;">
                Подтвердить заказ
            </button>
        </form>
    </div>

</div>
</body>
</html>
```

---

## Страница «Мои заказы»

### 11.1. OrderController

`src/main/java/com/example/shop/controller/OrderController.java`:

```java
package com.example.shop.controller;

import com.example.shop.entity.Order;
import com.example.shop.repository.OrderRepository;
import com.example.shop.service.OrderService;
import com.example.shop.util.SecurityUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @Autowired
    private OrderRepository orderRepository;

    @GetMapping
    public String myOrders(Model model) {
        model.addAttribute("orders", orderService.findMyOrders());
        return "orders";
    }

    @GetMapping("/{id}")
    public String orderDetails(@PathVariable Integer id, Model model) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Заказ не найден"));

        // Проверяем, что заказ принадлежит текущему пользователю
        String current = SecurityUtils.getCurrentUsername();
        if (order.getUser() != null && !order.getUser().getUsername().equals(current)) {
            throw new SecurityException("Нет доступа к чужому заказу");
        }

        model.addAttribute("order", order);
        return "order-details";
    }
}
```

### 11.2. Шаблон `orders.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Мои заказы', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Мои заказы</h1>

    <div th:if="${orders.isEmpty()}" style="padding: 30px; background: white; border-radius: 8px;">
        <p>У вас пока нет заказов.</p>
        <a th:href="@{/catalog}">Перейти в каталог</a>
    </div>

    <table th:if="${!orders.isEmpty()}">
        <thead>
        <tr>
            <th>№</th>
            <th>Дата создания</th>
            <th>Дата выдачи</th>
            <th>Статус</th>
            <th>Пункт выдачи</th>
            <th>Код выдачи</th>
            <th></th>
        </tr>
        </thead>
        <tbody>
        <tr th:each="o : ${orders}">
            <td th:text="${o.id}"></td>
            <td th:text="${o.createDate}"></td>
            <td th:text="${o.deliveryDate}"></td>
            <td th:text="${o.status.title}"></td>
            <td th:text="${o.pickupPoint.address}"></td>
            <td th:text="${o.getCode}"></td>
            <td>
                <a th:href="@{/orders/{id}(id=${o.id})}">Подробнее</a>
            </td>
        </tr>
        </tbody>
    </table>

</div>
</body>
</html>
```

### 11.3. Шаблон `order-details.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Заказ', content=~{::content})}">
<body>
<div th:fragment="content">

    <div th:if="${param.success}" 
         style="background: #d4edda; color: #155724; padding: 15px; border-radius: 6px; margin-bottom: 20px;">
        ✅ Заказ успешно оформлен!
    </div>

    <h1>Заказ №<span th:text="${order.id}"></span></h1>

    <div style="background: white; padding: 20px; border-radius: 8px;">
        <p><b>Дата создания:</b> <span th:text="${order.createDate}"></span></p>
        <p><b>Дата выдачи:</b> <span th:text="${order.deliveryDate}"></span></p>
        <p><b>Статус:</b> <span th:text="${order.status.title}"></span></p>
        <p><b>Пункт выдачи:</b> <span th:text="${order.pickupPoint.address}"></span></p>
        <p style="font-size: 24px; color: #0066cc;">
            <b>Код выдачи: <span th:text="${order.getCode}"></span></b>
        </p>

        <h2>Состав заказа</h2>
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

Добавьте в `layout.html` в блок навигации:

```html
<a th:if="${#authorization.expression('isAuthenticated()')}"
   th:href="@{/orders}">Мои заказы</a>
```

---

## OrderUtils: генерация кода выдачи

`src/main/java/com/example/shop/util/OrderUtils.java`:

```java
package com.example.shop.util;

import java.security.SecureRandom;

public final class OrderUtils {

    private static final SecureRandom RANDOM = new SecureRandom();

    private OrderUtils() {
    }

    /**
     * Генерирует 3-значный код выдачи заказа.
     * От 100 до 999 — так исключаем ведущие нули.
     */
    public static int generatePickupCode() {
        return 100 + RANDOM.nextInt(900);
    }
}
```

> 💡 **`SecureRandom`** используется для генерации кодов, которые можно использовать в реальной жизни. `Random` предсказуем — для реального магазина это плохо.

---

## Проверка

### 13.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | `Cart`, `CartItem` созданы | |
| 2 | `CartService` работает с `HttpSession` | |
| 3 | `CartController` принимает POST `/cart/add`, `/cart/remove`, `/cart/update` | |
| 4 | Кнопка «В корзину» есть в каталоге и карточке товара | |
| 5 | Страница `/cart` открывается | |
| 6 | В шапке виден счётчик корзины | |
| 7 | `Order` и `OrderProduct` созданы | |
| 8 | `OrderService.createOrder` создаёт заказ и его позиции | |
| 9 | Страница `/checkout` работает | |
| 10 | После оформления корзина очищается | |
| 11 | Страница `/orders` показывает заказы пользователя | |
| 12 | Страница `/orders/{id}` показывает детали | |
| 13 | Код выдачи генерируется случайно | |

### 13.2. Сценарий для проверки

1. Войдите как `maia`.
2. Откройте каталог, добавьте 2–3 товара в корзину.
3. Откройте `/cart` — проверьте итог.
4. Нажмите «Оформить заказ», выберите пункт выдачи.
5. Подтвердите — перекинет на `/orders/{id}?success=true`.
6. Откройте `/orders` — заказ должен быть в списке.
7. Убедитесь, что корзина пуста.

---

## Задание

1. **Создайте страницу `/orders/{id}` для администратора**:
   - Показывает все заказы (`OrderRepository.findAll()`).
   - Доступна только для роли `Админист`.
   - В таблице: `id`, пользователь, статус, дата, код выдачи.
   - Ссылка на детали каждого заказа.

2. **Добавьте изменение статуса заказа**:
   - `POST /admin/orders/{id}/status` с параметром `statusId`.
   - Форма со сменой статуса на странице деталей заказа.
   - Доступно только для `Админист` и `Менеджер`.

3. **Проверьте остаток товара при добавлении в корзину**:
   - В `CartService.addToCart` проверяйте, что `quantity <= product.getQuantityInStock()`.
   - Если превышает — бросайте исключение или просто ограничивайте.
   - В `CartController.addToCart` обрабатывайте это и возвращайте ошибку через `RedirectAttributes`.

4. **Создайте `util/CartUtils.java`**:
   - `sumTotal(Cart)` — итоговая сумма.
   - `countItems(Cart)` — количество позиций.
   - `hasProduct(Cart, String productId)` — есть ли товар в корзине.
   - Используйте в `CartService` вместо дублирования логики.

5. **Добавьте страницу успеха `/order-success`**:
   - Отдельная страница после оформления заказа.
   - Показывает номер заказа и код выдачи.
   - Кнопка «Перейти в каталог».

6. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 7](Lesson7.md) |
