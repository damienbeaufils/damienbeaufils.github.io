---
title: "How to properly encrypt data using JPA converters and Spring Data JPA"
date: "2017-07-21"
draft: false
tags: [Java, Spring Data JPA, Spring Boot, encryption, converter, unit testing, integration testing]
featured: true
---

**TL;DR: all code is directly available on GitHub: [https://github.com/damienbeaufils/spring-data-jpa-encryption-example](https://github.com/damienbeaufils/spring-data-jpa-encryption-example)**

Each time I want to encrypt entity fields values with JPA converters, I end up reading [this blog post](https://www.thoughts-on-java.org/how-to-use-jpa-type-converter-to/). This example is clear and functional, but **has no unit or integration tests**, and I thought **the code could be more decoupled to avoid duplication when having multiple converters**.

So I wrote an example using Spring Boot and Spring Data JPA, with a [`User`](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/blob/master/src/main/java/com/example/spring/data/jpa/encryption/domain/User.java) entity which have different fields: `id` (a `Long`), `firstName` (a `String`), `lastName` (a `String`), `email` (a `String`), `birthDate` (a `LocalDate`) and `creationDate` (a `LocalDateTime`). All fields except `id` are encrypted in database using AES algorithm.

Encryption is enabled on fields using different JPA converters: [`StringCryptoConverter`](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/blob/master/src/main/java/com/example/spring/data/jpa/encryption/converters/StringCryptoConverter.java), [`LocalDateCryptoConverter`](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/blob/master/src/main/java/com/example/spring/data/jpa/encryption/converters/LocalDateCryptoConverter.java) and [`LocalDateTimeCryptoConverter`](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/blob/master/src/main/java/com/example/spring/data/jpa/encryption/converters/LocalDateTimeCryptoConverter.java). **This is verified with [`UserRepositoryTest`](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/blob/master/src/test/java/com/example/spring/data/jpa/encryption/domain/UserRepositoryTest.java) integration test, and [all converters are unit tested](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/tree/master/src/test/java/com/example/spring/data/jpa/encryption/converters).**

Encryption key is empty by default (see `example.database.encryption.key` configuration key in [`application.yml`](https://github.com/damienbeaufils/spring-data-jpa-encryption-example/blob/master/src/main/resources/application.yml)). You have to provide an encryption key in configuration or specify it in options when running application.

_Feel free to fork & enjoy!_
