# Java 注解
Java 注解学习笔记，原文：[Java中的注解原来是这样回事的](https://blog.csdn.net/swpu_ocean/article/details/83352128)

通俗的理解注解就是类似书页上的注解一样，当你看到一个名词时，对它有一定初步的认识，当进一步了解注解时，那么便会了解一些额外信息。Java 中注解需要我们自己定义（当然有已有的），可以自由的添加一些额外信息，当在对被注解的对象进行应用时，就可以对这些附带的额外信息进行解释（注解处理）。

## 内置注解

- @Override
- @Deprecated
- @SupperessWarnings

## 元注解

元注解即用来描述注解的注解。按 OO 的思想，假设注解是一个对象，那么谁来定义注解呢，那就是元注解。

### @Target

一表胜千言

|参数|说明|
|:-|:-|
|CONSTRUCTOR|构造器的声明|
|FIELD|域声明（包括enum实例）|
|LOCAL_VARIABLE|局部变量声明|
|METHOD|方法声明|
|PACKAGE|包声明|
|PARAMETER|参数声明|
|TYPE|类、接口（包括注解类型）或enum声明|

### @Retention

|参数|说明|
|:-|:-|
|SOURCE|注解将被编译器丢弃|
|CLASS|注解在class文件中可用，但会被JVM丢弃|
|RUNTIME|JVM将在运行期也保留注解，因此可以通过反射机制读取注解的信息|

### @Documented


### @Inherited


## 如何定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {

}
```

在后续的使用中直接 `@Test` 就可以使用我们自己定义的注解了。但是因为这个注解没有实现任何功能，所它什么也不会做。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    String name() default "";

    String catalog() default "";

    String schema() default "";

    UniqueConstraint[] uniqueConstraints() default {};

    Index[] indexes() default {};
}
```

## 注解处理器

定义一个简单的注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Person{
    String name() default "I don't have name";
	int age() default 21;
}
```

应用到实体类中：

```java
public class MyLove {

    @Person(name = "My name is zhy")
    public String zhy(){
        return "zhy";
    }

    @Person(name = "My name is xyx", age = 19)
    public String xyx(){
        return "xyx";
    }

}
```

相应的注解处理器：

```java
import java.lang.reflect.Method;
import java.util.List;

public class MyLoveTest {

	public static void myLoveTest(List<Integer> ages, Class<?> cl) {
		Method[] methods = cl.getDeclaredMethods();
		for (Method method :
				methods) {
			Person person = method.getAnnotation(Person.class);
			if (person != null) {
				System.out.println("My name is " + person.name() + "and I'm " + person.age());
				ages.remove(person.age());
			}
		}

		for (int i :
				ages) {
			System.out.print("Missing age is " + i);
		}
	}
}
```