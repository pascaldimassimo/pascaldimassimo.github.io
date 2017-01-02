---
layout: post
title: 'Spring AOP: Prevent advising test classes'
---

Recently, I've been using <a href="http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/html/testing.html#integration-testing-annotations">@ContextConfiguration</a> to annotate my Spring test classes. This allows to have a context available for the test, without having to load it manually in a @Before method. It also gives to opportunity to use @Autowired within the test class.

But when using this feature with Spring AOP, you can run into a situation where test classes get advised. I often use the same package name for my business and test classes. For example, if my business classes are in the package 'com.company.xyz', then I put my tests into a another source folder (src/test/java), but using the same 'com.company.xyz' package name for the test classes. With this scheme, if you define a pointcut for all public methods of the 'com.company.xyz' packages, Spring will try to advise the public method of your test classes. This happens because, thanks to the @ContextConfiguration, the test classes are now part of the Spring context.

What is the problem with having the test classes advised? Most of the time, my test classes do not implement any interface. When Spring AOP tries to advise a class that does not implement an interface, it needs to create a <a href="http://cglib.sourceforge.net/">cglib</a> proxy, instead of a JDK proxy.  Since all the business classes that I need to advise implements at least one interface, I don't need to include cglib in the project. So when trying to advise my test classes, Spring throws an exception saying that cglib is not available.

So, I needed a way to tell Spring AOP not to advise my test classes. It turns out that the pointcut expression syntax allows to tell Spring not to advise a class if it implements a specific interface. So, using <a href="http://stackoverflow.com/questions/2011089/aspectj-pointcut-for-all-methods-of-a-class-with-specific-annotation#2522821">this Stackoverflow's answer</a>, I came up with this code:

```java
@Pointcut("within(@org.springframework.test.context.ContextConfiguration *)")
public void isTestClass() {}

@Pointcut("execution(public * com.company.xyz.*.*(..))")
public void isPublicMethod() {}

@Around("isPublicMethod() && !isTestClass()")
public void aroundMethod(ProceedingJoinPoint pjp)
{
  // ..
}
```

So here, we define one pointcut for classes implementing the @ContextConfiguration interface and another one for all the public methods of the com.company.xyz package. We can then use these two pointcut expressions to prevent Spring from advising the test classes (notice the !isTestClass).
