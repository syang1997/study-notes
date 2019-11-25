# Lambda表达式

## 为何需要 Lambda表达式

- 在Java中,我们无法将函数作为参数传递给方法,也无法声明返回一个函数的方法
- 在 Javascript中,函数参数是一个函数,返回值是另一个函数的情况是非常常见的; Javascript是
  门非常典型的函数式语言

## Lambda的基本结构

``` java
(param1,param2,param3)->{
    //执行体
}
```

- Lambda表达式可以有零个或多个参数
- 参数的类型既可以明确声明,也可以根据上下文来推断。例如:(inta)与(a)效果相同
- 所有参数需包含在圆括号内,参数之间用逗号相隔。例如:(a,b)或(inta,intb)或( String a,fintb, float c)
- 空圆括号代表参数集为空。例如:()->42
- 当只有一个参数,且其类型可推导时,圆括号()可省略。例如:a-> return a'a
- Lambda表达式的主体可包含零条或多条语句
- 如果 Lambda表达式的主体只有一条语句,花括号可省略。匿名函数的返回类型与该主体表达式一致 
- 如果 Lambda表达式的主体包含一条以上语句,则表达式必须包含在花括号아中(形成代码块)。匿名函数的返回类型与代码块的返回类型一致,若没有返回则为空

---------

注:

``` java
List<Integer> list=Arrays.asList(1,2,3,4);
list.forEach(new Consumer<Integer>(){
    public void accept(Integer integer){
        System.out.println(integer);
    }
    }
})
list.forRach(i->{
    System.out.println(i);
})
```

Consumer是一个函数式接口

关于函数式接口:

1. 如果一个接口只有一个抽象方法,那么该接口就是一个函数式接口

2. 如果我们在某个接口上声明了 Functiona1 Interface注解,那么编译器就会按照函数式接口的定义来要求该接口

3. 如果某个接口只有一个抽象方法,但我们井没有给该接口声明 Punctionalinterface注解,那么编译器依旧会将该接口看作是函数式接口。

   

-----------------

## Lambda表达式作用

- Lambda表达式为Java添加了缺失的函数式编程特性,使我们能将函数当做一等公民看待
- 在将函数作为一等公民的语言中, Lambda表达式的类型是函数。但在Java中, Lambda表达式是==对象==,他们必须依附于一类特别的对象类型函数式接口( (functional interface)
- 传递行为,而不仅仅是值
  - 提升抽象层次
  - API重用性
  - 更好更加灵活

  