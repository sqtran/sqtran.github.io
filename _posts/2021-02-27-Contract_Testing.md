---
layout: single
title: Contract Testing
date: 2021-02-27
tags: testing openapi swagger mockito junit
---

## Background
Contracts are formal definitions of an API.  The contract defines what valid requests must look like, and what the responses will be.  It also defines error codes and conditions, and specifies how the API will respond.  Contracts should be treated as a bond between a producer and their consumer(s).  If a producer completely satisifes their end of the contract, the consumer does not - and should not - care about the the producer's implementation details.

### OpenAPI
Although a contract can be written in any form, one of the most popular formats today is `OpenAPI` - formerly known as `Swagger` API.  The `Swagger` team saw how important having a universal contract format was, and open-sourced their format to the `OpenAPI` project.

### Tools
On the producer side, developers can leverage tools to get started.  These tools read in an `OpenAPI` Contract and scaffold out the backend infrastructure - requiring the developer to fill in the blanks.  I generally lean away from code-generation because it typically brings a lot of bloat - but, I know there's a time and place for it.

On the consumer side, developers can generate stubs that mock the expected responses, so consumers aren't dependent on having a real producer.  All the requests and responses are defined in the Contract, so consumers can build their services without being blocked.


## Java Example

In the Java world, there are libraries that compare the output of a `Code-First` Contract to an `API-First` contract, and determine if the code satisfies the contract.  This is an appropriate method if you prefer code-generation, but I have my own preferred approach.

`RestAssured` is a library that provides a framework for easily creating `http` calls.  When `JUnit` is used with a mocking framework such as `Mockito`, and is combined with a platform that produces standalone-runnable applications, such as `Quarkus/SpringBoot`, I believe this is a cleaner and elegant solution.

One of RestAssured core functionalities is the configuration of `filters`.  This is where the `OpenAPI` contract is specified.  Once configured, the filter will verify that all requests and responses conform to the OpenAPI specifications.  This is an easy way to validate that your Controllers are behaving as required.


Here's a code sample on how to set up the filter and write a test.


```java

@Test
public void myEndpointTest() {
    OpenApiValidationFilter openAPIFilter =
        new OpenApiValidationFilter("src/test/resources/mycontract.json");

    given()
        .port(8080)
        .filter(openAPIFilter)
    .when()
        .get("/endpoint")
    .then()
        .assertThat().statusCode(200);

}
```

It's worth pointing out that there was no mention about Mockito in the code above.  For a very basic test, there's probably no need to create a mock if your endpoint doesn't need it.  The power of Mockito comes into play when you need to validate that a request was properly built and returned from your controller.

**Do NOT mock the Controller's response!  Again, do NOT mock the Controller's response!!**

Mockito should mock something further downstream, such as a database call, or an external web response.  Your controller should call its service, and its service will consume a mocked response from the downstream system.  You might as well not even write any tests if you're testing your own mocked responses!


Setting all this up is super simple with Maven.  Just include the following in the pom, but swap out the spring-boot test starter if you're not using spring-boot.


```xml
<project>
    ...
    <properties>
        <java.version>11</java.version>
        <!-- The latest restassured (4.3.3) didn't work for me for some reason -->
        <restassured-version>4.2.0</restassured-version>
        <swagger-request-validator-version>2.15.1</swagger-request-validator-version>
    </properties>
    ...

    <dependencies>
        ...
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.atlassian.oai</groupId>
            <artifactId>swagger-request-validator-restassured</artifactId>
            <version>${swagger-request-validator-version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>${rest-assured-version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>json-path</artifactId>
            <version>${restassured-version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>xml-path</artifactId>
            <version>${restassured-version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

That's all you need to do to get started!  What's left is to write more tests that cover the different scenarios your OpenAPI specification defines.  Think about input edge cases, mock downstream system failures, and test the different error responses.  The happy-path scenario is typically the first test people write, but it's the most uninteresting test of them all.