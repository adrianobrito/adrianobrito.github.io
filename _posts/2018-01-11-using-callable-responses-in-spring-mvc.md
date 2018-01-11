---
layout: post
title: Using callable responses in Spring MVC
---
Spring MVC is definitively the [leading Java Web framework nowadays]({{ site.baseurl }}/public/images/top_web_framework.png), offering a strong abstraction for Servlet API, making easier to develop Java Web applications. It's used as one of base components of Spring Cloud, a suite of many different Spring frameworks necessary to build a microservice platform.

One of the advantages of use Spring MVC, is the fact that you have a simple idiomatic abstraction to develop applications over HTTP protocol. Different from Servlet API, you don't need to worry about low level details like `HttpServletRequest` or `HttpServletResponse`, you just care about the application's logic, like it's showed in the example below with a basic code:

```java
@Controller
public class HelloWorldController {

	@GetMapping("/helloWorld")
	@ResponseBody
	public String helloWorld() {
		return "Hello, World";
	}

}
```

Sometimes it is necessary to execute intensive I/O operations or do network tasks, such as handling file uploads or processing a huge volume of data coming from clients. In this particular situation, there's a simple way of doing that, like is showed in example controller implementation below:

```java
@Controller
@RequestMapping("/georeference")
public class GeoReferenceController {

	@Autowired
	private GeoReferenceService geoReferenceService;

	@PostMapping
	public ResponseEntity<?> saveGeoreferences(
		List<GeoReference> georeferences
	) {
		 geoReferenceService.saveGeoreferences(georeferences);
		 return ResponseEntity.ok().build();
	}

	@PostMapping("upload")
	public ResponseEntity<?> uploadGeoreferenceFile(
		@RequestParam MultipartFile geoferenceFile
	) {
		geoReferenceService.writeToDatabase(geoferenceFile);
		return ResponseEntity.ok().build();
	}

}
```

The example above is nice and easy to understand but it has a specific problem that developers usually don't care about: **blocking servlet threads due to long term requets**. Yeah, that's a problem that has a direct impact in a Java web application scalability, but to understand that specific problem it's necessary to dive in first into the:

## Servlet API Thread Pool

As you should know, typical servlet container implementations (Tomcat, Jetty, Undertow) handle each HTTP request as a thread, that is obtained from a thread pool in container. These thread pools are necessary to control the amount of threads that are being executed simultaneously. In a regular basis, each thread consumes ~1MB of memory just to allocate a single thread stack, so 1000 simultaneous requests could use ~1GB of memory only for the thread stacks. Therefore, thread pool comes as a solution to limit the amount of threads being executed, fitting the application to a scenario where it doesn't throw an `OutOfMemoryError`.

![Thread Pool]({{ site.baseurl }}/public/images/request_flow_thread.png)
<small>_Thread per request model_</small>

_Thread per request model_ assumes that threads obtained from pool will execute small tasks, and then those threads could be released and become available in the pool to process subsequent requests. When a request is dispatched against an endpoint that does a intensive job, the request's thread will be blocked waiting for the end of that job. In the case of many concurrent requests of this nature, the server can reach a scenario that the thread pool has any available threads and the incoming requests will have to wait for it ([starvation](https://docs.oracle.com/javase/tutorial/essential/concurrency/starvelive.html)).

![Thread Pool](https://res.infoq.com/articles/Java-Thread-Pool-Performance-Tuning/en/resources/queue-cartoon.jpg)
<small>_A busy thread pool analogy_</small>

## Servlet API 3.0 and Asynchronous processing

Since Servlet API 3.0, it was possible to implement [servlets that are capable to handle Asynchronous HTTP requests](https://docs.oracle.com/javaee/7/tutorial/servlets012.htm), making it easy to use threads inside servlet implementation like we can check in the example below:

```java
@WebServlet(
    name = "simple",
    value = {"/simple"},
    asyncSupported = true
)
public class SimpleAsyncServlet extends HttpServlet {

    public void service(ServletRequest req, final ServletResponse res)
      throws ServletException, IOException {

      final AsyncContext ctx = req.startAsync();
      ctx.setTimeout(3000);

      // Here ctx.start() receive a Runnable instance as parameter
      ctx.start(() -> {
          intenseJob();
          ctx.complete();
      });     
    }

}
```

Asynchronous requests run in a different thread than the usual HTTP threads, giving the possibility of executing heavy jobs without locking or idling servlet thread pool's threads. In order to avoid a situation that some thread runs infinitely in `AsyncContext`, it was specified a thread execution timeout after getting a `AsyncContext` instance.

Thus, this can be a solution for the `GeoReferenceController` problem that was mentioned before, but how is it done in a Spring MVC application?

## Callable responses

Spring MVC has a idiomatic way to handle situations where it is necessary to use asynchronous requests. It works by using the `Callable` interface from `java.util.concurrent` package, which is kinda like `Runnable`, except that it returns something at the end of its execution.

I created a little [project in github](https://github.com/adrianobrito/callable-controller) with some callable response implementations. In this project, there's a [class](https://github.com/adrianobrito/callable-controller/blob/master/src/main/java/com/example/callablecontroller/CallableController.java) with practical methods that I will use to show some callable response examples, starting from the most basic:

```java
@Controller
@RequestMapping("/async/callable")
public class CallableController {

    //..

    @GetMapping("/response-body")
    public @ResponseBody Callable<String> callable() {
        return () -> {
            Thread.sleep(2000);
            return "Callable result";
        };
    }

    //..

}
```

Spring MVC handles controller methods that return `Callable` as asynchronous requests, so the code inside the lambda is executed in a servlet's `AsyncContext` behind the scenes. It's recommendable to put only the heavy work inside the `Callable` anonymous function, in case of having extra basic logic into the controller method code.

In the `SimpleAsyncServlet` class, a timeout value was specified in `AsyncContext`. So, how can it be done in Spring MVC? Just by using `WebAsyncTask` as return type of a controller method, like is showed in the example below:

```java
@Controller
@RequestMapping("/async/callable")
public class CallableController {

    private final Integer TIMEOUT = 3000;

    @GetMapping("/custom-timeout-handling")
    public @ResponseBody WebAsyncTask<String> callableTimeout() {
        Callable<String> callable = () -> {
            Thread.sleep(2000);
            return "Callable result";
        };

        return new WebAsyncTask<String>(TIMEOUT, callable);
    }

    //..

}
```

Now, we have all the necessary components to refactor `GeoReferenceController` in a way that it doesn't block threads from servlet thread pool when performing a heavy task. Such thing can be verified at the following code:


```java
@Controller
@RequestMapping("/georeference")
public class GeoReferenceController {

    private final int uploadTimeout = 5000;

    @Autowired
    private GeoReferenceService geoReferenceService;

    @PostMapping
    public Callable<ResponseEntity<?>> saveGeoreferences(
        List<GeoReference> georeferences
    ) {
        return () -> {
          geoReferenceService.saveGeoreferences(georeferences);
          return ResponseEntity.ok().build();
        };
    }

    @PostMapping("upload")
    public WebAsyncTask<ResponseEntity<?>> uploadGeoreferenceFile(
        @RequestParam MultipartFile geoferenceFile
    ) {
        return WebAsyncTask<ResponseEntity<?>> (uploadTimeout, () -> {
            geoReferenceService.writeToDatabase(geoferenceFile);
            return ResponseEntity.ok().build();
        });    
    }

}
```

## Good Reads

Here are some good links for you to read and think more about details in servlet asynchronous requests and callable responses in Spring MVC.

* [The importance of tuning your thread pools; Andrew Brampton](https://blog.bramp.net/post/2015/12/17/the-importance-of-tuning-your-thread-pools/) - This blog post is a very complete guide about thread pool tuning, and the author talk about very good details of how servlet thread pool works behind the scenes in servlet container.
*  [Making a Controller Method Asynchronous; Rossen Stoyanchev](https://spring.io/blog/2012/05/10/spring-mvc-3-2-preview-making-a-controller-method-asynchronous/) : A brief discussion about the Spring MVC components that can be used to help implementation of asynchronous controller methods.
