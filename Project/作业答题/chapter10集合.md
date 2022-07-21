# work

# chapter10集合

1	有些可以重复，有些不可以

​		有	可以

​	无	不可以

​	键值对	值value	键key



2	

- I

~~~java
 System.out.println(list);
~~~

- II

  [Hello,Java,Learn,World]

- |||

  ~~~java
  List<String> list =new LinkedList<>();
  使用上：
      大体方法相同，linkedList多了首尾添加删除方法
  实现：
      ArrayList是依据array实现的，利于随机访问
      LinkedList是依据链表实现的，利于增删
  ~~~

- 略



3	Hello

​		Learn



4	C



5	

Worker.class

~~~java
	package question5;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public class Worker {
    private String name;
    private int age;
    private double salary;

    public static void main(String[] args) {
        //第一问
        List<Worker> list=new ArrayList<>();
        Worker w1 = new Worker("zhang3", 18, 3000);
        Worker w2 = new Worker("li4", 25, 3500);
        Worker w3 = new Worker("wang5", 22, 3200);
        list.add(w1);
        list.add(w2);
        list.add(w3);
        //第二问
        list.add(1,new Worker("zhao6",24,3300));
        //第三问
        list.remove(w3);
        //第四问
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i).name+" "+list.get(i).age+" "+list.get(i).salary);
        }
        //第五问
        for (Worker worker : list) {
            worker.work();
        }

    }
    //第六问
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Worker worker = (Worker) o;
        return age == worker.age && Double.compare(worker.salary, salary) == 0 && Objects.equals(name, worker.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, salary);
    }

    public void work(){
        System.out.println(name+"work");
    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Worker(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public Worker() {
    }
}

~~~



6	B



7	重写equeals错误

~~~java
代码跟第五题一样
~~~



8	

~~~java
package question5_8;

import java.util.*;

public class Worker {
    private String name;
    private int age;
    private double salary;

    public static void main(String[] args) {
        //第8题
        Set<Worker> set=new HashSet<>();
        Worker z1 = new Worker("zhao6", 24, 3300);
        Worker z2 = new Worker("zhao7", 24, 3300);
        Worker z3 = new Worker("zhao6", 24, 3300);
        Worker z4= new Worker("zhao8", 24, 3300);
        System.out.println(z1.hashCode());
        System.out.println(z3.hashCode());
        System.out.println(z1.equals(z3));
        set.add(z1);
        set.add(z2);
        set.add(z3);
        set.add(z4);
        for (Worker worker : set) {
            System.out.println(worker);
        }

    }
    //第六问
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Worker worker = (Worker) o;
        return age == worker.age && Double.compare(worker.salary, salary) == 0 && Objects.equals(name, worker.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age, salary);
    }

    @Override
    public String toString() {
        return "Worker{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", salary=" + salary +
                '}';
    }

    public void work(){
        System.out.println(name+"work");
    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Worker(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public Worker() {
    }
}

~~~



1、重写hashcode是为了保证两个对象内容相等（equals）时哈希表key唯一的原则

2、实际上重写了hashcode的两个对象的内存地址还是不相同的



9	覆盖	插入

​	1	删除

​	获取key的val	key	key对应的value

​	 keySet	set<Key>

​	values value



10

~~~java
package question10;

import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class TestMap {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        int i = scan.nextInt();
        Map<Integer, String> map = new HashMap<>();
        map.put(2006, "意大利");
        map.put(2002, "巴西");
        map.put(1998, "法国");
        if (map.get(i)!=null) {
            System.out.println(map.get(i));
        } else {
            System.out.println("没有");
        }
    }
}

~~~



11

~~~java
package question11;

import java.util.HashMap;
import java.util.Map;

public class TestMap {
    public static void main(String[] args) {
        Map<String,String> map=new HashMap<>();
        map.put("Tom","CoreJava");
        map.put("John","Oracle");
        map.put("Susan","Oracle");
        map.put("Jerry","JDBC");
        map.put("Jim","Unix");
        map.put("Kevin","JSP");
        map.put("Lucy","JSP");
        //2
        map.put("Allen","JDBC");
        //3
        map.replace("Lucy","CoreJava");
        //4
        for (String s : map.keySet()) {
            System.out.println(s+":"+map.get(s));
        }
        //5
        System.out.println("====");
        for (String s : map.keySet()) {
            if (map.get(s).equals("JSP"))System.out.println(s+":"+map.get(s));
        }
    }
}

~~~



12

b	hashcode会报错，重写了hashcode，stu1的name为空



13

TestMap.class

~~~java
package question13;

import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class TestMap {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        String s = scan.next();
        Map<Integer, String> map = new HashMap<>();
        map.put(2006, "意大利");
        map.put(2002, "巴西");
        map.put(1998, "法国");
        map.put(2003, "巴西");
        map.put(2005, "巴西");
        boolean flag=true;
        //第一种方法
//        for (Integer integer : map.keySet()) {
//            if (map.get(integer).equals(s)) {
//                System.out.println(integer + "、");
//                i++;
//            }
//        }
//        if (i==0) System.out.println("没有获得冠军");
        //第二种方法
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            if (entry.getValue().equals(s)) {
                System.out.println(entry.getKey());
                flag=false;
            }
        }if (flag) System.out.println("没有获得过世界杯");
    }
}

~~~



14

TestMap.class

~~~java
package question14;

import java.util.HashMap;
import java.util.Map;

public class TestMap {
    public static void main(String[] args) {
        String s="aaasfadgadgsdg";
        Map<Character,Integer> map=new HashMap<>();
        for (int i = 0; i < s.length(); i++) {
            if (map.get(s.charAt(i))==null)map.put(s.charAt(i),1);
            else map.put(s.charAt(i),(map.get(s.charAt(i))+1) );
        }
        System.out.println(map.entrySet());
    }
}
~~~

