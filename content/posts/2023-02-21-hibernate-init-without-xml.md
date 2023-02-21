+++ 
draft = false
date = 2023-02-21T17:16:00+02:00
title = "Hibernate Initialisation without XML"
description = ""
slug = "2023-02-21-hibernate-init-without-xml"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["hibernate", "java"]
categories = []
externalLink = ""
series = []
+++
# Hibernate Initialisation without XML

While working on some R&D last weekend, I was surprised when every introduction to Hibernate started with creating a `persistence.xml` file.

Ultimately this is probably because many developers implicitly
assume that ones JPA persistence unit will ultimately be managed
by an application server. But for small containerised
applications without application server, having multiple
mechanisms to configure the application - i.e. `persistence.xml`
and whatever native mechanism one prefers - is extremely unwieldy.
As an example, I have seen some really convoluted approaches for referencing environment variables to pull in database credentials.

I was surprised that even web-searches like "Hibernate without
persistence.xml" turned out quite unhelpful.
So I finally ended up looking into [integration tests of Hibernate](https://github.com/hibernate/hibernate-reactive/blob/1.1.9/hibernate-reactive-core/src/test/java/org/hibernate/reactive/BaseReactiveTest.java) to find out how this might work. And to top things off, it is extremely straightforward to do, too.

In this code, configuration is pulled from a [Vert.x configuration object](https://vertx.io/docs/vertx-config/java/).

```kotlin
    lateinit var jdbcConfig: JdbcConfig

    // can't be `by lazy` because it needs to be `suspend`
    private suspend fun initializeDatabaseConfiguration() {
        if (Boot::jdbcConfig.isInitialized) {
            return
        }

        val config = configRetriever.config.await()
        val dbConfig = config.getJsonObject("database")
        val url = dbConfig.getString("url")
        val username = dbConfig.getString("username")
        val password = dbConfig.getString("password")

        jdbcConfig = JdbcConfig("jdbc:$url", username, password)
    }
```

Now that this information is there, Hibernate can be set up.
Here I'm looking into [Hibernate Reactive](https://hibernate.org/reactive/), so some custom configuration is required:

```kotlin
    private fun initializeJPA(pluginManager: PluginManager): SessionFactory {
        val configuration = Configuration()
        configuration
            .setProperty(
                Settings.JPA_PERSISTENCE_PROVIDER,
                "org.hibernate.reactive.provider.ReactivePersistenceProvider"
            )
            .setProperty(Settings.DIALECT, "org.hibernate.dialect.PostgreSQL10Dialect")
            .setProperty(Settings.URL, jdbcConfig.url)
            .setProperty(Settings.USER, jdbcConfig.username)
            .setProperty(Settings.PASS, jdbcConfig.password)
```

Next, one will need to add the entity classes to the configuration, which will probably work a bit differently for each application.
In this case, I'm using [PF4J](https://pf4j.org/) and pulling in Hibernate entities from an [ExtensionPoint](https://pf4j.org/doc/extensions.html).

```kotlin
        for (extension in pluginManager.getExtensions(HibernateConfiguration::class.java)) {
            for (clz in extension.annotatedClasses()) {
                configuration.addAnnotatedClass(clz)
            }
        }
```

Since PF4J creates separate `ClassLoader`s for each plugin,
Hibernate needs to be made aware of these through a custom
`ClassLoaderService` instance. After that, the configuration
is ready to be used and the `SessionFactory` can be instantiated:

```kotlin
        val allClassLoaders = mutableListOf(this.javaClass.classLoader)
            .also { it.addAll(pluginManager.plugins.map { it.pluginClassLoader }) }
        val classLoaderService = ClassLoaderServiceImpl(allClassLoaders, TcclLookupPrecedence.BEFORE)
        val serviceRegistry = ReactiveServiceRegistryBuilder().addService(
            ClassLoaderService::class.java, classLoaderService
        )
            .applySettings(configuration.properties)
            .build()
        val jpaSessionFactory = configuration.buildSessionFactory(serviceRegistry)
        return jpaSessionFactory.unwrap(SessionFactory::class.java)
    }
```

That's all for now. I'll probably write a separate article about combining Vert.x, PF4J, Hibernate Reactive and [Kodein](https://kosi-libs.org/kodein/) soon.
