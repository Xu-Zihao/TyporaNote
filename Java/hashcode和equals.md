### equals方法

我们常用的对比变量的方法有`==`和`equals()`。

`==`运算符用于比较变量对应的内存中所存储的数值是否相同，要比较两个**基本类型**的数据或两个**引用变量**是否相等，只能使用“==”运算符。

> 可以理解为==对比的是在栈中的变量和值，其中包括基本类型的变量名和值，还有各种对象的引用变量名和引用地址。如 `user1=addr1`，
>
> `user2=addr2`，表达式`user1==user2`就是在对比`addr1`是否等于`addr2`。



Object类中的equals方法用于检测一个对象是否等于另外一个对象。Object类中实现的equals方法将确定两个对象的引用是否相等。

> 这很合理：如果两个对象的引用相同，那么这两个对象一定相同

但是，经常需要基于状态检测对象的相等性，如果两个对象的状态相同，才认为这两个对象是相等的。比如，两个员工对象的姓名、薪水、雇佣日期都一样，则认为它们是相同的员工（实际一般比较ID）。

```java
public class Employee
{
    ...
    public boolean equals(Object otherObject)
    {
        //快速对比引用
        if(this==otherObject) return true;
        //保证otherObject不为空
        if(otherObject==null) return false;
        //对比是否属于同一个类，这里有一个值得讨论的地方，看下面的解释
    	if(getClass()!=otherObject.getClass())
            return  false;
        //把Object转为相应的类型
        Employee other=(Employee) OtherObject;
        //对比内容
        return name.equals(other.name)
            &&salary==other.salary
            &&hireDay.equals(other.hireDay);
    }
}
```

> 谈一谈`equals`中的**对称性原则**，即`o1.equals(o2)`返回`true`，则依据对称性原则，`o2.equals(o1)`也应该返回`true`。
>
> 考虑这一种情况:
>
> `e.equals(m)`
>
> 在这里`e`是一个`Employee`的对象，`m`是`Manager`对象，并且两个对象拥有相同的姓名、薪水和雇佣日期。如果在`Employee.equals`中用`instanceof`进行检测，这个调用将返回`true`。但是在反过来调用：
>
> `m.equals(e)`
>
> 也需要返回`true`。对称性原则不允许这个调用返回`false`或抛出异常。
>
> 这就出现了很不合理的情况，`Manager`类受到了束缚。这个类的`equals`方法必须要将自己与任何一个它的父类对象进行对比，在这种情况下`Manager`只能使用自己与父类共有的属性进行对比而丢弃自己特有的部分。这样感觉`instanceof`也不是很好用。
>
> 很多人认为`equals()`的`getClass`检测是有问题的，因为它违反了替换原则。例如，在`AbstractSet`类的`equals`方法，它检测两个集合是否有相同的元素。`AbstractSet`有两个具体子类：`TreeSet`和`HashSet`，它们分别使用不同的算法查找集合元素。但是无论采用什么方法，你肯定希望可以比较任意的两个集合。在这个例子中，应该将`AbstractSet.equals`声明为`final`，因为对于相等性的定义（对比集合是否有相同的元素）在`AbstractSet`中就已经定好，子类只需要沿用该定义就行，但是为了让子类采用更适合自己的数据结构的高效算法进行比较，`AbstractSet.equals`并没有声明为`final`，子类也覆写了相应的`equals`方法。
>
> 有两种情况：
>
> - 如果子类可以有自己的相等性概念，则为了满足对称性需求应该使用`getClass`检测。
> - 如果由父类决定相等性概念，那就可以使用`instanceof`检测，这样就可以在不同的子类对象之间进行对比。

编写equals的几点建议：

1. 显示参数命名为`otherObject`，稍后需要将它强制转换成另一个名为`other`的变量。、、

2. 检测`this`与`otherObject`是否相等

   `if(this==otherObject) return true;`  这是一个优化，可以在相等的时候免去后面的繁琐对比。

3. 检测`otherObject`是否为`null`，如果为`null`，则返回`false`。

   `if(otherObject==null) return false;`

4. 比较`this`与`otherObject`的类。

   如果`equals`的语义可以在子类中改变，则用`getClass`检测：

   `if(getClass()!=otherObject.getClass()) return  false;`

   如果所有子类都有相同的相等性语义，则用`instanceof`检测：

   `if(!(otherObject instanceof ClassName)) return false;`

5. 将`otherObject`强制转换成相应类型的变量。

   `Employee other=(Employee) OtherObject;`

6. 根据相等性概念的要求来比较字段。使用==比较基本类型，使用`eqials`比较对象字段。

如果在子类中重新定义`equals`，需要在其中包含一个`super.equals(other)`调用。







### hashCode方法

hash code就是哈希码、散列码的意思。是由对象导出的一个整数值。散列码没有规律，两个不同的对象散列码基本上不会相同。

String类计算散列码：

```java
int hash=0;
for(int i=0;i<length();i++)
    hash=31*hash+charAt(i);
```

`hashCode`方法定义在`Object`类中，因此每个对象都有一个默认的散列码，值由对象的存储地址得出。散列码在散列表中有非常重要的作用，为了使得散列码内容与对象的内容有关，我们在定义一个类时，通常会重写`hashCode`方法。

：如果`x.equals(y)`返回`true`，那么`x.hashCode()`就必须与`y.hashCode()`返回相同的值。

`hashCode()`方法和equal()方法的作用其实一样，在[Java](https://links.jianshu.com/go?to=http%3A%2F%2Flib.csdn.net%2Fbase%2Fjava)里都是用来对比两个对象是否相等一致，那么equal()既然已经能实现对比的功能了，为什么还要`hashCode()`呢？
 因为重写的`equal()`里一般比较的比较全面比较复杂，这样效率就比较低，而利用`hashCode()`进行对比，则只要生成一个hash值进行比较就可以了，效率很高，但是`hashCode()`并不是完全可靠，有时候不同的对象他们生成的hash code也会一样，所以`hashCode()`只能说是大部分时候可靠，并不是绝对可靠，所以我们可以得出：

1. `equals()`相等的两个对象他们的`hashCode()`肯定相等，也就是用`equals()`对比是绝对可靠的。
2. `hashCode()`相等的两个对象他们的equal()不一定相等，也就是`hashCode()`不是绝对可靠的。

所以在容器中，用到hash的地方，比如`HashMap`，我们知道`HashMap`的键是不能重复的，在插入一个新的元素时首先通过新元素的键的hash code找插入的位置，如果桶的位置为空就插入，如果桶不为空，说明已经有一个hash code的键在`HashMap`，这时是不是就可以直接覆盖新值了呢？ 不是的，看上面的总结，hash code相同的两个对象（Key也是对象）不一定就是`equals()`的，所以还要调用`==`或者`equals()`进一步对比。而且我们知道hash code不同的两个对象一定是不同的对象，所以在没有发生哈希冲突的情况下（即无需进一步对比）对比hash code 比调用`equals()`效率高得多。

```java
 if (p.hash == hash //先对比hash code
     &&((k = p.key) == key || (key != null && key.equals(k)))) //进一步的对比
            e = p;
```

但是上面例子得前提是`equals`与`hashCode`的定义相容的情况。试想，我们新定义了一个类Student，重写了该类得equals方法对比的内容是ID，而`hashCode`方法继承沿用Object的默认方法，在实例化两个ID相同的`Student`对象，其在内存中的地址是不同的，在插入`hashMap`时，首先对比hash code（假设`Key`的类型是`Student`）就把这两个内容一样的对象（调用`equals()`返回`true`）当成不同的键插入到`HashMap`中，这是非常不合理的，所以在定义一个类时，如果这个类的对象将会是hash容器的元素，一定要使`equals()`和`HashMap()`相容.

