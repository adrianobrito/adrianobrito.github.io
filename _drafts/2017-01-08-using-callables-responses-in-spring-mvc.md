---
layout: post
title: Using callable responses in Spring MVC
---
[Spring MVC]({{ site.baseurl }}/public/images/top_web_framework.png) is definitively the leading Java Web framework nowadays, offering a strong abstraction for Servlet API, making easier to develop Java web applications. It's used as one of base components of Spring Cloud, a suite of many different Spring frameworks necessary to build a microservice platform.

One of the advantages of use Spring MVC, is the fact that you have a simple idiomatic abstraction to develop applications over HTTP protocol. Different from Servlet API, you don't need to worry about low level details like `HttpServletRequest` or `HttpServletResponse`, you just care about the application's logic, like it's showed in the example below with a basic example:

```java
@Controller
public class SampleController {

	@Autowired
	private HelloWorldService helloWorldService;

	@GetMapping("/helloWorld")
	@ResponseBody
	public String helloWorld() {
		return "Hello, World";
	}

}
```

## Servlet 3 and Asynchronous processing



[Expose the problem of long term threads in serlvet API contexts]
[Show a solution based in Callable Controller]
[Explain the thread flow]
[Conlclude with the advantage approach]
