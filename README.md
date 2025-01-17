Lab 04 - Jackson & JPA
==========

Before you start
----------
The purpose of this lab is to reinforce and build upon the lecture material concerning Jackson and JPA / Hibernate.

As with previous labs, you must first join a "group" on Canvas and then set up
your Github teams.

Make sure you include your `Team.md` file with a brief description of the roles and contribution of each member. Please also include, for each member, what their Github account name is. 

Exercise One - Build the JAX-RS Parolee project
----------
Project `lab-jpa-parolee` is a complete project that implements the Parolee Web service introduced in the previous lab. It includes a domain model, a DTO (data transmission object) class for `Parolee`, and makes use of the Jackson framework for converting between Java objects and JSON. The project is a multi-module project comprising two modules:

- `lab-jpa-parolee-domain-model`. This module implements the Parolee domain model. It also provides converters (subclasses of `StdSerializer` and `StdDeserializer`) for marshalling and unmarshalling instances of the `java.time` classes.
 
- `lab-jpa-parolee-web-service`. This implements the JAX-RS Web service, including the necessary `Application` subclass and resource class (`ParoleeResource`) that handles HTTP requests. In addition it contains the `Parolee` DTO and a mapping class `ParoleeMapper` to convert between `Parolee` domain objects and DTOs. Finally, this project includes an integration test class (`ParoleeWebServiceIT`) that exercises the Web service.

This project should be a useful resource as it illustrates how to use many aspects of the JAX-RS framework and API.

#### (a) Import, Build, Run project

This is a multi-module Maven project, so following the usual process for importing it into your IDE, building, and running (`verify` goal) it.

The POM is configured to build a WAR file, to run an embedded servlet container that hosts the Web service, and to run the integration tests - just as for Lab 03. The integration tests should pass.

-
-
-

#### (b) Reflect on the project

Make sure you understand how the project works. The main difference between this project and the Parolee JAX-RS project from Lab03 is that this project leverages JAX-RS' capability to automate marshalling/unmarshalling to/from different data formats (in this case JSON through MessageBodyReader/Writer implementations that use Jackson). In addition it illustrates the DTO concept.

Points to consider
----------

A (non-exhaustive) list of questions to consider include:


- How do we specify that web methods should produce / consume JSON?
- How do we marshal / unmarshal objects not natively supported by Jackson?
- We want to make some objects *immutable* (e.g. `Movement` and `GeoPosition` in this case). This necessitates the removal of any *setter* methods in those classes. How do we allow Jackson to unmarshal instances of these classes when it usually relies on setter methods?
-  Consider the use of the `Parolee` *DTO* (data transmission object) - a simplified version of the domain `Parolee` class, specifically used for data transfer between client and service. Why do you think this has been done here? More generally, under what circumstances do you think this appropriate? Under what circumstances do you think this is inappropriate or unnecessary? What are the benefits and drawbacks of this approach?
- In this project, identify an example of HATEOAS being employed?
- How do we marshal / unmarshal generically-typed objects?


You may want to refer to this project in the future when working on the project. It shows additional HTTP message processing and how to work with generically-typed objects that are to be marshalled and unmarshalled.


-
-
-
-
-
-
-
-
-
-
-
-

Exercise Two - Add JSON support to the Concert service
----------
Further develop the Concert Web service from Lab03 to allow clients to exchange both JSON-based and Java Serialization representations of `Concert`s. Unlike the supplied Parolee project from Lab03, where JSON was hand-crafted, you want to leverage the JAX-RS framework to automate JSON marshalling and unmarshalling with Jackson.

#### (a) Set up

Spend some time exploring what has been provided. It should be familiar from Lab03. Complete the following steps:

- The implementation of `ConcertResource` included in the `lab-jpa-concert` project is blank.

- Run the Maven `verify` goal. You will see failed tests are reported. Examine the report to determine where the information about the failed tests is found

- Examine the reports for the failed test (the `txt` files are easiest to understand but it is worth looking at the `xml` files as well). Knowing with is missing from the implementation should help you interpret the test reports.

#### (a) Modify the project artefacts

Complete the following steps

- Replace  the empty implementation of `ConcertResource` with your complete `ConcertResource` implementation you developed in Lab03. 

- Add the Jackson dependency to the project's POM file. The artefact you need is RESTEasy's  `resteasy-jckson2-provider`. See the POM for `lab-jpa-parolee-web-service` from Exercise One - it necessarily includes the dependency.

- Modify the `@Produces` / `@Consumes` annotations in the `ConcertResource` class to add the JSON MIME type.

- Identify any `Concert` properties which may not be correctly serialized by default using Jackson. Add any required custom serializers / deserializers to your project, and annotate `Concert`, as required, to allow the use of these classes.

In changing the `@Produces` / `@Consumes` annotations, you can specify them at the class level (rather than on individual methods) if you wish - they then apply to all methods in the class. Also, it's good practice to use MIME typed constants rather than string literals, e.g:

-
-
-

```java
@Produces({
    javax.ws.rs.core.MediaType.APPLICATION_JSON,
    SerializationMessageBodyReaderAndWriter.APPLICATION_JAVA_SERIALIZED_OBJECT
})
public class Foo { }
```

#### (b) Build and run the project

Once you've amended the code, build and run the project. With the  test cases supplied in `ConcertResourceJavaSerializationIT` and `ConcertResourceJsonIT`, the integration tests should demonstrate that the Web service offers both Java serialization and JSON representations of resources. To see the JSON in the HTTP message bodies, configure logging output for the namespace `org.apache.http` to `DEBUG` (see Lab03).

Points to consider
----------

- How does the JAX-RS framework separate the concerns of resource representation from application logic?
- What would you need to do to add support for another data format, e.g. XML?
- What quality attribute is promoted by the way that JAX-RS manages data formats?

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-


Exercise Three - Complete the Concert database application
----------
For this exercise, complete the `lab-jpa-database` project so that the unit tests run successfully.

The project is a simple Maven project using JPA / Hibernate to enable the persistence of `Concert`s and `Performer`s. It contains the following classes:

- Domain classes `Concert` and `Performer`. `Concert` has a unique ID, a title, a date and one `Performer`. `Performer` has a unique ID, a name, image file and `Genre`. A `Performer` can feature in many `Concert`s, hence there's a one-to-many relationship between `Performer` and `Concert`. The relationship is unidirectional, from `Concert` to `Performer`.

- Class `ConcertTest`, a unit test that tests the Hibernate `EntityManagerFactory` and `EntityManager` classes to access the database.
 
- `db-init.sql`, a database initialisation script that includes SQL DDL and DML statements to create tables for `Concert` and `Performer`, and to populate the tables with data. This script is run automatically at the beginning of each unit test.

- `persistence.xml`, containing the definition of the Hibernate *persistence unit* (rules determining which database to connect to; how to connect; whether to automatically generate tables; and which classes to persist in that database). In this case, the persistence unit is set up **not** to automatically generate tables, as we are creating them manually in `db-init.sql`. The `<exclude-unlisted-classes>` tag will force Hibernate to scan your codebase and include all `@Entity`s when set to `false`.

#### (a) Annotate the domain classes
For this task, annotate the `Concert` and `Performer` classes. Important things to consider:

- Both `Concert` and `Performer` are important *entities* that can exist independently of one another.

- The relationship between the two entities is as follows. Each `Concert` only has one performer. Each `Performer` may perform in any number of concerts. The only field in the Java classes describing this relationship is the `performer` field in `Concert`. Use an appropriate annotation here.

- Think about how you might want to *cascade* your persistence. For example, when a new `Concert` is created, with a new `Performer`, one might with to persist both entities at once with only a single `persist` call.

- `Performer`'s `genre` field is an `enum` type. We want to persist this in the database as a `String`. Investigate how we can configure this using the `@Enumerated` annotation.

- The pre-defined database in `db-init.sql` uses `AUTO_INCREMENT` to automatically generate and assign valid IDs to newly persisted entities. To force Hibernate to utilise the `AUTO_INCREMENT` functionality rather than its own generation strategy, we can set the `strategy` property of the `@GeneratedValue` annotation to `GenerationType.IDENTITY`.

#### (b) Run the unit tests
Once you've annotated your classes, simply run the unit tests from your IDE (there is no need to run a Maven goal for this project, as we are not running integration tests which require an active server). The unit tests should pass. If they do not, modify the annotations from task (a) until they do. You should not need to modify anything in the project, other than adding JPA annotations.

Points to consider
----------

- Can you think of any other ways to design a Concert database?
- Are there any benefits / trade-offs with your alternative approach?

#### Use of H2
Class `ConcertTest` configures the H2 database connection to ensure that any changes to the database are persisted to the local file system (the connection could have been configured such that the database exists only in memory, in which case its data would be lost once the JVM running the database  shuts down). Having the data persisted on disk is convenient as H2 includes a console application that you can use to access the database used by the application. Using the H2 Console, you can interact with the database, e.g. to run queries and see their results.

The H2 Console runs in a Web browser. See below for acquiring it.
The default username and password for connecting is `sa` and `sa`.

When configured as described above, there can be at most one connection to the database. If you are using the H2 Console and are connected, you cannot run the test cases. When the program tries to connect to the database, a connection exception will be thrown:

```
Database may be already in use: "Locked by another process".
```

You should first disconnect from the H2 console, by clicking the `Disconnect` button (at the top left of the H2 console). If you have disconnected but the exception is still thrown, you will need to forcefully close any lingering connections. To do this from the H2 Console, having clicked the `Disconnect` button, click on the `Preferences` link in the `Login` screen. Under `Active Sessions` click the `Shutdown` button.

The POM for project `lab-jpa-database` includes a dependency on the H2 library. 

-

#### GitHub Classroom
After you finishes all exercises and push all source code to the Github, please ensure that `GitHub Classroom Workflow` for autograding successfully runs without any errors. 

#### Resources

Useful resources for H2  include the H2 website:

<http://www.h2database.com/html/main.html>

From here, you can download the H2 Console for your own machines. The website also has useful information, e.g. the SQL grammar for H2.
