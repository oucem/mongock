<p align="center">
    <img src="https://raw.githubusercontent.com/cloudyrock/mongock/master/misc/logo.png" width="200" />
</p>

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.cloudyrock.mongock/mongock/badge.png)](https://search.maven.org/artifact/com.github.cloudyrock.mongock/mongock)
[![Build Status](https://travis-ci.org/cloudyrock/mongock.svg?branch=master)](https://travis-ci.org/cloudyrock/mongock)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=com.github.cloudyrock.mongock&metric=bugs)](https://sonarcloud.io/component_measures?id=com.github.cloudyrock.mongock&metric=bugs)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=com.github.cloudyrock.mongock&metric=vulnerabilities)](https://sonarcloud.io/component_measures?id=com.github.cloudyrock.mongock&metric=vulnerabilities)
[![Hex.pm](https://img.shields.io/hexpm/l/plug.svg)](https://github.com/dieppa/mongock/blob/master/LICENSE)

## LAST NEWS :bangbang::bangbang::collision::collision:

> **ROADMAP** for next features. We are designing the roadmap for the next features and we think that it's crucial that we understand the user's needs and preferences. For this we have created a quick survey to understand what is important for you and also to allow you to give us any idea or need you have and we haven't thought about. [Mongock roadmap survey](https://forms.gle/uk9jmtKvxJzi5Bda7)

> **3.2.4 is released** 

# Mongock: MongoDB version control tool for Java

**Mongock** is a java MongoDB tool for tracking, managing and applying database schema changes accross all your environments based on a coding approach.  

## Table of contents

  * [Why Mongock](#why-mongock)
  * [Sample projects](#sample-projects)
  * [Contributing](#contributing)
  * [Add a dependency](#add-a-dependency)
     * [With Maven](#with-maven)
     * [With Gradle](#with-gradle)
  * [Usage with Spring...Mongock as a Bean](#usage-with-springmongock-as-a-bean)
  * [Usage with SpringBoot...When you need to inject your own dependencies](#usage-with-springbootwhen-you-need-to-inject-your-own-dependencies)
  * [Standalone usage](#standalone-usage)
  * [Creating change logs](#creating-change-logs)
     * [@ChangeLog](#changelog)
     * [@ChangeSet](#changeset)
        * [Annotation parameters:](#annotation-parameters)
        * [Defining ChangeSet methods](#defining-changeset-methods)
        * [Defining ChangeSet methods with versions()](#defining-changeset-methods-with-versions)
  * [Injecting custom dependencies to change logs](#injecting-custom-dependencies-to-change-logs)
  * [Using Spring profiles](#using-spring-profiles)
     * [Enabling @Profile annotation (option)](#enabling-profile-annotation-option)
  * [Adding metadata](#adding-metadata)
  * [Configuring Lock](#configuring-lock)
  * [Known issues](#known-issues)
     * [Mongo java driver conflicts](#mongo-java-driver-conflicts)
  * [Mongo transaction limitations](#mongo-transaction-limitations)
  * [Code of conduct](#code-of-conduct)
  * [LICENSE](#license)
  
## Why Mongock
There are several good reasons to use Mongock in your project. Here we give you some of them:

* Solid solution which really works.
* **Works well with sharded collections**: Unlike other similar projects using javascript, which requires `db.eval()`. [Documentation](https://docs.mongodb.com/manual/reference/method/db.eval/#sharded-data).
* Works with [Mongo Atlas](https://www.mongodb.com/cloud/atlas).
* Distributed solution with solid locking mechanism.
* We are very responsive for community license and maximum 2 business days for professional license.
* Well maintained and regularly updated.
* Used by several tech companies in different industries.
* Can be used together with most, if not all, frameworks.
* Provides great integration for Spring, allowing you to inject any dependency you want to your changelog method.
* We walk with you to production. Get more information about our support model at dev@cloudyrock.io

## Sample projects
In [here](https://github.com/cloudyrock/mongock-samples) you can find some sample projects that show you how to use Mongock.

## Contributing
If you would like to contribute to Mongock project, please read [how to contribute](././community/CONTRIBUTING.md) for details on our collaboration process and standards.



## Add a dependency

Mongock can be used standalone or with Spring. When Using `mongock-spring` you don't need to import `mongock-core`, 
as it's already imported out of the box.

#### With Maven
```xml
<!-- To use standalone (i.e., w/o Spring ) -->
<dependency>
  <groupId>com.github.cloudyrock.mongock</groupId>
  <artifactId>mongock-core</artifactId>
  <version>3.2.4</version>
</dependency>

<!-- To use with Spring-->
<dependency>
  <groupId>com.github.cloudyrock.mongock</groupId>
  <artifactId>mongock-spring</artifactId>
  <version>3.2.4</version>
</dependency>


```
#### With Gradle
```groovy
compile 'org.javassist:javassist:3.18.2-GA'          // workround for ${javassist.version} placeholder issue*
compile 'com.github.cloudyrock.mongock:mongock-core:3.2.4'    // standalone
compile 'com.github.cloudyrock.mongock:mongock-spring:3.2.4'  // with Spring (in addition to mongock-core)
```

## Usage with Spring...Mongock as a Bean

You need to instantiate mongock object and provide some configuration.
If you use Spring, mongock can be instantiated as a singleton bean in the Spring context. 
In this case the migration process will be executed automatically on startup.

```java
@Bean
public SpringMongock mongock() {
  MongoClient mongoclient = new MongoClient(new MongoClientURI("yourDbName", yourMongoClientBuilder));
  return new SpringMongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setLockQuickConfig()
      .build();
}
```

## Usage with SpringBoot...When you need to inject your own dependencies

The main benefit of using SpringBoot integration is that it provides a totally flexible way to inject dependencies,
so you can inject any object to your change logs by using SpringBoot ApplicationContext.

In order to use this feature you need to instantiate the SpringBoot mongock class and provide the required configuration. 
Mongock will run as an [ApplicationRunner][ApplicationRunner] within SpringBoot.
In terms of execution, it will be very similar to the standard Spring implementation, the key difference is that ApplicationRunner beans run *after* (as opposed to during) the context is fully initialized. 

>**Note:** Using this implementation means you need all the dependencies in your changelogs(parameters in methods annotated with ```@ChangeSet```) declared as Spring beans.

>**Note:** The dependencies injected by the ApplicationContext (other than [MongoTemplate][MongoTemplate], [MongoDatabase][MongoDatabase] and [DB][DB]) won't be covered by the lock. This means
that if you are accessing to Mongo through a different mechanism to the ones mentioned, the lock synchronization is not guaranteed as Mongock only ensures synchronization when Mongo is accessed via either [MongoTemplate][MongoTemplate], [MongoDatabase][MongoDatabase] or [DB][DB]. 
For more information, please consult the [lock section](#configuring-lock)

```java
@Bean
public SpringBootMongock mongock(ApplicationContext springContext, MongoClient mongoClient) {
  return new SpringBootMongockBuilder(mongoClient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setApplicationContext(springContext) 
      .setLockQuickConfig()
      .build();
}
```

## Standalone usage
Using mongock standalone.

```java

  MongoClient mongoclient = new MongoClient(new MongoClientURI("yourDbName", yourMongoClientBuilder));
  Mongock runner=  new MongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setLockQuickConfig()
      .build();
  runner.execute();         //  ------> starts migration changesets
```

Above examples provide minimal configuration. The various `Mongock` builders provide some other possibilities (setters) 
to make the tool more flexible:

```java
builder.setChangelogCollectionName(logColName);   // default is dbchangelog, collection with applied change sets
builder.setLockCollectionName(lockColName);       // default is mongocklock, collection used during migration process
builder.setEnabled(shouldBeEnabled);              // default is true, migration won't start if set to false
```

[More about URI](http://mongodb.github.io/mongo-java-driver/3.5/javadoc/)


## Creating change logs

`ChangeLog` contains bunch of `ChangeSet`s. `ChangeSet` is a single task (set of instructions made on a database). In 
other words `ChangeLog` is a class annotated with `@ChangeLog` and containing methods annotated with `@ChangeSet`.

```java 
package com.example.yourapp.changelogs;

@ChangeLog
public class DatabaseChangelog {
  
  @ChangeSet(order = "001", id = "someChangeId", author = "testAuthor")
  public void importantWorkToDo(DB db){
     // task implementation
  }


}
```
### @ChangeLog

Class with change sets must be annotated by `@ChangeLog`. There can be more than one change log class but in that 
case `order` argument should be provided:

```java
@ChangeLog(order = "001")
public class DatabaseChangelog {
  //...
}
```
ChangeLogs are sorted alphabetically by `order` argument and changesets are applied due to this order.

### @ChangeSet

Method annotated by @ChangeSet is taken and applied to the database. History of applied change sets is stored in a 
collection called `dbchangelog` (by default) in your MongoDB

#### Annotation parameters:

`order` - string for sorting change sets in one changelog. Sorting in alphabetical order, ascending. It can be a number, 
a date etc.

`id` - name of a change set, **must be unique** for all change logs in a database

`author` - author of a change set

`runAlways` - _[optional, default: false]_ changeset will always be executed but only first execution event will be 
stored in dbchangelog collection

`version` - _[optional, default: "0"]_ defines a version on which this changeset is relate to. E.g. "0.1" means, this changeset should be applied to schema version 0.1 of your MongoDB. See [Defining ChangeSet methods with versions](#defining-changeset-methods-with-versions) for more information.

#### Defining ChangeSet methods
Method annotated by `@ChangeSet` can have one of the following definition:

```java
@ChangeSet(order = "001", id = "someChangeWithoutArgs", author = "testAuthor")
public void someChange1() {
   // method without arguments can do some non-db changes
}

@ChangeSet(order = "002", id = "someChangeWithMongoDatabase", author = "testAuthor")
public void someChange2(MongoDatabase db) {
  // type: com.mongodb.client.MongoDatabase : original MongoDB driver v. 3.x, operations allowed by driver are possible
  // example: 
  MongoCollection<Document> mycollection = db.getCollection("mycollection");
  Document doc = new Document("testName", "example").append("test", "1");
  mycollection.insertOne(doc);
}

@ChangeSet(order = "005", id = "someChangeWithSpringDataTemplate", author = "testAuthor")
public void someChange5(MongoTemplate mongoTemplate) {
  // type: org.springframework.data.mongodb.core.MongoTemplate
  // Spring Data integration allows using MongoTemplate in the ChangeSet
  // example:
  mongoTemplate.save(myEntity);
}

@ChangeSet(order = "006", id = "someChangeWithSpringDataTemplate", author = "testAuthor")
public void someChange6(MongoTemplate mongoTemplate, Environment environment) {
  // type: org.springframework.data.mongodb.core.MongoTemplate
  // type: org.springframework.core.env.Environment
  // Spring Data integration allows using MongoTemplate and Environment in the ChangeSet
}
```

#### Defining ChangeSet methods with versions
Method annotated by `@ChangeSet` have also the possibility to contain a version. 
This is a useful feature from a consultancy point of view. The more descriptive scenario is where a software provider has 
several customers  who he provides his software to. The clients may be using different versions of the software at the same time. 
So when he install the product in a customer, the changeSets need to be applied depending on the product version. 
With this solution, he can tag every changeSet with his product version and will tell mongock which version range to apply.

```java
@ChangeSet(order = "001", id = "someChangeToVersionOne", author = "testAuthor", version = "1")
public void someChange1(MongoDatabase db) {
}

@ChangeSet(order = "002", id = "someChangeToVersionOneDotOne", author = "testAuthor", version = "1.1")
public void someChange2(MongoDatabase db) {
}

@ChangeSet(order = "003", id = "someChangeToVersionTwoDotFiveDotOne", author = "testAuthor", systemVersion = "2.5.1")
public void someChange3(MongoDatabase db) {
}

@ChangeSet(order = "004", id = "someChangeToVersionTwoDotFiveDotFive", author = "testAuthor", systemVersion = "2.5.5")
public void someChange5(MongoDatabase db) {
}

@ChangeSet(order = "005", id = "someChangeToVersionTwoDotSix", author = "testAuthor", systemVersion = "2.6")
public void someChange6(MongoDatabase db) {
}
```

With specifying versions you are able to upgrade to specific versions:

```java

  MongoClient mongoclient = new MongoClient(new MongoClientURI("yourDbName", yourMongoClientBuilder));
  Mongock runner=  new MongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setLockQuickConfig()
      .setStartSystemVersion("1")
      .setEndSystemVersion("2.5.5")
      .build();
  runner.execute();         //  ------> starts migration changesets from systemVersion 1 to 2.5.5
```

This example will execute `ChangeSet` 1, 2 and 3, 
because the specified systemVersion in the changeset should be greater equals the `startSystemVersion` and lower than `endSystemVersion`.


## Injecting custom dependencies to change logs
Right now this is possible by using SpringBoot Application Context. 
As explained in section [Usage with Spring...Mongock as a Bean](#usage-with-springmongock-as-a-bean), once you have injected the Spring ApplicationContext, you can use your beans in Mongock changeSet methods via method parameter. Don't use @autowired annotation.

For example having a springdata repository  'PersonRepository' in your project, that you wish to use in your changeSet, you can use it like follow

```java
@ChangeLog(order = "1")
@Profile("test")
public class ChangelogForTestEnv{
  @ChangeSet(author = "testuser", id = "myTestChangest", order = "01")
  public void testingEnvOnly(MongoTemplate template, PersonRepository repository){
    List<Person> allPersons = repository.findAll();
  } 
}
```

Notice that you shouldn't use the repository to write to Mongo, as it won't be covered by the lock.(this feature which allows you to use your own repositories freely in yout changeSet methods will be availble in future releases)

## Using Spring profiles
     
**mongock** accepts Spring's `org.springframework.context.annotation.Profile` annotation. If a change log or change set 
class is annotated  with `@Profile`, then it is activated for current application profiles.

> Mongock will soon support the new Profile expression approach from Spring 5 in 

_Example 1_: annotated change set will be invoked for a `dev` profile
```java
@Profile("dev")
@ChangeSet(author = "testuser", id = "myDevChangest", order = "01")
public void devEnvOnly(MongoDatabase db){
  // ...
}
```
_Example 2_: all change sets in a changelog will be invoked for a `test` profile
```java
@ChangeLog(order = "1")
@Profile("test")
public class ChangelogForTestEnv{
  @ChangeSet(author = "testuser", id = "myTestChangest", order = "01")
  public void testingEnvOnly(MongoDatabase db){
    // ...
  } 
}
```

### Enabling @Profile annotation (option)
      
To enable the `@Profile` integration, please inject `org.springframework.core.env.Environment` to your runner.

```java      
@Bean 
@Autowired
public SpringMongock mongock(Environment environment) {
  SpringMongock runner = new SpringMongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setSpringEnvironment(environment)
      .setLockQuickConfig()
      .build();

  //... etc
}
```

## Adding metadata
Sometimes there is the need of adding some extra information to the mongockChangeLog documents at execution time. This 
is address by Mongock allowing to set a Map object to the MongockBuilder(core, Spring and Springboot) with the metadata which will be added later 
to each mongockChangeLog document inserted in the Mongock 'transaction'.
```java      
Map<String, Object> metadata = new HashMap<>();
metadata.put("string_key", "string_value");

Mongock runner = new MongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
    .setLockQuickConfig()
    .withMetadata(metadata)
    .build();

  //... etc
}
```

## Configuring Lock 
In order to execute the changelogs, mongock needs to manage the lock to ensure only one instance executes a changelog at a time.
By default the lock is reserved 24 hours and, in case the lock is held by another mongock instance, will ignore the execution
and no exception will be sent, unless the parameter throwExceptionIfCannotObtainLock is set to true.

There are 3 parameters to configure:

`lockAcquiredForMinutes` - Number of minutes mongock will acquire the lock for. It will refresh the lock when is close 
to be expired anyway. 

`maxTries` - Max tries when the lock is held by another mongock instance.

`maxWaitingForLockMinutes` - Max minutes mongock will wait for the lock in every try. 

To configure these parameters there are two methods: setLockConfig and `setLockConfig` and `setLockQuickConfig`. Both
 will set the parameter throwExceptionIfCannotObtainLock to true.
 ```java      
 @Bean @Autowired
 public Mongock mongock(Environment environment) {
   Mongock runner = new mongock(uri);
   runner.setLockConfig(5, 6, 3);
 }
 ```
 or quick config with 3 minutes for lockAcquiredFor, 3 max tries and 4 minutes for maxWaitingForLock
 ```java      
  @Bean @Autowired
  public Mongock mongock(Environment environment) {
    Mongock runner = new mongock(uri);
    runner.setLockQuickConfig();
  }
  ```
 
 

## Known issues

### Mongo java driver conflicts

**mongock** depends on `mongo-java-driver`. If your application has mongo-java-driver dependency too, there could be 
library conflicts in some cases.

**Exception**:
```
com.mongodb.WriteConcernException: { "serverUsed" : "localhost" , 
"err" : "invalid ns to index" , "code" : 10096 , "n" : 0 , 
"connectionId" : 955 , "ok" : 1.0}
```

**Workaround**:

You can exclude mongo-java-driver from **mongock**  and use your dependency only. Maven and gradle examples below:
```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.4.0</version>
</dependency>

<dependency>
  <groupId>com.github.cloudyrock.mongock</groupId>
  <artifactId>mongock-core</artifactId>
  <version>3.2.4</version>
  <exclusions>
    <exclusion>
      <groupId>org.mongodb</groupId>
      <artifactId>mongo-java-driver</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

```gradle
    // build.gradle
    compile "org.mongodb:mongo-java-driver:3.4.0"
    compile("com.github.cloudyrock.mongock:mongock:3.2.4") {
        exclude group: 'org.mongodb', module: 'mongo-java-driver'
    }

```

## Mongo transaction limitations

Due to Mongo limitations, there is no way to provide atomicity at ChangelogSet level. So a Changelog could need 
more than one execution to be finished, as any interruption could happen, leaving the changelog in a inconsistent state.
If that happens, the next time mongock is executed it will try to finish the changelog execution, but it could already be 
half executed.

For this reason, the developer in charge of the changelog's design, should make sure that:
 
- **Changelog is idempotent**: As changelog can be interrupted at any time, it will need to be executed again. 
- **Changelog is Backward compatible (If high availability is required)**: While the migration process is taking place, 
the old version of the software is still running. During this time could happen(and probably will) that the old version 
of the software is dealing with the new version of the data. Could even happen that the data is a mix between old and 
new version. This means the software must still work regardless of the status of the database. In case the developer is aware of 
this and still decides to provide a non-backward-compatible changeSet, he should know it's a detriment to high 
availability.
- **Changelog reduces its execution time in every iteration**: This is harder to explain. As said, a changelog can be 
interrupted at any time. This means an specific changelog needs to be re-run. In the undesired scenario where the 
changelog's execution time is grater than the interruption time(could be Kubernetes initial delay), that changelog won't 
be ever finished. So the changelog needs to be developed in such a way that every iteration reduces its execution time, 
so eventually, after some iterations, the changelog finished. 
- **Changelog's execution time is shorter than interruption time**: In case the previous condition cannot be ensured, 
could be enough if the changelog's execution time is shorter than the interruption time. This is not ideal as the 
execution time depends on the machine, but in most case could be enough.

## Code of conduct
Please read the [code of conduct](././community/CODE_OF_CONDUCT.md) for details on our code of conduct.

## LICENSE
Mongock propject is licensed under the [Apache License Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html). See the [LICENSE](./LICENSE.md) file for details


[ApplicationRunner]: https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html
[MongoTemplate]: https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html
[MongoDatabase]: http://mongodb.github.io/mongo-java-driver/3.6/javadoc/?com/mongodb/client/MongoDatabase.html
[DB]: http://mongodb.github.io/mongo-java-driver/3.6/javadoc/?com/mongodb/DB.html
