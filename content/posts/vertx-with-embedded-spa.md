+++ 
draft = false
date = 2021-04-05T13:31:04+02:00
title = "Deploy a Vert.x app with an embedded SPA"
description = ""
slug = "2021-04-04-deploy-vertx-with-embedded-spa"
authors = []
tags = ["Vert.x", "Vue 3", "Kotlin", "Gradle"]
categories = []
externalLink = ""
series = ["Vert.x basics"]
+++
# Deploy a Vert.x application with an embedded SPA

As a developer I have come to enjoy the versatility and power of the [Vert.x](https://vertx.io/) platform. And although it contains many utilities for server-side rendering, called [Vert.x Web](https://vertx.io/docs/vertx-web/java/), there are situations where you might want to use a single-page application (SPA) instead.

Concrete reasons for and against SPAs are best kept separate from this. One thing, however, that makes SPAs cumbersome for small teams is having to deploy them separately from the API they will talk to, so having a setup that allows for deploying everything in one go.

As such, here are a few tips for setting up your project so that the SPA will be deployed along with the rest of your application as static resources: a single `.jar` file that includes everything required to run the application.


### Setting up

To get the basic directory structure going, it's easiest to go to [`start.vertx.io`](https://start.vertx.io/). As the language of choice I chose Kotlin. Also, we'll use the Gradle build system.

After extracting the generated archive, you'll have the following file and directory structure (only listing those files that will be of interest during the rest of this article):

```
build.gradle.kts        
src/
src/main/
src/main/kotlin/com/example/starter/MainVerticle.kt  
src/test/
gradlew.bat             
gradlew                 
```

The infrastructure for building and running the JVM application is prepared now and `./gradlew run` will start it up. Feel free to try this and point your browser to `http://localhost:8888/`. There you should see "Hello from Vert.x!" Great!


### Adding the SPA

The SPA will be built separately from the JVM-based projects. You might even want to put it into a separate repository (using Git submodules) and it usually contains the tests as part of its folder structure. That is why we'll place it into a separate folder called `src/webapp`, next to the `src/main` and `src/test` directories.

To easily bootstrap this part, open a shell in the `src` directory and run `npm init @vitejs/app webapp`. Choose a flavour of your choice (I usually prefer `vue-ts`) and witness the `webapp` directory being created.
If you want to test whether everything worked, you can follow the instructions shown on the screen:

```
cd webapp
npm install
npm run dev
```

And you should finally see
```
> webapp@0.0.0 dev /tmp/src/webapp
> vite

Pre-bundling dependencies:
  vue
(this will be run only when your dependencies or config have changed)

  vite v2.1.5 dev server running at:

  > Local:    http://localhost:3000/
  > Network:  http://[YOUR-NETWORK-IP]:3000/

  ready in 205ms.
```

Open your browser at `http://localhost:3000/` and after confirming that this does work indeed, we can stop the web server again. This server automatically hot-reloads any changes made on the fly will be used during development of the SPA.

## Building the SPA along with the Vert.x application

Now for the tasty bits: what we ultimately want is that the application doesn't just say "Hello from Vert.x!" anymore but instead serves up the compiled SPA.
For this to work, three steps are necessary:

1. the SPA needs to be built along the JVM code
2. the generated HTML & JS need to be bundled into the `.jar` file
3. the Vert.x application needs to serve up the generated assets

So, let's get to work!

### Building the SPA with Gradle

In order to integrate the NPM build process into the existing configuration, we'll use a [Gradle Plugin for integration NodeJS](https://github.com/node-gradle/gradle-node-plugin). Add it to the `plugins` section of your `build.gradle.kts`:

```kotlin
plugins {
  // ...
  id("com.github.node-gradle.node") version "3.0.1"
}
```

Now we can configure the build cycle and integrate the build process:

```kotlin
node {
  nodeProjectDir.set(file("src/webapp"))
  npmInstallCommand.set(if (getenv("CI") != null) { "ci" } else { "install" })
  download.set(false)
}

tasks.register<NpxTask>("buildFrontEnd") {
  dependsOn("npmInstall")
  inputs.files(fileTree("src/webapp/src"))
  outputs.dir("build/resources/main/webapp")
  command.set("vite")
  args.set(listOf("build", "--mode", "production"))
}
```

This configures the `node` plugin by pointing it to the previously-generated `src/webapp` directory. NPM dependencies will be installed using `npm install` on regular systems and `npm ci` when running on CI systems. 
Also, I decided to disable downloading of Node.js distributions via this plugin because I usually prefer to do this separately.

Finally, we'll need to ensure that the newly-created `buildFrontEnd` task is invoked along with the existing build steps. I chose the following configuration for now, but will still have to look more closely at that part to determine whether there are better alternatives.

```kotlin
tasks.named("build") {
  dependsOn("buildFrontEnd")
}

tasks.named("assemble") {
  dependsOn("buildFrontEnd")
}

tasks.named("run") {
  dependsOn("build")
}
```

Done. Now the SPA will be built and packaged along with the rest of the application.

### Serving the SPA with Vert.x

Only one step left: the compiled application will need to be served via Vert.x. Open up the generated `MainVerticle.kt` and adapt its implementation:

```kotlin
class MainVerticle : AbstractVerticle() {

  override fun start(startPromise: Promise<Void>) {
    val router = Router.router(vertx)
    router.route("/*").handler(StaticHandler.create("webapp"))
    router.get().handler { routingContext: RoutingContext ->
      routingContext.response().sendFile("webapp/index.html")
    }
    vertx
      .createHttpServer()
      .requestHandler(router)
      .listen(8888) { asyncResult ->
        if (asyncResult.succeeded()) {
          startPromise.complete()
          println("HTTP server started on port 8888")
        } else {
          startPromise.fail(asyncResult.cause());
        }
      }
  }
}
```

If you look back at the Gradle configuration, you will see that the SPA will be compiled into the `webapp` directory inside the bundle:
```kotlin
  outputs.dir("build/resources/main/webapp")
```
This directory is served from the root of the Vert.x application. Also, we'll set up a backup-route that will serve the generated `index.html` whenever there isn't any other matching route. This will allow our SPA to use arbitrary routes and deep links without being forced to only used client-side hash-based routing.

That's it: if you now invoke `./gradlew run` again, you will see the SPA being served on port 8888, i.e. `http://localhost:8888`. Congratulations, you now have everything you need to easily deploy a Vert.x-based application with a Node.js-based SPA front-end. 🥳