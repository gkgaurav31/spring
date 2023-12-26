# USING DATA SOURCES IN SPRING APPS

![data-sources](images/data-sources.png)

The JDK itself does not provide the implementation for connecting to any specific database. It offers abstraction through interfaces such as `java.sql.Connection`, `java.sql.ResultSet`, and `java.sql.Statement`. The actual implementation for connecting to a particular database is provided by JDBC drivers, which are offered by various vendors. While it is possible to directly use the JDBC driver (`JDBC DriverManager`) to establish connections to the database when necessary, this approach can introduce significant overhead. For instance, it requires authentication for each connection request made to the database. To alleviate this, `DataSource` objects come into play, facilitating the management of connections to the database. They optimize connection requests within your application, enhancing the performance of operations in the persistence layer. In SpringBoot, the default data source implementation utilizes [HikariCP](https://github.com/brettwooldridge/HikariCP) for efficient connection pooling.

## USING JDBC TEMPLATE TO WORK WITH PERSISTED DATA

When working with a database, we implement all the capabilities related to the
persistence layer in classes we (by convention) name repositories.

## DEPENDENCIES NEEDED

We will start by using an `in-memory` database `H2`.

The app only needs the database and the JDBC driver at runtime. The app doesn't need them for compilation. To instruct Maven we only want these dependencies at runtime,
we add the `scope` tag with the value `runtime`.

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

### SCHEMA FILE FOR THE DATBASE

Create a `schema.sql` file and keep it under `resources` folder.

```sql
CREATE TABLE IF NOT EXISTS purchase (
 id INT AUTO_INCREMENT PRIMARY KEY,
 product varchar(50) NOT NULL,
 price double NOT NULL
);
```

> In a real-world example, we will need to use a dependency that also allows us to version our database scripts. [Flywaydb](https://flywaydb.org/) and [Liquibase](https://www.liquibase.org/) are good options for it.

### IMPLEMENTATION

Let us start by creating the model class for a `Product`.

> When you want to store a floating-point value accurately and make sure you don't lose decimal precision when executing various operations with the values, use `BigDecimal` and not `double` or `float`

```java
package com.example.demo.model;

import java.math.BigDecimal;

public class Product {

    private int id;
    private String product;
    private BigDecimal price;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getProduct() {
        return product;
    }

    public void setProduct(String product) {
        this.product = product;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }
}
```

Now let us create the repository class `PurchaseRepository`:

> When Spring Boot saw you added the H2 dependency in `pom.xml`, it automatically configured a data source and a `JdbcTemplate` instance, which have used in this example.

> If we use Spring but not Spring Boot, we need to define the `DataSource` bean and the `JdbcTemplate` bean (we can add them in the Spring context using the `@Bean` annotation in the configuration class)

```java
package com.example.demo.repository;

import com.example.demo.model.Product;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class PurchaseRepository {

    private final JdbcTemplate jdbcTemplate;

    public PurchaseRepository(JdbcTemplate jdbcTemplate){
        this.jdbcTemplate = jdbcTemplate;
    }

    public void purchaseProduct(Product product){

        String sql = "INSERT INTO purchase VALUES(NULL, ?, ?)";

        jdbcTemplate.update(sql, product.getProduct(), product.getPrice());

    }

}
```

The `JdbcTemplate` `update()` method sends the query to the database server. The first parameter the method gets is the query, and the next parameters are the values for the parameters. These values replace, in the same order, each question mark in the query.

In Spring's `JdbcTemplate`, a SELECT query retrieves data from a database. To map this data into specific model objects (like a `Purchase` class), a `RowMapper` interface is implemented. It defines how each row from the `ResultSet` is transformed into an instance of the designated model class. The `RowMapper` serves as the mapping logic, assigning `ResultSet` values to object attributes, enabling the database data to be represented as instances of the model class for use within the Java application.
