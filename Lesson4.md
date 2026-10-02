# Занятие 4. Spring Security: аутентификация, роли и защита админки

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 5](Lesson5.md)

## План
1. [Введение: зачем нужна безопасность](#введение)
2. [Аутентификация vs авторизация](#аутентификация-vs-авторизация)
3. [Как работает Spring Security (фильтры)](#как-работает-spring-security)
4. [Entity User и Role](#entity-user-и-role)
5. [UserRepository и UserService](#userrepository-и-userservice)
6. [BCrypt: почему нельзя хранить пароли в открытом виде](#bcrypt)
7. [Конфигурация SecurityConfig](#конфигурация-securityconfig)
8. [Страница входа (login)](#страница-входа)
9. [Кнопка выхода и отображение пользователя](#кнопка-выхода)
10. [Ограничение доступа по ролям](#ограничение-доступа-по-ролям)
11. [SecurityUtils в пакете util](#securityutils-в-пакете-util)
12. [Проверка и чек-лист](#проверка)
13. [Задание](#задание)

---

## Введение

За три занятия мы построили приложение, которое:
- читает товары и категории из БД,
- добавляет, редактирует и удаляет товары,
- имеет аккуратную слоистую архитектуру.

**Проблема:** кто угодно может открыть `/admin/products/new` и удалить все товары. Это недопустимо для реального интернет-магазина.

**Сегодня:** закроем админку паролем, добавим роли (клиент, менеджер, администратор) и научим приложение показывать имя текущего пользователя.

> 💡 В вашей БД уже есть таблицы `user` и `role` — мы их используем. Если их нет, загляните в `ScriptWithData.sql`.

---

## Аутентификация vs авторизация

> 💡 Эти два слова часто путают. Разница принципиальная.

| Термин | Вопрос, на который отвечает | Пример |
|--------|----------------------------|--------|
| **Аутентификация** (authentication) | «Кто ты?» | Ввод логина и пароля |
| **Авторизация** (authorization) | «Что тебе можно?» | Роль `ADMIN` может удалять товары, роль `CLIENT` — нет |

**Порядок такой:**
1. Сначала приложение проверяет, кто вы (аутентификация).
2. Потом смотрит, что вам разрешено (авторизация).

Если аутентификация не пройдена — вы анонимный гость. Если пройдена, но роль не подходит — вы получаете **403 Forbidden**.

---

## Как работает Spring Security

> 💡 **Spring Security — это цепочка сервлет-фильтров.** Каждый HTTP-запрос проходит через них по очереди, ещё до того, как он дойдёт до вашего контроллера.

Упрощённо:

```
HTTP-запрос
    ↓
[Filter 1] SecurityContextPersistenceFilter   ← загружает пользователя из сессии
    ↓
[Filter 2] UsernamePasswordAuthenticationFilter ← обрабатывает POST /login
    ↓
[Filter 3] AuthorizationFilter                ← проверяет права на URL
    ↓
Ваш Controller
```

**Что это значит на практике:**
- Если запрос идёт на защищённый URL и пользователь не залогинен — Spring перенаправит его на `/login`.
- Если пользователь залогинен, но у него нет нужной роли — вернётся 403.
- Если всё ок — запрос дойдёт до контроллера.

> 💡 **Важно:** Spring Security сам по себе не знает, где хранятся пользователи. Мы должны дать ему `UserDetailsService`, который умеет искать пользователя по логину.

---

## Entity User и Role

В нашей БД есть таблица `user` с полями `username`, `password`, `role_id` и таблица `role`. Создадим для них Entity.

### 4.1. Entity Role

`src/main/java/com/example/shop/entity/Role.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "role")
@Data
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "title", nullable = false, length = 20)
    private String title;
}
```

### 4.2. Entity User

`src/main/java/com/example/shop/entity/User.java`:

```java
package com.example.shop.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "\"user\"")
@Data
public class User {

    @Id
    @Column(name = "username", length = 50)
    private String username;

    @Column(name = "password", nullable = false, length = 50)
    private String password;

    @Column(name = "first_name", nullable = false, length = 30)
    private String firstName;

    @Column(name = "second_name", nullable = false, length = 30)
    private String secondName;

    @Column(name = "middle_name", length = 30)
    private String middleName;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "role_id")
    private Role role;
}
```

> ⚠️ **Важно:** `@Table(name = "\"user\"")` — в PostgreSQL `user` — зарезервированное слово, поэтому имя таблицы нужно экранировать двойными кавычками. В Java-строке кавычки экранируются как `\"`.

> 💡 Обратите внимание: `username` — это не `id`, а сам первичный ключ (строковый). Так устроена ваша БД.

---

## UserRepository и UserService

### 5.1. UserRepository

`src/main/java/com/example/shop/repository/UserRepository.java`:

```java
package com.example.shop.repository;

import com.example.shop.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, String> {

    Optional<User> findByUsername(String username);
}
```

> 💡 **`findByUsername`** — Spring Data JPA сам сгенерирует реализацию по имени метода. Никакого SQL писать не нужно: `findBy<Поле>` → `SELECT * FROM user WHERE username = ?`.

### 5.2. UserService

`src/main/java/com/example/shop/service/UserService.java`:

```java
package com.example.shop.service;

import com.example.shop.entity.User;
import com.example.shop.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> findAll() {
        return userRepository.findAll();
    }

    public Optional<User> findByUsername(String username) {
        return userRepository.findByUsername(username);
    }

    public User save(User user) {
        return userRepository.save(user);
    }
}
```

---

## BCrypt: почему нельзя хранить пароли в открытом виде

> 💡 **Никогда** не храните пароли в открытом виде. Если БД утечёт, злоумышленник получит все пароли. Вместо этого хранят **хеш**.

**Что такое хеш:**
- Хеш-функция превращает строку (`qwerty123`) в непонятный набор символов (`$2a$10$N9qo8uLO...`).
- Из хеша **невозможно** восстановить исходный пароль.
- Но можно проверить: если хеш от введённого пароля совпадает с сохранённым — пароль верный.

> 💡 **BCrypt** — специальный алгоритм хеширования, который:
> 1. Медленный (защита от брутфорса).
> 2. Использует «соль» — случайные данные, добавляемые к паролю. Это значит, что одинаковые пароли у разных пользователей дают **разные** хеши.

**В чём проблема нашей БД:** в таблице `user` пароли хранятся в открытом виде (например, `'1'`, `'2'`). Для учебного проекта это ок, но в реальной жизни так нельзя.

### 6.1. Что мы сделаем

1. Добавим `PasswordEncoder` (BCrypt) в конфигурацию.
2. При создании нового пользователя будем хешировать пароль.
3. Для **уже существующих** пользователей из `ScriptWithData.sql` — закодируем пароли вручную через SQL.

### 6.2. Генерация BCrypt-хеша

Создайте временный класс `util/PasswordHashGenerator.java`:

```java
package com.example.shop.util;

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class PasswordHashGenerator {
    public static void main(String[] args) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        System.out.println("Пароль 'demo': " + encoder.encode("demo"));
        System.out.println("Пароль 'admin': " + encoder.encode("admin"));
    }
}
```

Запустите — получите что-то вроде:

```
Пароль 'demo': $2a$10$wH8Qx...
Пароль 'admin': $2a$10$K7pMz...
```

### 6.3. Обновите пароли в БД

В DBeaver выполните:

```sql
UPDATE "user" SET password = '$2a$10$...' WHERE username = 'maia';
UPDATE "user" SET password = '$2a$10$...' WHERE username = 'damir';
-- и так далее для всех пользователей
```

Или создайте SQL-скрипт со всеми хешами.

> ⚠️ **Важно:** поле `password` в вашей БД — `varchar(50)`. BCrypt-хеш — 60 символов. Нужно **расширить поле**:
> ```sql
> ALTER TABLE "user" ALTER COLUMN password TYPE varchar(100);
> ```

После этого удалите `PasswordHashGenerator.java` — он нужен был только один раз.

---

## Конфигурация SecurityConfig

Создайте `src/main/java/com/example/shop/config/SecurityConfig.java`:

```java
package com.example.shop.config;

import com.example.shop.entity.User;
import com.example.shop.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private UserRepository userRepository;

    /**
     * Шифрование паролей через BCrypt.
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * Как Spring ищет пользователя по логину.
     */
    @Bean
    public UserDetailsService userDetailsService() {
        return username -> {
            User user = userRepository.findByUsername(username)
                    .orElseThrow(() -> new UsernameNotFoundException("Пользователь не найден: " + username));

            return org.springframework.security.core.userdetails.User
                    .withUsername(user.getUsername())
                    .password(user.getPassword())
                    .roles(user.getRole() != null ? user.getRole().getTitle() : "CLIENT")
                    .build();
        };
    }

    /**
     * Правила доступа к URL.
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/login", "/css/**", "/js/**", "/images/**").permitAll()
                        .requestMatchers("/", "/products-page", "/products-jpa-page",
                                         "/categories", "/products-jpa").permitAll()
                        .requestMatchers("/admin/**").hasAnyRole("Админист", "Менеджер")
                        .anyRequest().authenticated()
                )
                .formLogin(form -> form
                        .loginPage("/login")
                        .defaultSuccessUrl("/products-jpa-page", true)
                        .permitAll()
                )
                .logout(logout -> logout
                        .logoutUrl("/logout")
                        .logoutSuccessUrl("/products-jpa-page")
                        .permitAll()
                );
        return http.build();
    }
}
```

### 7.1. Разбор конфигурации

| Часть | Что делает |
|-------|-----------|
| `@Configuration` | Помечает класс как источник настроек |
| `@EnableWebSecurity` | Активирует Spring Security |
| `passwordEncoder()` | Создаёт бин BCrypt для кодирования/проверки паролей |
| `userDetailsService()` | Определяет, как искать пользователя по логину |
| `filterChain()` | Основные правила: что защищено, что открыто, куда редиректить |
| `.permitAll()` | Разрешить всем (в том числе анонимам) |
| `.hasAnyRole(...)` | Разрешить только с ролями |
| `.anyRequest().authenticated()` | Всё остальное — только после логина |

> ⚠️ **Роли в вашей БД:** `"Админист"`, `"Менеджер"`, `"Клиент"`. Spring Security по умолчанию ожидает роли в виде `ROLE_XXX`. Метод `.roles("Админист")` сам добавит префикс `ROLE_` — не добавляйте его вручную.

### 7.2. Почему `/login` — кастомная страница

Spring Security по умолчанию предоставляет свою страницу входа. Она некрасивая. Мы сделаем свою, а Spring будет обрабатывать POST `/login` сам.

---

## Страница входа

Создайте `src/main/java/com/example/shop/controller/LoginController.java`:

```java
package com.example.shop.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class LoginController {

    @GetMapping("/login")
    public String loginPage() {
        return "login";
    }
}
```

Создайте `src/main/resources/templates/login.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Вход</title>
    <style>
        body { font-family: Arial, sans-serif; display: flex; justify-content: center;
               align-items: center; height: 100vh; margin: 0; background: #f4f4f4; }
        .login-box { background: white; padding: 30px; border-radius: 8px;
                     box-shadow: 0 2px 10px rgba(0,0,0,0.1); width: 320px; }
        h1 { margin-top: 0; text-align: center; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input { width: 100%; padding: 8px; box-sizing: border-box; }
        button { width: 100%; padding: 10px; background: #007bff; color: white;
                 border: none; cursor: pointer; border-radius: 4px; }
        .error { color: red; text-align: center; margin-bottom: 15px; }
        .success { color: green; text-align: center; margin-bottom: 15px; }
    </style>
</head>
<body>
    <div class="login-box">
        <h1>Вход</h1>

        <div class="error" th:if="${param.error}">Неверный логин или пароль</div>
        <div class="success" th:if="${param.logout}">Вы вышли из системы</div>

        <form th:action="@{/login}" method="post">
            <div class="form-group">
                <label>Логин</label>
                <input type="text" name="username" required autofocus/>
            </div>
            <div class="form-group">
                <label>Пароль</label>
                <input type="password" name="password" required/>
            </div>
            <button type="submit">Войти</button>
        </form>
    </div>
</body>
</html>
```

> ⚠️ **Имена полей обязательны:** `username` и `password`. Spring Security ищет именно их в POST-запросе. Если переименовать — аутентификация не сработает.

---

## Кнопка выхода

Добавьте в `templates/products-jpa.html` (или в общий фрагмент) блок:

```html
<div style="text-align:right; margin-bottom: 20px;">
    <span th:if="${#authorization.expression('isAuthenticated()')}">
        Вы вошли как: <b th:text="${#authentication.name}">guest</b>
        <form th:action="@{/logout}" method="post" style="display:inline; margin-left:10px;">
            <button type="submit">Выйти</button>
        </form>
    </span>
    <a th:if="${!#authorization.expression('isAuthenticated()')}" th:href="@{/login}">Войти</a>
</div>
```

> 💡 **`#authentication.name`** — встроенный объект Thymeleaf, содержит логин текущего пользователя.
> **`#authorization.expression('isAuthenticated()')`** — проверяет, залогинен ли пользователь.

> ⚠️ Logout должен быть POST, а не GET — это требование Spring Security по умолчанию.

---

## Ограничение доступа по ролям

Теперь `/admin/**` защищён. Попробуйте:

1. Выйдите из системы (если залогинены).
2. Откройте `http://localhost:8080/admin/products/new`.
3. Вас перекинет на `/login`.
4. Войдите как `maia` (роль Клиент) — вернётся **403 Forbidden**.
5. Войдите как `damir` (роль Админист) — форма откроется.

> 💡 **Проверка роли в шаблонах.** Если нужно скрыть кнопку «Удалить» от не-админов:
>
> ```html
> <form th:if="${#authorization.expression('hasRole(''Админист'')')}"
>       th:action="@{/admin/products/delete/{id}(id=${p.id})}" method="post">
>     <button type="submit">Удалить</button>
> </form>
> ```

---

## SecurityUtils в пакете util

> 💡 Если вам часто нужно получать текущего пользователя в коде — вынесите это в утилиту.

`src/main/java/com/example/shop/util/SecurityUtils.java`:

```java
package com.example.shop.util;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;

public final class SecurityUtils {

    private SecurityUtils() {
    }

    /**
     * Возвращает логин текущего пользователя или null, если аноним.
     */
    public static String getCurrentUsername() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()
                || "anonymousUser".equals(auth.getPrincipal())) {
            return null;
        }
        return auth.getName();
    }

    /**
     * Проверяет, что текущий пользователь имеет указанную роль.
     */
    public static boolean hasRole(String role) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null) {
            return false;
        }
        return auth.getAuthorities().stream()
                .anyMatch(a -> a.getAuthority().equals("ROLE_" + role));
    }

    /**
     * Проверяет, что пользователь залогинен.
     */
    public static boolean isAuthenticated() {
        return getCurrentUsername() != null;
    }
}
```

**Использование в Service:**

```java
import com.example.shop.util.SecurityUtils;

@Transactional
public Product createFromForm(ProductForm form) {
    if (!SecurityUtils.hasRole("Админист")) {
        throw new SecurityException("Только администратор может создавать товары");
    }
    // ...
}
```

**Использование в Controller:**

```java
@GetMapping("/my-profile")
public String myProfile(Model model) {
    String username = SecurityUtils.getCurrentUsername();
    if (username == null) {
        return "redirect:/login";
    }
    model.addAttribute("user", userService.findByUsername(username).orElseThrow());
    return "profile";
}
```

---

## Проверка

### 12.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | Entity `User` и `Role` созданы | |
| 2 | `UserRepository.findByUsername` работает | |
| 3 | BCrypt-хеши сгенерированы и проставлены в БД | |
| 4 | `SecurityConfig` компилируется | |
| 5 | Страница `/login` открывается | |
| 6 | Вход под `damir` (Админист) работает | |
| 7 | Вход под `maia` (Клиент) — редирект на 403 при `/admin/**` | |
| 8 | Кнопка «Выйти» работает | |
| 9 | На странице видно имя текущего пользователя | |
| 10 | Незалогиненный пользователь не может зайти в `/admin/**` | |

### 12.2. Сценарий для проверки

1. Откройте `/admin/products/new` без логина → редирект на `/login`.
2. Войдите как `maia` (Клиент) → 403.
3. Войдите как `damir` (Админист) → форма открывается.
4. Создайте товар.
5. Нажмите «Выйти» → редирект на `/products-jpa-page`.

---

## Задание

1. **Создайте страницу регистрации `/register`**:
   - DTO `RegistrationForm` с полями: `username`, `password`, `firstName`, `secondName`, `email`.
   - Валидация: логин ≥ 4 символов, пароль ≥ 6, email корректный.
   - При регистрации пароль хешируется через `PasswordEncoder`.
   - Новому пользователю присваивается роль `Клиент`.
   - Откройте `/register` всем (без аутентификации).

2. **Добавьте страницу профиля `/profile`**:
   - Показывает данные текущего пользователя.
   - Доступна только залогиненным.
   - Использует `SecurityUtils.getCurrentUsername()`.

3. **Создайте `util/DateUtils.java`**:
   - `format(LocalDate)` — в вид `дд.мм.гггг`.
   - `parse(String)` — из строки в `LocalDate`.
   - Используйте в профиле для отображения даты регистрации (если добавите поле).

4. **Скройте кнопки «Редактировать» и «Удалить»** в `products-jpa.html` для не-админов:
   - `th:if="${#authorization.expression('hasRole(''Админист'')')}"`.

5. **Создайте `CategoryAdminController`** с ролями:
   - Только `Админист` может создавать/удалять категории.
   - `Менеджер` может только просматривать.

6. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 5](Lesson5.md) |

---

## 💡 Что дальше (анонс урока 5)

На следующем занятии:
- Добавим **навигационное меню** через **Thymeleaf Fragments** (общий шаблон).
- Сделаем **главную страницу** с популярными товарами.
- Добавим **каталог** с фильтрацией по категориям.
- Реализуем **карточку товара** с фотографией.
- Введём **пагинацию** для больших списков.
