# 继承及类型参数化    
> 刘新春    
> 2017年1月2日

*本章主要参考《Scala编程》第10章*

本章主要比较类之间的两种基本关系：组合与继承。组合是指一个类持有另一个类的引用，借助被引用的类完成任务。继承是超类/子类的关系。Java中的继承关系一般如下所示：    
```java
class 类名 extends 父类 implements 接口1, 接口2... {
}

```
而Scala中的继承关系一般为：
```java
class 类名 extends 父类 with 特质1 with 特质2 with ...{
}
```   
特质的具体内容请参阅[trait](...)。另外，本文会涉及部分泛型的内容。    

[TOC]
## 1. 抽象类   
在SDK中设计抽象类的目的一般是不同类型的子类间共有却有所不同的行为抽象而成的方法。例如，在动物中一般通过生殖的方式繁衍后代。具体细分假设只有哺乳动物和鸟类：哺乳动物一般通过胎生的方式，而鸟类通过卵生的方式进行繁衍。为此，我们在设计类时可以抽象出Animal类。每种动物都有自己的出生方式。为此，具有`birthMethod()`，该方法返回该类动物的繁衍方式。Animal类的形式可以如下所示：    
```java
BirthMethod extends Enumeration {
    type BirthMethod = Value

    /**
     * 胎生
     */
    val VIVIPARITY = Value

    /**
     * 卵生
     */
    val OVIPARITY = Value
}
```
```java
abstract class Animal {
    def birthMethod(): BirthMethod
}
```
在该类中，`birthMethod()`方法并没有具体的实现，即：`birthMethod`方法为`Animal`类的抽象成员函数。*具有抽象成员的类本身必须声明为抽象类*，具体的实现细节为在该类关键字class前加修饰符abstract即可：
```java
abstract class 类名 ...
```

abstract修饰符说明该类有未实现的成员方法，因此不能被实例化为一个具体的对象。如下面操作，编译器会报错：
```shell
scala> new Animal
<console>:5: error: class Animal is abstract;
    cannot be instantiated
    new Animal
        ^
```
某个类中只要有一个方法未实现（即：没有等号或方法体），即认为该类是抽象类。与Java不同，抽象方法前并没有abstract修饰符（也不允许）。    

## 2. 定于无参数方法
在以上定义中，方法`birthMethod()`并没有参数列表，甚至连空列表都没有。在Animal抽象类中定义`birthMethod()`方法如下：
```Java
def birthMethod(): BirthMethod
```
更多时候在Scala中无参数方法定义使用如下方法：
```Java
def birthMethod: BirthMethod
```

无参数方法在scala中非常普遍。相较于带空括号的方法定义，如def birthMethod(): BirthMethod，该方法被称为“空括号方法”。惯例如下：
*只要方法中没有参数并且方法仅能通过读取所包含对象的属性去访问可变状态（特指方法不能改变可变状态），就使用无参数方法。*    

## 3. 扩展类    
由于有很多种哺乳动物和鸟类，因此，我们继续抽象一层，将哺乳动物和鸟类抽象出来。并实现`birthMethod`。具体如下：    

### 哺乳动物    
假设后续我们会按照家养的还是野生的来分类哺乳动物，因此，抽象出生存方式`liveMethod`方法：
```java
object LiveMethod extends Enumeration {
    type LiveMethod = Value

    /**
     * 圈养
     */
     val CAPTIVE = Value

    /**
     * 野生
     */
     val WILD = Value
}
```
```java
abstract class Mammal extends Animal {

    override def birthMethod = BirthMethod.VIVIPARITY    

    def liveMethod: LiveMethod   

    def information = "This is a Mammal animal."
}
```
### 鸟类    
假设我们按照会不会飞分类鸟类：
```java
object FlyModel extends A {

    type FlyModel = Value

    /**
     * 会飞
     */
     val FLY = Value("fly")

    /**
     * 不会飞
     */
     val NO_FLY = Value("no_fly")
}
```
```java
abstract class Bird extends Animal {

    override def birthMethod = BirthMethod.OVIPARITY

    def flyModel: FlyModel
}
```
与Java类似，在类的后面需要使用`extends`表示该类是某个类的子类:
```Java
... extends Animal ...
```
extends子句具有两个效果：
* 使`Mammal`(`Bird`)类继承了`Animal`类的所有非私有成员；
* 让`Mammal`(`Bird`)类成为`Animal`类的子类（反过来，`Animal`是`Mammal`(`Bird`)的父类）。

如果一个类没有extends子句，Scala编译器将隐式的假设该类扩展了Scala.AnyRef，这与Java的java.lang.Object相同。因此，`Man`类隐式的扩展了`AnyRef`类。因此，上面类的继承关系如下图所示:
![jicheng](./jicheng.png)

继承(inheritance)表示超类的所有成员也是子类的成员，但以下两种情况例外：
* 超类的私有成员不会被子类继承；
* 超类的成员若与子类中实现的成员具有相同名称和参数则不会被子类继承。这种情况被称为子类的成员重写(override)了超类的成员。如果子类中的成员具体实现的是超类的抽象方法，也可以说是具体的成员实现(implement)了抽象的成员。

以上两个类`Mammal`和`Bird`也是抽象类，也不能使用new来实例话一个对象。以`Mammal`类为例，定义两个常规的类`AfricanBuffalo`和`Cattle`
```java
class AfricanBuffalo extends Mammal {

    private var address: Option[String] = None

    def liveModel = LiveModel.WILD

    override def information = "This is an AfricanBuffalo, it's also an Animal"

    def setAddress(address: String): Unit = {
           this.address = Some(address)
    }

    def getAddress: Option[String] = address
}


子类的值可以在任何需要超类值得地方使用（称为子类型化）。如下所示：
```java
val bufffalo: Mammal = new AfricanBuffalo
```
变量buffalo定义为`Mammal`类型，所以初始化的值应该也为`Mammal`类型。但实际上初始化的值为`AfricanBuffalo`类型，这是因为`AfricanBuffalo`类扩展了`Mammal`类，`AfricanBuffalo`类与`Mammal`类是兼容的。但此时，buffalo只能调用`Mammal`的方法，而不能调用`AfricanBuffalo`的自有方法。

## 4. 重写方法和字段
Scala里的字段与方法属于相同的命名空间，这样，字段就可以重写无参的方法。如下面的Cattle类可以如下实现information方法：
```java
class Cattle extends Mammal {
  private var host: Option[String] = None

  override def liveMethod: LiveMethod = LiveMethod.CAPTIVE

  override val information = super.information + "And it's also a Cattle."

  def setHost(h: String): Unit = {
    host = Some(h)
  }

  def getHost = host
}
```

在Scala中禁止在同一个类里用相同的名称定义字段和方法（Java是允许的），如下面这段java代码是可以顺利通过编译的：
```java
class CompilesFine {
  private int f = 0;
  public int f() {
    return 1;
  }
}
```
但相应的Scala类不能通过编译：
```java

class WontCompile {
  private var f = 0  //不能编译通过，因为字段和方法重名
  def f = 1
}
```
**关于overrride**
Scala要求：
* 若子类成员重写了父类的具体成员则必须带有override修饰符;    
* 若子类成员实现了父类的同名抽象成员，则这个修饰符是可选的;    
* 若子类成员并未重写或实现什么其他超类里的成员则禁止使用法这个修饰符。    
## 5. 定义参数化字段
参数化字段即：在类的参数中就定义类的内部字段。如下面的类定义：
```java
class Dog(val name：String) extends Mammal
```
该类内部定义了一个名为name的字段，即：可以如下方式使用：
```java
val dog = new Dog("Shinzo Abe")
println(dog.name)
```
其中参数化的字段可以使用private、protected、override来进行修饰。如：
```java
class Dog2(private val age: Int, name: String)

val dog2 = new Dog2(90, "Lee-Teng Hui")
println(dog2.age)
println(dog2.name)
```
dog2.age会报错。

```java
class Dog3(var name: String){
    val age = 5
}
class YellowDog(override val age, var name: String) extends Dog(name) {

}
```

## 6. 调用超类的构造器
在上面的例子中，Dog3的主构造器需要传入一个参数，由于YellowDog扩展了Dog3,因此，YellowDog需要传递给超类的主构造器一个参数。要调用超类的构造器，只要简单的把要传递的参数或参数列表放在超类名之后的括号中即可。(似乎不能使用超类的辅助构造器)   
```java
class Dog4(var name: String) {

    def getName = name

    def this() = this("littleblack")
}
```
```java
class BlackDog(val age: Int, override var name: String) extends Dog4 {
}
```
```java
val littleBlack = new BlackDog(3, "X")
println(littleBlack.getName)
```
## 7. 多态和动态绑定    
* 多态    
在上文中，我们定义了AfricanBuffalo类和Cattle类，其中，两者都是Mammal的子类。因此，Mammal类的变量，既可以指向AfricanBuffalo的类对象，又可以指向Cattle类的对象。
```java
val mammal1: Mammal = new AfricanBuffalo()
val mammal2: Mammal = new Cattle()
```
即：Mammal的对象可以有多种形式，这种形式称为“多态”。
* 动态绑定    
```java
def getInfo(m: Mammal): String = {

    m.information
}
```
如果传入的是一个`AfricanBuffalo`的对象，则会调用`AfricanBuffalo`的infomation方法，如果传入的是`Cattle`的对象，则会调用`Cattle`的information的方法.    

# 类型参数化    

## 2.1 泛型类
在Java和C++一样，类和特质可以带类型参数。在Java中，类型的泛型使用"<>"来表示，而在Scala中可以使用"[]"来表示泛型。如下所示：
```java
class Pair[T, S](val first: T, val second: S)
```
以上代码定义了一个带有两个参数类型为T和S的类。可以使用声明的参数类型来定义变量、方法参数以及返回值的类型。这种带有一个或多个类型参数的类称为泛型类。如果把类型参数替换成实际的类型，将得到一个普通的类，如Pair[Int, String]、Pair[String, Stirng].    
另外，Scala会从构造参数推断出实际类型：    
```java
val p = new Pair("Hello", "Peak") // 这是一个Pair[String, String]类    
val q = new Pair(365, "Happy Day") // 这是一个Pair[Int, String]类
```
泛型类的形式如下：
```java
class 类名[类型参数1, 类型参数2...](参数列表) {  
}
```
## 2.2 泛型函数    

函数也可以才有类型参数：    
```java
def getMiddle[T](a: Array[T]) = a(a.length / 2)
```
泛型类的定义如下：    
```java
def 函数名[类型参数1, 类型参数2....](参数类型): 返回值类型 = {
}
```
如果有必要，可以指定类型：    
```java
val f = getMiddle[String] _
```

## 类型变量界定    

有时需要对类型变量进行限制。例如，假设有一个Pair类，要求两个入参是相同的类型，可以这样定义:    
```java
class Pair[T](val first: T, val second: T)
```
### 上界定界    
假设想要添加一个方法，比较first和second，并返回较小的一个值：    
```java
class Pair[T](val first: T, val second: T) {
  def smaller = if (first.compareTo(second) < 0) first else second
}
```
但我们并不直到first是否具有compareTo方法，因此，T类型必须具有compareTo方法，即：T是`Comparable[T]`的子类。    
```java
class Pair[T <: Comparable[T]] (val first: T, val second: T) {
  def smaller = if (first.compareTo(second) < 0) first else second
}
```
### 下界定界    
假设`Person`类是`Student`类的超类，假设Pair存在一种方法，用第一个参数的超类来替换第一个参数，由于first和second是相同的类型。因此，返回值应该是Pair[T的超类]:
class Pair[T](val first: T, val second: T) {
  def replaceFirst[R >: T](newFirst: R): Pair[R] = new Pair[R](newFirst, second)
}

### 视图定界（隐式转换）    
### 上下文定界    
### Manifest上下文定界
### 多重定界
