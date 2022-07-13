# java中遇到的方法与函数

# 1、遇到的方法

##  split()---String数组的分割

Split(expression[, delimiter[, count[, compare]]])

Split函数语法有如下几部分：

| **部分**     | **必要性** | **描述**                                                     |
| ------------ | ---------- | ------------------------------------------------------------ |
| *expression* | 必需的。   | 包含子字符串和分隔符的字符串表达式。如果*expression*是一个长度为零的字符串("")，**Split**则返回一个空数组，即没有元素和数据的数组。 |
| *delimiter*  | 可选的。   | 用于标识子字符串边界的字符串字符。如果忽略，则使用空格字符(" ")作为分隔符。如果*delimiter*是一个长度为零的字符串，则返回的数组仅包含一个元素，即完整的*expression*字符串。 |
| *count*      | 可选的。   | 要返回的子字符串数，–1表示返回所有的子字符串。               |
| *compare*    | 可选的。   | 数字值，表示判别子字符串时使用的比较方式。关于其值，请参阅“设置值”部分。 |

使用：

~~~java
String str=“123￥45￥67￥8”;
strs[]=Stringstr.split("￥");
strs[0]=123;
strs[1]=45;
strs[2]=67;
strs[3]=8;
~~~



## matches()---String字符串的匹配字符串 

matches() 方法用于检测字符串是否匹配给定的正则表达式。

语法

```
public boolean matches(String regex)
```

参数

- regex -- 匹配字符串的正则表达式。

返回值

在字符串匹配给定的正则表达式时，返回 true。

实例

```
public class Test {
    public static void main(String args[]) {
        String Str = new String("www.runoob.com");

        System.out.print("返回值 :" );
        System.out.println(Str.matches("(.*)runoob(.*)"));
        
        System.out.print("返回值 :" );
        System.out.println(Str.matches("(.*)google(.*)"));

        System.out.print("返回值 :" );
        System.out.println(Str.matches("www(.*)"));
    }
}


String a="12-12+34-5/2";
        String[] b=a.split("- | / | \\+ ");
        for(int i=0;i<b.length;i++){
            System.out.println(b[i]);
        }
        boolean matches = b[0].matches("\\d");
        System.out.println(matches);
```

以上程序执行结果为：

```
返回值 :true
返回值 :false
返回值 :true
```



## peek--观察stack栈顶元素

~~~java
Stacck<String> s=new Stack<String>();
s.add("1");
s.add("2");
s.add("3");
s.add("4");
sout(s.peek);
结果：4
~~~



## Arrays.方法

### Arrays.sort(数组[])--排序

想用sort函数做排序，有俩种表现方法：
（设定好一个数组为num）
第一种：Arrays.sort(num);

第二种：Arrays.sort(num,起始的下标，想要排序的数的数量)；

样例：
我想要输出全部数组，那么为：Arrays.sort(count,0,n);

我想要排序3个数字，那么为Arrays.sort(count,0,3);（排序的对应下标为：0，1，2）



### Arrays.asList(数组[])--把数组转换为List

```java
int[] a={1,2,3,4}; 
List<Integer> list=Arrays.asList(a);

```



### Arrays.stream(数组).sum()/ --用流求int数组的sum值

```jav
输入:
int[] intArray = {1, 2, 3, 4, 5};
System.out.println(  Arrays.stream(intArray).sum()  );
输出:
15
```



### Arrays.stream(数组).average().getAsDouble() )--用流求int数组的average值

```java
输入:
int[] intArray = {1, 2, 3, 4, 5};
System.out.println(  Arrays.stream(intArray).average().getAsDouble()  );
输出:
3.0
```



### Arrays.asList,toArray--String数组与list相互转换

```java
	    String[] a={"1","2","3"};
        ArrayList<String> lsit = new ArrayList<>(Arrays.asList(a));
        System.out.println(lsit.get(0));
        System.out.println(lsit.get(1));
        lsit.toArray(a);

        for (int i = 0; i < a.length; i++) {
            System.out.println(a[i]);
        }

输出：
1
2
1
2
3
```





## Math.pow(a,b)--a的b次方

```java
Math.pow(a,b); 
```



## Collections.resver(List<> list)--反转元素

![1656935666063](C:\Users\zouzhao\AppData\Local\Temp\1656935666063.png)



## Collections.swap(List<> list, int i, int j)--交换元素

![1656938292172](C:\Users\zouzhao\AppData\Local\Temp\1656938292172.png)



## Math.abs()--绝对值

abs(param) 返回参数的绝对值。参数可以是 int, float, long, double, short, byte类型。 



## remove() 方法--删除动态数组里的单个元素 

~~~java
// 删除指定元素
arraylist.remove(Object obj)

// 删除指定索引位置的元素
arraylist.remove(int index)
~~~



## HashMap getOrDefault(Object key, V defaultValue)--获取指定 key 对应对 value

```java
		HashMap<Integer, String> sites = new HashMap<>();

        // 往 HashMap 添加一些元素
        sites.put(1, "Google");
        sites.put(2, "Runoob");
        sites.put(3, "Taobao");
        System.out.println("sites HashMap: " + sites);

        // key 的映射存在于 HashMap 中
        // Not Found - 如果 HashMap 中没有该 key，则返回默认值
        String value1 = sites.getOrDefault(1, "Not Found");
        System.out.println("Value for key 1:  " + value1);

        // key 的映射不存在于 HashMap 中
        // Not Found - 如果 HashMap 中没有该 key，则返回默认值
        String value2 = sites.getOrDefault(4, "Not Found");
        System.out.println("Value for key 4: " + value2);
```







# 2、其他指令 

## 刷新DNS解析缓存:ipconfig/flushdns 