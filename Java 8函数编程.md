### 1. 为什么引入函数编程
为了编写这类处理批量数据的并行类库，需要在语言层面上修改现有的Java：**增加Lambda表达式**。

### 2. 什么是函数编程
在思考问题时，使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

#### 2.1 高阶函数
高阶函数是指接受另外一个函数作为参数或返回一个函数的函数。

高阶函数不难辨认：看函数签名就够了。如果函数的参数列表里包含函数接口或该函数返回一个函数接口，那么该函数就是高阶函数。

#### 2.2 副作用
所谓"副作用"（side effect），指的是函数内部与外部互动（最典型的情况，就是修改全局变量的值），产生运算以外的其他结果。

函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值.

### 3. Lambda表达式
Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

Lambda表达式通过**invokedynamic**指令实现，书写Lambda表达式不会产生新的类。

lambda 表达式的语法格式如下：
```
(parameters) -> expression
```
```
(parameters) -> { statements; }
```

#### 3.1 Lambda表达式的不同形式
```
# 无参数
Runnable noArguments = () -> System.out.println("Hello World");
```

```
# 只包含一个参数，可省略参数的括号
ActionListener oneArgument = event -> System.out.println("button clicked");
```

```
# 主体不仅可以是一个表达式，而且也可以是一段代码块
Runnable multiStatement = () -> {
    System.out.print("Hello");
    System.out.println(" World");
};
```

```
# 两个参数
BinaryOperator<Long> add = (x, y) -> x + y;
```

```
# 显式声明参数类型
BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;
```

Lambda 表达式的类型依赖于上下文环境， 是由编译器推断出来的。 目标类型也不是一个全新的概念, Java 中初始化数组时， 数组的类型就是根据上下文推断出来的。
 
 
#### 3.2 方法引用

通常与Lambda表达式联合使用，可以直接引用已有Java类或对象的方法。

方法引用的标准形式是：类名::方法名。（注意：只需要写方法名，不需要写括号）

有以下四种形式的方法引用：


类型 |	示例
--- |----
引用静态方法  |  ContainingClass::staticMethodName
引用某个对象的实例方法 | ContainingObject::instanceMethodName
引用某个类型的任意对象的实例方法 | ContainingType::methodName
引用构造方法 | ClassName::new

凡是使用Lambda 表达式的地方，就可以使用方法引用。

#### 3.3 函数接口

函数接口是只有一个抽象方法的接口，用作Lambda 表达式的类型。

##### @FunctionalInterface注解
该注释会强制javac 检查一个接口是否符合函数接口的标准。如果该注释添加给一个枚举类型、类或另一个注释，或者接口包含不止一个抽象方法，javac就会报错。重构代码时，使用它能很容易发现问题。

### 4. 流

Stream 是用函数式编程方式在集合类上进行复杂操作的工具。

#### 4.1 惰性求值和及早求值

如果返回值是Stream，
那么是惰性求值；如果返回值是另一个值或为空，那么就是及早求值。

通用形式为：

> Stream.惰性求值.惰性求值. ... .惰性求值.及早求值

#### 4.2 内部迭代和外部迭代
- 外部迭代，顾名思义，就是迭代是发生在Collection外层，由外部程序控制。
```
List<String> alphabets = Arrays.asList(new String[]{"a","b","b","d"});
 
for(String letter: alphabets){
    System.out.println(letter.toUpperCase());
}
```
```
List<String> alphabets = Arrays.asList(new String[]{"a","b","b","d"});
 
Iterator<String> iterator = alphabets.listIterator();
while(iterator.hasNext()){
    System.out.println(iterator.next().toUpperCase());
}
```
- 对于内部迭代，表示迭代完全有Collection内部控制，进行迭代。
```
List<String> alphabets = Arrays.asList(new String []{"a","b","b","d"});

alphabets.forEach(l -> System.out.println(l.toUpperCase()));

```

#### 4.3 创建stream

- 使用Stream.of(val1,val2,val3...)
```
Stream<Integer> stream = Stream.of(1,2,3,4,5,6,7,8,9);
stream.forEach(p -> System.out.println(p));

Stream<Integer> stream = Stream.of(1,2,3,4,5,6,7,8,9);
stream.forEach(System.out::println);
```
- 使用 Stream.of(arrayOfElements)
```
Stream<Integer> stream = Stream.of(new Integer[]{1,2,3,4,5,6,7,8,9} );
stream.forEach(p -> System.out.println(p));

List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
Stream<List<Integer>> listStream = Stream.of(list);
listStream.forEach(System.out::println);
# output:[1, 2, 3]
```

- 使用 List.stream()
```
Stream<Integer> stream = list.stream();
stream.forEach(p -> System.out.println(p));

int[] ints = new int[]{1,3,4,2,5};
IntStream intStream= Arrays.stream(ints);
intStream.forEach(System.out::print);
```

- 使用Stream.generator()

```
Stream<Date> stream = Stream.generate(() -> { return new Date();});
stream.forEach(p -> System.out.println(p));

```

- 使用String.chars() 或者 tokens
```
IntStream stream = "12345_abcdefg".chars();
stream.forEach(p -> System.out.println(p));
 
//OR
 
Stream<String> stream = Stream.of("A$B$C".split("\\$"));
stream.forEach(p -> System.out.println(p));
```


#### 4.4 处理顺序
```
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return true;
    })
    .forEach(s -> System.out.println("forEach: " + s));
```
处理结果
```
filter:  d2
forEach: d2
filter:  a2
forEach: a2
filter:  b1
forEach: b1
filter:  b3
forEach: b3
filter:  c
forEach: c 
```



操作类 | 接口方法
--- | ---
中间操作 |	concat distinct  filter flatMap limit map peek skip sorted parallel sequential unordered
结束操作 | allMatch anyMatch collect count findAny findFirst forEach forEachOrdered max min noneMatch reduce toArray

#### 4.5 中间操作
中间操作返回的是Stream本身，所以多个中间操作可以作为处理链处理Stream



#### 1. map
将一个流中的值转换成一个新的流
```
List<String> collected = Stream.of("a", "b", "hello")
                               .map(string -> string.toUpperCase())
                               .collect(toList());
```

mapToInt将Stream中的元素转换成int类型的
```
Stream.of(1, 2, 3).mapToInt(data -> data * 10).forEach(System.out::println);
```
mapToLong和mapToDouble与此类似。


#### 2. filter
对stream中所有元素进行过滤
```
List<String> beginningWithNumbers = 
        Stream.of("a", "1abc", "abc1")
              .filter(value -> isDigit(value.charAt(0)))
              .collect(toList());
```
#### 3. flatMap
可用 Stream 替换值，然后将多个 Stream 连接成一个 Stream
```
List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
                               .flatMap(numbers -> numbers.stream())
                               .collect(toList());
```
- flatMapToInt
- flatMapToLong
- flatMapToDouble

flatMap和map的区别
```
List<String> list2 = Arrays.asList("hello", "hi", "你好");
List<String> list3 = Arrays.asList("zhangsan", "lisi", "wangwu", "zhaoliu");

list2.stream()
        .map(item -> list3.stream().map(item2 -> item + " " + item2))
        .collect(Collectors.toList())
        .forEach(stream -> stream.forEach(System.out::println));
System.out.println("----------");
list2.stream()
        .flatMap(item -> list3.stream().map(item2 -> item + " " + item2))
        .collect(Collectors.toList())
        .forEach(System.out::println);
```
#### 4. sorted
返回一个排序的Stream 视图，sorted() 默认自然排序，同时也可以定制Comparator
```
memberNames.stream().sorted()
                    .map(String::toUpperCase)
                    .forEach(System.out::println);
```
自然排序
```
list.stream().sorted() 
```

自然序逆序元素，使用Comparator 提供的reverseOrder() 方法

```
list.stream().sorted(Comparator.reverseOrder()) 
```
使用Comparator 来排序一个list

```
list.stream().sorted(Comparator.comparing(Student::getAge)) 

```
把上面的元素逆序
```
list.stream().sorted(Comparator.comparing(Student::getAge).reversed()) 

```
list直接排序
```
list.sort(Comparator.comparing(Integer::intValue));

list.sort(Comparator.comparing(Integer::intValue).reversed());

list.sort(Comparator.comparing(Student::getAge));

list.sort(Comparator.comparing(Student::getAge).reversed());
```
#### 5. limit
对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素

#### 6. skip
返回一个丢弃原Stream的前N个元素后剩下元素组成的新Stream，如果原Stream中包含的元素个数小于N，那么返回空Stream.

#### 7. peek
peek方法也是接收一个Consumer功能型接口，它与forEach的区别就是它会返回Stream接口，也就是说forEach是一个Terminal操作，而peek是一个Intermediate操作，forEach完了以后Stream就消费完了，不能继续再使用，而peek还可以继续使用。
```
Stream<String> stream = Stream.of("I", "love", "you");
        stream.peek(System.out::println).forEach(System.out::println);
```
输出结果：
```
I
I
love
love
you
you
```

#### 8. distinct
对于Stream中包含的元素进行去重操作（去重逻辑依赖元素的equals方法），新生成的Stream中没有重复的元素

```
List<Integer> numList = Arrays.asList(1, 1, null, 2, 3, 4, null, 5, 6, 7, 8, 9, 10);
System.out.println("sum is " + numList.stream().filter(num -> num != null).
        distinct().mapToInt(num -> num * 2).
        peek(System.out::println).skip(2).limit(4).sum());
```

#### 9. concat
将两个Stream合并成一个，这个方法一次只能用来合并两个Stream，不能一次多个Stream合并。
```
Stream<Integer>stream1=Arrays.asList(1,2,3).stream();
Stream<String>stream2=Arrays.asList("a","b","c").stream();
Stream.concat(stream1,stream2).forEach(System.out::print);
```
#### 流复用
Java 8 streams不能被复用，当你执行完任何一个最终操作（terminal operation）的时候流就被关闭了。
```
Stream<String> stream =
    Stream.of("d2", "a2", "b1", "b3", "c")
        .filter(s -> s.startsWith("a"));

stream.anyMatch(s -> true);    // ok
stream.noneMatch(s -> true);   // exception
```
可以通过为每个最终操作（terminal operation）创建一个新的stream链的方式来解决上面的重用问题，Stream api中已经提供了一个stream supplier类来在已经存在的中间操作（intermediate operations ）的stream基础上构建一个新的stream。

```
Supplier<Stream<String>> streamSupplier =
    () -> Stream.of("d2", "a2", "b1", "b3", "c")
            .filter(s -> s.startsWith("a"));

streamSupplier.get().anyMatch(s -> true);   // ok
streamSupplier.get().noneMatch(s -> true);  // ok
```
streamSupplier的每个get()方法会构造一个新的stream，我们可以在这个stream上执行期望的最终操作（terminal operation）。

#### 4.6 终止操作
终止操作会返回一个特定的类型，而不是返回Stream本身。

#### collect
collect(toList()) 方法由 Stream 里的值生成一个列表，是一个及早求值操作。可以理解为 Stream 向 Collection 的转换。

注意这边的 toList() 其实是 Collectors.toList()，因为采用了静态倒入，看起来显得简洁。

```
List<String> collected = Stream.of("a", "b", "c")
                               .collect(Collectors.toList());
```

##### toList
toList收集器通过使用List的add方法将元素添加到一个结果List列表中，toList收集器使用ArrayList作为List的实现。

##### toSet
toSet 方法采用HashSet作为Set的实现来储存结果集。

##### toMap
```
List<Person> list = new ArrayList<>();
list.add(new Person(1, "haha"));
list.add(new Person(2, "rere"));
list.add(new Person(3, "fefe"));
Map<Integer, Person> map = list.stream()
        .collect(Collectors.toMap(Person::getId, Function.identity()));
System.out.println(map);
Map<Integer, String> newmap = list.stream()
        .collect(Collectors.toMap(Person::getId, Person::getName));
System.out.println(newmap);
```
Function.identity()-->返回stream中的元素

使用Collectors.toMap方法时的两个问题： 

1、当key重复时，会抛出异常：java.lang.IllegalStateException: Duplicate key **

2、当value为null时，会抛出异常：java.lang.NullPointerException
```
List<User> userList = new ArrayList<>();
userList.add(new User(1L, "aaa"));
userList.add(new User(2L, "bbb"));
userList.add(new User(3L, "ccc"));
userList.add(new User(2L, "ddd"));
userList.add(new User(3L, "eee"));
```
```
Map<Long, String> map = userList.stream()
        .collect(Collectors.toMap(User::getId, User::getUserName, (v1, v2) -> v1));
System.out.println(map);
```
```
{1=aaa, 2=bbb, 3=ccc}

```
```
Map<Long, String> map = userList.stream()
    .collect(Collectors.toMap(User::getId, User::getUserName, (v1, v2) -> v2));
System.out.println(map);
```
```
{1=aaa, 2=ddd, 3=eee}
```
#### foreach
遍历stream所有的元素
```
memberNames.forEach(System.out::println);
```
```
// 使用forEach()结合Lambda表达式迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.forEach((k, v) -> System.out.println(k + " = " + v));
```
```
# getOrDefault()
该方法跟Lambda表达式没关系，但是很有用。方法签名为V getOrDefault(Object key, V defaultValue)，作用是按照给定的key查询Map中对应的value，如果没有找到则返回defaultValue。使用该方法程序员可以省去查询指定键值是否存在的麻烦．
// 查询Map中指定的值，不存在时使用默认值
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
// Java7以及之前做法
if(map.containsKey(4)){ // 1
    System.out.println(map.get(4));
} else {
    System.out.println("NoValue");
}
// Java8使用Map.getOrDefault()
System.out.println(map.getOrDefault(4, "NoValue")); // 2

```

forEachOrdered
```
List<String> list = Arrays.asList("x", "y", "z");

list.parallelStream().forEach(System.out::print);
System.out.println();
list.parallelStream().forEachOrdered(System.out::print);
```
#### match
用来判断Stream中的某个元素是否符合某项断言

Stream 有三个 match 方法，从语义上说：

- allMatch：Stream 中全部元素符合传入的 predicate，返回 true
- anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
- noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

```
boolean matchedResult = memberNames.stream()
                    .anyMatch((s) -> s.startsWith("A"));
 
System.out.println(matchedResult);
 
matchedResult = memberNames.stream()
                    .allMatch((s) -> s.startsWith("A"));
 
System.out.println(matchedResult);
 
matchedResult = memberNames.stream()
                    .noneMatch((s) -> s.startsWith("A"));
```
#### count
统计stream的元素个数，返回long
```
long totalMatched = memberNames.stream()
                    .filter((s) -> s.startsWith("A"))
                    .count();
```


#### max和min 

求最大值和最小值

```
List<Integer> list = Lists.newArrayList(3, 5, 2, 9, 1);
int maxInt = list.stream()
                 .max(Integer::compareTo)
                 .get();
int minInt = list.stream()
                 .min(Integer::compareTo)
                 .get();
```
#### sum
求和
```
int sum = Stream.of(1, 2, 4).mapToInt(Integer::intValue).sum();

or

int sum = Stream.of(1, 2, 4).mapToInt(x -> x.intValue()).sum();
```
#### reduce
reduce 操作可以实现从一组值中生成一个值。
```
int result = Stream.of(1, 2, 3, 4)
                   .reduce(0, (acc, element) -> acc + element);
```
```
Optional<String> reduced = memberNames.stream()
                    .reduce((s1,s2) -> s1 + "#" + s2);
```
```
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
                filter(x -> x.compareTo("Z") > 0).
                reduce("", String::concat);
```
#### summaryStatistics
```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
IntSummaryStatistics stats = numbers
        .stream()
        .mapToInt((x) -> x)
        .summaryStatistics();
System.out.println(stats.toString());
System.out.println("List中最大的数字 : " + stats.getMax());
System.out.println("List中最小的数字 : " + stats.getMin());
System.out.println("所有数字的总和   : " + stats.getSum());
System.out.println("所有数字的平均值 : " + stats.getAverage());
```
#### 例子

```
List<String> biu = Stream.of("a", "b", "hello").map(String::toUpperCase).collect(toList());
System.out.println(biu.toString());

List<Transaction> groceryTransactions = new Arraylist<>();
for(Transaction t: transactions){
    if(t.getType() == Transaction.GROCERY){
        groceryTransactions.add(t);
    }
}
Collections.sort(groceryTransactions, new Comparator(){
    public int compare(Transaction t1, Transaction t2){
        return t2.getValue().compareTo(t1.getValue());
    }
});
List<Integer> transactionIds = new ArrayList<>();
for(Transaction t: groceryTransactions){
    transactionsIds.add(t.getId());
}
```
#### Java 8 的排序、取值实现
```
List<Integer> transactionsIds = transactions.parallelStream().
 filter(t -> t.getType() == Transaction.GROCERY).
 sorted(comparing(Transaction::getValue).reversed()).
 map(Transaction::getId).
 collect(toList());
```
### 5. 收集器
收集器可用来计算流的最终值，是 reduce 方法的模拟。
```
// 指定收集类型
public void toCollectionTreeset() {
    Stream<Integer> stream = Stream.of(1, 2, 3);
    stream.collect(Collectors.toCollection(TreeSet::new));
}

// 最大值
public Optional<Artist> biggestGroup(Stream<Artist> artists) {
    Function<Artist, Long> getCount = artist -> artist.getMembers().count();
    return artists.collect(Collectors.maxBy(comparing(getCount)));
}

// 平均值
public double averageNumberOfTracks(List<Album> albums) {
    return albums.stream()
            .collect(Collectors.averagingInt(album -> album.getTrackList().size()));
}

// 数组分块，使用Predicate函数接口判断，ture一块；false一块
public Map<Boolean, List<Artist>> bandsAndSolo(Stream<Artist> artists) {
    return artists.collect(Collectors.partitioningBy(Artist::isSolo));
}

// 数据分组
public Map<Artist, List<Album>> albumsByArtist(Stream<Album> albums) {
    return albums.collect(Collectors.groupingBy(Album::getMainMusician));
}

// 字符串合并
public static String formatArtists(List<Artist> artists) {
    return artists.stream()
            .map(Artist::getName)
            .collect(Collectors.joining(", ", "[", "]"));
}

// 分组后获取每组数量的总数
public Map<Artist, Long> numberOfAlbums(Stream<Album> albums) {
    return albums.collect(Collectors.groupingBy(Album::getMainMusician, Collectors.counting()));
}

// 分组后获取每组数据中的映射数据
public Map<Artist, List<String>> nameOfAlbums(Stream<Album> albums) {
    return albums.collect(Collectors.groupingBy(Album::getMainMusician,
            Collectors.mapping(Album::getName, Collectors.toList())));
}
```
```
Map<Integer, List<Person>> personGroups = Stream.generate(new PersonSupplier()).
 limit(100).
 collect(Collectors.groupingBy(Person::getAge));
Iterator it = personGroups.entrySet().iterator();
while (it.hasNext()) {
 Map.Entry<Integer, List<Person>> persons = (Map.Entry) it.next();
 System.out.println("Age " + persons.getKey() + " = " + persons.getValue().size());
}
```
```
public static void main(String[] args) {
    List<String> languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");

    System.out.println("Languages which starts with J :");
    filter(languages, (str) -> str.startsWith("J"));

    System.out.println("Languages which ends with a ");
    filter(languages, (str) -> str.endsWith("a"));

    System.out.println("Print all languages :");
    filter(languages, (str) -> true);

    System.out.println("Print no language : ");
    filter(languages, (str) -> false);

    System.out.println("Print language whose length greater than 4:");
    filter(languages, (str) -> str.length() > 4);
}

public static void filter(List<String> names, Predicate<String> condition) {
    names.stream().filter((name) -> (condition.test(name)))
            .forEach((name) -> System.out.println(name + " "));
}
```
Output
```
Languages which starts with J :
Java 
Languages which ends with a 
Java 
Scala 
Print all languages :
Java 
Scala 
C++ 
Haskell 
Lisp 
Print no language : 
Print language whose length greater than 4:
Scala 
Haskell 
```

#### 总结
总之，Stream 的特性可以归纳为：

- 不是数据结构
- 它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。
- 它也绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。
- 所有 Stream 的操作必须以 lambda 表达式为参数
- 不支持索引访问
- 你可以请求第一个元素，但无法请求第二个，第三个，或最后一个。不过请参阅下一项。
- 很容易生成数组或者 List
- 惰性化
- 很多 Stream 操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始。
Intermediate 操作永远是惰性化的。
- 并行能力
当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。
- 可以是无限的
    - 集合有固定大小，Stream 则不必。limit(n) 和 findFirst() 这类的 short-circuiting 操作可以对无限的 Stream 进行运算并很快完成。


### 6. 测试或调试
peek 方法能记录中间值，在调试时非常有用。

对 Stream API 的调试，IDEA 官方开发了一个 Plugin──Java Stream Debugger 来扩展 IDEA 中的 debug 工具。在 debug 的工具栏上增加了 Trace Current Stream Chain 按钮。

这个工具超级厉害！