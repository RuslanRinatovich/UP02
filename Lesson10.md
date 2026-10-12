# Занятие 10. Отчёты, экспорт данных, графики и улучшение UX

Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[В начало](readme.md) | [Содержание](readme.md) | [Урок 11](Lesson11.md)

## План
1. [Введение: что было и куда идём](#введение)
2. [Что такое отчёты и какие они бывают](#что-такое-отчёты)
3. [Фильтры по датам в админ-заказах](#фильтры-по-датам)
4. [Запросы для отчётов](#запросы-для-отчётов)
5. [Экспорт в CSV](#экспорт-csv)
6. [Экспорт в Excel (Apache POI)](#экспорт-excel)
7. [Страница отчётов в админке](#страница-отчётов)
8. [Графики через Chart.js](#графики-chartjs)
9. [Пагинация в админских таблицах](#пагинация-в-админке)
10. [Email-уведомления о смене статуса](#email-уведомления)
11. [Проверка и чек-лист](#проверка)
12. [Задание](#задание)

---

## Введение

За девять занятий мы сделали:
- Полный функционал магазина: каталог, корзина, заказы, админка.
- Слоистую архитектуру, безопасность, REST API.
- Логи, тесты, файловое хранилище, кэширование.

**Проблемы, которые остались:**
1. **Админ не может выгрузить заказы** в Excel, чтобы отправить в бухгалтерию.
2. **Нет статистики** — сколько продали за месяц, какие товары популярны.
3. **Все заказы в админке видны одной простыней** — нет фильтра по датам и пагинации.
4. **Пользователь не знает, что его заказ перешёл в новый статус** — нет уведомлений.

**Сегодня:**
- Фильтры по датам.
- Экспорт в CSV и Excel.
- Отчёты и графики.
- Email-уведомления.

---

## Что такое отчёты

> 💡 **Отчёт** — это представление данных в удобном для анализа виде. В админ-панели обычно бывают:
>
> | Вид отчёта | Что показывает | Кому нужен |
> |-----------|---------------|-----------|
> | Список заказов за период | Все заказы с фильтром | Менеджер |
> | Топ-N товаров | Что покупают чаще всего | Маркетолог |
> | Выручка по дням | Динамика продаж | Директор |
> | Средний чек | Средняя сумма заказа | Аналитик |
> | Отчёт по категориям | Какие категории продаются лучше | Менеджер |

**Форматы вывода:**
- **HTML-таблица** — для просмотра на странице.
- **CSV** — простой текстовый формат, открывается в Excel.
- **XLSX (Excel)** — форматированный, с несколькими листами.
- **График (Chart.js)** — визуализация тренда.

---

## Фильтры по датам

### 3.1. DTO для фильтров

`src/main/java/com/example/shop/dto/OrderFilter.java`:

```java
package com.example.shop.dto;

import lombok.Data;
import org.springframework.format.annotation.DateTimeFormat;

import java.time.LocalDate;

@Data
public class OrderFilter {

    private Integer statusId;

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
    private LocalDate from;

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
    private LocalDate to;
}
```

> 💡 **`@DateTimeFormat(iso = ISO.DATE)`** — Spring автоматически распарсит строку из формы в формате `2026-10-01` в `LocalDate`.

### 3.2. Метод в OrderRepository

```java
@Query("""
       SELECT o FROM Order o
       WHERE (:statusId IS NULL OR o.status.id = :statusId)
         AND (:from IS NULL OR o.createDate >= :from)
         AND (:to IS NULL OR o.createDate <= :to)
       ORDER BY o.createDate DESC
       """)
Page<Order> findWithFilters(@Param("statusId") Integer statusId,
                            @Param("from") LocalDate from,
                            @Param("to") LocalDate to,
                            Pageable pageable);
```

> 💡 **`(:param IS NULL OR ...)`** — типичный приём для «опциональных» фильтров в JPQL. Если параметр `null` — условие пропускается.
>
> ⚠️ **`@Param`** обязателен, если имена параметров метода не совпадают с именами в запросе (или если компилятор без `-parameters`).

### 3.3. Обновляем OrderAdminController

```java
@GetMapping
public String list(@ModelAttribute OrderFilter filter,
                   @RequestParam(defaultValue = "0") int page,
                   Model model) {
    Page<Order> orders = orderRepository.findWithFilters(
            filter.getStatusId(), filter.getFrom(), filter.getTo(),
            PageRequest.of(page, 20, Sort.by("createDate").descending())
    );
    model.addAttribute("orders", orders);
    model.addAttribute("filter", filter);
    model.addAttribute("statuses", statusRepository.findAll());
    model.addAttribute("currentPage", page);
    return "admin/orders";
}
```

### 3.4. Форма фильтра в `admin/orders.html`

```html
<form method="get" th:action="@{/admin/orders}"
      style="background: white; padding: 15px; border-radius: 8px; margin-bottom: 20px; display: flex; gap: 10px; align-items: end;">
    <div>
        <label>Статус</label>
        <select name="statusId" th:field="*{filter.statusId}" style="display: block;">
            <option value="">Все</option>
            <option th:each="s : ${statuses}"
                    th:value="${s.id}"
                    th:text="${s.title}"></option>
        </select>
    </div>
    <div>
        <label>С</label>
        <input type="date" name="from" th:value="${filter.from}" style="display: block;"/>
    </div>
    <div>
        <label>По</label>
        <input type="date" name="to" th:value="${filter.to}" style="display: block;"/>
    </div>
    <button type="submit">Применить</button>
    <a th:href="@{/admin/orders}">Сбросить</a>
</form>
```

> ⚠️ При работе с `@ModelAttribute OrderFilter filter` используйте `th:value` и `name="..."`, а не `th:field`, если форма не обёрнута в `th:object`. Иначе будет ошибка.

---

## Запросы для отчётов

Создайте `repository/ReportRepository.java` — отдельный репозиторий для агрегатных запросов:

```java
package com.example.shop.repository;

import com.example.shop.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDate;
import java.util.List;

public interface ReportRepository extends JpaRepository<Order, Integer> {

    /**
     * Выручка по дням за период.
     * Возвращает: дата, сумма
     */
    @Query("""
           SELECT o.createDate, SUM(op.count * p.cost)
           FROM Order o
           JOIN o.items op
           JOIN op.product p
           WHERE o.createDate BETWEEN :from AND :to
           GROUP BY o.createDate
           ORDER BY o.createDate
           """)
    List<Object[]> revenueByDay(@Param("from") LocalDate from, @Param("to") LocalDate to);

    /**
     * Топ-N товаров по количеству продаж.
     */
    @Query("""
           SELECT p.title, SUM(op.count)
           FROM OrderProduct op
           JOIN op.product p
           GROUP BY p.title
           ORDER BY SUM(op.count) DESC
           """)
    List<Object[]> topProducts(Pageable pageable);

    /**
     * Средний чек за период.
     */
    @Query("""
           SELECT AVG(o.total)
           FROM (SELECT SUM(op.count * p.cost) AS total
                 FROM OrderProduct op
                 JOIN op.product p
                 WHERE op.order.createDate BETWEEN :from AND :to
                 GROUP BY op.order.id) o
           """)
    Double averageCheck(@Param("from") LocalDate from, @Param("to") LocalDate to);
}
```

> 💡 **`Object[]`** — Hibernate возвращает «сырые» результаты для агрегатов. Каждый элемент массива — колонка из SELECT. Это некрасиво, но быстро. Для сложных отчётов используют **проекции** (интерфейсы DTO), мы рассмотрим их в задании.

> ⚠️ **JPQL подзапрос в FROM** — не поддерживается в стандартном JPQL! Для `averageCheck` придётся использовать `nativeQuery = true`. Ниже — исправленный вариант.

**Исправленный метод `averageCheck`:**

```java
@Query(value = """
       SELECT AVG(sub.total)
       FROM (
           SELECT SUM(op.count * p.cost) AS total
           FROM order_product op
           JOIN product p ON p.id = op.product_id
           JOIN "order" o ON o.id = op.order_id
           WHERE o.create_date BETWEEN :from AND :to
           GROUP BY o.id
       ) sub
       """, nativeQuery = true)
Double averageCheck(@Param("from") LocalDate from, @Param("to") LocalDate to);
```

> 💡 **Нативный SQL** — пишется как в DBeaver. Работает с конкретной СУБД. Не переносится на MySQL без правок. Используйте только когда JPQL не справляется.

---

## Экспорт в CSV

### 5.1. CSV — это просто

**CSV** = comma-separated values. Текстовый формат: каждая строка — запись, поля разделены `;` или `,`.

**Правила:**
- Кодировка — **UTF-8 с BOM** (иначе Excel не поймёт русский).
- Разделитель в русской локали — `;`.
- Если в поле есть `;`, `,`, `"`, перенос строки — оборачиваем в кавычки.

### 5.2. Утилита `util/CsvUtils.java`

```java
package com.example.shop.util;

public final class CsvUtils {

    private CsvUtils() {
    }

    /**
     * Экранирует значение для CSV.
     */
    public static String escape(Object value) {
        if (value == null) {
            return "";
        }
        String s = value.toString();
        if (s.contains(";") || s.contains(",") || s.contains("\"") || s.contains("\n")) {
            return "\"" + s.replace("\"", "\"\"") + "\"";
        }
        return s;
    }

    /**
     * Собирает строку CSV из массива значений.
     */
    public static String line(Object... values) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < values.length; i++) {
            if (i > 0) sb.append(";");
            sb.append(escape(values[i]));
        }
        return sb.append("\n").toString();
    }

    /**
     * BOM для Excel (чтобы русский отображался корректно).
     */
    public static byte[] withBom(String content) {
        byte[] bom = new byte[]{(byte) 0xEF, (byte) 0xBB, (byte) 0xBF};
        byte[] body = content.getBytes(java.nio.charset.StandardCharsets.UTF_8);
        byte[] result = new byte[bom.length + body.length];
        System.arraycopy(bom, 0, result, 0, bom.length);
        System.arraycopy(body, 0, result, bom.length, body.length);
        return result;
    }
}
```

### 5.3. Метод экспорта в OrderAdminController

```java
@GetMapping("/export/csv")
public ResponseEntity<byte[]> exportCsv(@ModelAttribute OrderFilter filter) {
    List<Order> orders = orderRepository.findWithFilters(
            filter.getStatusId(), filter.getFrom(), filter.getTo(),
            Pageable.unpaged()
    ).getContent();

    StringBuilder sb = new StringBuilder();
    sb.append(CsvUtils.line("№", "Дата", "Пользователь", "Статус",
                            "Пункт выдачи", "Код выдачи", "Сумма"));

    for (Order o : orders) {
        BigDecimal sum = o.getItems().stream()
                .map(i -> i.getProduct().getCost()
                        .multiply(BigDecimal.valueOf(i.getCount())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);

        sb.append(CsvUtils.line(
                o.getId(),
                o.getCreateDate(),
                o.getUser() != null ? o.getUser().getUsername() : "—",
                o.getStatus().getTitle(),
                o.getPickupPoint().getAddress(),
                o.getGetCode(),
                sum
        ));
    }

    byte[] bytes = CsvUtils.withBom(sb.toString());
    String filename = "orders_" + LocalDate.now() + ".csv";

    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"" + filename + "\"")
            .contentType(MediaType.parseMediaType("text/csv; charset=UTF-8"))
            .body(bytes);
}
```

> 💡 **`Content-Disposition: attachment; filename="..."`** — заставляет браузер скачать файл, а не открыть его.

**Добавьте кнопку в `admin/orders.html`:**

```html
<a th:href="@{/admin/orders/export/csv(statusId=${filter.statusId}, from=${filter.from}, to=${filter.to})}"
   style="background: #28a745; color: white; padding: 8px 16px; border-radius: 4px; text-decoration: none;">
    📥 Экспорт CSV
</a>
```

---

## Экспорт в Excel (Apache POI)

> 💡 **Apache POI** — библиотека для создания файлов Excel (`.xlsx`) из Java.

### 6.1. Зависимость в `pom.xml`

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

### 6.2. Сервис `service/ExcelExportService.java`

```java
package com.example.shop.service;

import com.example.shop.entity.Order;
import com.example.shop.entity.OrderProduct;
import lombok.extern.slf4j.Slf4j;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.stereotype.Service;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.math.BigDecimal;
import java.time.format.DateTimeFormatter;
import java.util.List;

@Slf4j
@Service
public class ExcelExportService {

    private static final DateTimeFormatter DATE_FMT = DateTimeFormatter.ofPattern("dd.MM.yyyy");

    public byte[] exportOrders(List<Order> orders) {
        try (Workbook workbook = new XSSFWorkbook();
             ByteArrayOutputStream out = new ByteArrayOutputStream()) {

            Sheet sheet = workbook.createSheet("Заказы");

            // Стиль заголовка
            CellStyle headerStyle = createHeaderStyle(workbook);

            // Заголовок
            Row header = sheet.createRow(0);
            String[] columns = {"№", "Дата", "Пользователь", "Статус",
                                "Пункт выдачи", "Код", "Сумма, ₽"};
            for (int i = 0; i < columns.length; i++) {
                Cell cell = header.createCell(i);
                cell.setCellValue(columns[i]);
                cell.setCellStyle(headerStyle);
            }

            // Данные
            int rowNum = 1;
            for (Order order : orders) {
                Row row = sheet.createRow(rowNum++);

                row.createCell(0).setCellValue(order.getId());
                row.createCell(1).setCellValue(
                        order.getCreateDate().format(DATE_FMT));
                row.createCell(2).setCellValue(
                        order.getUser() != null ? order.getUser().getUsername() : "—");
                row.createCell(3).setCellValue(order.getStatus().getTitle());
                row.createCell(4).setCellValue(order.getPickupPoint().getAddress());
                row.createCell(5).setCellValue(order.getGetCode());
                row.createCell(6).setCellValue(calculateTotal(order).doubleValue());
            }

            // Автоширина колонок
            for (int i = 0; i < columns.length; i++) {
                sheet.autoSizeColumn(i);
            }

            workbook.write(out);
            log.info("Экспортировано {} заказов в Excel", orders.size());
            return out.toByteArray();

        } catch (IOException e) {
            throw new RuntimeException("Ошибка экспорта в Excel", e);
        }
    }

    private CellStyle createHeaderStyle(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        Font font = workbook.createFont();
        font.setBold(true);
        style.setFont(font);
        style.setFillForegroundColor(IndexedColors.GREY_25_PERCENT.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        style.setBorderBottom(BorderStyle.THIN);
        return style;
    }

    private BigDecimal calculateTotal(Order order) {
        return order.getItems().stream()
                .map(i -> i.getProduct().getCost()
                        .multiply(BigDecimal.valueOf(i.getCount())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

### 6.3. Endpoint для скачивания

```java
@GetMapping("/export/xlsx")
public ResponseEntity<byte[]> exportExcel(@ModelAttribute OrderFilter filter) {
    List<Order> orders = orderRepository.findWithFilters(
            filter.getStatusId(), filter.getFrom(), filter.getTo(),
            Pageable.unpaged()
    ).getContent();

    byte[] bytes = excelExportService.exportOrders(orders);
    String filename = "orders_" + LocalDate.now() + ".xlsx";

    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"" + filename + "\"")
            .contentType(MediaType.parseMediaType(
                    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"))
            .body(bytes);
}
```

**Добавьте кнопку:**

```html
<a th:href="@{/admin/orders/export/xlsx(statusId=${filter.statusId}, from=${filter.from}, to=${filter.to})}"
   style="background: #1d6f42; color: white; padding: 8px 16px; border-radius: 4px; text-decoration: none; margin-left: 10px;">
    📊 Экспорт Excel
</a>
```

> 💡 **`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`** — официальный MIME-тип `.xlsx`. Запомнить сложно, но нужно.

---

## Страница отчётов

### 7.1. ReportController

`src/main/java/com/example/shop/controller/admin/ReportController.java`:

```java
package com.example.shop.controller.admin;

import com.example.shop.repository.ReportRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.time.LocalDate;
import java.util.List;

@Controller
@RequestMapping("/admin/reports")
public class ReportController {

    @Autowired
    private ReportRepository reportRepository;

    @GetMapping
    public String reports(
            @RequestParam(required = false)
            @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate from,
            @RequestParam(required = false)
            @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate to,
            Model model) {

        if (from == null) from = LocalDate.now().minusDays(30);
        if (to == null) to = LocalDate.now();

        List<Object[]> revenueByDay = reportRepository.revenueByDay(from, to);
        List<Object[]> topProducts = reportRepository.topProducts(PageRequest.of(0, 10));
        Double avgCheck = reportRepository.averageCheck(from, to);

        model.addAttribute("from", from);
        model.addAttribute("to", to);
        model.addAttribute("revenueByDay", revenueByDay);
        model.addAttribute("topProducts", topProducts);
        model.addAttribute("avgCheck", avgCheck);
        return "admin/reports";
    }
}
```

### 7.2. Шаблон `admin/reports.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      th:replace="~{layout :: page(title='Отчёты', content=~{::content})}">
<body>
<div th:fragment="content">

    <h1>Отчёты</h1>

    <form method="get" style="background: white; padding: 15px; border-radius: 8px;
                              margin-bottom: 20px; display: flex; gap: 10px; align-items: end;">
        <div>
            <label>С</label>
            <input type="date" name="from" th:value="${from}" style="display: block;"/>
        </div>
        <div>
            <label>По</label>
            <input type="date" name="to" th:value="${to}" style="display: block;"/>
        </div>
        <button type="submit">Показать</button>
    </form>

    <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">

        <div style="background: white; padding: 20px; border-radius: 8px;">
            <h2>Выручка по дням</h2>
            <canvas id="revenueChart" height="200"></canvas>
        </div>

        <div style="background: white; padding: 20px; border-radius: 8px;">
            <h2>Средний чек</h2>
            <p style="font-size: 36px; color: #0066cc;"
               th:text="${avgCheck != null ? #numbers.formatDecimal(avgCheck, 1, 2) : '0.00'} + ' ₽'"></p>
        </div>

    </div>

    <div style="background: white; padding: 20px; border-radius: 8px; margin-top: 20px;">
        <h2>Топ-10 товаров по продажам</h2>
        <table>
            <thead>
            <tr>
                <th>#</th>
                <th>Товар</th>
                <th>Продано, шт.</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="row, iter : ${topProducts}">
                <td th:text="${iter.index + 1}"></td>
                <td th:text="${row[0]}"></td>
                <td th:text="${row[1]}"></td>
            </tr>
            </tbody>
        </table>
    </div>

</div>

<script th:src="'https://cdn.jsdelivr.net/npm/chart.js'"></script>
<script th:inline="javascript">
    const labels = /*[[${revenueByDay.![T(java.time.LocalDate).parse(#root[0][0].toString())]}]]*/ [];
    const values = /*[[${revenueByDay.![#root[0][1]]}]]*/ [];

    const ctx = document.getElementById('revenueChart');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Выручка, ₽',
                data: values,
                borderColor: '#0066cc',
                backgroundColor: 'rgba(0, 102, 204, 0.1)',
                fill: true,
                tension: 0.3
            }]
        },
        options: {
            responsive: true,
            plugins: { legend: { display: false } }
        }
    });
</script>

</body>
</html>
```

> ⚠️ Передача данных из Thymeleaf в JavaScript через `th:inline="javascript"` — **мощный, но капризный** механизм. Если объект сложный (например, `Object[]` из JPQL), Thymeleaf может не справиться. Проще:
>
> 1. В контроллере преобразовать `List<Object[]>` в `List<Map<String, Object>>`.
> 2. Передать в модель как JSON-строку через `ObjectMapper`.
> 3. В шаблоне использовать `[[${json}]]` без `th:inline`.
>
> **Готовый вариант:**

**В контроллере:**

```java
@Autowired
private ObjectMapper objectMapper;

// ...
List<Map<String, Object>> revenueData = revenueByDay.stream()
        .map(r -> Map.of("date", r[0].toString(), "revenue", r[1]))
        .toList();

model.addAttribute("revenueJson", objectMapper.writeValueAsString(revenueData));
```

**В шаблоне:**

```html
<script th:src="'https://cdn.jsdelivr.net/npm/chart.js'"></script>
<script>
    const revenueData = [[${revenueJson}]];
    const labels = revenueData.map(r => r.date);
    const values = revenueData.map(r => r.revenue);

    const ctx = document.getElementById('revenueChart');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Выручка, ₽',
                data: values,
                borderColor: '#0066cc',
                backgroundColor: 'rgba(0, 102, 204, 0.1)',
                fill: true,
                tension: 0.3
            }]
        },
        options: { responsive: true, plugins: { legend: { display: false } } }
    });
</script>
```

> 💡 **`[[${revenueJson}]]`** — Thymeleaf подставит JSON-строку как есть. В JavaScript это будет литерал объекта/массива.

---

## Пагинация в админских таблицах

Мы уже добавили `Pageable` в `findWithFilters`. Осталось вывести пагинацию в шаблоне.

### 9.1. Фрагмент `fragments/pagination.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<nav th:fragment="pager(page, baseUrl)" class="pagination"
     th:if="${page.totalPages > 1}">

    <a th:if="${page.hasPrevious()}"
       th:href="@{${baseUrl}(page=${page.number - 1})}">← Назад</a>

    <span th:each="i : ${#numbers.sequence(0, page.totalPages - 1)}"
          th:classappend="${i == page.number} ? 'current'">
        <a th:href="@{${baseUrl}(page=${i})}" th:text="${i + 1}"></a>
    </span>

    <a th:if="${page.hasNext()}"
       th:href="@{${baseUrl}(page=${page.number + 1})}">Вперёд →</a>
</nav>
</body>
</html>
```

### 9.2. Использование в `admin/orders.html`

```html
<div th:replace="~{fragments/pagination :: pager(${orders}, '/admin/orders')}"></div>
```

> ⚠️ При использовании `@{${baseUrl}(...)}` параметры фильтра (statusId, from, to) теряются. Чтобы их сохранить — передавайте их в `baseUrl` как query-string:
>
> ```html
> <div th:replace="~{fragments/pagination :: pager(${orders}, '/admin/orders?statusId=' + ${filter.statusId != null ? filter.statusId : ''} + '&from=' + ${filter.from != null ? filter.from : ''} + '&to=' + ${filter.to != null ? filter.to : ''})}"></div>
> ```
>
> Это некрасиво. Правильнее — в фрагменте использовать `th:href="@{/admin/orders(page=..., statusId=..., from=..., to=...)}"` с явной передачей всех параметров. Оставьте это как **задание**.

---

## Email-уведомления

> 💡 **JavaMailSender** — встроенный в Spring механизм отправки писем. Работает через SMTP-сервер (Gmail, Mail.ru, Yandex).

### 10.1. Зависимость

В `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

### 10.2. Настройка `application-dev.properties`

```properties
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

app.mail.from=your-email@gmail.com
```

> ⚠️ **Пароль Gmail** — не обычный, а **app password** (создаётся в настройках аккаунта). Обычный пароль не сработает из-за 2FA.
>
> 💡 Если нет желания настраивать реальный SMTP — используйте [MailHog](https://github.com/mailhog/MailHog) в Docker. Он перехватывает письма и показывает их в веб-интерфейсе `http://localhost:8025`.
>
> ```bash
> docker run -d -p 1025:1025 -p 8025:8025 mailhog/mailhog
> ```
>
> И в `application-dev.properties`:
> ```properties
> spring.mail.host=localhost
> spring.mail.port=1025
> ```

### 10.3. `service/EmailService.java`

```java
package com.example.shop.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class EmailService {

    @Autowired(required = false)
    private JavaMailSender mailSender;

    @Value("${app.mail.from:noreply@shop.local}")
    private String from;

    @Async
    public void sendOrderStatusChanged(String to, Integer orderId, String newStatus) {
        if (mailSender == null) {
            log.warn("JavaMailSender не настроен. Письмо для {} не отправлено.", to);
            return;
        }
        try {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setFrom(from);
            message.setTo(to);
            message.setSubject("Статус заказа №" + orderId + " изменён");
            message.setText("Здравствуйте!\n\n" +
                    "Статус вашего заказа №" + orderId + " изменён на: " + newStatus + ".\n\n" +
                    "С уважением,\nМагазин канцтоваров");

            mailSender.send(message);
            log.info("Письмо отправлено: {} (заказ {})", to, orderId);

        } catch (Exception e) {
            log.error("Ошибка отправки письма на {}", to, e);
        }
    }
}
```

> 💡 **`@Async`** — метод выполняется в отдельном потоке. Если SMTP медленный, пользователь не ждёт. Для работы нужно добавить `@EnableAsync` в `ShopApplication`.

### 10.4. Обновляем `OrderAdminController.changeStatus`

```java
@PostMapping("/{id}/status")
public String changeStatus(@PathVariable Integer id,
                           @RequestParam Integer statusId) {
    Order order = orderRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("Заказ не найден"));

    Status newStatus = statusRepository.findById(statusId)
            .orElseThrow(() -> new IllegalArgumentException("Статус не найден"));

    order.setStatus(newStatus);
    orderRepository.save(order);

    if (order.getUser() != null) {
        emailService.sendOrderStatusChanged(
                order.getUser().getUsername() + "@mail.ru",
                order.getId(),
                newStatus.getTitle()
        );
    }

    return "redirect:/admin/orders/" + id;
}
```

> ⚠️ **Проблема:** в вашей БД нет поля `email` у пользователя. Логин `maia` — а email нужен `maia@mail.ru`. Мы **додумываем** его (приписываем `@mail.ru`). В реальности было бы `user.getEmail()`. Оставьте это как замечание для студентов.

### 10.5. Включаем асинхронность

В `ShopApplication`:

```java
@SpringBootApplication
@EnableCaching
@EnableAsync
public class ShopApplication { ... }
```

---

## Проверка

### 11.1. Чек-лист

| # | Проверка | ✅ |
|---|----------|---|
| 1 | Фильтр по датам работает в `/admin/orders` | |
| 2 | Фильтр по статусу сохраняется | |
| 3 | Экспорт в CSV скачивает файл | |
| 4 | CSV открывается в Excel с русскими буквами | |
| 5 | Экспорт в Excel работает | |
| 6 | Страница `/admin/reports` открывается | |
| 7 | График выручки строится | |
| 8 | Топ-10 товаров отображается | |
| 9 | Средний чек считается | |
| 10 | Смена статуса заказа отправляет email | |
| 11 | Пагинация в админ-заказах работает | |

### 11.2. Сценарии

1. Откройте `/admin/orders`, задайте даты, примените фильтр.
2. Нажмите «Экспорт CSV» — откройте в Excel.
3. Нажмите «Экспорт Excel» — откройте.
4. Откройте `/admin/reports` — увидите график и топ товаров.
5. Войдите как `maia`, оформите заказ.
6. Войдите как `damir`, смените статус заказа.
7. Проверьте письмо (в MailHog или на почте).

---

## Задание

1. **Добавьте экспорт товаров в CSV и Excel**:
   - `/admin/products/export/csv`
   - `/admin/products/export/xlsx`
   - Колонки: артикул, название, цена, остаток, категория.

2. **Создайте отчёт «Продажи по категориям»**:
   - `SELECT c.title, SUM(op.count * p.cost) FROM ...`
   - Таблица на странице `/admin/reports`.

3. **Улучшите пагинацию**:
   - Сохраняйте параметры фильтра при переходе между страницами.
   - Используйте фрагмент `pagination.html` со всеми параметрами.

4. **Создайте `util/DateUtils.java`**:
   - `format(LocalDate)` — в формат `дд.мм.гггг`.
   - `parse(String)` — обратно.
   - `startOfMonth(LocalDate)`, `endOfMonth(LocalDate)` — для отчётов.
   - Используйте в ReportController для фильтров по умолчанию.

5. **Добавьте график «Продажи по категориям»** (doughnut/pie):
   - Chart.js.
   - Данные из нового отчёта.

6. **Реализуйте уведомления через БД**:
   - Таблица `notification` (id, username, message, created_at, is_read).
   - При смене статуса заказа — запись в таблицу.
   - На странице `/orders` отображается значок «У вас N новых уведомлений».

7. **Загрузите результат на Gogs** в репозиторий `Lesson1`.

---

| Предыдущее занятие | &nbsp; | Следующее занятие |
|:----------------:|:----------:|:----------------:|
| [В начало](readme.md) | [Содержание](readme.md) | [Урок 11](Lesson11.md) |

---

## 💡 Что дальше (анонс урока 11)

На следующем занятии:
- **Swagger / OpenAPI** — автодокументация REST API.
- **Postman-коллекция** — сохранение всех запросов.
- **Dockerfile** и **docker-compose.yml** — запуск приложения + PostgreSQL в контейнерах.
- **CI**: сборка и тесты при push в Gogs.
- **Финальный README** — инструкция запуска.

---

## 📚 Полезные ссылки

- 🇬🇧 [Baeldung: Spring Boot File Download](https://www.baeldung.com/spring-controller-return-image-file)
- 🇬🇧 [Apache POI Quick Guide](https://poi.apache.org/components/spreadsheet/quick-guide.html)
- 🇬🇧 [Chart.js Documentation](https://www.chartjs.org/docs/latest/)
- 🇬🇧 [Baeldung: Spring Mail](https://www.baeldung.com/spring-email)
- 🇬🇧 [Baeldung: @Async in Spring](https://www.baeldung.com/spring-async)
- 🇬🇧 [MailHog для тестирования email](https://github.com/mailhog/MailHog)
- 🇷🇺 [Хабр: Экспорт данных в Excel на Java](https://habr.com/ru/articles/) *(поиск: «apache poi export excel java»)*
- 🇷🇺 [Хабр: Chart.js с Thymeleaf](https://habr.com/ru/articles/) *(поиск: «chartjs thymeleaf spring boot»)*
