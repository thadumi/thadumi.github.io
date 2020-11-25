---
title: Different ways for creating a Servlet 
tags: Java JakartaEE
---

In a Web Application a Servlet can be defined in three ways:
- using the WebServlet annotation
- writing the configuration in the deployment descriptor
- programmatically using ServerContext and ServletRegistration.Dynamic APIs

# Using the WebServlet annotation

The [WebServlet](https://jakarta.ee/specifications/platform/8/apidocs/javax/servlet/annotation/WebServlet.html) annotation can be used only on classes that inherit [HttpServlet](https://jakarta.ee/specifications/servlet/4.0/apidocs/javax/servlet/http/HttpServlet.html).

```java
@WebServlet(name="Foo", urlPatterns={"/foo"})
public class FooService extends HttpServlet {
	// servlet code
}
```

A class annotated with `WebService` will be found by the container at deploy time. The corresponding Servlet will be available at the specified URL patterns with the using the parameter `urlPatterns` . If `urlPatterns` is the only parameter used, then we can use the `value` parameter e.g:

```java
@WebServlet("/foo")
public class FooService extends HttpServlet {
	// servlet code
}
```

The name parameter is optional. If it's not specified is the fully qualified class name of the Servlet (e.g com.thadumi.FooService) will be used as default.

For the full list of parameters check out the [JakartaEE documentation](https://jakarta.ee/specifications/platform/8/apidocs/javax/servlet/annotation/WebServlet.html).

# Creating a Servlet using the deployment descriptor

In the above section, we saw that for using the `WebServlet` annotation we must inherit from `HttpServlet`. However, a Servlet could also inherit from [GenericServlet](https://jakarta.ee/specifications/servlet/4.0/apidocs/javax/servlet/GenericServlet.html) or implement the [Servlet](https://jakarta.ee/specifications/servlet/4.0/apidocs/javax/servlet/Servlet.html) interface. So, who can we deploy these kinds of servlets?

```java
public class FooService extends GenericServlet {
	// servlet code
}
```

For deploying a Servlet of any kind we can use the deployment descriptor. It is the file named web.xml, and located inside the app's WAR under `./WEB-INF/web.xml`. The configuration syntax is given by the [deployment descriptor schema](https://jakarta.ee/specifications/servlet/5.0/jakarta-servlet-spec-5.0.html#deployment-descriptor). The next snipet is an example of how to write the configuration for FooService.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="3.1"
         xmlns="https://jakarta.ee/xml/ns/jakartaee/"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee/ https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd">
         
    <servlet>
            <servlet-name>Foo</servlet-name>
            <servlet-class>com.thadumi.FooService</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>Foo</servlet-name>
        <url-pattern>/foo</url-pattern>
    </servlet-mapping>

</web-app>
```

# Programmatically using the APIs

We are not going to discuss in-depth this approach here. It should be used only when we are writing a framework or complex libraries.

In short, we can implement the [ServletContextListener](https://jakarta.ee/specifications/servlet/4.0/apidocs/javax/servlet/ServletContextListener.html) for creating a context listener. This allows us to be notified about the ServletContext lifecycle changes. Thus, when the web application initialization process is starting we can access the [ServletContext](https://jakarta.ee/specifications/servlet/5.0/jakarta-servlet-spec-5.0.html#servlet-context) and add to it further servlets programmatically.

```java
@WebListener
public class ContextListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ServletContext ctx = sce.getServletContext();

        ServletRegistration.Dynamic srd = ctx.addServlet("Foo", "com.thadumi.FooService");
        srd.addMapping("/foo");
        srd.setLoadOnStartup(1);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        // ...
    }
}
```

# References

1. https://docs.oracle.com/javase/tutorial/java/annotations/basics.html
2. https://jakarta.ee/specifications/platform/9/platform-spec-9-SNAPSHOT.html
3. https://jakarta.ee/specifications/servlet/5.0/servlet-spec-5.0-SNAPSHOT.html