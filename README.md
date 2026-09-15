# Particle41 DevOps Challenge: legacy enterprise app: "Snowman"

Welcome to the Particle41 DevOps Team Challenge.

This challenge is for candidates who want to join the Particle41 DevOps team.

It is designed to assess your level of familiarity with common modern development and operations tools and concepts.

## Summary

We aim to hire software engineers who embrace the DevOps mindset, especially taking an infrastructure-as-code approach to software and infrastructure deployment.

This challenge is designed to evaluate your abilities in the following technologies and concepts:

- Software development (in general), by troubleshooting a legacy "production-like" application. We're using a community project called "Snowman" as the sample application.

- Containers, by containerizing a legacy application and making it safe to run.

- Cloud infrastructure using Terraform/OpenTofu, including networking and compute resources on the cloud
  provider of your choice.

- Documentation, for the devops team (your colleagues) to run the application.

This assessment asks you to take a preexisting legacy application, fix it, containerize it, and deploy it end-to-end to a cloud provider of your choice using Terraform/OpenTofu. There is an extra-credit section at the end for overachievers.

## Documentation is MANDATORY

It is mandatory to include documentation for your repository explaining how to use it.

Imagine that someone with less experience than you will need to clone your repository and deploy your containerized application, or deploy your Terraform/OpenTofu infrastructure.

With that in mind, you must provide all the instructions they will need to do that successfully. These must include any prerequisites for deployment; mention of needed tools and links to their installation pages; how to configure credentials for the tool of your choice; and what commands to run for deploying your code.

_We want to see your ability to properly document and communicate about your work with the team._

- Update the `README.md` at the root directory of your project, with instructions for the team to deploy the projects you created. Include any notes that you might add to the README if this were a real project.

## AI usage policy

AI usage is actually a prerequisite to complete this challenge. This reflects current practice on the job, and makes this challenge a close similarity of a real task you'll like encounter on the job. The intention is to make the challenge reflect how we use AI day-to-day on the team.

This shapes the challenge and the way we will grade it, completely.

The first principle is: we want to be sure that you know what you're doing. The challenge is designed to prevent the mindless one-shot attempt i.e. simply copying the CHALLENGE.md into an agent prompt and asking it to solve it. That is explicitly below the baseline in our grading process.

Of course assume we'll use AI for grading your results. We've also used it to build the challenge. The acceptance criteria for the solution are clearly defined below being derived from this principle.

## The original Snowman app

The original Snowman app we're using as the starting point application for the challenge is a surprisingly good fit for us, because it reflects a task you may encounter live on the job: patch a legacy enterprise application and bring it to the cloud, ensure it runs smoothly and safely.

We've made little to no modification to the original in this respect. You can find the original `README.md` text below.

**Original README.md:**

---

<!-- markdownlint-disable MD025 MD051 -->

# Enterprise Application : Snowman

[![Build Status](https://travis-ci.org/colinbut/enterprise-application.svg?branch=master)](https://travis-ci.org/colinbut/enterprise-application)

## Table of Contents

- [Preamble](#preamble)
- [Pre-requisites](#prerequisites)
- [Software Architecture](#architecture)
- [Database Design](#db-design)
- [Data Access](#data-access)
  - [Java - JDBC](#jdbc)
  - [Spring - JdbcTemplate](#jdbctemplate)
  - [Object Relational Mapping](#orm)
- [Messaging](#messaging)
- [Caching](#caching)
- [Uber Jar](#uber-jar)
- [Embedded Jetty](#embedded-jetty)
- [Database Migration](#database-migration)
- [Scalability](#scalability)
- [Clustering](#clustering)
- [Resiliency](#resiliency)

This project aims to provide a skeleton example of a common traditional (perhaps now viewed as legacy) enterprise
like application.

This project's concepts are an extension of another demo project Sales-Order-System where it showcase more advanced common features that
is normally seen in a traditional big-scale enterprise application.

_Note, this project is not a complete application and i have no intention of making it complete, it only serves the purpose of providing a variety of specific
demonstrations only._

### <a name="preamble"></a>Preamble

Snowman is an fictional enterprise scale/ready employee management system (EMS). This project uses Snowman to
demonstrate the characteristics of a stereotypical backend enterprise application.

![Image of a Snowman](etc/snowman.jpeg)

Snowman exposes its functionality via REST (-like) endpoints. Essentially this is a
backend web service.

### <a name="prerequisites"></a>Pre - Requisites

- Java JDK 7
- Maven
- MySQL

1. Start up MySQL Server
2. run `run.sh` script

### <a name="architecture"></a>Software Architecture

Instead of using a Layered Architecture where you commonly have 3 layers with one directional flow, this project showcase
a Hexagonal Architecture (Ports and Adapters). The core domain comprises of the main business logic would be the inner and
the application infrastructure (Database, Message Queues, REST endpoints) would be the outer layers.

![Image of a Hexagonal Architecture](etc/HexagonalArchitecture.png)

This is how the system components fit together:

![Image of System Components](etc/SystemComponents.png)

### <a name="db-design"></a>Database Design

![Image of ER diagram](etc/entity-relationship.png)

Database table structure:

![Image of Table Diagram](etc/relation-table-schema.png)

### <a name="data-access"></a>Data Access

Rather than having a logical Data Access Layer within a 3 layered architecture, in a hexagonal architecture data access are
in an outer layer.

#### <a name="jdbc"></a>Java - JDBC

One way of accessing the db from the application is using raw JDBC. This is quite low level. See ApplicationInfoDaoImpl.java

#### <a name="spring-jdbctemplate"></a>Spring - JDBC

The application framework in Spring provides a level of abstraction of data access by giving the usage of JdbcTemplate. This
class from Spring implements the GoF Template design pattern. By using this class, it relieves away a lot of the 'pain' of writing
so called 'boilerplate' code in setting up db connections, exception handling etc. See UserDaoImpl.java

#### <a name="orm"></a>Object-Relational-Mapping(ORM)

Object Relational Mapping (ORM) is the mapping of relational database tables to objects and vice-versa.
We use JPA standard along the the Hibernate implementation to achieve this. Model objects (Entities) are part of thus 'anemic'
data model. The DAOs encapsulates Hibernate EntityManager to do the basic CRUD operations.

### <a name="messaging"></a>Messaging

Communication to 3rd party external systems is primarily achieved via messaging using JMS & ActiveMQ as the
implementing underlying Messaging System (Message Brokers, Message Queues, Topics).

Apache ActiveMQ is JMS compliant belonging to open source apache foundation. One of the commonly used MQ out there.

### <a name="caching"></a>Caching

[TBD]

### <a name="uber-jar"></a>Uber Jar

As part of the deployment process, Application is packaged up in a "uber" runnable executable jar.
This jar contains all dependencies copied in. And can be easily run from a command line with:

```java
java -jar target/Snowman.jar
```

This whole process is achieved by the [Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/)

### <a name="embedded-jetty"></a>Embedded Jetty

Rather than deploying the application in either a full blown JavaEE application server (JBoss/Wildfly, Websphere, Weblogic, Geronimo etc)
or a Web/JSP Container (Tomcat, Jetty, UnderTow), we embed a HTTP listener into the application in the form of
"Embedded Jetty".

All dependencies are bundled/packaged together in an Uber jar file so it works. This means it is therefore
wasn't necessary required to be provided with specific JavaEE dependencies from the JavaEE platform.

### <a name="database-migration"></a>Database Migration

Database updates are implemented using patches via changesets with a database migration tool in [liquibase](http://www.liquibase.org/)
to execute them in order to 'patch' up the database.

To run, execute the maven liquibase plugin by...

Running update patches:

```xml
mvn liquibase:update
```

And to rollback those particular patches:

```xml
mvn liquibase:rollback -Dliquibase.rollbackCount=1
```

### <a name="scalability"></a>Scalability

1. Horizonantal Scalability
2. Vertical Scalability

#### Horizontal Scalability a.k.a "Scaling Out"

To support horizontal scalability you run multiple instances of the same application.

You can do this manually like:

```java
java -jar -Dport=[port number] target/Snowman.jar
```

where port number is an unused port

#### Veritcal Scalability a.k.a "Scaling Up"

This option is limited kind of a way. You can opt to run this application in a higher spec
machine. Alternatively, you can give more memory to the JVM by tuning the min and max
parameters like the following:

```java
java -jar -Xms256m -Xmx2048m target/Snowman.jar
```

### <a name="clustering"></a>Clustering

No clustering options. This application does not run in a managed cluster. Horizontal scalability
was chosen as demonstration in favour of running Cluster Servers to achieve Clustering.

See [Scalability](#scalability) for more info.

### <a name="resiliency"></a>Resiliency / Fault Tolerance

1. Failover and Recovery
2. Disaster and Recovery

#### Failover and Recovery

No F&R is supported.

You can failover the application easily (assuming you are running
multiple instances to mimic multiple nodes servers). Just do a:

```bash
kill -9 [pid]
```

where pid is the process id of the particular application node server.

But there is no Recovery. Because no Clustering options. Refer to [Clustering](#clustering) section.

#### Disaster and Recovery

Although not specifically supported here, you can achieve this also. Manual intervention is required
though in order to start up the application (or group of applications) together.

To do Disaster Recovery a.k.a DR, you need to have the concept of a "Site". Commonly, use
multiple sites. For simplicity, assume 2 sites - 1 Primary Site and the other is Standby Site.
Both sites would house the set of applications but only one site is up.

If Primary site considered fail (i.e. multiple failovers of one or more applications (or application components
if it is a Distributed Component Architecture)), then the Primary site should shutdown and thus Standby site
would be required to be started up.

## Author

Colin But.
