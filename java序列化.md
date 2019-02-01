[TOC]

### 什么是序列化 (Serialization)

百度百科官方解释：

序列化是将对象的状态信息转换为**可以存储或传输的形式**的过程。在序列化期间，对象将其**当前状态**写入到**临时或持久性存储区**。以后，可以通过从存储区中读取或反序列化对象的状态，**重新创建**该对象。

维基百科官方解释：

序列化是指将**数据结构或对象状态**转换成**可取用格式**（例如存成文件，存于缓冲，或经由网络中发送），以留待后续在**相同或另一台**计算机环境中，能**恢复原先状态**的过程。依照序列化格式**重新获取字节**的结果时，可以利用它来产生与原始对象相同语义的**副本**。从一系列字节提取数据结构的反向操作，是反序列化。

### 序列化的特点

平台无关性：在一个平台上序列化的对象可以在另一个完全不同的平台上反序列化该对象。

对于Java序列化来说整个过程都是 Java 虚拟机（JVM）独立的

### 序列化的应用场景

- 将对象变为持久化对象
- 远程方法调用（RMI）
- 网络传输

### Java中如何实现序列化

同样引用维基百科的解释：

Java 提供**自动序列化**，需要以java.io.Serializable接口的实例来标明对象（即该对象需要实现serializable接口）。实现接口将类别标明为“可序列化”，然后Java在内部处理序列化。在Serializable接口上并没有预先定义序列化的方法，但可序列化类别可任意定义某些特定名称和签署的方法，如果这些方法有定义了，可被调用运行序列化/反序列化部分过程。

**原生类型以及永久和非静态的对象引用**（*序列化不会关注静态变量*），会被编码到字节流之中。序列化对象引用的**每个对象**，**若其中未标明为transient的字段，也必须被序列化**(也就是说如果想要对象的某个字段不被序列化，就要将该字段表明为transient类型)；如果整个过程中，引用到的任何永久对象不能序列化，则这个过程会失败。开发人员可将对象标记为暂时的，或针对对象重新定义的序列化，来影响序列化的处理过程，以截断引用图的某些部分而不序列化。Java并不使用构造函数来序列化对象。

总的来说就是两点：

- 该类必须实现 java.io.Serializable 对象。

- 该类的**所有属性必须是可序列化**的。如果有一个属性不是可序列化的，则该属性**必须注明是短暂transient**的。

源码中也这样解释到：

>```
> The serialization interface has no methods or fields
>and serves only to identify the semantics of being serializable.
>```

### 哪些情况不能实现序列化

在默认情况下有三个主要原因使对象无法被序列化。

- 在序列化状态下并不是所有的对象都能获取到有用的语义。例如，Thread对象绑定到当前Java虚拟机的状态，对Thread对象状态的反序列化环境来说，没有意义。
- 其对象的序列化状态构成其类别兼容性缔结（compatibility contract）的某一部分。在维护可序列化类别之间的兼容性时，需要额外的精力和考量。所以，使类别可序列化需要慎重的设计决策而非默认情况
- 序列化允许访问类别的**永久私有成员**，包含**敏感信息（例如，密码）的类别不应该是可序列化**的，也不能外部化。

### 序列化的实现DEMO

创建一个user对象

```java
import java.io.Serializable;

public class Person implements Serializable {

    private static final long serialVersionUID = 7592930394427200495L;

    private int id ;
    private String name;
    private int age;
    private static int telephone;
    private transient String password;

    public static long getSerialVersionUID() {
        return serialVersionUID;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
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

    public static int getTelephone() {
        return telephone;
    }

    public static void setTelephone(int telephone) {
        Person.telephone = telephone;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", password='" + password + '\'' +
                '}';
    }
}

```



ObjectOutputStream对象进行序列化

```
ObjectOutputStream:Creates an ObjectOutputStream that writes to the specified OutputStream

```

```java
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;

public class SerializeDemo {
    public static void main(String[] args){

        Person person = new Person();
        person.setId(1);
        person.setAge(23);
        person.setPassword("1234");
        person.setName("kangkang");
        System.out.println("创建一个对象："+person);

        try {
            System.out.println("进行序列化，写入文件tempfile中");
            ObjectOutputStream fileOut = new ObjectOutputStream(new FileOutputStream("tempFile"));
            fileOut.writeObject(person);
            fileOut.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```
创建一个对象：Person{id=1, name='kangkang', age=23, password='1234'}
进行序列化，写入文件tempfile中
```



ObjectInputStream进行反序列化

```
ObjectInputStream:
Read an object from the ObjectInputStream.  The class of the object, the
* signature of the class, and the values of the non-transient and
* non-static fields of the class and all of its supertypes are read.
* Default deserializing for a class can be overridden using the writeObject
* and readObject methods.
ObjectInputStream执行从ObjectInputStream中读取一个对象，该对象的类，该类的签名，和所有非transient属性的值，以及非静态字段，和所有的父类都会被读取
```

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.ObjectInputStream;

public class DeserializationDemo {
    public static void main(String[] args) throws ClassNotFoundException {

        try {
            ObjectInputStream fileIn = new ObjectInputStream(new FileInputStream("tempFile"));
            Person p = (Person) fileIn.readObject();
            System.out.println("反序列化");
            System.out.println(p);
            fileIn.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

```
反序列化
Person{id=1, name='kangkang', age=23, password='null'}
```

可以看到这里的password字段都被设置成为了null，是因为private transient String password;中password字段为transient字段，所以该字段的值不会被序列化，而会被自动设置为null。如果我们非要将password也要进行序列化的话，不妨在序列化的过程中进行加密处理，就可以保证数据的安全性了。

小贴士😊

并不是只要被transient修饰就一定不能被序列化，如果该类实现的是Externalizable接口而不是Serializable接口的话，那么同样能够被序列化。因为Externalizable接口不再像Serializable那样自动序列化接口，而是人为指定想要序列化的变量。还要注意的是transient关键字只能用来修饰字段，不能用来修饰方法



参考博客

[深入分析Java的序列化与反序列化](https://www.hollischuang.com/archives/1140)
