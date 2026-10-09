<table style="width: 100%;">
  <tr>
    <td style="text-align: center; border: none;"> 
        Министерство образования и науки РФ <br/>
        ГАПОУ "Зеленодольский механический колледж"
    </td>
  </tr>
  <tr>
    <td style="text-align: center; border: none; height: 45em;">
        <h2>
            Курс занятий по УП 02<br/>
            Разработка веб-приложений на Java
        </h2>
        <p>
            Spring Boot · Spring Data JPA · Spring Security · Thymeleaf · PostgreSQL
        </p>
    </td>
  </tr>
  <tr>
    <td style="text-align: right; border: none; height: 20em;">
        <div style="float: right;" align="left">
            <b>Разработал</b>: <br/>
            Сафиулин Руслан Ринатович
        </div>
    </td>
  </tr>
  <tr>
    <td style="text-align: center; border: none; height: 1em;">
        г. Зеленодольск, 2026
    </td>
  </tr>
</table>

<div style="page-break-after: always;"></div>

# Содержание

## 📚 Материалы курса

[Руководство по стилю SQL · SQL Style Guide](SQLStyleGuide.MD)<br/>
[Задание на разработку.pdf](taskfordoiing.pdf)<br/>
[Скрипт БД для интернет-магазина канцтоваров](ScriptWithData.sql)<br/>
[Шаблон словаря данных](docs/DataDictionary_Template.xlsx)

## 🔑 Учётные данные

[Учётные данные 235 Группа](215.md)<br/>
[Учётные данные 237 Группа](217.md)

## 🗓️ Занятия

| № | Тема | Ключевые навыки |
|:-:|------|-----------------|
| 1 | [Занятие 1. Подключение к БД, запуск скрипта, первый Spring Boot](Lesson1.md) | DBeaver, SQL-скрипт, Spring Initializr, `JdbcTemplate`, первый контроллер, Thymeleaf-шаблон, словарь данных |
| 2 | [Занятие 2. Spring Data JPA: Entity и Repository](Lesson2.md) | `@Entity`, `@ManyToOne`, `JpaRepository`, связи между таблицами, REST + HTML |
| 3 | [Занятие 3. Формы, валидация и архитектура проекта](Lesson3.md) | DTO, Bean Validation, пакеты `entity/repository/dto/service/controller/util`, CRUD, загрузка фото, редактирование и удаление |
| 4 | [Занятие 4. Spring Security: аутентификация и роли](Lesson4.md) | `UserDetailsService`, BCrypt, роли, `/admin/**`, страница входа, `SecurityUtils` |
| 5 | [Занятие 5. Thymeleaf Fragments, каталог и карточка товара](Lesson5.md) | `layout.html`, фрагменты, фильтры, пагинация, отдача фото, главная страница |
| 6 | [Занятие 6. Корзина, оформление заказа и «Мои заказы»](Lesson6.md) | `HttpSession`, `Cart`, `Order`, `OrderProduct`, генерация кодов выдачи |
| 7 | [Занятие 7. Админ-панель и REST API](Lesson7.md) | Дашборд, управление заказами, смена статусов, `/api/**`, `@RestControllerAdvice` |
| 8 | [Занятие 8. Логирование, профили и тесты](Lesson8.md) | SLF4J, `dev`/`prod`, `@ControllerAdvice`, страницы 404/403/500, JUnit 5 + Mockito |
| 9 | [Занятие 9. Интеграционные тесты, файлы и оптимизация](Lesson9.md) | `@DataJpaTest`, `@SpringBootTest`, TestContainers, `JOIN FETCH`, файловое хранилище, кэширование |
| 10 | [Занятие 10. Отчёты, экспорт и улучшение UX](Lesson10.md) | CSV/Excel (Apache POI), графики (Chart.js), email-уведомления, фильтры по датам |
| 11 | [Занятие 11. Swagger, Docker и деплой](Lesson11.md) | OpenAPI/Swagger, Postman, Dockerfile, docker-compose, CI |
| 12 | [Занятие 12. Финализация и защита проекта](Lesson12.md) | Чек-лист, презентация, скринкаст, защита, подготовка к демо-экзамену |
| — | [**Зачётное задание**](VAR2/FINALTASK2.MD) | Итоговая работа |

## 🎯 Итоговый проект

К концу практики каждый студент разрабатывает **интернет-магазин канцтоваров** с полным функционалом:

- 📦 Каталог товаров с фильтрами, сортировкой и пагинацией
- 🖼️ Карточка товара с фотографией
- 🛒 Корзина на основе сессии
- 📋 Оформление заказа с выбором пункта выдачи
- 👤 Личный кабинет с историей заказов
- 🔐 Аутентификация и роли (`Клиент`, `Менеджер`, `Админист`)
- 🛠️ Админ-панель с дашбордом, CRUD и управлением заказами
- 🌐 REST API с документацией Swagger
- 🧪 Покрытие тестами (>60%)
- 🐳 Docker-деплой
- 📄 README с описанием и инструкцией запуска

## 🧰 Стек технологий

| Технология | Назначение |
|------------|-----------|
| Java 21 | Язык программирования |
| Spring Boot 3.x | Фреймворк |
| Spring Data JPA | Доступ к БД |
| Spring Security | Аутентификация и авторизация |
| Thymeleaf | Серверный шаблонизатор |
| PostgreSQL 16 | База данных |
| Maven | Сборка проекта |
| DBeaver | Клиент БД |
| IntelliJ IDEA | IDE |
| Gogs | Git-сервер |
| JUnit 5 + Mockito | Тестирование |
| Docker | Контейнеризация |

## 📖 Вспомогательные материалы

[Что такое Git и с чем его едят?](https://skillbox.ru/media/code/chto_takoe_git_obyasnyaem_na_skhemakh/)<br/>
[Как отправить данные на gogs-server](https://drive.google.com/file/d/1dTC3Px5rwt--s7hT8Ei2QXamTh_lJAjU/view?usp=drive_link)<br/>
[Руководство по стилю кода на Java](https://google.github.io/styleguide/javaguide.html)<br/>
[Официальная документация Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/)<br/>
[Baeldung — лучшие туториалы по Spring](https://www.baeldung.com/)

## 🎓 Преподавателю

### Требования к студенту к концу практики

- Уметь разворачивать БД из SQL-скрипта и подключаться к ней.
- Проектировать слоистую архитектуру (`entity`, `repository`, `service`, `controller`, `dto`, `util`).
- Работать с формами, валидацией, безопасностью.
- Писать unit- и интеграционные тесты.
- Оформлять проект в Git и пушить в Gogs.
- Писать чистый код с логированием и обработкой ошибок.

### Критерии оценивания

| Критерий | Вес |
|----------|-----|
| Работающий функционал (каталог, корзина, заказы) | 30% |
| Архитектура и чистота кода | 20% |
| Безопасность (роли, пароли) | 15% |
| Тесты и покрытие | 15% |
| Оформление (README, Docker, Swagger) | 10% |
| Защита проекта (объяснение кода) | 10% |

## 📬 Обратная связь

[Мой аккаунт на Boosty](https://boosty.to/itmagic)

---

<div align="center">
    <i>По вопросам обращаться к Сафиулину Руслану Ринатовичу</i>
</div>
