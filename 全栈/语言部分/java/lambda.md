[toc]



## lambda表达式

> 用于简化函数式接口的匿名内部类

- 函数式接口:l有且仅有一个抽象方法的接口,上面都可能会有一个@FunctionalInterface的注解，该注解用于约束当前接口必须是函数式接口

```java
Arrays.sort(students, new Comparator<Student>() {
    @Override         
    public int compare(Student o1, Student o2) {
        return o1.getAge() - o2.getAge(); 
    }        
});

```

使用lambda表达式的类型

```Java
Arrays.sort(students, (Student o1, Student o2) -> {
	return o1.getAge() - o2.getAge();
});

```



### 省略规则

* 参数类型全部可以省略不写。

* 如果只有一个参数，参数类型省略的同时“()”也可以省略，但多个参数不能省略“()”

* 如果Lambda表达式中只有一行代码，大括号可以不写，同时要省略分号“;”如果这行代码是return语句，也必须去掉return



### 静态方法的引用

**类名::静态方法**

* 如果某个Lambda表达式里只是调用一个静态方法，并且“→”前后参数的形式一致，就可以使用静态方法引用

```java
Arrays.sort(students, (o1, o2) -> Student2.compareAge(o1, o2));
↓
Arrays.sort(students, Student2::compareAge);
```



### 实例方法的引用

**对象名::实例方法**

- 如果某个Lambda表达式里只是通过对象名称调用一个实例方法，并且“→”前后参数的形式一致，就可以使用实例方法引用

```Java
Arrays.sort(students, (o1, o2) -> t.compareHeight(o1, o2));
↓
Arrays.sort(students, t::compareHeight);
```



### 特定类型方法的引用

**特定类的名称::方法**

- 如果某个Lambda表达式里只是调用一个特定类型的实例方法，并且前面参数列表中的第一个参数是作为方法的主调，后面的所有参数都是作为该实例方法的入参的，则此时就可以使用特定类型的方法引用

```Java
Arrays.sort(names, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return o1.compareToIgnoreCase(o2);
    }
});
↓
Arrays.sort(names, (String o1, String o2) -> o1.compareToIgnoreCase(o2));
↓
Arrays.sort(names, String::compareToIgnoreCase);
```



### 构造器的引用

**类名::new**

* 如果某个Lambda表达式里只是在创建对象，并且“→”前后参数情况一致，就可以使用构造器引用

```Java
CarFactory cf = new CarFactory() {
    @Override
    public Car getCar(String name) {
        return new Car(name);
    }
};
↓
CarFactory cf = name -> new Car(name);
↓
CarFactory cf = Car::new;
```

