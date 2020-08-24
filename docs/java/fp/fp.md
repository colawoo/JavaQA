
### 1 `Lambda`表达式
实现接口  ===> 匿名内部类 ===> `Lambda`表达式


#### 1.1 `Lambda`语法

- ->：`JDK8`引入的一个新操作符，`Lambda`操作符。它将`Lambda`表达式分为左右两部分。
- 左侧：参数列表
    - 无参：必须使用括号`()`表示空参数列表。
    - 单参：可以不需要括号`()`，也可以使用括号`()`包裹单参。
    - 多参：将参数列表放在括号`()`中。
- 右侧：方法体
    - 方法体是单行：该表达式的结果自动成为`Lambda`表达式的返回值，在此处使用`return`关键字是非法的。
    - 方法体是多行，则必须将这些行放在花括号中。在这种情况下，就需要使用`return`。

注意：
`Lambda`表达式中使用到的局部变量，和`JDK7`之前的匿名内部类情况相同，都需要声明为`final`，只不过`JDK8`中可以不显式声明为`final`。

### 2 函数式接口

只包含一个抽象方法声明的接口，称为**函数式接口**。但接口内可以有默认方法（`default`修饰）、静态方法（`static`修饰）、对`Object`类中方法的重写方法。
可以使用注解`@FunctionalInterface`修饰。`Lambda`表达式需要函数式接口的支持。



#### 2.1 `JDK8`内置四类核心函数式接口

| 类               | 类型       | 方法               |
| ---------------- | ---------- | ------------------ |
| `Consumer<T>`    | 消费型接口 | void accept(T t);  |
| `Supplier<T>`    | 供给型接口 | T get();           |
| `Function<T, R>` | 函数型接口 | R apply(T t);      |
| `Predicate<T>`   | 断言型接口 | boolean test(T t); |



#### 2.2 函数式接口汇总一览

|                                       | 方法                                                         | 类                                                           |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Runnable`<br />0参数，无返回值       | public abstract void run();                                  | `Runnable`                                                   |
| `Callable`<br />0参数，有返回值       | V call() throws Exception;                                   | `Callable<V>`                                                |
| `Comparator`<br />2参数，有返回值     | int compare(T o1, T o2);                                     | `Comparator<T>`                                              |
| `Supplier` <br />0参数，有返回值      | T get();<br />boolean getAsBoolean();<br />int getAsInt();<br />long getAsLong();<br />double getAsDouble(); | `Supplier<T>`<br />`BooleanSupplier`<br />`IntSupplier`<br />`LongSupplier`<br />`DoubleSupplier` |
| `Comsumer`<br />1或2参数，无返回值    | void accept(T t);<br />void accept(T t, U u);<br />void accept(int value);<br />void accept(long value);<br />void accept(double value);<br />----------------------------------------------------<br />void accept(T t, int value);<br />void accept(T t, long value);<br />void accept(T t, double value); | Consumer<T><br />BiConsumer<T, U><br />IntConsumer<br />LongConsumer<br />DoubleConsumer<br />------------------------------------------------<br />ObjIntConsumer<T><br />ObjLongConsumer<T><br />ObjDoubleConsumer<T> |
| `Predicate`<br />1或2参数，返回布尔值 | boolean test(T t);<br />boolean test(T t, U u);<br />boolean test(int value);<br />boolean test(long value);<br />boolean test(double value); | Predicate<T><br />BiPredicate<T, U><br />IntPredicate<br />LongPredicate<br />DoublePredicate |
| `Function`<br />1或2参数，有返回值    | R apply(T t);<br />R apply(T t, U u);<br />----------------------------------------------------<br />R apply(int value);<br />long applyAsLong(int value);<br />double applyAsDouble(int value);<br />----------------------------------------------------<br />R apply(long value);<br />int applyAsInt(long value);<br />double applyAsDouble(long value);<br />----------------------------------------------------<br />R apply(double value);<br />int applyAsInt(double value);<br />long applyAsLong(double value);<br />----------------------------------------------------<br />int applyAsInt(T value);<br />long applyAsLong(T value);<br />double applyAsDouble(T value);<br />----------------------------------------------------<br />int applyAsInt(T t, U u);<br />long applyAsLong(T t, U u);<br />double applyAsDouble(T t, U u); | Function<T, R><br />BiFunction<T, U, R><br />------------------------------------------------<br />IntFunction<R><br />IntToLongFunction<br />IntToDoubleFunction<br />------------------------------------------------<br />LongFunction<R><br />LongToIntFunction<br />LongToDoubleFunction<br />------------------------------------------------<br />DoubleFunction<R><br />DoubleToIntFunction<br />DoubleToLongFunction<br />------------------------------------------------<br />ToIntFunction<T><br />ToLongFunction<T><br />ToDoubleFunction<T><br />------------------------------------------------<br />ToIntBiFunction<T, U><br />ToLongBiFunction<T, U><br />ToDoubleBiFunction<T, U> |
| `Operator`<br />1或2参数，有返回值    | T apply(T t);<br />int applyAsInt(int operand);<br />long applyAsLong(long operand);<br />double applyAsDouble(double operand);<br />----------------------------------------------------<br />T apply(T t1, Tt2);<br />int applyAsInt(int left, int right);<br />long applyAsLong(long left, long right);<br />double applyAsDouble(double left, double right); | `UnaryOperator<T>extends Function<T, T>`<br />`IntUnaryOperator`<br />`LongUnaryOperator`<br />`DoubleUnaryOperator`<br />------------------------------------------------<br />`BinaryOperator<T> extends BiFunction<T,T,T>`<br />`IntBinaryOperator`<br />`LongBinaryOperator`<br />`DoubleBinaryOperator` |




### 3 方法引用

方法引用可用于在不调用方法的情况下引用方法。它将方法视为Lambda表达式。它们只能作为语法糖来减少一些lambdas的冗长。

有些情况下，我们用Lambda表达式仅仅是调用一些已经存在的方法，除了调用动作外，没有其他任何多余的动作，在这种情况下，我们倾向于通过方法名来调用它，使得Lambda在调用那些已经拥有方法名的方法的代码更简洁、更容易理解。



#### 3.1 基本语法

- 类名或对象名
- ::
- 方法名
  - 注意没有方法参数列表



#### 3.2 方法引用分类

| 类型         | 语法             | 示例                                                         | 备注                                                         |
| ------------ | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 实例方法引用 | 实例::实例方法名 | User user = new User();<br />Supplier<String> s = () -> user.getName();<br />Supplier<String> s1 = user::getName; // 引用<br />s1.get(); // 方法调用 |                                                              |
| 静态方法引用 | 类名::静态方法名 | Comparator<Integer> c = (x, y) -> Integer.*compare*(x, y);<br />Comparator<Integer> c1 = Integer::*compare*; |                                                              |
| 对象方法引用 | 类名::实例方法名 | BiPredicate<String, String> bp = (x, y) -> x.equals(y);<br />BiPredicate<String, String> bp1 = String::equals; | 第一个参数即x，是实例方法的调用者；<br />第二个参数是实例方法的参数。 |
| 构造函数引用 | 类名::new        | 示例一：<br />Supplier<User> s = () -> new User();<br />Supplier<User> s1 = () -> User::new;<br />示例二：<br />Function<Integer, User> f = x -> new User(x);<br />Function<Integer, User> f1 = User::new; |                                                              |
| 数组引用     | 类型[]::new      | Function<Integer, String[]> f = x -> new String[x];<br />Function<Integer, String[]> f1 = String[]::new; |                                                              |



注意：

保证要调用的方法的参数列表和返回值，与我们使用的函数式接口内的抽象方法保持一致。



