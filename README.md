# Quarkus-MicroProfile-Fundamentals

## Prerequisite
- Install JDK 11 or higher
- Install Maven 3.6.2 or higher 3.8 is recommended
- Install docker, this will be used for the database

## Inventory service

### Hello world Quarkus
Create a Quarkus project at https://quarkus.io with the following settings:

    groupId org.acme
    artifactIdi: inventory-service

For extensions choose  

    SmallRy OpenAPI
    resteasy-jackson

Import this project into your IDE, and start the application (details are in the readme.md of the generated project)

Tip: if you are working with a light themed console, then some logging might be hard to read, add the following line to the `appilcation.properties`. Quarkus was build with the dark theme in mind ;)

    quarkus.log.console.darken=100


#### Lombok
If you are a fan of lombok you can add the following dependency:

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.24</version>
        <scope>provided</scope>
    </dependency>

## Rest endpoint
See https://quarkus.io/guides/rest-json

- Create a rest endpoint that will return a list of `ProductDTO`'s:

```java
    class ProductDTO {
        int id;
        String name;
        String brand;
        String category;
        short modelYear;
        BigDecimal listPrice;
    }
```
  Test the endpoint with swagger (http://localhost:8080/q/swagger-ui/)
- Create a rest assured `@QuakrusTest` unit test to very that the endpoint returns the correct list`
  Tip: have a look at https://quarkus.io/guides/getting-started-testing#recap-of-http-based-testing-in-jvm-mode and https://github.com/rest-assured/rest-assured/wiki/Usage

## Document API
See https://github.com/eclipse/microprofile-open-api/blob/2.0.1-RC1/spec/src/main/asciidoc/microprofile-openapi-spec.adoc and look for `Operation`

- Document the product resources and verify that swagger now shows documentations on the API

## Postgres
In the postgres folder there is `docker-compose.yml`. Head over to this folder and run `docker-compose up` to start the database

Goto http://localhost:8181 and login to the database using **Adminer**; Use the following settings:

    System: PostgreSql
    Server: db
    Username: postgres
    Password: example
    Database: [Leave  blank]


In the SQL command view execute the following commands:

    CREATE DATABASE inventory;

and

    CREATE DATABASE shop;

## Hibernate panache
See https://quarkus.io/guides/hibernate-orm-panache

In the documentation there is an example for the `quarkus-hibernate-orm-panache` and `quarkus-jdbc-postgresql` dependency. 
Add those to the application.
> Note if you don't configure the datasource in the application quarkus will spin up a postgres docker container, as we have our own we don't what that!

Configure the datasource according to the documentation.

## Liquibase or Flyway
To create a database schema and create an inital data set we will be using either Liquibase or Flyway as a database migration tool. You can pick your favourite to use in this course.

See https://quarkus.io/guides/liquibase or See https://quarkus.io/guides/flyway

In the postgres folder there are 3 `.sql` files we will be using 2 of the `create.sql` and `data.sql` configure the database migration tool to first load the `create.sql` then the `data.sql`

After this exercise you should have al least the 3 tables (categories, brands, products) and some data in them, check them out with **Adminer**.

## Implement database
Set the following property in the `appilication.properties`

    quarkus.hibernate-orm.database.generation=validate

This will check the current schema with the database. Implement the `@Entity`'s for the 3 tables. 

Hint:

    @ManyToOne
    @JoinColumn(name = "brand_id")
    Brand brand;

- Ensure the application starts again, and let the endpoint return the actual products from the database.

## Testing
- Also fix the now broken unit test. Use a `@InjectMock` for the rest test. See https://quarkus.io/blog/mocking/
- Implement a search api where users can search for a product given a name
- Implement a database test with h2.
  - Choose if you want to run tests with a clean db and let hibernate generate the database tables, or let the migrate tool do that for you. The scrips will also work on h2. 
  - See https://quarkus.io/guides/config-reference#profiles, so you can configure a different datasource for the test profile
  - See https://quarkus.io/guides/getting-started-testing for testing, checkout "Tests and transactions" and "Starting services before the Quarkus application starts" to start the h2 db.
  
## Shop service
Create a Quarkus project at https://quarkus.io with the following settings:

    groupId org.acme
    artifactIdi: shop-service

For extensions choose

    SmallRy OpenAPI
    resteasy-jackson
    postgreSQL
    Hiberante ORM with panache
    REST Client Jackson

- Configure the datasource like in the inventory-service but change the db to `shop`
- Change the http port configuration to 8070 with `quarkus.http.port` 

## Shop service security
- First secure the application with a jwt token see: https://quarkus.io/guides/security-jwt
  - Note that open-api (swagger) needs to know about security, first add the `RestApplication` class to your resource package in the shop-sevice
  
  ```
  @SecurityScheme(securitySchemeName = RestApplication.API_KEY, type = SecuritySchemeType.HTTP, scheme = "Bearer")
  public class RestApplication extends javax.ws.rs.core.Application {
  ```
  Then add to every method that requires authorization 
  ```
  @SecurityRequirement(name = API_KEY)
  ```

## Shop service implementation
- A `Customer` can be an object that can be persisted, or just be the login name of the jwt token. The choice is yours. 
- Create a `Cart` object that can be persisted. A cart belongs to a `Customer` and has a total price and a list of `Products`.
  Note that as we are building a Microservice we don't want/need to store a `Product` in the shop database, only a reference to it.
- Update the rest endpoint with the following methods:
  - Get `Cart` fot the current logged-in user
  - Update `Cart` fot the current logged-in user

## Call product service
See https://quarkus.io/guides/rest-client
When products are added/removed form the cart then the total price needs to be updated. We need to get the pricing from the product service.

- Add an endpoint to the product service, that can calculate the price for a list of products
- Update the total price with the result of the product service

## Health
See https://quarkus.io/guides/smallrye-health
- Add a heath endpoint that also checks that the database is "up" to both services.

## Metrics
See https://quarkus.io/guides/micrometer
- Add a timer to all the database calls in the inventory service
- Ensure that a histogram for the 90 end 95 percentiles are exposed

### Bonus Metrics
- Setup a docker image of https://hub.docker.com/r/prom/prometheus that runs prometheus and scraps your endpoint. Also see https://prometheus.io/docs/prometheus/latest/getting_started/#downloading-and-running-prometheus
- Try and get a graph from the timer metrics

## Configuration
See https://quarkus.io/guides/config
- Add a price multiplier configuration property `price.multipier` to the inventory service, that multiplies the price by the amount in the property

See https://quarkus.io/guides/config-mappings
- Rebuild the config, so it injects a `PriceConfiguration` object, please make sure it still works if the property is not configured, and use `1` as value.

## Fault tolerance
See https://quarkus.io/guides/smallrye-fault-tolerance
- Our Application is way too stable ;) lets make the calculate pricing rest call fail every and then succeed
  ```
  private AtomicLong counter = new AtomicLong(0);
  private void maybeFail() {
      // introduce some artificial failures
      final Long invocationNumber = counter.getAndIncrement();
      if (invocationNumber % 2 > 1) { // alternate 1 successful and 1 failing invocations
          throw new RuntimeException("Service failed.");
      }
  }
  ```
- Ensure that the Cart service retries the pricing operation if it fails

## Native image
See https://quarkus.io/guides/building-native-image 
Build a native image of your services.

## BONUS: Tracing
See https://quarkus.io/guides/opentracing
- Add open tracing to the shop service
- Start the jaeger tracing docker container `docker run -p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 jaegertracing/all-in-one:latest`
  - UI can be found on: http://localhost:16686/
- Add jdbc tracing
- Also add it to the inventory service and see that it works if you have the shop service call the inventory service
