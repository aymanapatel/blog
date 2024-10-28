---
title: "Upgrading Java and Spring without the headache"
seoTitle: "Upgrading Java and Spring without the headache"
seoDescription: "🤔 Want to upgrade Java, Spring Boot versions is a methodical way"
datePublished: Wed Apr 10 2024 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: cm2swmtmf000c08jp941w8ut3
slug: upgrading-java-and-spring-without-the-headache
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/hko-iWhYdYE/upload/0cb8781a42e1bfaf5426a505649e0b69.jpeg
tags: java, springboot

---

🤔 Want to upgrade Java, Spring Boot versions is a methodical way  
  
There is a platform for that!  
  
💡 Moderne's OpenRewrite platform helps with managing complexities in upgrading your Java based projects  
  
Some common use cases include:  
1\. Upgrading to Java 17  
2\. Upgrading Spring Boot 2 to Spring Boot 3  
  
🙋Why a tool?  
Knowing the API changes in libraries for every dependency is difficult. Also, a lot of time can be wasted in a particular solution that does not work well with a particular dependency.  
  
🔧How does it work?  
  
You can import custom recipes as Maven/Gradle task and run them.  
For example, you want to migrate from JUnit 4 to JUnit 5, you can import a recipe for it in Gradle like this:

```java
activeRecipe("org.openrewrite.java.testing.junit5.JUnit5BestPractices")
```

And run the gradle task:

```java
gradlew rewriteRun
```

Since these recipes are chainable, you can add steps to make code changes incrementally.  
Other features include:  
1\. Writing your own custom [recipes](https://docs.openrewrite.org/authoring-recipes/recipe-development-environment) for your use-case  
2\. Dashboard to track the recipes done in your project to track your level of completion of version migration.  
  
📖 Resources:  
1\. Moderne's site: [](https://www.moderne.io/)[https://www.moderne.io/  
2](https://www.moderne.io/￼2). OpenRewrite recipes: [https://docs.openrewrite.org/](https://docs.openrewrite.org/)