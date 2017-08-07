title: Kotlin State of the Server
class: animation-fade
layout: true

<!-- This slide will serve as the base layout for all your slides -->
.bottom-bar[
  {{title}}
]

---

class: impact

# {{title}}
## High Level Overview of how to use Kotlin Server-Side
Victor J. Reventos

---

# Topics Covered

* REST API
* Asynchronous Programming

---

# REST API

## Frameworks

**Everything** on the jvm ecosystem

* [Jooby](http://jooby.org/)
* [Spring Boot](https://projects.spring.io/spring-boot/)
* [sparkjava](http://sparkjava.com/)
* [ktor](https://github.com/Kotlin/ktor)
* Insert your favorite framework here...

---

# Jooby

## Features

* MVC programming model
* Dependency Injection using [Guice](https://github.com/google/guice)
* Plugabble Server Architecture
 * Netty
 * Jetty
 * Undertow
* [Module System](http://jooby.org/modules/)
 * Database
 * Caching
 * etc



---

# Jooby Sample

[Project Example](https://github.com/vjames19/kotlin-microservice-example)

```kotlin
import org.jooby.*

fun main(args:Array<String>) {
  run(*args) {
    get("/hello") { req, resp ->
        val name = req.param("name").toOptional().orElseGet { "World" }
        "Hello ${name}"
    }
  }
}
```

---

# Asynchronous Programming

## Futures

* A traditional Future represents the result of an asynchronous computation
 
 * A computation that may or may not have finished producing a result yet. 
 
 * A Future can be a handle to an in-progress computation, a promise from a service to supply us with a result.

--

* [CompletableFuture](http://www.baeldung.com/java-completablefuture) - Java 8 future

--

* [ListenableFuture](https://github.com/google/guava/wiki/ListenableFutureExplained) - Pre Java 8 way to deal with Future

---

# Futures in Kotlin
 
* [kotlin-futures](https://github.com/vjames19/kotlin-futures) - A collections of extension functions to make the JVM Future, CompletableFuture, ListenableFuture API more functional and Kotlin like.

--

### map - Transform a future by applying a function

```kotlin
val future = Future { 10 }
	.map { "Hello user with id: $it" }
```

vs

```kotlin
val future = Future { 10 }
    .thenApplyAsync(Function { userId -> "Hello user with id: $userId" }, ForkJoinExecutor)
```

---

# Combining Futures - Kotlin

```kotlin
// Fetching the posts depends on fetching the User
val posts = fetchUser(1).flatMap { fetchPosts(it) }

// Fetching both the user and the posts and then combining them into one
val userPosts =  fetchUser(1).flatMap { user ->
    fetchPosts(user).map { UserPosts(user, it) }
}
```

---

# Combining Futures - Java

```kotlin
val posts = fetchUser(1)
        .thenComposeAsync(Function { fetchPosts(it) }, ForkJoinExecutor)

val userPosts = fetchUser(1)
        .thenComposeAsync(Function { user ->
            fetchPosts(user).thenApplyAsync(Function { posts ->
                UserPosts(user, posts)
            }, ForkJoinExecutor)
        }, ForkJoinExecutor)
```


---

# Enter Coroutines

* Coroutines are a way to write asynchronous code sequentially. 
* Instead of messing around with callbacks, you can write your lines of code one after the other. 

---

```kotlin
data class User(val id: Long, val name: String)
data class Post(val id: Long, val content: String)
data class UserPosts(val user: User, val posts: List<Post>)

fun fetchUser(id: Long): CompletableFuture<User> = Future { User(1, "Victor")}
fun fetchPosts(user: User): CompletableFuture<List<Post>> = Future { emptyList<Post>() }

fun userPosts(): CompletableFuture<UserPosts> = future { 

	// all of this is happening async
    val user = fetchUser(1).await()

    // await will yield the thread until its finished
    val posts = fetchPosts(user).await()
    
    // returns a new future with UserPosts
    UserPosts(user, posts)
}
```

---

class: middle, center

# Questions?