---
layout: post
title: 'Building a singleton in Java'
---

Usually, I don’t use singletons that objects need to fetch using a static method. I prefer using dependency injection to give each object what it needs. But for lower level concerns, like logging or security, it is common to use singletons that are accessible from a static method. Like the way you get a logger with <a href="http://www.slf4j.org/">SLF4J</a>:

```java
Logger logger = LoggerFactory.getLogger(HelloWorld.class);
```

The logger object is a singleton obtained via a static method.

So how should we implement those static methods to return a singleton?

<h2 id="lazy-or-not">Lazy or not?</h2>

The first question you need to ask is whether or not you need lazy initialiation of the singleton. According to <a href="http://en.wikipedia.org/wiki/Lazy_initialization">Wikipedia</a>, “lazy initialization is the tactic of delaying the creation of an object, the calculation of a value, or some other <strong>expensive</strong> process until the first time it is needed” (emphasis mine). So if building your singleton is not really expansive, there is probably no need to build it lazily. In that case, the easiest way to build a singleton is to use a static field:

```java
public class SingletonHolder
{
   public static final Singleton INSTANCE = new Singleton();
}
```

Then, to get this singleton, you only need to grab the instance:

```java
Singleton singleton = SingletonHolder.INSTANCE;
```

Of course, if you prefer, you could add a getter to the class to return the instance instead of having it public.

The important thing about this singleton implementation is the use of a static field to hold the instance. Static fields are initialized once when the class is loaded. So INSTANCE will be unique because it will be instanciated only once. But beware of serialization and reflection hacks that could wreak havoc and make INSTANCE not unique anymore. Check Item 3 of Joshua Bloch’s excellent <a href="http://www.informit.com/store/effective-java-9780321356680">Effective Java 2nd Edition</a> for details.

If you are using Java 1.5+, maybe the best approach to create a singleton is by using enums:

```java
public enum EnumSingleton
{
   INSTANCE;
}
```

To get the instance:

```java
EnumSingleton.INSTANCE;
```

The advantage of this method is that the singleton is guaranteed to be unique, even in the face of serialization or reflection. And you have to admit it is very simple!

According to <a href="http://stackoverflow.com/questions/3635396/pattern-for-lazy-thread-safe-singleton-instantiation-in-java#3635619">Michael Borgwardt in this Stackoverflow answer</a>, 99.99% of the time, this is all you need to build a singleton in Java. I’m sure that 99.99% is more a figure of speech than a scientifically validated number, but you got the idea!

<h2 id="im-lazy">I’m lazy!</h2>

If your singleton is really that expensive to build, maybe you want to defer its creation to the moment it is first accessed.

One challenge when implementing a lazy-initiliazed singleton is to correctly handle multiple threads. For a singleton that is expansive to build, we want the first thread that needs it to trigger its creation, while all other threads that need the singleton afterward to simply grab it. We don’t want to have multiple threads triggering the creation of multiple singletons (it won’t be a singleton anymore!).

Probably the most simple way to acheive this is to synchronize the method use to get and build the singleton, like this:

```java
public class SingletonHolder
{
   private static Singleton INSTANCE;

   public static synchronized Singleton getInstance()
   {
      if (INSTANCE == null)
      {
         INSTANCE = new Singleton();
      }
      return INSTANCE;
   }
}
```

Nothing complicated about this. For many implementations that needs lazy initialization, this is perfectly fine. The main drawback of it is that under heavy thread usage, it might be a performance bottleneck (but please, do profile before jumping to that conclusion). As we can see, the whole getInstance method is synchronized, but what we really need to protect is the creation of the singleton. Once it is created, there is no need for the method to be synchronized anymore. How can we acheive this?

One method that was devised to solve that problem is <a href="http://en.wikipedia.org/wiki/Double-checked_locking">double-checked locking</a>. It goes like this:

```java
public class SingletonHolder
{
   private static Singleton INSTANCE;

   public static Singleton getInstance()
   {
      if (INSTANCE == null)
      {
         synchronized (SingletonHolder.class)
         {
            if (INSTANCE == null)
            {
               INSTANCE = new Singleton();
            }
         }
      }
      return INSTANCE;
   }
}
```

As we can see, the creation of the singleton is protected by a synchronized block. But the method itself is not. Once the singleton is built, the INSTANCE is returned and the synchronized block won’t be used anymore. Intituively, it seems to be ok. The main problem with this is, well, it don’t work in Java! Because of memory visibility and reordoring of instructions by the compiler and/or runtime, even if a thread sees the INSTANCE as non-null, it might not be properly initialized yet (as if the constructor as not been run). See the famous <a href="http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html">The “Double-Checked Locking is Broken” Declaration</a> for more details.

But wait! Thanks to changes in the Java memory model from version 1.5, you can make double-checked locking work by either making INSTANCE volatile, or by making sure the Singleton class is immutable. See again the Declaration for details.

So, should we use this method to build lazy-initialized singleton? In is <a href="http://www.ibm.com/developerworks/library/j-jtp03304/">excellent article on the subject</a>, Brian Goetz says that even though it now works since Java 1.5, it might not be the best choice (see the heading <a href="https://www.ibm.com/developerworks/library/j-jtp03304/#3.2">Does this fix the double-checked locking problem?</a> in his article). It turns out that there is a better approach:

```java
public class SingletonHolder
{
   private static class Holder
   {
      static Singleton INSTANCE = new Singleton();
   }

   public static Singleton getInstance()
   {
      return Holder.INSTANCE;
   }
}
```

That method is called <a href="http://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom">Initialization-on-demand holder</a>. Notice that the singleton is hold by a private static class. But is it lazy and thread-safe? Will the singleton only be initialized on the first call to getInstance? Quoting Brian Goetz:
<blockquote>This idiom derives its thread safety from the fact that operations that are part of class initialization, such as static initializers, are guaranteed to be visible to all threads that use that class, and its lazy initialization from the fact that the inner class is not loaded until some thread references one of its fields or methods.</blockquote>
So INSTANCE will only be initialized the first time it is accessed and all other threads will get the initialized INSTANCE afterward. The beauty of this method is its simplicity. Less code and no synchronized. And according to Brian Goetz, it is faster.

In Item 71 of <a href="http://www.informit.com/store/effective-java-9780321356680">Efective Java 2nd Edition</a>, Joshua Bloch recommends this last method for lazy initialization of a static field. For lazy initialization of instance fields, he recommends to use double-checked locking (of course, only if you really need it).

<h2 id="conclusion">Conclusion</h2>

Like I said in the introduction, in general, I think it is best to use dependency injection instead of singletons accessible from a static method. But if you do decide that it is what you need, you should go with the simplest solutions, like a singleton initialized on a static field.
