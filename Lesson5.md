Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[Урок 4](Lesson4.md) | [Содержание](readme.md) | [Урок 6](Lesson6.md)

# Урок 5. Реализация корзины
1. [Добавление нового артефакта](#добавление-нового-артефакта)
   * [pom.xml](#pomxml)
2. [Добавление и изменение сущностей](#добавление-и-изменение-сущностей)
   * [Category](#класс-category)
   * [Manufacturer](#класс-manufacturer)
   * [Order](#класс-order)
   * [OrderProduct](#класс-orderproduct)
   * [OrderProductId](#класс-orderproductid)
   * [PickupPoint](#класс-pickuppoint)
   * [Product](#класс-product)
   * [Status](#класс-status)
   * [Role](#класс-role)
   * [Supllier](#класс-supllier)
   * [Unittype](#класс-unittype-)
   * [User](#класс-user-)
   * [hibernate.cfg.xml](#hibernatecfgxml)
3. [Создание макета каталога товаров](#создание-и-изменение-существующих-макетов)
   * [main-view.fxml](#main-viewfxml)
   * [login-view.fxml](#login-viewfxml)
   * [category-table-view.fxml](#category-table-viewfxml)
   * [order-view.fxml](#order-viewfxml)
   * [ordercell-view.fxml](#ordercell-viewfxml)
   * [product-edit-view.fxml](#product-edit-viewfxml)
   * [productcell-view.fxml](#productcell-viewfxml)
   * [products-table-view.fxml](#products-table-viewfxml)
4. [Создание и изменение существующих контроллеров](#создание-и-изменение-существующих-контроллеров)
   * [LoginController](#класс-logincontroller)
   * [MainWindowController](#класс-mainwindowcontroller)
   * [ListCellController](#класс-listcellcontroller)
   * [CategoryTableViewController](#класс-categorytableviewcontroller)
   * [OrderCell](#класс-ordercell)
   * [OrderCellController](#класс-ordercellcontroller)
   * [OrderViewController](#класс-orderviewcontroller)
   * [ProductCell](#класс-productcell)
   * [ProductEditViewController](#класс-producteditviewcontroller)
   * [ProductTableViewController](#класс-producttableviewcontroller)
5. [Пакет util](#пакет-util)
   * [Manager](#managerjava)
   * [Basket](#класс-basket)
   * [Item](#класс-item)
6. [Добавление репозиториев](#добавление-репозиториев)
   * [BaseDao](#класс-basedao)
   * [CategoryDao](#класс-categorydao)
   * [ManufacturerDao](#класс-manufacturerdao)
   * [OrderDao](#класс-orderdao)
   * [OrderProductDao](#класс-orderproductdao)
   * [PickupPointDao](#класс-pickuppointdao)
   * [ProductDao](#класс-productdao)
   * [StatusDao](#класс-statusdao)
   * [SupplierDao](#класс-supplierdao)
   * [UnittypeDao](#класс-unittypedao)
   * [UserDao](#класс-userdao)
   * [RoleDao](#класс-roledao)
7. [Добавление сервисов](#добавление-сервисов)
   * [CategoryService](#класс-categoryservice)
   * [ManufacturerService](#класс-manufacturerservice)
   * [OrderProductService](#класс-orderproductservice)
   * [OrderService](#класс-orderservice)
   * [PickupPointService](#класс-pickuppointservice)
   * [ProductService](#класс-productservice)
   * [RoleService](#класс-roleservice)
   * [StatusService](#класс-statusservice)
   * [SupplierService](#класс-supplierservice)
   * [UnittypeService](#класс-unittypeservice)
   * [UserService](#класс-userservice)
8. [Запуск приложения](#запуск-приложения)
9. [Задания](#задания)

## Добавление нового артефакта
1. Откройте файл pom.xml и замените его содержимое.
### pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>ru.trade</groupId>
    <artifactId>trade-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>trade-app</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.9.2</junit.version>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.gluonhq/charm-glisten -->
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>8.0.1.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>6.2.7.Final</version>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.4</version>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-controls</artifactId>
            <version>21-ea+24</version>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-swing</artifactId>
            <version>13.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-fxml</artifactId>
            <version>21-ea+24</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>22</source>
                    <target>22</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.openjfx</groupId>
                <artifactId>javafx-maven-plugin</artifactId>
                <version>0.0.8</version>
                <executions>
                    <execution>
                        <!-- Default configuration for running with: mvn clean javafx:run -->
                        <id>default-cli</id>
                        <configuration>
                            <mainClass>ru.trade.tradeapp/ru.trade.tradeapp.TradeApp</mainClass>
                            <launcher>app</launcher>
                            <jlinkZipName>app</jlinkZipName>
                            <jlinkImageName>app</jlinkImageName>
                            <noManPages>true</noManPages>
                            <stripDebug>true</stripDebug>
                            <noHeaderFiles>true</noHeaderFiles>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>3.3.1</version>
                <configuration>
                    <configLocation>checkstyle.xml</configLocation>
                    <includeTestSourceDirectory>true</includeTestSourceDirectory>
                    <failOnViolation>true</failOnViolation>
                    <logViolationsToConsole>true</logViolationsToConsole>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>check</goal>
                        </goals>
                        <phase>compile</phase>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## Добавление и изменение сущностей


1. В папке models создайте и замените следующие классы

![img.png](Lesson5Images/img.png)
### класс Category
```java
package ru.demo.tradeapp.model;


import jakarta.persistence.*;

import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

@Entity
@Table(name = "categories", schema = "public")
public class Category {

    @Id
    @Column(name = "category_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long categoryId;
    @Column(name = "title", nullable = false, length = 200)
    private String title;
    @OneToMany(mappedBy = "category")
    private Set<Product> products = new HashSet<Product>();

    public Category() {

    }

    public Category(Long categoryId, String title) {
        this.categoryId = categoryId;
        this.title = title;

    }

    public Set<Product> getProducts() {
        return products;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Category category)) return false;
        return Objects.equals(categoryId, category.categoryId) && Objects.equals(title, category.title);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * categoryId.hashCode() + 31 * title.hashCode();
        return hashCode;
    }

    public Long getCategoryId() {
        return categoryId;
    }

    public void setCategoryId(Long categoryId) {
        this.categoryId = categoryId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return title;
    }
}

```
### класс Manufacturer
```java
package ru.demo.tradeapp.model;


import jakarta.persistence.*;

import java.util.Objects;

@Entity
@Table(name = "manufacturers", schema = "public")
public class Manufacturer {


    @Id
    @Column(name = "manufacturer_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long manufacturerId;

    @Column(name = "title", nullable = false, length = 200)
    private String title;


    public Manufacturer() {

    }

    public Manufacturer(Long manufacturerId, String title) {
        this.manufacturerId = manufacturerId;
        this.title = title;

    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Manufacturer that)) return false;
        return Objects.equals(manufacturerId, that.manufacturerId) && Objects.equals(title, that.title);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * manufacturerId.hashCode() + 31 * title.hashCode();
        return hashCode;
    }

    public Long getManufacturerId() {
        return manufacturerId;
    }

    public void setManufacturerId(Long manufacturerId) {
        this.manufacturerId = manufacturerId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return title;
    }
}


```
### класс Order
```java
package ru.demo.tradeapp.model;

import jakarta.persistence.*;

import java.time.LocalDate;
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

@Entity
@Table(name = "orders", schema = "public")

public class Order {

    @Id
    @Column(name = "order_id")
    private Long orderId;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order order)) return false;
        return Objects.equals(orderId, order.orderId) && Objects.equals(status, order.status) && Objects.equals(pickupPoint, order.pickupPoint) && Objects.equals(createDate, order.createDate) && Objects.equals(deliveryDate, order.deliveryDate) && Objects.equals(user, order.user) && Objects.equals(getCode, order.getCode) && Objects.equals(orderProducts, order.orderProducts);
    }

    @Override
    public int hashCode() {


        return Objects.hash(orderId, status, pickupPoint, createDate, deliveryDate, user, getCode, orderProducts);
    }

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "status_id", nullable = false)
    private Status status;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "pickuppoint_id", nullable = false)
    private PickupPoint pickupPoint;

    @Column(name = "create_date", nullable = false)
    private LocalDate createDate;

    @Column(name = "delivery_date", nullable = false)
    private LocalDate deliveryDate;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "username", nullable = true)
    private User user;
    @Column(name = "get_code", nullable = false)
    private Integer getCode;

    @OneToMany(mappedBy = "order")
    private Set<OrderProduct> orderProducts = new HashSet<OrderProduct>();

    public Order() {
    }

    public Set<OrderProduct> getOrderProducts() {
        return orderProducts;
    }

    public void setOrderProducts(Set<OrderProduct> orderProducts) {
        this.orderProducts = orderProducts;
    }

    public Long getOrderId() {
        return orderId;
    }

    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
    }

    public PickupPoint getPickupPoint() {
        return pickupPoint;
    }

    public void setPickupPoint(PickupPoint pickupPoint) {
        this.pickupPoint = pickupPoint;
    }

    public LocalDate getCreateDate() {
        return createDate;
    }

    public void setCreateDate(LocalDate createDate) {
        this.createDate = createDate;
    }

    public LocalDate getDeliveryDate() {
        return deliveryDate;
    }

    public void setDeliveryDate(LocalDate deliveryDate) {
        this.deliveryDate = deliveryDate;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public Integer getGetCode() {
        return getCode;
    }

    public void setGetCode(Integer getCode) {
        this.getCode = getCode;
    }
}

```
### класс OrderProduct
```java
package ru.demo.tradeapp.model;

import jakarta.persistence.*;

import java.util.Objects;


@Entity
@IdClass(OrderProductId.class)
@Table(name = "order_products", schema = "public")
public class OrderProduct {

    @Id
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id", nullable = false)
    private Order order;

    @Id
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;

    @Column(name = "count", nullable = false)
    private Long count;

    public OrderProduct(Order order, Product product, Long count) {
        this.order = order;
        this.product = product;
        this.count = count;
    }

    public OrderProduct(Product product, Long count) {
        this.product = product;
        this.count = count;
    }

    public Order getOrder() {
        return order;
    }

    public void setOrder(Order order) {
        this.order = order;
    }

    public Product getProduct() {
        return product;
    }

    public void setProduct(Product product) {
        this.product = product;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderProduct that)) return false;
        return Objects.equals(order, that.order) && Objects.equals(product, that.product) && Objects.equals(count, that.count);
    }

    @Override
    public int hashCode() {
        return 17 * order.getOrderId().hashCode() + 31 * product.getProductId().hashCode() + 17 * count.hashCode();
    }

    public Long getCount() {
        return count;
    }

    public void setCount(Long count) {
        this.count = count;
    }
}

```
### класс OrderProductId
```java
package ru.demo.tradeapp.model;

import java.io.Serializable;
import java.util.Objects;

public class OrderProductId implements Serializable {
    private Order order;

    private Product product;

    public OrderProductId() {
    }

    public OrderProductId(Order order, Product product) {
        this.order = order;
        this.product = product;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderProductId that)) return false;
        return Objects.equals(order, that.order) && Objects.equals(product, that.product);
    }

    @Override
    public int hashCode() {
        return Objects.hash(order, product);
    }

    // equals() and hashCode()
}

```
### класс PickupPoint
```java
package ru.demo.tradeapp.model;

import jakarta.persistence.*;

import java.util.Objects;


@Entity
@Table(name = "pickup_points", schema = "public")
public class PickupPoint {

    @Id
    @Column(name = "pickup_point_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long pickupPointId;
    @Column(name = "address", nullable = false)
    private String address;

    public PickupPoint() {
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PickupPoint that)) return false;
        return Objects.equals(pickupPointId, that.pickupPointId) && Objects.equals(address, that.address);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * pickupPointId.hashCode() + 31 * address.hashCode();
        return hashCode;
    }

    @Override
    public String toString() {
        return address;
    }

    public Long getPickupPointId() {
        return pickupPointId;
    }

    public void setPickupPointId(Long pickupPointId) {
        this.pickupPointId = pickupPointId;
    }

    public String getTitle() {
        return address;
    }

    public void setTitle(String address) {
        this.address = address;
    }
}

```
### класс Product
```java
package ru.demo.tradeapp.model;


import jakarta.persistence.*;
import javafx.beans.property.SimpleStringProperty;
import javafx.beans.property.StringProperty;
import javafx.embed.swing.SwingFXUtils;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import ru.demo.tradeapp.TradeApp;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

@Entity
@Table(name = "products", schema = "public")

public class Product {

    @Id
    @Column(name = "product_id", nullable = false, length = 100)
    private String productId;
    @Column(name = "title", nullable = false, length = 100)
    private String title;
    @Column(name = "description")
    private String description;
    @Column(name = "cost", nullable = false)
    private Double cost;
    @Column(name = "max_discount_amount")
    private Integer maxDiscountAmount;
    @Column(name = "discount_amount")
    private Integer discountAmount;
    @Column(name = "quantity_in_stock", nullable = false)
    private Integer quantityInStock;
    @OneToMany(mappedBy = "product")

    private Set<OrderProduct> orderProducts = new HashSet<OrderProduct>();
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "unittype_id", nullable = false)
    private Unittype unittype;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "manufacturer_id", nullable = false)
    private Manufacturer manufacturer;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "supplier_id", nullable = false)
    private Supplier supplier;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private Category category;
    @Column(name = "photo")
    private byte[] photo;
    public Product() {
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Product product)) return false;
        return Objects.equals(productId, product.productId) && Objects.equals(title, product.title) && Objects.equals(description, product.description) && Objects.equals(cost, product.cost) && Objects.equals(maxDiscountAmount, product.maxDiscountAmount) && Objects.equals(discountAmount, product.discountAmount) && Objects.equals(quantityInStock, product.quantityInStock) && Objects.equals(unittype, product.unittype) && Objects.equals(manufacturer, product.manufacturer) && Objects.equals(supplier, product.supplier) && Objects.equals(category, product.category);
    }

    @Override
    public int hashCode() {
        return Objects.hash(productId, title, description, cost, maxDiscountAmount, discountAmount, quantityInStock, unittype, manufacturer, supplier, category);
    }

    public StringProperty getPropertyTitle() {
        return new SimpleStringProperty(this.title);
    }

    public Set<OrderProduct> getOrderProducts() {
        return orderProducts;
    }

    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Double getCost() {
        return cost;
    }

    public void setCost(Double cost) {
        this.cost = cost;
    }

    public Integer getMaxDiscountAmount() {
        return maxDiscountAmount;
    }

    public void setMaxDiscountAmount(Integer maxDiscountAmount) {
        this.maxDiscountAmount = maxDiscountAmount;
    }

    public Integer getDiscountAmount() {
        return discountAmount;
    }

    public void setDiscountAmount(Integer discountAmount) {
        this.discountAmount = discountAmount;
    }

    public Integer getQuantityInStock() {
        return quantityInStock;
    }

    public void setQuantityInStock(Integer quantityInStock) {
        this.quantityInStock = quantityInStock;
    }

    public Unittype getUnittype() {
        return unittype;
    }

    public void setUnittype(Unittype unittype) {
        this.unittype = unittype;
    }

    public Manufacturer getManufacturer() {
        return manufacturer;
    }

    public void setManufacturer(Manufacturer manufacturer) {
        this.manufacturer = manufacturer;
    }

    public Supplier getSupplier() {
        return supplier;
    }

    public void setSupplier(Supplier supplier) {
        this.supplier = supplier;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    public boolean isHasPhoto() {
        return photo != null;
    }

    public Image getPhoto() throws IOException {
        if (photo == null)
            return new Image(TradeApp.class.getResourceAsStream("picture.png"));
        BufferedImage capture = ImageIO.read(new ByteArrayInputStream(photo));
        return SwingFXUtils.toFXImage(capture, null);
    }

    public void setPhoto(Image img) throws IOException {
        BufferedImage buf = SwingFXUtils.fromFXImage(img, null);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ImageIO.write(buf, "jpg", baos);
        byte[] bytes = baos.toByteArray();
        this.photo = bytes;
    }

    public ImageView getImage() throws IOException {
        ImageView image = new ImageView();
        image.setImage(getPhoto());
        image.setFitHeight(60);
        image.setPreserveRatio(true);
        return image;
    }

    public Double getPriceWithDiscount() {
        return cost * (1 - discountAmount / 100.0);
    }
}

```
### класс Status
```java
package ru.demo.tradeapp.model;

import jakarta.persistence.*;

import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

@Entity
@Table(name = "statuses", schema = "public")
public class Status {

    @Id
    @Column(name = "status_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long statusId;
    @Column(name = "title", nullable = false, length = 200)
    private String title;
    @OneToMany
    @JoinColumn(name = "status_id")
    private Set<Order> orders = new HashSet<Order>();

    public Status() {

    }

    public Status(Long statusId, String title) {
        this.statusId = statusId;
        this.title = title;

    }

    @Override
    public String toString() {
        return title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Status status)) return false;
        return Objects.equals(statusId, status.statusId) && Objects.equals(title, status.title);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * statusId.hashCode() + 31 * title.hashCode();
        return hashCode;
    }

    public Long getStatusId() {
        return statusId;
    }

    public void setStatusId(Long statusId) {
        this.statusId = statusId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Set<Order> getOrders() {
        return orders;
    }

    public void setOrders(Set<Order> orders) {
        this.orders = orders;
    }
}

```
### класс Role
```java
package ru.demo.tradeapp.model;

import jakarta.persistence.*;

import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

@Entity
@Table(name = "roles", schema = "public")
public class Role {

    @Id
    @Column(name = "role_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long roleId;
    @Column(name = "title", nullable = false, length = 200)
    private String title;
    @OneToMany
    @JoinColumn(name = "role_id")
    private Set<User> users = new HashSet<User>();

    public Role() {
    }

    public Role(Long categoryId, String title) {
        this.roleId = categoryId;
        this.title = title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Role role)) return false;
        return Objects.equals(roleId, role.roleId) && Objects.equals(title, role.title);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * roleId.hashCode() + 31 * title.hashCode();
        return hashCode;
    }


    public Long getRoleId() {
        return roleId;
    }

    public void setRoleId(Long categoryId) {
        this.roleId = categoryId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Set<User> getUsers() {
        return users;
    }

    public void setUsers(Set<User> users) {
        this.users = users;
    }
}
```

### класс Supllier
```java
package ru.demo.tradeapp.model;



import jakarta.persistence.*;

import java.util.Objects;

@Entity
@Table(name = "suppliers", schema = "public")
public class Supplier {


    @Id
    @Column(name = "supplier_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long supplierId;

    @Column(name = "title", nullable = false, length = 200)
    private String title;

    public Supplier() {

    }
    public Supplier(Long supplierId, String title) {
        this.supplierId = supplierId;
        this.title = title;

    }

    public Long getSupplierId() {
        return supplierId;
    }

    public void setSupplierId(Long supplierId) {
        this.supplierId = supplierId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Supplier supplier)) return false;
        return Objects.equals(supplierId, supplier.supplierId) && Objects.equals(title, supplier.title);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * supplierId.hashCode() + 31 * title.hashCode();
        return hashCode;
    }
}


```


### класс Unittype 

```java
package ru.demo.tradeapp.model;

import jakarta.persistence.*;

import java.util.Objects;

@Entity
@Table(name = "unittypes", schema = "public")
public class Unittype {


    @Id
    @Column(name = "unittype_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long unittypeId;

    @Column(name = "title", nullable = false, length = 200)
    private String title;

    public Unittype() {

    }

    public Unittype(Long unittypeId, String title) {
        this.unittypeId = unittypeId;
        this.title = title;

    }

    public Long getUnittypeId() {
        return unittypeId;
    }

    public void setUnittypeId(Long categoryId) {
        this.unittypeId = unittypeId;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Unittype unittype)) return false;
        return Objects.equals(unittypeId, unittype.unittypeId) && Objects.equals(title, unittype.title);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * unittypeId.hashCode() + 31 * title.hashCode();
        return hashCode;
    }
}
```

### класс User 

```java

package ru.demo.tradeapp.model;

// Java Program to Illustrate Creation of Simple POJO Class

// Importing required classes

import jakarta.persistence.*;

import java.util.Objects;

@Entity
@Table(name = "users", schema = "public")
// POJO class
public class User {
    @Id
    @Column(name = "username")
    private String username;

    @Column(name = "first_name")
    private String firstName;
    @Column(name = "second_name")
    private String secondName;
    @Column(name = "middle_name")
    private String middleName;

    @Column(name = "password")
    private String password;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "role_id", nullable = false)
    private Role role;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getSecondName() {
        return secondName;
    }

    public void setSecondName(String secondName) {
        this.secondName = secondName;
    }

    public String getMiddleName() {
        return middleName;
    }

    public void setMiddleName(String middleName) {
        this.middleName = middleName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Role getRole() {
        return role;
    }

    public void setRole(Role role) {
        this.role = role;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User user)) return false;
        return Objects.equals(username, user.username) && Objects.equals(firstName, user.firstName) && Objects.equals(secondName, user.secondName) && Objects.equals(middleName, user.middleName) && Objects.equals(password, user.password);
    }

    @Override
    public int hashCode() {
        final int hashCode = 17 * username.hashCode() + 31 * firstName.hashCode() + 17 * secondName.hashCode() + 31 * middleName.hashCode() + 17 * password.hashCode();
        return hashCode;
    }


}

```

4. Измените содержимое файла hibernate.cfg.xml(Мы добавили mapping на новые добавленные классы)
### hibernate.cfg.xml
```xml
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Set URL -->
        <property name="hibernate.connection.url">jdbc:postgresql://192.168.2.202:5432/trade</property>
        <!-- Set User Name -->
        <property name="hibernate.connection.username">postgres</property>
        <!-- Set Password -->
        <property name="hibernate.connection.password">root</property>
        <!-- Set Driver Name -->
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.show_sql">true</property>
        <!-- Optional: Auto-generate schema -->
        <!-- <property name = "hibernate.hbm2ddl.auto">create</property> -->
        <mapping class="ru.demo.tradeapp.model.Category"/>
        <mapping class="ru.demo.tradeapp.model.Manufacturer"/>
        <mapping class="ru.demo.tradeapp.model.OrderProduct"/>
        <mapping class="ru.demo.tradeapp.model.Order"/>
        <mapping class="ru.demo.tradeapp.model.PickupPoint"/>
        <mapping class="ru.demo.tradeapp.model.Product"/>
        <mapping class="ru.demo.tradeapp.model.Role"/>
        <mapping class="ru.demo.tradeapp.model.Status"/>
        <mapping class="ru.demo.tradeapp.model.Supplier"/>
        <mapping class="ru.demo.tradeapp.model.Unittype"/>
        <mapping class="ru.demo.tradeapp.model.User"/>
    </session-factory>
</hibernate-configuration>


```

## Создание и изменение существующих макетов

1.  Создайте или замените код существующих файлов
![img_1.png](Lesson5Images/img_1.png)
### main-view.fxml
```fxml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="400.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/17.0.2-ea" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.MainWindowController">
    <children>
        <BorderPane prefHeight="200.0" prefWidth="200.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
            <center>
                <BorderPane fx:id="BorderPaneMainFrame" BorderPane.alignment="CENTER">
                    <top>
                        <FlowPane minHeight="-Infinity" nodeOrientation="LEFT_TO_RIGHT" rowValignment="TOP" BorderPane.alignment="CENTER">
                            <children>
                                <TextField fx:id="TextFieldSearch" onAction="#TextFieldSearchAction" prefHeight="25.0" prefWidth="262.0" promptText="Введите название для поиска" />
                                <ComboBox fx:id="ComboBoxCategory" onAction="#ComboBoxCategoryAction" promptText="тип продукта" />
                                <ComboBox fx:id="ComboBoxDiscount" onAction="#ComboBoxDiscountAction" promptText="скидка" />
                                <ComboBox fx:id="ComboBoxSort" onAction="#ComboBoxSortAction" promptText="сортировка" />
                            </children>
                        </FlowPane>
                    </top>
                    <center>
                        <ListView fx:id="ListViewProducts" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" prefHeight="201.0" prefWidth="600.0" BorderPane.alignment="CENTER" />
                    </center>
               <bottom>
                  <FlowPane BorderPane.alignment="CENTER">
                     <children>
                        <Button fx:id="BtnBasket" mnemonicParsing="false" onAction="#BtnBasketAction" text="Корзина" />
                        <Label fx:id="LabelBasketInfo" prefHeight="17.0" prefWidth="182.0" text="Label" />
                     </children>
                  </FlowPane>
               </bottom>
                </BorderPane>
            </center>
            <bottom>
                <Label fx:id="LabelInfo" text="Label" BorderPane.alignment="CENTER_LEFT" />
            </bottom>
         <top>
            <HBox BorderPane.alignment="CENTER">
               <children>
                        <MenuBar maxWidth="1.7976931348623157E308" prefHeight="25.0" prefWidth="381.0" HBox.hgrow="ALWAYS">
                            <menus>
                                <Menu mnemonicParsing="false" text="Файл">
                                    <items>
                                        <MenuItem fx:id="MenuItemProducts" mnemonicParsing="false" onAction="#MenuItemProductsAction" text="Товары" />
                                        <MenuItem fx:id="MenuItemOrders" mnemonicParsing="false" onAction="#MenuItemOrdersAction" text="Заказы" />
                                    </items>
                                </Menu>


                            </menus>
                        </MenuBar>
                        <Label fx:id="LabelUser" alignment="CENTER_RIGHT" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="user-label" text="Label" HBox.hgrow="ALWAYS">
                     <padding>
                        <Insets right="20.0" />
                     </padding></Label>
               </children>
            </HBox>
         </top>
        </BorderPane>
    </children>
</AnchorPane>


```
### login-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.image.*?>
<?import javafx.scene.layout.*?>

<AnchorPane maxHeight="200.0" maxWidth="350.0" minHeight="200.0" minWidth="350.0" prefHeight="200.0" prefWidth="350.0" xmlns="http://javafx.com/javafx/17.0.2-ea" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.LoginController">
    <children>
        <GridPane layoutX="25.0" layoutY="34.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
            <columnConstraints>
                <ColumnConstraints hgrow="SOMETIMES" maxWidth="100.0" minWidth="100.0" prefWidth="100.0" />
                <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
            </columnConstraints>
            <rowConstraints>
                <RowConstraints maxHeight="40.0" minHeight="40.0" prefHeight="40.0" vgrow="SOMETIMES" />
                <RowConstraints maxHeight="30.0" minHeight="30.0" prefHeight="30.0" vgrow="SOMETIMES" />
                <RowConstraints maxHeight="30.0" minHeight="30.0" prefHeight="30.0" vgrow="SOMETIMES" />
                <RowConstraints fx:id="ThirdRow" maxHeight="-Infinity" minHeight="0.0" prefHeight="50.0" vgrow="SOMETIMES" />
                <RowConstraints maxHeight="1.7976931348623157E308" minHeight="50.0" prefHeight="50.0" vgrow="SOMETIMES" />
            </rowConstraints>
            <children>
                <HBox alignment="CENTER" fillHeight="false" nodeOrientation="LEFT_TO_RIGHT" prefHeight="100.0" prefWidth="200.0" spacing="10.0" GridPane.columnSpan="2" GridPane.hgrow="ALWAYS" GridPane.rowIndex="4" GridPane.vgrow="SOMETIMES">
                    <children>
                        <Button fx:id="BtnOk" alignment="CENTER" defaultButton="true" maxHeight="-Infinity" maxWidth="-Infinity" mnemonicParsing="false" onAction="#BtnOkActon" prefHeight="34.0" prefWidth="100.0" text="OK" textAlignment="CENTER" HBox.hgrow="ALWAYS" />
                        <Button fx:id="BtnCancel" alignment="CENTER" cancelButton="true" maxHeight="-Infinity" maxWidth="-Infinity" mnemonicParsing="false" onAction="#BtnCancelAction" prefHeight="34.0" prefWidth="100.0" text="Cancel" textAlignment="CENTER" HBox.hgrow="ALWAYS" />
                    </children>
                    <opaqueInsets>
                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0" />
                    </opaqueInsets>
                    <padding>
                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0" />
                    </padding>
                    <GridPane.margin>
                        <Insets />
                    </GridPane.margin>
                </HBox>
                <Label alignment="CENTER" prefHeight="40.0" prefWidth="374.0" styleClass="header-label" text="ООО РУЧКИ" textAlignment="CENTER" GridPane.columnSpan="2" GridPane.vgrow="SOMETIMES" />
                <PasswordField fx:id="PasswordField" promptText="Пароль" GridPane.columnSpan="2" GridPane.rowIndex="2" GridPane.vgrow="ALWAYS">
                    <GridPane.margin>
                        <Insets left="30.0" right="30.0" />
                    </GridPane.margin></PasswordField>
                <TextField fx:id="TextFieldUsername" promptText="Логин" GridPane.columnSpan="2" GridPane.rowIndex="1" GridPane.vgrow="ALWAYS">
                    <GridPane.margin>
                        <Insets left="30.0" right="30.0" />
                    </GridPane.margin></TextField>
                <ImageView fitHeight="33.0" fitWidth="85.0" pickOnBounds="true" preserveRatio="true" GridPane.halignment="CENTER" GridPane.valignment="CENTER">
                    <image>
                        <Image url="@pen.png" />
                    </image>
                </ImageView>
                <ImageView fx:id="ImageViewCaptcha" fitHeight="40.0" fitWidth="150.0" pickOnBounds="true" preserveRatio="true" GridPane.columnSpan="2" GridPane.rowIndex="3">
                    <GridPane.margin>
                        <Insets bottom="5.0" left="60.0" top="5.0" />
                    </GridPane.margin></ImageView>
                <TextField fx:id="TextFieldCaptcha" maxHeight="-Infinity" maxWidth="-Infinity" prefHeight="26.0" prefWidth="125.0" promptText="Введите капчу" GridPane.columnIndex="1" GridPane.rowIndex="3">
                    <GridPane.margin>
                        <Insets left="130.0" right="5.0" />
                    </GridPane.margin>
                </TextField>
                <Button fx:id="BtnRenewCaptcha" alignment="CENTER" contentDisplay="CENTER" mnemonicParsing="false" onAction="#BtnRenewCaptchaAction" prefHeight="41.0" prefWidth="21.0" text="O" textAlignment="CENTER" GridPane.rowIndex="3">
                    <GridPane.margin>
                        <Insets bottom="5.0" left="5.0" top="5.0" />
                    </GridPane.margin></Button>
            </children>
        </GridPane>
    </children>
</AnchorPane>

```
### category-table-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.Menu?>
<?import javafx.scene.control.MenuBar?>
<?import javafx.scene.control.MenuItem?>
<?import javafx.scene.control.TableColumn?>
<?import javafx.scene.control.TableView?>
<?import javafx.scene.control.TextField?>
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.layout.BorderPane?>
<?import javafx.scene.layout.FlowPane?>
<?import javafx.scene.layout.HBox?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="400.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/22" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.CategoryTableViewController">
   <children>
      <BorderPane prefHeight="200.0" prefWidth="200.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
         <center>
            <BorderPane BorderPane.alignment="CENTER">
               <top>
                  <FlowPane nodeOrientation="LEFT_TO_RIGHT" rowValignment="TOP" BorderPane.alignment="CENTER">
                     <children>
                        <TextField fx:id="TextFieldSearch" onAction="#TextFieldSearchAction" prefHeight="25.0" prefWidth="262.0" promptText="Введите название для поиска" />
                     </children>
                  </FlowPane>
               </top>
               <center>
                  <TableView fx:id="TableViewCategories" prefHeight="200.0" prefWidth="200.0" tableMenuButtonVisible="true" BorderPane.alignment="CENTER">
                    <columns>
                        <TableColumn id="TableColumnCountInStock" fx:id="TableColumnId" prefWidth="82.0" text="Id" />
                      <TableColumn id="TableColumnTitle" fx:id="TableColumnTitle" editable="false" maxWidth="1.7976931348623157E308" prefWidth="300.0" text="Наименование" />
                    </columns>
                  </TableView>
               </center>
            </BorderPane>
         </center>
         <bottom>
            <Label fx:id="LabelInfo" text="Label" BorderPane.alignment="CENTER_LEFT" />
         </bottom>
         <top>
            <HBox BorderPane.alignment="CENTER">
               <children>
                  <MenuBar prefHeight="25.0" prefWidth="381.0" HBox.hgrow="ALWAYS">
                     <menus>
                        <Menu mnemonicParsing="false" text="Файл">
                           <items>
                              <MenuItem fx:id="MenuItemBack" mnemonicParsing="false" onAction="#MenuItemBackAction" text="Назад" />
                           </items>
                        </Menu>
                        <Menu mnemonicParsing="false" text="Правка">
                           <items>
                              <MenuItem fx:id="MenuItemAdd" mnemonicParsing="false" onAction="#MenuItemAddAction" text="Добавить" />
                              <MenuItem fx:id="MenuItemUpdate" mnemonicParsing="false" onAction="#MenuItemUpdateAction" text="Изменить" />
                              <MenuItem fx:id="MenuItemDelete" mnemonicParsing="false" onAction="#MenuItemDeleteAction" text="Удалить" />
                           </items>
                        </Menu>

                           </menus>
                  </MenuBar>
                  <Label fx:id="LabelUser" alignment="CENTER_RIGHT" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" prefWidth="120.0" styleClass="user-label" text="Label" HBox.hgrow="ALWAYS">
                     <padding>
                        <Insets right="20.0" />
                     </padding>
                  </Label>
               </children>
            </HBox>
         </top>
      </BorderPane>
   </children>
</AnchorPane>

```
### order-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Button?>
<?import javafx.scene.control.ComboBox?>
<?import javafx.scene.control.DatePicker?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.ListView?>
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.layout.BorderPane?>
<?import javafx.scene.layout.FlowPane?>
<?import javafx.scene.layout.HBox?>
<?import javafx.scene.layout.VBox?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="600.0" prefWidth="800.0" xmlns="http://javafx.com/javafx/22" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.OrderViewController">
    <children>
        <BorderPane prefHeight="200.0" prefWidth="200.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
            <center>
                <BorderPane fx:id="BorderPaneMainFrame" BorderPane.alignment="CENTER">
                    <top>
                  <VBox BorderPane.alignment="CENTER">
                     <children>
                        <ComboBox fx:id="ComboStatus" prefWidth="150.0" promptText="статус заказа" />
                        <DatePicker fx:id="DatePickerOrderCreateDate" prefHeight="25.0" prefWidth="249.0" promptText="дата заказа" />
                                <ComboBox fx:id="ComboBoxPickupPoint" onAction="#ComboBoxPickupPointAction" prefHeight="25.0" prefWidth="249.0" promptText="пункт выдачи" />
                        <DatePicker fx:id="DatePickerOrderDeliveryDate" prefHeight="25.0" prefWidth="249.0" promptText="дата доставки" />
                        <Label fx:id="LabelOrderGetCode" text="Label" />
                     </children>
                  </VBox>
                    </top>
                    <center>
                        <ListView fx:id="ListViewProducts" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" prefHeight="201.0" prefWidth="600.0" BorderPane.alignment="CENTER" />
                    </center>
               <bottom>
                  <FlowPane BorderPane.alignment="CENTER">
                     <children>
                        <Label fx:id="LabelBasketInfo" prefHeight="17.0" prefWidth="579.0" text="Label" />
                        <Button fx:id="BtnDelete" mnemonicParsing="false" onAction="#BtnDeleteAction" text="Удалить" />
                        <Button fx:id="BtnOk" mnemonicParsing="false" onAction="#BtnOkAction" text="Оформить" />
                        <Button mnemonicParsing="false" text="Распечатать" />
                     </children>
                  </FlowPane>
               </bottom>
                </BorderPane>
            </center>
         <top>
            <HBox BorderPane.alignment="CENTER">
               <children>
                  <Label fx:id="LabelOrderNumber" prefHeight="17.0" prefWidth="366.0" text="Label" />
                        <Label fx:id="LabelUser" alignment="CENTER_RIGHT" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="user-label" text="Label" HBox.hgrow="ALWAYS">
                     <padding>
                        <Insets right="20.0" />
                     </padding></Label>
               </children>
            </HBox>
         </top>
        </BorderPane>
    </children>
</AnchorPane>

```
### ordercell-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.ContextMenu?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.MenuItem?>
<?import javafx.scene.image.ImageView?>
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.layout.ColumnConstraints?>
<?import javafx.scene.layout.GridPane?>
<?import javafx.scene.layout.HBox?>
<?import javafx.scene.layout.RowConstraints?>
<?import javafx.scene.text.Font?>

<AnchorPane fx:id="CellAnchorPane" maxHeight="-Infinity" maxWidth="1.7976931348623157E308" minHeight="-Infinity" minWidth="-Infinity" prefHeight="142.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/22" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.OrderCellController">
   <children>
      <GridPane layoutY="14.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
        <columnConstraints>
          <ColumnConstraints minWidth="100.0" prefWidth="100.0" />
            <ColumnConstraints hgrow="ALWAYS" maxWidth="1.7976931348623157E308" minWidth="10.0" prefWidth="100.0" />
          <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
        </columnConstraints>
        <rowConstraints>
          <RowConstraints maxHeight="-Infinity" minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
          <RowConstraints maxHeight="-Infinity" minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
            <RowConstraints maxHeight="-Infinity" minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
          <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
            <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
            <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
        </rowConstraints>
         <children>
            <ImageView fx:id="ImageViewPhoto" fitHeight="140.0" fitWidth="90.0" pickOnBounds="true" preserveRatio="true" GridPane.rowSpan="6">
               <GridPane.margin>
                  <Insets bottom="5.0" left="5.0" right="5.0" top="5.0" />
               </GridPane.margin></ImageView>
            <Label fx:id="LabelPercent" alignment="CENTER" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="percent-label" text="Label" textAlignment="CENTER" GridPane.columnIndex="2" GridPane.rowSpan="4">
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                    <items>
                      <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                    </items>
                  </ContextMenu>
               </contextMenu></Label>
            <Label fx:id="LabelTitle" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="title-label" text="Label" GridPane.columnIndex="1" GridPane.hgrow="ALWAYS">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
            <Label fx:id="LabelDescription" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="product-cell" text="Label" GridPane.columnIndex="1" GridPane.rowIndex="1">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
            <Label fx:id="LabelManufacturer" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="product-cell" text="Label" GridPane.columnIndex="1" GridPane.rowIndex="2">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
            <HBox alignment="CENTER_LEFT" prefHeight="100.0" prefWidth="200.0" spacing="10.0" GridPane.columnIndex="1" GridPane.halignment="CENTER" GridPane.hgrow="ALWAYS" GridPane.rowIndex="3" GridPane.valignment="CENTER" GridPane.vgrow="ALWAYS">
               <children>
                  <Label fx:id="LabelBasePrice" lineSpacing="10.0" maxHeight="1.7976931348623157E308" maxWidth="-Infinity" prefWidth="100.0" text="Label" HBox.hgrow="ALWAYS">
                     <opaqueInsets>
                        <Insets />
                     </opaqueInsets>
                     <contextMenu>
                        <ContextMenu>
                           <items>
                              <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                           </items>
                        </ContextMenu>
                     </contextMenu>
                  </Label>
                  <Label fx:id="LabelPriceWithDiscount" maxHeight="1.7976931348623157E308" maxWidth="-Infinity" prefWidth="100.0" text="Label" HBox.hgrow="ALWAYS">
                     <contextMenu>
                        <ContextMenu>
                           <items>
                              <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                           </items>
                        </ContextMenu>
                     </contextMenu></Label>
               </children>
               <GridPane.margin>
                  <Insets bottom="10.0" left="10.0" right="10.0" top="10.0" />
               </GridPane.margin>
            </HBox>
            <HBox alignment="CENTER_LEFT" layoutX="120.0" layoutY="100.0" prefHeight="100.0" prefWidth="200.0" spacing="10.0" GridPane.columnIndex="1" GridPane.rowIndex="4">
               <children>
                  <Label fx:id="LabelCountInBasket" lineSpacing="10.0" maxHeight="1.7976931348623157E308" maxWidth="-Infinity" prefWidth="100.0" text="Label" HBox.hgrow="ALWAYS">
                     <opaqueInsets>
                        <Insets />
                     </opaqueInsets>
                     <contextMenu>
                        <ContextMenu>
                           <items>
                              <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                           </items>
                        </ContextMenu>
                     </contextMenu>
                     <padding>
                        <Insets left="10.0" />
                     </padding>
                  </Label>
                  <Label fx:id="LabelCountInStock" maxHeight="1.7976931348623157E308" maxWidth="-Infinity" prefHeight="24.0" prefWidth="293.0" text="Label" HBox.hgrow="ALWAYS">
                     <contextMenu>
                        <ContextMenu>
                           <items>
                              <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                           </items>
                        </ContextMenu>
                     </contextMenu>
                     <padding>
                        <Insets left="10.0" />
                     </padding>
                  </Label>
               </children>
            </HBox>
            <Label fx:id="LabelInfo" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="product-cell" text="Label" GridPane.columnIndex="1" GridPane.rowIndex="5">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
         </children>
      </GridPane>
   </children>
</AnchorPane>

```
### product-edit-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Button?>
<?import javafx.scene.control.ButtonBar?>
<?import javafx.scene.control.ComboBox?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.TextArea?>
<?import javafx.scene.control.TextField?>
<?import javafx.scene.image.ImageView?>
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.layout.BorderPane?>
<?import javafx.scene.layout.ColumnConstraints?>
<?import javafx.scene.layout.GridPane?>
<?import javafx.scene.layout.RowConstraints?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="600.0" prefWidth="800.0" xmlns="http://javafx.com/javafx/22" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.ProductEditViewController">
   <children>
      <BorderPane layoutX="170.0" layoutY="47.0" prefHeight="200.0" prefWidth="200.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
         <bottom>
            <ButtonBar prefHeight="40.0" prefWidth="200.0" BorderPane.alignment="CENTER">
              <buttons>
                <Button fx:id="BtnSave" defaultButton="true" mnemonicParsing="false" onAction="#BtnSaveAction" text="Сохранить" />
                  <Button fx:id="BtnCancel" cancelButton="true" mnemonicParsing="false" onAction="#BtnCancelAction" text="Отмена" />
              </buttons>
               <padding>
                  <Insets right="20.0" />
               </padding>
            </ButtonBar>
         </bottom>
         <center>
            <GridPane BorderPane.alignment="CENTER">
              <columnConstraints>
                <ColumnConstraints hgrow="SOMETIMES" maxWidth="190.0" minWidth="10.0" prefWidth="190.0" />
                <ColumnConstraints hgrow="SOMETIMES" maxWidth="680.0" minWidth="10.0" prefWidth="610.0" />
              </columnConstraints>
              <rowConstraints>
                <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="ALWAYS" />
                <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="100.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
                <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
                  <RowConstraints minHeight="10.0" prefHeight="30.0" />
              </rowConstraints>
               <children>
                  <TextField fx:id="TextFieldArtikul" GridPane.columnIndex="1" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Артикул" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Название" GridPane.rowIndex="2" />
                  <TextField fx:id="TextFieldTitle" GridPane.columnIndex="1" GridPane.rowIndex="2" />
                  <TextArea fx:id="TextAreaDescription" prefHeight="200.0" prefWidth="200.0" GridPane.columnIndex="1" GridPane.rowIndex="4" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Описание" GridPane.rowIndex="4" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Стоимость за единицу" GridPane.rowIndex="5" />
                  <TextField fx:id="TextFieldCost" GridPane.columnIndex="1" GridPane.rowIndex="5" />
                  <ComboBox fx:id="ComboBoxCategory" prefHeight="25.0" prefWidth="300.0" GridPane.columnIndex="1" GridPane.rowIndex="3" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Категория товара" GridPane.rowIndex="3" />
                  <ComboBox fx:id="ComboBoxUnittype" prefHeight="25.0" prefWidth="300.0" GridPane.columnIndex="1" GridPane.rowIndex="7" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Единицы измерения" GridPane.rowIndex="7" />
                  <ComboBox fx:id="ComboBoxManufacturer" prefHeight="25.0" prefWidth="300.0" GridPane.columnIndex="1" GridPane.rowIndex="8" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Производитель" GridPane.rowIndex="8" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Поставщик" GridPane.rowIndex="9" />
                  <ComboBox fx:id="ComboBoxSupplier" prefHeight="25.0" prefWidth="300.0" GridPane.columnIndex="1" GridPane.rowIndex="9" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" prefHeight="17.0" prefWidth="184.0" text="Размер максимальной скидки" GridPane.rowIndex="10" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Количество на складе" GridPane.rowIndex="6" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" text="Изображение" GridPane.rowIndex="1" />
                  <Label maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" prefHeight="17.0" prefWidth="184.0" text="Размер действующей скидки" GridPane.rowIndex="11" />
                  <ImageView fx:id="ImageViewPhoto" fitHeight="150.0" fitWidth="200.0" pickOnBounds="true" preserveRatio="true" GridPane.columnIndex="1" GridPane.rowIndex="1" />
                  <Button fx:id="BtnLoadImage" mnemonicParsing="false" onAction="#BtnLoadImageAction" text="Загрузить" GridPane.columnIndex="1" GridPane.rowIndex="1" GridPane.valignment="BOTTOM">
                     <GridPane.margin>
                        <Insets left="200.0" />
                     </GridPane.margin>
                  </Button>
                  <TextField fx:id="TextFieldCountInStock" GridPane.columnIndex="1" GridPane.rowIndex="6" />
                  <TextField fx:id="TextFieldDiscountAmountMax" GridPane.columnIndex="1" GridPane.rowIndex="10" />
                  <TextField fx:id="TextFieldDiscountAmount" GridPane.columnIndex="1" GridPane.rowIndex="11" />
               </children>
            </GridPane>
         </center>
      </BorderPane>
   </children>
</AnchorPane>

```

### productcell-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.ContextMenu?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.MenuItem?>
<?import javafx.scene.image.ImageView?>
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.layout.ColumnConstraints?>
<?import javafx.scene.layout.GridPane?>
<?import javafx.scene.layout.HBox?>
<?import javafx.scene.layout.RowConstraints?>
<?import javafx.scene.text.Font?>

<AnchorPane fx:id="CellAnchorPane" maxHeight="-Infinity" maxWidth="1.7976931348623157E308" minHeight="-Infinity" minWidth="-Infinity" prefHeight="142.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/22" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.ListCellController">
   <children>
      <GridPane layoutY="14.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
        <columnConstraints>
          <ColumnConstraints minWidth="100.0" prefWidth="100.0" />
            <ColumnConstraints hgrow="ALWAYS" maxWidth="1.7976931348623157E308" minWidth="10.0" prefWidth="100.0" />
          <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
        </columnConstraints>
        <rowConstraints>
          <RowConstraints maxHeight="-Infinity" minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
          <RowConstraints maxHeight="-Infinity" minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
            <RowConstraints maxHeight="-Infinity" minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
          <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
        </rowConstraints>
         <children>
            <ImageView fx:id="ImageViewPhoto" fitHeight="140.0" fitWidth="90.0" pickOnBounds="true" preserveRatio="true" GridPane.rowSpan="4">
               <GridPane.margin>
                  <Insets bottom="5.0" left="5.0" right="5.0" top="5.0" />
               </GridPane.margin></ImageView>
            <Label fx:id="LabelPercent" alignment="CENTER" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="percent-label" text="Label" textAlignment="CENTER" GridPane.columnIndex="2" GridPane.rowSpan="4">
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                    <items>
                      <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                    </items>
                  </ContextMenu>
               </contextMenu></Label>
            <Label fx:id="LabelTitle" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="title-label" text="Label" GridPane.columnIndex="1" GridPane.hgrow="ALWAYS">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
            <Label fx:id="LabelDescription" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="product-cell" text="Label" GridPane.columnIndex="1" GridPane.rowIndex="1">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
            <Label fx:id="LabelManufacturer" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" styleClass="product-cell" text="Label" GridPane.columnIndex="1" GridPane.rowIndex="2">
               <padding>
                  <Insets left="10.0" />
               </padding>
               <font>
                  <Font size="14.0" />
               </font>
               <contextMenu>
                  <ContextMenu>
                     <items>
                        <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                     </items>
                  </ContextMenu>
               </contextMenu>
            </Label>
            <HBox alignment="CENTER_LEFT" prefHeight="100.0" prefWidth="200.0" spacing="10.0" GridPane.columnIndex="1" GridPane.halignment="CENTER" GridPane.hgrow="ALWAYS" GridPane.rowIndex="3" GridPane.valignment="CENTER" GridPane.vgrow="ALWAYS">
               <children>
                  <Label fx:id="LabelBasePrice" lineSpacing="10.0" maxHeight="1.7976931348623157E308" maxWidth="-Infinity" prefWidth="100.0" text="Label" HBox.hgrow="ALWAYS">
                     <opaqueInsets>
                        <Insets />
                     </opaqueInsets>
                     <contextMenu>
                        <ContextMenu>
                           <items>
                              <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                           </items>
                        </ContextMenu>
                     </contextMenu>
                  </Label>
                  <Label fx:id="LabelPriceWithDiscount" maxHeight="1.7976931348623157E308" maxWidth="-Infinity" prefWidth="100.0" text="Label" HBox.hgrow="ALWAYS">
                     <contextMenu>
                        <ContextMenu>
                           <items>
                              <MenuItem mnemonicParsing="false" onAction="#AddProductInBasket" text="Добавить в корзину" />
                           </items>
                        </ContextMenu>
                     </contextMenu></Label>
               </children>
               <GridPane.margin>
                  <Insets bottom="10.0" left="10.0" right="10.0" top="10.0" />
               </GridPane.margin>
            </HBox>
         </children>
      </GridPane>
   </children>
</AnchorPane>

```
### products-table-view.fxml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.CheckBox?>
<?import javafx.scene.control.ComboBox?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.Menu?>
<?import javafx.scene.control.MenuBar?>
<?import javafx.scene.control.MenuItem?>
<?import javafx.scene.control.TableColumn?>
<?import javafx.scene.control.TableView?>
<?import javafx.scene.control.TextField?>
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.layout.BorderPane?>
<?import javafx.scene.layout.FlowPane?>
<?import javafx.scene.layout.HBox?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="600.0" prefWidth="800.0" xmlns="http://javafx.com/javafx/22" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ru.demo.tradeapp.controller.ProductTableViewController">
   <children>
      <BorderPane prefHeight="200.0" prefWidth="200.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0">
         <top>
            <HBox BorderPane.alignment="CENTER_LEFT">
               <children>
                  <MenuBar maxWidth="1.7976931348623157E308" prefHeight="25.0" prefWidth="381.0" HBox.hgrow="ALWAYS">
                    <menus>
                      <Menu mnemonicParsing="false" text="Файл">
                        <items>
                          <MenuItem fx:id="MenuItemBack" mnemonicParsing="false" onAction="#MenuItemBackAction" text="Назад" />
                        </items>
                      </Menu>
                      <Menu mnemonicParsing="false" text="Правка">
                        <items>
                          <MenuItem fx:id="MenuItemAdd" mnemonicParsing="false" onAction="#MenuItemAddAction" text="Добавить" />
                              <MenuItem fx:id="MenuItemUpdate" mnemonicParsing="false" onAction="#MenuItemUpdateAction" text="Изменить" />
                              <MenuItem fx:id="MenuItemDelete" mnemonicParsing="false" onAction="#MenuItemDeleteAction" text="Удалить" />
                        </items>
                      </Menu>
                      <Menu mnemonicParsing="false" text="Справочники">
                        <items>
                          <MenuItem fx:id="MenuItemCategories" mnemonicParsing="false" onAction="#MenuItemCategoriesAction" text="Категории" />
                              <MenuItem fx:id="MenuItemManufacturers" mnemonicParsing="false" onAction="#MenuItemManufacturersAction" text="Производители" />
                              <MenuItem fx:id="MenuItemSuppliers" mnemonicParsing="false" onAction="#MenuItemSuppliersAction" text="Поставщики" />
                              <MenuItem fx:id="MenuItemUnittypes" mnemonicParsing="false" onAction="#MenuItemUnittypesAction" text="Единицы измерения" />
                        </items>
                      </Menu>
                    </menus>
                  </MenuBar>
                  <Label fx:id="LabelUser" alignment="CENTER_RIGHT" maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308" prefWidth="120.0" styleClass="user-label" text="Label" textAlignment="RIGHT" HBox.hgrow="ALWAYS">
                     <padding>
                        <Insets right="20.0" />
                     </padding>
                  </Label>
               </children>
            </HBox>
         </top>
         <center>
            <BorderPane BorderPane.alignment="CENTER">
               <top>
                  <FlowPane nodeOrientation="LEFT_TO_RIGHT" rowValignment="TOP" BorderPane.alignment="CENTER">
                     <children>
                        <TextField fx:id="TextFieldSearch" onAction="#TextFieldSearchAction" onInputMethodTextChanged="#TextFieldTextChanged" prefHeight="25.0" prefWidth="262.0" promptText="Введите название для поиска" />
                        <ComboBox fx:id="ComboBoxProductType" onAction="#ComboBoxProductTypeAction" prefWidth="150.0" promptText="тип продукта" />
                        <ComboBox fx:id="ComboBoxDiscount" onAction="#ComboBoxDiscountAction" prefHeight="26.0" prefWidth="143.0" promptText="скидка" />
                        <ComboBox fx:id="ComboBoxSort" onAction="#ComboBoxSortAction" prefWidth="150.0" promptText="сортировка" />
                     </children>
                  </FlowPane>
               </top>
               <center>
                  <TableView fx:id="TableViewProducts" prefHeight="200.0" prefWidth="200.0" tableMenuButtonVisible="true" BorderPane.alignment="CENTER">
                    <columns>
                        <TableColumn id="TableColumnPhoto" fx:id="TableColumnPhoto" prefWidth="96.0" resizable="false" text="Фото" />
                      <TableColumn id="TableColumnProductId" fx:id="TableColumnProductId" minWidth="0.0" prefWidth="86.0" text="Артикул" />
                      <TableColumn id="TableColumnTitle" fx:id="TableColumnTitle" maxWidth="1.7976931348623157E308" prefWidth="142.0" resizable="false" text="Наименование" />
                        <TableColumn id="TableColumnCountInStock" fx:id="TableColumnCountInStock" minWidth="0.0" prefWidth="179.0" text="Количество на складе" />
                        <TableColumn fx:id="TableColumnDiscount" prefWidth="81.0" text="Скидка" />
                        <TableColumn fx:id="TableColumnCost" prefWidth="133.0" text="Цена со скидкой" />
                    </columns>
                  </TableView>
               </center>
            </BorderPane>
         </center>
         <bottom>
            <Label fx:id="LabelInfo" text="Label" BorderPane.alignment="CENTER_LEFT" />
         </bottom>
      </BorderPane>
      <CheckBox layoutX="-196.0" layoutY="-157.0" mnemonicParsing="false" text="CheckBox" />
   </children>
</AnchorPane>

```


## Создание и изменение существующих контроллеров


1. Добавьте классы контроллеров или замените код существующих
![img_2.png](Lesson5Images/img_2.png)

### Класс LoginController
```java
package ru.demo.tradeapp.controller;

import jakarta.persistence.Query;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.FXMLLoader;
import javafx.fxml.Initializable;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.PasswordField;
import javafx.scene.control.TextField;
import javafx.scene.image.ImageView;
import javafx.scene.layout.RowConstraints;
import javafx.stage.Stage;
import org.hibernate.Session;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.User;
import ru.demo.tradeapp.util.HibernateSessionFactoryUtil;
import ru.demo.tradeapp.util.MakeCaptcha;
import ru.demo.tradeapp.util.Manager;

import java.io.IOException;
import java.net.URL;
import java.util.*;

import static ru.demo.tradeapp.util.Manager.ShowErrorMessageBox;
import static ru.demo.tradeapp.util.Manager.screenSize;

public class LoginController implements Initializable {

    boolean isWrongCaptha;
    boolean isShowCaptha;
    String captchaCode;
    int secondsLeft;
    @FXML
    RowConstraints ThirdRow;
    @FXML
    Button BtnRenewCaptcha;
    @FXML
    private Button BtnCancel;
    @FXML
    private Button BtnOk;
    @FXML
    private PasswordField PasswordField;
    @FXML
    private TextField TextFieldUsername;
    @FXML
    private TextField TextFieldCaptcha;
    @FXML
    private ImageView ImageViewCaptcha;

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        initController();
    }

    @FXML
    void BtnRenewCaptchaAction(ActionEvent event) {
        generateCaptcha();
    }

    @FXML
    void BtnCancelAction(ActionEvent event) {
        Manager.ShowPopup();
    }

    @FXML
    void BtnOkActon(ActionEvent event) {
        try (Session session = HibernateSessionFactoryUtil.getSessionFactory().openSession()) {
            Query query = session.createQuery("from User", User.class);
            List<User> users = query.getResultList();
            Optional<User> person = users.stream().filter(user -> user.getUsername().equals(TextFieldUsername.getText()) &&
                    user.getPassword().equals(PasswordField.getText())).findFirst();

            if (person.isEmpty() && isShowCaptha && !TextFieldCaptcha.getText().equals(captchaCode)) {
                System.out.println("Bad error");
                ShowErrorMessageBox("Не верный логин, пароль или текст капчи");
                blockButtons();
                return;
            }
            if (person.isEmpty() && (!isShowCaptha)) {
                System.out.println("Bad error");
                generateCaptcha();
                isShowCaptha = true;
                ThirdRow.setPrefHeight(50);
                ImageViewCaptcha.setVisible(true);
                TextFieldCaptcha.setVisible(true);
                BtnRenewCaptcha.setVisible(true);
                ShowErrorMessageBox("Не верный логин или пароль");
                return;
            }
            if (person.isPresent() && isShowCaptha && !TextFieldCaptcha.getText().equals(captchaCode)) {
                blockButtons();
                ShowErrorMessageBox("Не верный логин, пароль или текст капчи");
                return;
            }

            if (person.isPresent() && isShowCaptha && TextFieldCaptcha.getText().equals(captchaCode)) {
                showMainWindow(person.get());
                return;
            }

            if (person.isPresent() && !isShowCaptha) {
                showMainWindow(person.get());
            }
        }
    }

    public void generateCaptcha() {
        try {
            ImageViewCaptcha.setImage(MakeCaptcha.CreateImage(150, 40, 4));
            captchaCode = MakeCaptcha.captchaCode();
            System.out.println(captchaCode);
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }


    public void showMainWindow(User person) {
        Manager.currentUser = person;
        System.out.println(Manager.currentUser);
        Manager.mainStage.hide();
        Stage newWindow = new Stage();

        FXMLLoader fxmlLoader = new FXMLLoader(TradeApp.class.getResource("main-view.fxml"));
       // FXMLLoader fxmlLoader = new FXMLLoader(TradeApp.class.getResource("category-table-view.fxml"));
        Scene scene = null;
        try {
            scene = new Scene(fxmlLoader.load(), screenSize.getWidth(), screenSize.getHeight());
            scene.getStylesheets().add("base-styles.css");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        newWindow.setTitle("Вы вошли как " + Manager.currentUser.getFirstName());
        newWindow.setMaximized(true);
        newWindow.setScene(scene);
        newWindow.setOnCloseRequest(e -> {
            Manager.mainStage.show();
        });
        Manager.secondStage = newWindow;

        newWindow.show();
    }

    public void initTimer() {
        TimerTask task = new TimerTask() {
            public void run() {
                System.out.println("Task performed on: " + new Date() + "n" +
                        "Thread's name: " + Thread.currentThread().getName());
                secondsLeft--;
                if (secondsLeft == 0) ;
                {
                    BtnOk.setDisable(false);
                    BtnCancel.setDisable(false);
                    this.cancel();
                }
            }
        };
        Timer timer = new Timer("Timer");

        long delay = 10000L;
        timer.schedule(task, delay);
    }

    public void blockButtons() {
        initTimer();
        secondsLeft = 10;
        BtnOk.setDisable(true);
        BtnCancel.setDisable(true);
    }

    public void initController() {
        ThirdRow.setPrefHeight(0);
        TextFieldUsername.setText("maia");
        PasswordField.setText("1");
        TextFieldCaptcha.setVisible(false);
        BtnRenewCaptcha.setVisible(false);
        ImageViewCaptcha.setVisible(false);
        isWrongCaptha = false;
        isShowCaptha = false;
        captchaCode = "";
        secondsLeft = 0;
    }


}
```
### Класс MainWindowController
```java
package ru.demo.tradeapp.controller;

import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.FXMLLoader;
import javafx.fxml.Initializable;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.BorderPane;
import javafx.stage.Modality;
import javafx.stage.Stage;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.Category;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.service.CategoryService;
import ru.demo.tradeapp.service.ProductService;
import ru.demo.tradeapp.util.Manager;

import java.io.IOException;
import java.net.URL;
import java.util.Comparator;
import java.util.List;
import java.util.ResourceBundle;
import java.util.stream.Collectors;

import static ru.demo.tradeapp.util.Manager.currentUser;
import static ru.demo.tradeapp.util.Manager.screenSize;

public class MainWindowController implements Initializable {

    @FXML
    ComboBox<String> ComboBoxDiscount;
    @FXML
    Label LabelBasketInfo;

    @FXML
    Button BtnBasket;


    private int itemsCount;
    private CategoryService categoryService = new CategoryService();
    private ProductService productService = new ProductService();
    @FXML
    private ListView<Product> ListViewProducts;
    @FXML
    private ComboBox<Category> ComboBoxCategory;
    @FXML
    private ComboBox<String> ComboBoxSort;
    @FXML
    private Label LabelInfo;
    @FXML
    private Label LabelUser;
    @FXML
    private TextField TextFieldSearch;
    @FXML
    private BorderPane BorderPaneMainFrame;

    @FXML
    void TextFieldTextChanged(ActionEvent event) {
        filterData();
    }

    @FXML
    void ComboBoxCategoryAction(ActionEvent event) {
        filterData();
    }

    @FXML
    void ComboBoxDiscountAction(ActionEvent event) {
        filterData();
    }


    @FXML
    void MenuItemProductsAction(ActionEvent event) {
        Manager.LoadSecondStageScene("products-table-view.fxml");
    }

    @FXML
    void MenuItemOrdersAction(ActionEvent event) {
        Manager.LoadSecondStageScene("products-table-view.fxml");
    }

    @FXML
    void ComboBoxSortAction(ActionEvent event) {
        filterData();
    }

    @FXML
    void TextFieldSearchAction(ActionEvent event) {
        filterData();
    }

    @FXML
    void BtnBasketAction(ActionEvent event)
    {
        Stage newWindow = new Stage();
        FXMLLoader fxmlLoader = new FXMLLoader(TradeApp.class.getResource("order-view.fxml"));
        Scene scene = null;
        try {
            scene = new Scene(fxmlLoader.load(), screenSize.getWidth(), screenSize.getHeight());
            scene.getStylesheets().add("base-styles.css");

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        newWindow.initOwner(Manager.mainStage);
        newWindow.initModality(Modality.WINDOW_MODAL);
        newWindow.setScene(scene);
        newWindow.showAndWait();
    }

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        LabelUser.setText("Вы вошли как " + currentUser.getSecondName() + " " + Manager.currentUser.getFirstName());
        List<Category> categoryList = categoryService.findAll();
        categoryList.add(0, new Category(0L, "Все"));
        ObservableList<Category> categories = FXCollections.observableArrayList(categoryList);
        ComboBoxCategory.setItems(categories);
        ObservableList<String> discounts = FXCollections.observableArrayList("Все товары", "0-9.99%", "10-14.99%", "15% и более");
        ComboBoxDiscount.setItems(discounts);
        ObservableList<String> orders = FXCollections.observableArrayList("по возрастанию цены", "по убыванию цены");
        ComboBoxSort.setItems(orders);
        filterData();
    }

    public void loadProducts(Category category) {
        ListViewProducts.getItems().clear();
        List<Product> products = productService.findAll();
        itemsCount = products.size();
        LabelInfo.setText("Всего записей " + itemsCount + " из " + itemsCount);
        if (category != null) {
            products = products.stream().filter(product -> product.getCategory().getCategoryId().equals(category.getCategoryId())).collect(Collectors.toList());
            int filteredItemsCount = products.size();
            LabelInfo.setText("Всего записей " + filteredItemsCount + " из " + itemsCount);
        }
        for (Product product : products) {
            ListViewProducts.getItems().add(product);
        }
        ListViewProducts.setCellFactory(lv -> new ProductCell());
    }

    void filterData() {
        List<Product> products = productService.findAll();
        itemsCount = products.size();
        if (!ComboBoxCategory.getSelectionModel().isEmpty()) {
            Category category = ComboBoxCategory.getValue();
            if (category.getCategoryId() != 0) {
                products = products.stream().filter(product -> product.getCategory().getCategoryId().equals(category.getCategoryId())).collect(Collectors.toList());
            }
        }
        if (!ComboBoxDiscount.getSelectionModel().isEmpty()) {
            String discount = ComboBoxDiscount.getValue();
            if (discount.equals("0-9.99%")) {
                products = products.stream().filter(product -> product.getDiscountAmount() < 10).collect(Collectors.toList());
            }
            if (discount.equals("10-14.99%")) {
                products = products.stream().filter(product -> product.getDiscountAmount() >= 10 && product.getDiscountAmount() < 15).collect(Collectors.toList());
            }
            if (discount.equals("15% и более")) {
                products = products.stream().filter(product -> product.getDiscountAmount() >= 15).collect(Collectors.toList());
            }
        }
        if (!ComboBoxSort.getSelectionModel().isEmpty()) {
            String order = ComboBoxSort.getValue();
            if (order.equals("по возрастанию цены")) {
                products = products.stream().sorted(Comparator.comparing(Product::getPriceWithDiscount)).collect(Collectors.toList());
            }
            if (order.equals("по убыванию цены")) {
                products = products.stream().sorted(Comparator.comparing(Product::getPriceWithDiscount)).collect(Collectors.toList()).reversed();
            }
        }

        String searchText = TextFieldSearch.getText();
        if (!searchText.isEmpty()) {
            products = products.stream().filter(product -> product.getTitle().toLowerCase().contains(searchText.toLowerCase())).collect(Collectors.toList());
        }
        ListViewProducts.getItems().clear();
        for (Product product : products) {
            ListViewProducts.getItems().add(product);
        }
        ListViewProducts.setCellFactory(lv -> new ProductCell());
        int filteredItemsCount = products.size();
        LabelInfo.setText("Всего записей " + filteredItemsCount + " из " + itemsCount);
    }
}

```

### Класс ListCellController
```java
package ru.demo.tradeapp.controller;

import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.scene.control.Label;
import javafx.scene.image.ImageView;
import javafx.scene.layout.AnchorPane;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.util.Manager;

import java.io.IOException;

import static ru.demo.tradeapp.util.Manager.ShowErrorMessageBox;

public class ListCellController {

    @FXML
    private ImageView ImageViewPhoto;

    @FXML
    private Label LabelDescription;

    @FXML
    private Label LabelManufacturer;

    @FXML
    private Label LabelBasePrice;

    @FXML
    private Label LabelPriceWithDiscount;

    @FXML
    private Label LabelPercent;

    @FXML
    private Label LabelTitle;
    @FXML
    private AnchorPane CellAnchorPane;

    Product currentProduct;
    @FXML
    void AddProductInBasket(ActionEvent event) {

        ShowErrorMessageBox("Товар добавлен" + currentProduct.getTitle());
        Manager.mainBasket.addProductInBasket(currentProduct);


    }

    public void setProduct(Product product) throws IOException {
        currentProduct = product;
        ImageViewPhoto.setImage(product.getPhoto());
        LabelPercent.setText(product.getDiscountAmount().toString() + "%");
        LabelDescription.setText(product.getDescription());
        LabelTitle.setText(product.getTitle());
        LabelManufacturer.setText("Производитель: " + product.getManufacturer().getTitle());
        if (product.getDiscountAmount() >= 15) {
            CellAnchorPane.setStyle("-fx-background-color: #7fff00;");
        } else {
            CellAnchorPane.setStyle("-fx-background-color: #fff;");
        }

        LabelBasePrice.setText(String.format("%.2f", product.getCost()) + " руб.");
        LabelPriceWithDiscount.setVisible(false);
        LabelBasePrice.setStyle("-fx-text-fill: #000000;");
        if (product.getDiscountAmount() > 0) {
            LabelPriceWithDiscount.setText(String.format("%.2f", product.getPriceWithDiscount()) + " руб.");
            LabelPriceWithDiscount.setVisible(true);
            LabelPriceWithDiscount.setStyle("-fx-text-fill: #0000FF;");
            LabelBasePrice.setStyle("-fx-text-fill: #FF0000;");
        }

    }

}

```
### Класс CategoryTableViewController
```java
package ru.demo.tradeapp.controller;


import javafx.beans.property.SimpleStringProperty;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.FXMLLoader;
import javafx.fxml.Initializable;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.input.InputMethodEvent;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.Category;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.service.CategoryService;
import ru.demo.tradeapp.util.Manager;

import java.io.IOException;
import java.net.URL;
import java.util.List;
import java.util.Optional;
import java.util.ResourceBundle;
import java.util.stream.Collectors;

import static ru.demo.tradeapp.util.Manager.*;

public class CategoryTableViewController implements Initializable {

    @FXML
    private MenuItem MenuItemAdd;

    @FXML
    private MenuItem MenuItemBack;

    @FXML
    private MenuItem MenuItemDelete;


    @FXML
    private MenuItem MenuItemUpdate;


    @FXML
    private Label LabelInfo;

    @FXML
    private Label LabelUser;

    @FXML
    private TableColumn<Category, String> TableColumnId;

    @FXML
    private TableColumn<Category, String> TableColumnTitle;

    @FXML
    private TableView<Category> TableViewCategories;

    @FXML
    private TextField TextFieldSearch;
    private int itemsCount;
    private CategoryService categoryService = new CategoryService();


    @FXML
    void BtnBackAction(ActionEvent event) {
        FXMLLoader fxmlLoader = new FXMLLoader(TradeApp.class.getResource("products-table-view.fxml"));

        Scene scene = null;
        try {
            scene = new Scene(fxmlLoader.load(), screenSize.getWidth(), screenSize.getHeight());
            scene.getStylesheets().add("base-styles.css");
            Manager.secondStage.setMaximized(true);
            Manager.secondStage.setScene(scene);

            //Manager.mainStage.show();

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }


    @FXML
    void TextFieldSearchAction(ActionEvent event) {
        filterData();
    }


    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        initController();
    }

    public void initController() {
        LabelUser.setText("Вы вошли как " + currentUser.getSecondName()+ " "+ Manager.currentUser.getFirstName());
        setCellValueFactories();
        filterData();
    }

    void filterData() {
        List<Category> categories = categoryService.findAll();
        itemsCount = categories.size();

        String searchText = TextFieldSearch.getText();
        if (!searchText.isEmpty()) {
            categories = categories.stream().filter(product -> product.getTitle().toLowerCase().contains(searchText.toLowerCase())).collect(Collectors.toList());
        }
        TableViewCategories.getItems().clear();
        for (Category category : categories) {
            TableViewCategories.getItems().add(category);
        }

        int filteredItemsCount = categories.size();
        LabelInfo.setText("Всего записей " + filteredItemsCount + " из " + itemsCount);
    }

    private void setCellValueFactories() {

        TableColumnId.setCellValueFactory(cellData -> new SimpleStringProperty(cellData.getValue().getCategoryId().toString()));
        TableColumnTitle.setCellValueFactory(cellData -> new SimpleStringProperty(cellData.getValue().getTitle()));
    }

    @FXML
    void MenuItemAddAction(ActionEvent event) {
        Manager.currentProduct = null;
        //ShowEditProductWindow();
        filterData();
    }

    @FXML
    void MenuItemBackAction(ActionEvent event) {
        Manager.LoadSecondStageScene("products-table-view.fxml");
    }


    @FXML
    void MenuItemDeleteAction(ActionEvent event) {
        Category category = TableViewCategories.getSelectionModel().getSelectedItem();
        if (!category.getProducts().isEmpty()) {
            ShowErrorMessageBox("Ошибка целостности данных, у данного товара есть зависимые заказы");
            return;
        }

        Optional<ButtonType> result = ShowConfirmPopup();
        if (result.get() == ButtonType.OK) {
            categoryService.delete(category);
            filterData();
        }
    }


    @FXML
    void MenuItemUpdateAction(ActionEvent event) {
        Category category = TableViewCategories.getSelectionModel().getSelectedItem();
        // Manager.currentProduct = category;
        // ShowEditCategoryWindow();
        filterData();
    }

}

```
### Класс OrderCell
```java
package ru.demo.tradeapp.controller;

import javafx.fxml.FXMLLoader;
import javafx.scene.Parent;
import javafx.scene.control.ListCell;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.Order;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.util.Item;

import java.io.IOException;
import java.util.Map;

public class OrderCell extends ListCell<Item> {

    private final Parent root;
    private OrderCellController controller;

    public OrderCell() {
        try {
            FXMLLoader loader = new FXMLLoader(TradeApp.class.getResource("ordercell-view.fxml"));
            root = loader.load();
            root.getStylesheets().add("base-styles.css");
            controller = loader.getController();
        } catch (IOException exc) {
            throw new RuntimeException(exc);
        }
    }

    @Override
    protected void updateItem(Item item, boolean empty) {
        super.updateItem(item, empty);
        if (empty || item == null) {
            setGraphic(null);
        } else {
            try {
                controller.setItem(item);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
            setGraphic(root);
        }
    }
}

```
### Класс OrderCellController
```java
package ru.demo.tradeapp.controller;

import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.scene.control.Label;
import javafx.scene.image.ImageView;
import javafx.scene.layout.AnchorPane;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.util.Item;

import java.io.IOException;
import java.util.Map;

public class OrderCellController {

    @FXML
    private AnchorPane CellAnchorPane;

    @FXML
    private ImageView ImageViewPhoto;

    @FXML
    private Label LabelBasePrice;

    @FXML
    private Label LabelCountInBasket;

    @FXML
    private Label LabelCountInStock;

    @FXML
    private Label LabelDescription;

    @FXML
    private Label LabelInfo;

    @FXML
    private Label LabelManufacturer;

    @FXML
    private Label LabelPercent;

    @FXML
    private Label LabelPriceWithDiscount;

    @FXML
    private Label LabelTitle;

    @FXML
    void AddProductInBasket(ActionEvent event) {

    }

    public void setItem(Item item) throws IOException {
        Product product = item.getProduct();
        ImageViewPhoto.setImage(product.getPhoto());
        LabelPercent.setText(product.getDiscountAmount().toString() + "%");
        LabelDescription.setText(product.getDescription());
        LabelTitle.setText(product.getTitle());
        LabelManufacturer.setText("Производитель: " + product.getManufacturer().getTitle());

        LabelCountInBasket.setText("Количество: " + item.getCount());
        LabelCountInStock.setText("В наличии на складе: " + product.getQuantityInStock());

        LabelInfo.setText("Итого: " + item.getTotal() + "руб.");
        if (product.getDiscountAmount() >= 15) {
            CellAnchorPane.setStyle("-fx-background-color: #7fff00;");
        } else {
            CellAnchorPane.setStyle("-fx-background-color: #fff;");
        }

        LabelBasePrice.setText(String.format("%.2f", product.getCost()) + " руб.");
        LabelPriceWithDiscount.setVisible(false);
        LabelBasePrice.setStyle("-fx-text-fill: #000000;");
        if (product.getDiscountAmount() > 0) {
            LabelPriceWithDiscount.setText(String.format("%.2f", product.getPriceWithDiscount()) + " руб.");
            LabelPriceWithDiscount.setVisible(true);
            LabelPriceWithDiscount.setStyle("-fx-text-fill: #0000FF;");
            LabelBasePrice.setStyle("-fx-text-fill: #FF0000;");
        }

    }



}

```
### Класс OrderViewController
```java
package ru.demo.tradeapp.controller;

import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.*;
import javafx.scene.layout.BorderPane;
import ru.demo.tradeapp.model.*;
import ru.demo.tradeapp.service.OrderProductService;
import ru.demo.tradeapp.service.OrderService;
import ru.demo.tradeapp.service.PickupPointService;
import ru.demo.tradeapp.service.StatusService;
import ru.demo.tradeapp.util.Item;
import ru.demo.tradeapp.util.Manager;

import java.net.URL;
import java.time.LocalDate;
import java.util.*;
import java.util.stream.Collectors;

import static ru.demo.tradeapp.util.Manager.*;

public class OrderViewController implements Initializable {


    PickupPointService pickupPointService = new PickupPointService();
    StatusService statusService = new StatusService();
    OrderService orderService = new OrderService();
    OrderProductService orderProductService = new OrderProductService();
    Order newOrder;
    @FXML
    private BorderPane BorderPaneMainFrame;

    @FXML
    private Button BtnDelete;

    @FXML
    private Button BtnOk;

    @FXML
    private ComboBox<PickupPoint> ComboBoxPickupPoint;

    @FXML
    private ComboBox<Status> ComboStatus;

    @FXML
    private DatePicker DatePickerOrderCreateDate;

    @FXML
    private DatePicker DatePickerOrderDeliveryDate;

    @FXML
    private Label LabelBasketInfo;

    @FXML
    private Label LabelOrderGetCode;

    @FXML
    private Label LabelOrderNumber;

    @FXML
    private Label LabelUser;

    @FXML
    private ListView<Item> ListViewProducts;

    @FXML
    void BtnDeleteAction(ActionEvent event) {
        Item item = ListViewProducts.getSelectionModel().getSelectedItem();
        Alert alert = new Alert(Alert.AlertType.CONFIRMATION);
        alert.setTitle("Удаление");
        alert.setHeaderText("Вы действительно хотите удалить товар " + item.getProduct().getTitle() + " из корзины?");
        Optional<ButtonType> result = alert.showAndWait();
        if (result.get() == ButtonType.OK) {
            mainBasket.deleteProductFromBasket(item.getProduct());
            loadProducts();
        }
    }

    @FXML
    void BtnOkAction(ActionEvent event) {
        String error = checkFields().toString();
        if (!error.isEmpty()) {
            MessageBox("Ошибка", "Заполните поля", error, Alert.AlertType.ERROR);
            return;
        }
        newOrder.setUser(currentUser);
        newOrder.setStatus(ComboStatus.getValue());
        newOrder.setPickupPoint(ComboBoxPickupPoint.getValue());

        Set<OrderProduct> orderProducts = new HashSet<>();
        for (Item item : mainBasket.getBasket().values()) {
            OrderProduct orderProduct = new OrderProduct(newOrder, item.getProduct(), Long.valueOf(item.getCount()));
            orderProducts.add(orderProduct);
        }

        orderService.save(newOrder);



//       for (OrderProduct orderProduct :orderProducts) {
//            orderProductService.save(orderProduct);}


        MessageBox("Информация", "", "Данные сохранены успешно", Alert.AlertType.INFORMATION);

    }

    @FXML
    void ComboBoxPickupPointAction(ActionEvent event) {

    }

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        ComboStatus.setValue(new Status(1L, "Новый"));
        LabelUser.setText("Вы вошли как " + currentUser.getSecondName() + " " + Manager.currentUser.getFirstName());
        ComboBoxPickupPoint.setItems(FXCollections.observableArrayList(pickupPointService.findAll()));
        ComboStatus.setItems(FXCollections.observableArrayList(statusService.findAll()));
        newOrder = CreateNewOrder();
        LabelOrderNumber.setText("Заказ №" + newOrder.getOrderId() + " на имя " + currentUser.getSecondName() + " " + currentUser.getFirstName());
        DatePickerOrderCreateDate.setValue(newOrder.getCreateDate());
        DatePickerOrderDeliveryDate.setValue(newOrder.getDeliveryDate());
        LabelOrderGetCode.setText("Код выдачи: " + newOrder.getGetCode());
        loadProducts();
    }

    public void loadProducts() {
        ListViewProducts.getItems().clear();

        for (Map.Entry<Product, Item> entry : Manager.mainBasket.getBasket().entrySet()) {
            ListViewProducts.getItems().add(entry.getValue());
        }
        ListViewProducts.setCellFactory(lv -> new OrderCell());
        LabelBasketInfo.setText("Общая сумма заказа: " + Manager.mainBasket.getTotalCost() + ". Общий размер скидки: " + mainBasket.getTotalDiscount() + "%");
    }

    public Order CreateNewOrder() {
        Order order = new Order();

        Optional<Order> maxOrder = orderService.findAll().stream().max(Comparator.comparing(Order::getOrderId));
        if (maxOrder.isPresent())
            order.setOrderId(maxOrder.get().getOrderId() + 1);
        else {
            order.setOrderId(1L);
        }
        order.setCreateDate(LocalDate.now());
        if (Manager.mainBasket.isOnStock())
            order.setDeliveryDate(order.getCreateDate().plusDays(3));
        else
            order.setDeliveryDate(order.getCreateDate().plusDays(6));

        Random rnd = new Random();
        order.setGetCode(rnd.nextInt(100, 1000));
        return order;
    }

    StringBuilder checkFields() {
        StringBuilder error = new StringBuilder();

        if (ComboStatus.getValue() == null) {
            error.append("Выберите статус\n");
        }
        if (ComboBoxPickupPoint.getValue() == null) {
            error.append("Выберите пункт выдачи\n");
        }


        return error;
    }

}

```
### Класс ProductCell
```java
package ru.demo.tradeapp.controller;

import javafx.fxml.FXMLLoader;
import javafx.scene.Parent;
import javafx.scene.control.ListCell;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.Product;

import java.io.IOException;

public class ProductCell extends ListCell<Product> {

    private final Parent root;
    private ListCellController controller;

    public ProductCell() {
        try {
            FXMLLoader loader = new FXMLLoader(TradeApp.class.getResource("productcell-view.fxml"));
            root = loader.load();
            root.getStylesheets().add("base-styles.css");
            controller = loader.getController();
        } catch (IOException exc) {
            throw new RuntimeException(exc);
        }
    }

    @Override
    protected void updateItem(Product product, boolean empty) {
        super.updateItem(product, empty);
        if (empty || product == null) {
            setGraphic(null);
        } else {
            try {
                controller.setProduct(product);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
            setGraphic(root);
        }
    }
}

```
### Класс ProductEditViewController
```java
package ru.demo.tradeapp.controller;

import javafx.collections.FXCollections;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.*;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.stage.FileChooser;
import javafx.stage.Stage;
import ru.demo.tradeapp.model.*;
import ru.demo.tradeapp.service.*;
import ru.demo.tradeapp.util.Manager;

import java.io.File;
import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.ResourceBundle;

import static ru.demo.tradeapp.util.Manager.MessageBox;

public class ProductEditViewController implements Initializable {
    boolean imageLoaded = false;
    @FXML
    private Button BtnCancel;
    @FXML
    private Button BtnLoadImage;
    @FXML
    private Button BtnSave;
    private CategoryService categoryService = new CategoryService();
    private ManufacturerService manufacturerService = new ManufacturerService();
    private SupplierService supplierService = new SupplierService();
    private UnittypeService unittypeService = new UnittypeService();
    private ProductService productService = new ProductService();
    @FXML
    private ComboBox<Category> ComboBoxCategory;

    @FXML
    private ComboBox<Manufacturer> ComboBoxManufacturer;

    @FXML
    private ComboBox<Supplier> ComboBoxSupplier;

    @FXML
    private ComboBox<Unittype> ComboBoxUnittype;
    @FXML
    private ImageView ImageViewPhoto;
    @FXML
    private TextArea TextAreaDescription;

    @FXML
    private TextField TextFieldArtikul;

    @FXML
    private TextField TextFieldCost;

    @FXML
    private TextField TextFieldCountInStock;

    @FXML
    private TextField TextFieldDiscountAmount;

    @FXML
    private TextField TextFieldDiscountAmountMax;

    @FXML
    private TextField TextFieldTitle;

    @FXML
    void BtnLoadImageAction(ActionEvent event) throws MalformedURLException {
        FileChooser fileChooser = new FileChooser();
        fileChooser.getExtensionFilters().addAll(
                new FileChooser.ExtensionFilter("JPG", "*.jpg")
        );
        Stage stage = (Stage) BtnLoadImage.getScene().getWindow();
        File file = fileChooser.showOpenDialog(stage);

        if (file != null) {
            String imageUrl = file.toURI().toURL().toExternalForm();
            ImageViewPhoto.setImage(new Image(imageUrl));
            imageLoaded = true;
        }
    }

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        ComboBoxCategory.setItems(FXCollections.observableArrayList(categoryService.findAll()));
        ComboBoxSupplier.setItems(FXCollections.observableArrayList(supplierService.findAll()));
        ComboBoxManufacturer.setItems(FXCollections.observableArrayList(manufacturerService.findAll()));
        ComboBoxUnittype.setItems(FXCollections.observableArrayList(unittypeService.findAll()));
        if (Manager.currentProduct != null) {
            TextFieldArtikul.setEditable(false);
            TextFieldArtikul.setText(Manager.currentProduct.getProductId());
            TextFieldTitle.setText(Manager.currentProduct.getTitle());
            TextAreaDescription.setText(Manager.currentProduct.getDescription());
            TextFieldCost.setText(String.format("%.2f", Manager.currentProduct.getCost()));
            TextFieldCountInStock.setText(Manager.currentProduct.getQuantityInStock().toString());
            TextFieldDiscountAmount.setText(Manager.currentProduct.getDiscountAmount().toString());
            TextFieldDiscountAmountMax.setText(Manager.currentProduct.getMaxDiscountAmount().toString());
            try {
                ImageViewPhoto.setImage(Manager.currentProduct.getPhoto());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
            ComboBoxCategory.setValue(Manager.currentProduct.getCategory());
            ComboBoxSupplier.setValue(Manager.currentProduct.getSupplier());
            ComboBoxManufacturer.setValue(Manager.currentProduct.getManufacturer());
            ComboBoxUnittype.setValue(Manager.currentProduct.getUnittype());
        } else {
            Manager.currentProduct = new Product();
        }
    }

    @FXML
    void BtnCancelAction(ActionEvent event) {
        Stage stage = (Stage) BtnCancel.getScene().getWindow();
        // do what you have to do
        stage.close();
    }

    @FXML
    void BtnSaveAction(ActionEvent event) throws IOException {
        String error = checkFields().toString();
        if (!error.isEmpty()) {
            MessageBox("Ошибка", "Заполните поля", error, Alert.AlertType.ERROR);
            return;
        }
        Manager.currentProduct.setTitle(TextFieldTitle.getText());
        Manager.currentProduct.setCategory(ComboBoxCategory.getValue());
        Manager.currentProduct.setSupplier(ComboBoxSupplier.getValue());
        Manager.currentProduct.setUnittype(ComboBoxUnittype.getValue());
        Manager.currentProduct.setManufacturer(ComboBoxManufacturer.getValue());
        if (imageLoaded) {
            Manager.currentProduct.setPhoto(ImageViewPhoto.getImage());
        }
        String number = TextFieldCost.getText();
        number = number.replace(',', '.');
        Manager.currentProduct.setCost(Double.parseDouble(number));
        Manager.currentProduct.setDiscountAmount(Integer.parseInt(TextFieldDiscountAmount.getText()));
        Manager.currentProduct.setMaxDiscountAmount(Integer.parseInt(TextFieldDiscountAmountMax.getText()));
        Manager.currentProduct.setQuantityInStock(Integer.parseInt(TextFieldCountInStock.getText()));
        if (Manager.currentProduct.getProductId() == null) {
            Manager.currentProduct.setProductId(TextFieldArtikul.getText());
            productService.save(Manager.currentProduct);
            MessageBox("Информация", "", "Данные сохранены успешно", Alert.AlertType.INFORMATION);
        } else {
            productService.update(Manager.currentProduct);
            MessageBox("Информация", "", "Данные обновлены успешно", Alert.AlertType.INFORMATION);
        }
    }

    StringBuilder checkFields() {
        StringBuilder error = new StringBuilder();
        if (TextFieldArtikul.getText().isEmpty()) {
            error.append("Укажите артикул товара\n");
        }
        if (TextFieldTitle.getText().isEmpty()) {
            error.append("Укажите название товара\n");
        }
        if (TextFieldCost.getText().isEmpty()) {
            error.append("Укажите стоимость товара\n");
        }
        if (ComboBoxCategory.getValue() == null) {
            error.append("Выберите категорию\n");
        }
        if (ComboBoxManufacturer.getValue() == null) {
            error.append("Выберите производителя\n");
        }
        if (ComboBoxSupplier.getValue() == null) {
            error.append("Выберите поставщика\n");
        }
        if (ComboBoxUnittype.getValue() == null) {
            error.append("Выберите единицу измерения\n");
        }

        if (!IsInteger(TextFieldDiscountAmount.getText())) {
            error.append("Действующая скидка должна быть целым числом в диапазоне от 0% до 100%\n");
        }
        if (IsInteger(TextFieldDiscountAmount.getText()) && (Integer.parseInt(TextFieldDiscountAmount.getText()) < 0 || Integer.parseInt(TextFieldDiscountAmount.getText()) > 100)) {
            error.append("Действующая скидка должна быть целым числом в диапазоне от 0% до 100%\n");
        }
        if (!IsInteger(TextFieldDiscountAmountMax.getText())) {
            error.append("Максимальная скидка должна быть целым числом в диапазоне от 0% до 100%\n");
        }
        if (IsInteger(TextFieldDiscountAmountMax.getText()) && (Integer.parseInt(TextFieldDiscountAmountMax.getText()) < 0 || Integer.parseInt(TextFieldDiscountAmountMax.getText()) > 100)) {
            error.append("Максимальная скидка должна быть целым числом в диапазоне от 0% до 100%\n");
        }
        if (IsInteger(TextFieldDiscountAmountMax.getText()) && IsInteger(TextFieldDiscountAmount.getText())) {
            int maxDiscount = Integer.parseInt(TextFieldDiscountAmountMax.getText());
            int discount = Integer.parseInt(TextFieldDiscountAmount.getText());
            if (discount > maxDiscount)
                error.append("Действующая скидка не может быть больше максимальной\n");
        }
        if (!IsInteger(TextFieldCountInStock.getText())) {
            error.append("Количество товара на складе должно быть целым числом\n");
        }
        if (IsInteger(TextFieldCountInStock.getText()) && Integer.parseInt(TextFieldCountInStock.getText()) < 0) {
            error.append("Количество товара на складе должно быть положительным целым числом\n");
        }

        if (!IsDouble(TextFieldCost.getText())) {
            error.append("Стоимость должна быть положительным числом\n");
        }
        if (IsDouble(TextFieldCost.getText()) && Double.parseDouble(TextFieldCost.getText().replace(',', '.')) < 0) {
            error.append("Стоимость должна быть положительным числом\n");
        }

        return error;
    }

    boolean IsInteger(String number) {
        if (number == null) {
            return false;
        }
        try {
            int d = Integer.parseInt(number);
        } catch (NumberFormatException nfe) {
            return false;
        }
        return true;
    }

    boolean IsDouble(String number) {
        if (number == null) {
            return false;
        }
        try {
            number = number.replace(',', '.');
            double d = Double.parseDouble(number);
        } catch (NumberFormatException nfe) {
            return false;
        }
        return true;
    }

}

```
### Класс ProductTableViewController
```java
package ru.demo.tradeapp.controller;

import javafx.beans.property.SimpleIntegerProperty;
import javafx.beans.property.SimpleObjectProperty;
import javafx.beans.property.SimpleStringProperty;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.FXMLLoader;
import javafx.fxml.Initializable;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.image.ImageView;
import javafx.scene.input.InputMethodEvent;
import javafx.stage.Modality;
import javafx.stage.Stage;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.Category;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.service.CategoryService;
import ru.demo.tradeapp.service.ProductService;
import ru.demo.tradeapp.util.Manager;

import java.io.IOException;
import java.net.URL;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;
import java.util.ResourceBundle;
import java.util.stream.Collectors;

import static ru.demo.tradeapp.util.Manager.*;

public class ProductTableViewController implements Initializable {

    private int itemsCount;
    private CategoryService categoryService = new CategoryService();
    private ProductService productService = new ProductService();
    @FXML
    private ComboBox<String> ComboBoxDiscount;

    @FXML
    private ComboBox<Category> ComboBoxProductType;

    @FXML
    private ComboBox<String> ComboBoxSort;
    @FXML
    private MenuItem MenuItemAdd;

    @FXML
    private MenuItem MenuItemBack;

    @FXML
    private MenuItem MenuItemCategories;

    @FXML
    private MenuItem MenuItemDelete;

    @FXML
    private MenuItem MenuItemManufacturers;

    @FXML
    private MenuItem MenuItemSuppliers;

    @FXML
    private MenuItem MenuItemUnittypes;

    @FXML
    private MenuItem MenuItemUpdate;


    @FXML
    private TableColumn<Product, ImageView> TableColumnPhoto;

    @FXML
    private TableColumn<Product, Integer> TableColumnCountInStock;

    @FXML
    private TableColumn<Product, Integer> TableColumnDiscount;

    @FXML
    private TableColumn<Product, String> TableColumnCost;

    @FXML
    private TableColumn<Product, String> TableColumnProductId;


    @FXML
    private TableColumn<Product, String> TableColumnTitle;
    @FXML
    private Label LabelInfo;
    @FXML
    private Label LabelUser;
    @FXML
    private TextField TextFieldSearch;


    @FXML
    private TableView<Product> TableViewProducts;

    @FXML
    void ComboBoxDiscountAction(ActionEvent event) {
        filterData();
    }

    @FXML
    void ComboBoxProductTypeAction(ActionEvent event) {
        filterData();
    }

    @FXML
    void ComboBoxSortAction(ActionEvent event) {
        filterData();
    }

    @FXML
    void TextFieldSearchAction(ActionEvent event) {
        filterData();
    }


    void ShowEditProductWindow() {
        Stage newWindow = new Stage();
        FXMLLoader fxmlLoader = new FXMLLoader(TradeApp.class.getResource("product-edit-view.fxml"));

        Scene scene = null;
        try {
            scene = new Scene(fxmlLoader.load());
            scene.getStylesheets().add("base-styles.css");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        newWindow.setTitle("Изменить данные");
        newWindow.initOwner(Manager.secondStage);
        newWindow.initModality(Modality.WINDOW_MODAL);
        newWindow.setScene(scene);
        Manager.currentStage = newWindow;
        newWindow.showAndWait();
        Manager.currentStage = null;
        filterData();
    }

    @FXML
    void TextFieldTextChanged(InputMethodEvent event) {

    }

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        initController();
    }

    public void initController() {
        LabelUser.setText("Вы вошли как " + currentUser.getSecondName()+ " "+ Manager.currentUser.getFirstName());
        List<Category> categoryList = categoryService.findAll();
        categoryList.add(0, new Category(0L, "Все"));
        ObservableList<Category> categories = FXCollections.observableArrayList(categoryList);
        ComboBoxProductType.setItems(categories);
        ObservableList<String> discounts = FXCollections.observableArrayList("Все товары", "0-9.99%", "10-14.99%", "15% и более");
        ComboBoxDiscount.setItems(discounts);
        ObservableList<String> orders = FXCollections.observableArrayList("по возрастанию цены", "по убыванию цены");
        ComboBoxSort.setItems(orders);
        setCellValueFactories();
        filterData();
    }

    void filterData() {
        List<Product> products = productService.findAll();
        itemsCount = products.size();
        if (!ComboBoxProductType.getSelectionModel().isEmpty()) {
            Category category = ComboBoxProductType.getValue();
            if (category.getCategoryId() != 0) {
                products = products.stream().filter(product -> product.getCategory().getCategoryId().equals(category.getCategoryId())).collect(Collectors.toList());
            }
        }
        if (!ComboBoxDiscount.getSelectionModel().isEmpty()) {
            String discount = ComboBoxDiscount.getValue();
            if (discount.equals("0-9.99%")) {
                products = products.stream().filter(product -> product.getDiscountAmount() < 10).collect(Collectors.toList());
            }
            if (discount.equals("10-14.99%")) {
                products = products.stream().filter(product -> product.getDiscountAmount() >= 10 && product.getDiscountAmount() < 15).collect(Collectors.toList());
            }
            if (discount.equals("15% и более")) {
                products = products.stream().filter(product -> product.getDiscountAmount() >= 15).collect(Collectors.toList());
            }
        }
        if (!ComboBoxSort.getSelectionModel().isEmpty()) {
            String order = ComboBoxSort.getValue();
            if (order.equals("по возрастанию цены")) {
                products = products.stream().sorted(Comparator.comparing(Product::getPriceWithDiscount)).collect(Collectors.toList());
            }
            if (order.equals("по убыванию цены")) {
                products = products.stream().sorted(Comparator.comparing(Product::getPriceWithDiscount)).collect(Collectors.toList()).reversed();
            }
        }

        String searchText = TextFieldSearch.getText();
        if (!searchText.isEmpty()) {
            products = products.stream().filter(product -> product.getTitle().toLowerCase().contains(searchText.toLowerCase())).collect(Collectors.toList());
        }
        TableViewProducts.getItems().clear();
        for (Product product : products) {
            TableViewProducts.getItems().add(product);
        }

        int filteredItemsCount = products.size();
        LabelInfo.setText("Всего записей " + filteredItemsCount + " из " + itemsCount);
    }

    private void setCellValueFactories() {

        TableColumnPhoto.setCellValueFactory(cellData -> {
            try {
                return new SimpleObjectProperty<ImageView>(cellData.getValue().getImage());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        });
        TableColumnProductId.setCellValueFactory(cellData -> new SimpleStringProperty(cellData.getValue().getProductId()));
        TableColumnTitle.setCellValueFactory(cellData -> cellData.getValue().getPropertyTitle());
        TableColumnCountInStock.setCellValueFactory(cellData -> new SimpleIntegerProperty(cellData.getValue().getQuantityInStock()).asObject());
        TableColumnCost.setCellValueFactory(cellData -> new SimpleStringProperty(String.format(String.format("%.2f", cellData.getValue().getCost()) + " руб.")));
        TableColumnDiscount.setCellValueFactory(cellData -> new SimpleIntegerProperty(cellData.getValue().getDiscountAmount()).asObject());
    }

    @FXML
    void MenuItemAddAction(ActionEvent event) {
        Manager.currentProduct = null;
        ShowEditProductWindow();
        filterData();
    }

    @FXML
    void MenuItemBackAction(ActionEvent event) {
        Manager.LoadSecondStageScene("main-view.fxml");
    }

    @FXML
    void MenuItemCategoriesAction(ActionEvent event) {
        Manager.LoadSecondStageScene("category-table-view.fxml");
    }

    @FXML
    void MenuItemDeleteAction(ActionEvent event) {
        Product product = TableViewProducts.getSelectionModel().getSelectedItem();
        if (!product.getOrderProducts().isEmpty()) {
            ShowErrorMessageBox("Ошибка целостности данных, у данного товара есть зависимые заказы");
            return;
        }

        Optional<ButtonType> result = ShowConfirmPopup();
        if (result.get() == ButtonType.OK) {
            productService.delete(product);
            filterData();
        }
    }

    @FXML
    void MenuItemManufacturersAction(ActionEvent event) {
        Manager.LoadSecondStageScene("manufacturers-table-view.fxml");
    }

    @FXML
    void MenuItemSuppliersAction(ActionEvent event) {
        Manager.LoadSecondStageScene("suppliers-table-view.fxml");
    }

    @FXML
    void MenuItemUnittypesAction(ActionEvent event) {
        Manager.LoadSecondStageScene("unittypes-table-view.fxml");
    }

    @FXML
    void MenuItemUpdateAction(ActionEvent event) {
        Product product = TableViewProducts.getSelectionModel().getSelectedItem();
        Manager.currentProduct = product;
        ShowEditProductWindow();
        filterData();
    }
}


```

## Пакет util
1. Добавьте или измените код существующих классов
![img_3.png](Lesson5Images/img_3.png)
### Manager.java
```java
package ru.demo.tradeapp.util;

import javafx.application.Platform;
import javafx.fxml.FXMLLoader;
import javafx.geometry.Rectangle2D;
import javafx.scene.Scene;
import javafx.scene.control.Alert;
import javafx.scene.control.ButtonType;
import javafx.stage.Screen;
import javafx.stage.Stage;
import ru.demo.tradeapp.TradeApp;
import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.model.User;

import java.io.IOException;
import java.util.Optional;

public class Manager {
    public static Rectangle2D screenSize = Screen.getPrimary().getVisualBounds();
    public static User currentUser = null;
    public static Stage mainStage;
    public static Stage secondStage;
    public static Stage currentStage;
    public static Product currentProduct;

    public static Basket mainBasket = new Basket();

    public static void ShowPopup() {
        Alert alert = new Alert(Alert.AlertType.CONFIRMATION);
        alert.setTitle("Закрыть приложение");
        alert.setHeaderText("Вы хотите выйти из приложения?");
        alert.setContentText("Все несохраненные данные, будут утеряны");

        Optional<ButtonType> result = alert.showAndWait();
        if (result.get() == ButtonType.OK) {
            Platform.exit();
        }
    }

    public static void ShowErrorMessageBox(String message) {
        Alert alert = new Alert(Alert.AlertType.ERROR);
        alert.setTitle("Ошибка");
        alert.setHeaderText(message);
        alert.showAndWait();
    }

    public static void MessageBox(String title, String header, String message, Alert.AlertType alertType) {
        Alert alert = new Alert(alertType);
        alert.setTitle(title);
        alert.setHeaderText(header);
        alert.setContentText(message);
        alert.showAndWait();

    }

    public static Optional<ButtonType> ShowConfirmPopup() {
        Alert alert = new Alert(Alert.AlertType.CONFIRMATION);
        alert.setTitle("Удаление");
        alert.setHeaderText("Вы действительно хотите удалить запись?");
        alert.setContentText("Также будут удалены все зависимые от этой записи данные");
        Optional<ButtonType> result = alert.showAndWait();
        return result;
    }

    public static void LoadSecondStageScene(String fxmlFileName)
    {
        FXMLLoader fxmlLoader = new FXMLLoader(TradeApp.class.getResource(fxmlFileName));
        Scene scene = null;
        try {
            scene = new Scene(fxmlLoader.load(), screenSize.getWidth(), screenSize.getHeight());
            scene.getStylesheets().add("base-styles.css");
            Manager.secondStage.setScene(scene);

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }


    public static void LoadOrderScene(String fxmlFileName)
    {

    }

}

```

5. Добавьте в папку util новый класс Basket
### Класс Basket
```java
package ru.demo.tradeapp.util;

import ru.demo.tradeapp.model.Product;

import java.util.HashMap;
import java.util.Map;

public class Basket
{
    protected HashMap<Product, Item> basket  = new  HashMap<Product, Item>();
    /// <summary>
    /// Value словаря - количество товара и стоимость
    /// </summary>
    public HashMap<Product, Item> getBasket()
    {
        return basket;
    }


    /// <summary>
    /// Словарь хранит товар в качестве ключа и BuyItem в качестве значения
    /// </summary>



    // очистка корзины
    public void clearBasket()
    {
        this.basket.clear();
    }
    /// <summary>
    /// Добавление товара в корзину
    /// </summary>
    /// <param name="product">Добавляемый товар</param>
    public void addProductInBasket(Product product)
    {
        // если такой товар есть в корзине
        if (this.basket.containsKey(product))
        {
            // увеличиваем его количество на +1
            int k = this.basket.get(product).count + 1;
            // пересчистваем стоимость
            double p = product.getPriceWithDiscount() * k;
            double c = product.getCost() * k;
            this.basket.put(product, new Item (product, k,c, p));
        }

        else
        {
            // добавляем новый товар в корзину в количесьве 1 шт
            double p = product.getPriceWithDiscount();
            double c = product.getCost();
            this.basket.put(product, new Item (product,1,c, p));
        }
    }
    /// <summary>
    /// Изменяет количество товара product в корзине
    /// </summary>
    /// <param name="product">Товар</param>
    /// <param name="count">количество товара</param>
    public  void setCount(Product product, int count)
    {
        if (this.basket.containsKey(product))
        {
            int k = count;
            double p = product.getPriceWithDiscount() * k;
            double c = product.getCost() * k;
            this.basket.put(product, new Item (product, k,c, p));
            // если количество 0 или меньше 0 удаляем товар из корзины
            if (k <= 0)
            {
                getBasket().remove(product);
            }
        }
    }
    /// <summary>
    /// Удаляем товар product из корзины
    /// </summary>
    /// <param name="product">Удаляемый товар</param>
    public  void deleteProductFromBasket(Product product)
    {
        if (this.basket.containsKey(product))
        {
            basket.remove(product);
        }
    }
    /// <summary>
    /// Cтоимость всех товаров в корзине
    /// </summary>
    public  double getTotalCost()
    {

            double sum = 0;
            for (Item item : basket.values())
            {
                sum += item.total;
            }
            return sum;
    }

    public  double getCostWithoutDiscont()
    {
        double sum = 0;
        for (Item item : basket.values())
        {
            sum += item.cost;
        }
        return sum;
    }

    public int getTotalDiscount()
    {
            int discount = (int) Math.round((getCostWithoutDiscont() - getTotalCost()) / getTotalCost() * 100);
            if (discount < 0)
                discount = 0;
            return discount;
    }
    /// <summary>
    /// Количество товаров в корзине
    /// </summary>
    public  int getCount()
    {
            return basket.size();

    }
    /// <summary>
    /// Возвращает true, если на складе каждого товара не меньше 3 единиц
    /// </summary>
    public boolean isOnStock()
    {
            for (Product item: basket.keySet())
            {
                if (item.getQuantityInStock() < 3)
                {
                    return false;
                }
            }    return true;
    }
}


```

6. Добавьте в папку util класс Item
### Класс Item
```java
package ru.demo.tradeapp.util;

import ru.demo.tradeapp.model.Product;

public class Item {

    public Product getProduct() {
        return product;
    }

    public Integer getCount() {
        return count;
    }

    public Double getCost() {
        return cost;
    }

    public Double getTotal() {
        return total;
    }

    Product product;
    Integer count;
    Double cost;
    Double total;

    Item(Product product, Integer count, Double cost, Double total) {
        this.product = product;
        this.count = count;
        this.cost = cost;
        this.total = total;
    }
}
```

## Добавление репозиториев
1. Добавьте или измените классы из папки repository
![img_4.png](Lesson5Images/img_4.png)
### Класс BaseDao
```java
package ru.demo.tradeapp.repository;

import org.hibernate.Session;
import org.hibernate.Transaction;
import ru.demo.tradeapp.util.HibernateSessionFactoryUtil;

import java.util.List;

public abstract class BaseDao<T>  {
    private Class<T> clazz;

    public BaseDao(Class<T> clazz) {
        this.clazz = clazz;
    }

    protected Session getCurrentSession() {
        return HibernateSessionFactoryUtil.getSessionFactory().openSession();
    }


    public void save(final T entity) {
        Session session = getCurrentSession();
        Transaction tx1 = session.beginTransaction();
        session.persist(entity);
        tx1.commit();
        session.close();
    }

    public void update(final T entity) {
        Session session = getCurrentSession();
        Transaction tx1 = session.beginTransaction();
        session.merge(entity);
        tx1.commit();
        session.close();
    }

    public void delete(final T entity) {
        Session session = getCurrentSession();
        Transaction tx1 = session.beginTransaction();
        session.remove(entity);
        tx1.commit();
        session.close();
    }

    public void deleteById(final long entityId) {
        final T entity = findOne(entityId);
        delete(entity);
    }

    public T findOne(final long id) {
        return getCurrentSession().get(clazz, id);
    }


    public List<T> findAll() {
        List<T> items = (List<T>) HibernateSessionFactoryUtil.getSessionFactory().openSession().createQuery("from " + clazz.getName()).list();
        return items;
    }
}

```

### Класс CategoryDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Category;

public class CategoryDao extends BaseDao<Category> {
    public CategoryDao() {
        super(Category.class);
    }
}

```
### Класс ManufacturerDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Manufacturer;

public class ManufacturerDao extends BaseDao<Manufacturer> {
    public ManufacturerDao() {
        super(Manufacturer.class);
    }
}

```
### Класс OrderDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Order;

public class OrderDao  extends BaseDao<Order> {
    public OrderDao() {
        super(Order.class);
    }
}

```
### Класс OrderProductDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.OrderProduct;
public class OrderProductDao  extends BaseDao<OrderProduct> {
    public OrderProductDao() {
        super(OrderProduct.class);
    }
}

```
### Класс PickupPointDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.PickupPoint;

public class PickupPointDao extends BaseDao<PickupPoint> {
    public PickupPointDao() {
        super(PickupPoint.class);
    }
}

```
### Класс ProductDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Product;

public class ProductDao extends BaseDao<Product> {
    public ProductDao() {
        super(Product.class);
    }
}

```
### Класс StatusDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Status;

public class StatusDao extends BaseDao<Status> {
    public StatusDao() {
        super(Status.class);
    }
}

```
### Класс SupplierDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Supplier;

public class SupplierDao extends BaseDao<Supplier> {
    public SupplierDao() {
        super(Supplier.class);
    }
}

```
### Класс UnittypeDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Unittype;

public class UnittypeDao extends BaseDao<Unittype> {
    public UnittypeDao() {
        super(Unittype.class);
    }
}

```
### Класс UserDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.User;

public class UserDao extends BaseDao<User>{
    public UserDao() {
        super(User.class);
    }
}

```
### Класс RoleDao
```java
package ru.demo.tradeapp.repository;

import ru.demo.tradeapp.model.Role;

public class RoleDao extends BaseDao<Role> {
    public RoleDao() {
        super(Role.class);
    }
}

```

## Добавление сервисов
1. Добавьте или измените классы сервисов из папки service
![img_5.png](Lesson5Images/img_5.png)
### класс CategoryService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Category;
import ru.demo.tradeapp.repository.CategoryDao;

import java.util.List;

public class CategoryService {
    private CategoryDao categoryDao = new CategoryDao();

    public CategoryService() {
    }

    public List<Category> findAll() {
        return categoryDao.findAll();
    }

    public Category findOne(final long id) {
        return categoryDao.findOne(id);
    }

    public void save(final Category entity)
    {
        if (entity == null)
            return;
        categoryDao.save(entity);
    }

    public void update(final Category entity)
    {
        if (entity == null)
            return;
        categoryDao.update(entity);
    }

    public void delete(final Category entity)
    {
        if (entity == null)
            return;
        categoryDao.delete(entity);
    }

    public void deleteById(final Long id)
    {
        if (id == null)
            return;
        categoryDao.deleteById(id);
    }
}

```

### класс ManufacturerService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Manufacturer;
import ru.demo.tradeapp.repository.ManufacturerDao;

import java.util.List;

public class ManufacturerService {
    private ManufacturerDao manufacturerDao = new ManufacturerDao();

    public ManufacturerService() {
    }

    public List<Manufacturer> findAll() {
        return manufacturerDao.findAll();
    }

    public Manufacturer findOne(final long id) {
        return manufacturerDao.findOne(id);
    }

    public void save(final Manufacturer entity) {
        if (entity == null)
            return;
        manufacturerDao.save(entity);
    }

    public void update(final Manufacturer entity) {
        if (entity == null)
            return;
        manufacturerDao.update(entity);
    }

    public void delete(final Manufacturer entity) {
        if (entity == null)
            return;
        manufacturerDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        manufacturerDao.deleteById(id);
    }
}
```

### класс OrderProductService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.OrderProduct;
import ru.demo.tradeapp.repository.OrderProductDao;

import java.util.List;

public class OrderProductService {
    private OrderProductDao orderProductDao = new OrderProductDao();

    public OrderProductService() {
    }

    public List<OrderProduct> findAll() {
        return orderProductDao.findAll();
    }

    public OrderProduct findOne(final long id) {
        return orderProductDao.findOne(id);
    }

    public void save(final OrderProduct entity) {
        if (entity == null)
            return;
        orderProductDao.save(entity);
    }

    public void update(final OrderProduct entity) {
        if (entity == null)
            return;
        orderProductDao.update(entity);
    }

    public void delete(final OrderProduct entity) {
        if (entity == null)
            return;
        orderProductDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        orderProductDao.deleteById(id);
    }
}

```

### класс OrderService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Order;
import ru.demo.tradeapp.repository.OrderDao;

import java.util.List;

public class OrderService {
    private OrderDao orderDao = new OrderDao();

    public OrderService() {
    }

    public List<Order> findAll() {
        return orderDao.findAll();
    }

    public Order findOne(final long id) {
        return orderDao.findOne(id);
    }

    public void save(final Order entity) {
        if (entity == null)
            return;
        orderDao.save(entity);
    }

    public void update(final Order entity) {
        if (entity == null)
            return;
        orderDao.update(entity);
    }

    public void delete(final Order entity) {
        if (entity == null)
            return;
        orderDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        orderDao.deleteById(id);
    }
}


```
### класс PickupPointService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.PickupPoint;
import ru.demo.tradeapp.repository.PickupPointDao;

import java.util.List;

public class PickupPointService {
    private PickupPointDao pickupPointDao = new PickupPointDao();

    public PickupPointService() {
    }

    public List<PickupPoint> findAll() {
        return pickupPointDao.findAll();
    }

    public PickupPoint findOne(final long id) {
        return pickupPointDao.findOne(id);
    }

    public void save(final PickupPoint entity) {
        if (entity == null)
            return;
        pickupPointDao.save(entity);
    }

    public void update(final PickupPoint entity) {
        if (entity == null)
            return;
        pickupPointDao.update(entity);
    }

    public void delete(final PickupPoint entity) {
        if (entity == null)
            return;
        pickupPointDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        pickupPointDao.deleteById(id);
    }
}

```

### класс ProductService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Product;
import ru.demo.tradeapp.repository.ProductDao;

import java.util.List;

public class ProductService {
    private ProductDao productDao = new ProductDao();

    public ProductService() {
    }

    public List<Product> findAll() {
        return productDao.findAll();
    }

    public Product findOne(final long id) {
        return productDao.findOne(id);
    }

    public void save(final Product entity) {
        if (entity == null)
            return;
        productDao.save(entity);
    }

    public void update(final Product entity) {
        if (entity == null)
            return;
        productDao.update(entity);
    }

    public void delete(final Product entity) {
        if (entity == null)
            return;
        productDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        productDao.deleteById(id);
    }
}


```

### класс RoleService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Role;
import ru.demo.tradeapp.repository.RoleDao;

import java.util.List;

public class RoleService {
    private RoleDao roleDao = new RoleDao();

    public RoleService() {
    }

    public List<Role> findAll() {
        return roleDao.findAll();
    }

    public Role findOne(final long id) {
        return roleDao.findOne(id);
    }

    public void save(final Role entity) {
        if (entity == null)
            return;
        roleDao.save(entity);
    }

    public void update(final Role entity) {
        if (entity == null)
            return;
        roleDao.update(entity);
    }

    public void delete(final Role entity) {
        if (entity == null)
            return;
        roleDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        roleDao.deleteById(id);
    }
}

```

### класс StatusService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Status;
import ru.demo.tradeapp.repository.StatusDao;

import java.util.List;

public class StatusService {
    private StatusDao statusDao = new StatusDao();

    public StatusService() {
    }

    public List<Status> findAll() {
        return statusDao.findAll();
    }

    public Status findOne(final long id) {
        return statusDao.findOne(id);
    }

    public void save(final Status entity) {
        if (entity == null)
            return;
        statusDao.save(entity);
    }

    public void update(final Status entity) {
        if (entity == null)
            return;
        statusDao.update(entity);
    }

    public void delete(final Status entity) {
        if (entity == null)
            return;
        statusDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        statusDao.deleteById(id);
    }
}

```

### класс SupplierService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Supplier;
import ru.demo.tradeapp.repository.SupplierDao;

import java.util.List;

public class SupplierService {
    private SupplierDao supplierDao = new SupplierDao();

    public SupplierService() {
    }

    public List<Supplier> findAll() {
        return supplierDao.findAll();
    }

    public Supplier findOne(final long id) {
        return supplierDao.findOne(id);
    }

    public void save(final Supplier entity) {
        if (entity == null)
            return;
        supplierDao.save(entity);
    }

    public void update(final Supplier entity) {
        if (entity == null)
            return;
        supplierDao.update(entity);
    }

    public void delete(final Supplier entity) {
        if (entity == null)
            return;
        supplierDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        supplierDao.deleteById(id);
    }
}


```

### класс UnittypeService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.Unittype;
import ru.demo.tradeapp.repository.UnittypeDao;

import java.util.List;

public class UnittypeService {
    private UnittypeDao unittypeDao = new UnittypeDao();

    public UnittypeService() {
    }

    public List<Unittype> findAll() {
        return unittypeDao.findAll();
    }

    public Unittype findOne(final long id) {
        return unittypeDao.findOne(id);
    }

    public void save(final Unittype entity) {
        if (entity == null)
            return;
        unittypeDao.save(entity);
    }

    public void update(final Unittype entity) {
        if (entity == null)
            return;
        unittypeDao.update(entity);
    }

    public void delete(final Unittype entity) {
        if (entity == null)
            return;
        unittypeDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        unittypeDao.deleteById(id);
    }
}


```

### класс UserService
```java
package ru.demo.tradeapp.service;

import ru.demo.tradeapp.model.User;
import ru.demo.tradeapp.repository.UserDao;

import java.util.List;

public class UserService {
    private UserDao userDao = new UserDao();

    public UserService() {
    }

    public List<User> findAll() {
        return userDao.findAll();
    }

    public User findOne(final long id) {
        return userDao.findOne(id);
    }

    public void save(final User entity) {
        if (entity == null)
            return;
        userDao.save(entity);
    }

    public void update(final User entity) {
        if (entity == null)
            return;
        userDao.update(entity);
    }

    public void delete(final User entity) {
        if (entity == null)
            return;
        userDao.delete(entity);
    }

    public void deleteById(final Long id) {
        if (id == null)
            return;
        userDao.deleteById(id);
    }
}
```





# Запуск приложения
1. Запуcтите приложение. Введите учетные данные, например логин: ```maia``` , пароль: ```1```.
![img_6.png](Lesson5Images/img_6.png)
2. Просмотрите работу приложения. На данном этапе в черновом виде реализованы:
   - добавление товаров в корзину
   - просмотр корзины
   - удаление товара из корзины
   - изменены интерфейс и стили приложения
   - добавлен просмотр списка категории товаров




# Задания
1. На основе макета CRUD с товарами создайте формы для просмотра, добавления, удаления и редактирования следуюущих сущностей:
   * PickupPoint
   * Role
   * Status

   


Предыдущее занятие | &nbsp; | Следующее занятие
:----------------:|:----------:|:----------------:
[Урок 4](Lesson4.md) | [Содержание](readme.md) | [Урок 6](Lesson6.md)