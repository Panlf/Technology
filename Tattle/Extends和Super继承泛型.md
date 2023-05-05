# List<? extends T>与List<? super T>的区别

## 名词解释

### ？
？表示类型通配符，即具体传什么参数类型，在List定义时不用考虑。

### <T>
这里的 <> 表示泛型，T 表示泛型中装载的类型为T类型，等到需要的时候，我们可以具体这个 T。我们在使用动态数组实现 ArrayList 的时候，如果希望这个 ArrayList 不仅仅支持一个类型的话，我们可以给这个 ArrayList 定义泛型，泛型中存放的是T类型。在实际创建出这个 ArrayList 对象时，我们可以指定泛型中的具体类型。

### <? extends T>
类型上界，这里的 ? 可以是 T 类型或者 T 的子类类型。

### <? super T>
类型下界，这里的?可以是T类型或者T的超类类型，但不代表我们可以往里面添加任意超类类型的元素。

## 在List中引入通配符界限限制的假设

不管是`List<? extends T>`还是`List<? super T>`，如果能读取元素，那么这个元素一定能转化为 T 类型，注意不是强制类型转换，强制类型转换是容易出现问题。

显然`List<? extends T>`内都是 T 的子类类型，能够向上转型为 T 类型，因此该 list 可以读取。

而`List<? super T>`内可以是 T 的超类类型，T 的超类转 T 是有可能出现异常的。

那我干脆转化成 Object 类型不好吗，所有类的基类都是 Object，不属于强制类型转换。哥们，转换成 Object 了，那你还图个啥？转换为 Object 类型是没有意义的。

假设`List<? extends T>`能添加元素，那么需要满足添加的任意元素需要能够直接转化成 T 的任何一个子类，T 的子类 A 和子类 B 是不能相互转化的，显然该 list 是不能添加元素的。

假设`List<? super T>`能添加元素，那么同样需要满足添加的任意元素能够直接转化成 T 的任何一个超类。此时添加 T 的子类元素就能满足该要求，因为 T 的任意子类可以向上转型成 T 的任何超类。

## List<? extends T>

List<? extends T>是被设计用来读取数据的泛型，并且只能读取类型为 T 的元素。原因如下：

元素是可以进行向上转型的，因此，我们可以这样做来读取元素。
```
List<? extends Number> list = new ArrayList<>(); 
Number number = list.get(0);
```

可以读取，但不能写入，比如以下的代码就直接报错。
```
public class Main {

    static class A { }

    static class B extends A { }

    static class C extends A { }

    public static void main(String[] args) {
        List<? extends A> list = new ArrayList<>();
        list.add(new A());//编译报错
        list.add(new B());//编译报错
        list.add(new C());//编译报错
    }

}
```

A 的子类 B 与子类 C 是不能相互转换的，因此是不能往该 list 中添加元素。

虽然不能添加元素，但可以在初始化的时候，接受一个已经定义好的 list，而该 list 存放的类型一定相同。因此，List<? extends T>可直接接受一个定义好的 list。

```
public static List<Integer> getList(){
    List<Integer> list=new ArrayList<>();
    list.add(1);
    return list;
}

// ....

public static void main(String[] args) {
    List<? extends Number> list = new ArrayList<>();
    list=getList();
}
```

### List<? super T>

List<? super T>是被设计用来添加数据的泛型，并且只能添加 T 类型或其子类类型的元素。

为什么只能是 T 类型及其子类型元素，超类类型的元素不可以吗？

超类类型转化为 T 类型，是需要强制类型转换的，是容易出现异常的，无法保障的。

而传入 T 类型及其子类类型时，能够直接转化为 T 的任意超类类型。比如，下面的代码是可以运行的

```
public class Main {

    static class A { }

    static class B extends A { }

    static class C extends A { }

    public static void main(String[] args) {
        List<? super A> list = new ArrayList<>();
        list.add(new A());
        list.add(new B());
        list.add(new C());
    }
}
```

该 list 也可以读取其中的元素，从第二节可以得出，只能用 Object 接收，没多大意义。

```
List<? super Integer> list2 = new ArrayList<>();
list2.add(new Integer(1));
Object integer=list2.get(0);
```

如果我们使用 Object 类型来接收获取到的元素，那么元素本身的类型就会丢失，因此，我们不使用List<? super T>来获取元素。

如果我们非要使用List<? super Integer>中的 Integer 类型来接收获取到的元素，那么必须进行强制类型转换，是会出现异常的，无法保障。

```
List<? super Integer> list2 = new ArrayList<>();
list2.add(new Integer(1));
Integer integer1= (Integer) list2.get(0);
```

## 总结
（1）`List<? extends T>`适用于读取数据，读取出来的数据全部用T类型接收。如果我们往此 list 中添加 T 类型不同的子类的话，各种子类无法相互转换，因此不能添加元素，但可接受初始赋值。

（2）`List<? super T>`适用于添加元素，只能添加 T 类型或其子类类型。因为这些类型都能转换为T的任意超类类型（向上转型），因此我们可以对此 list 添加元素。只能用 Object 类型来接收获取到的元素，但是这些元素原本的类型会丢失。