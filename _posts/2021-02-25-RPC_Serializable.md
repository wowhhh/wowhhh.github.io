---
layout: post
title:  "kryo"
tags:   RPC
date:   2021-02-25 10:00:00 +0800
categories: [RPC]

---

**序列化的主要目的就是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中**

## 使用场景

- 对象在进行**网络传输**，比如远程方法调用RPC，之前需要先被序列化，接收到序列化的对象之后需要再进行反序列化
- 将对象**存储到文件**中需要进行序列化，将对象从文件中读取出来需要进行反序列化
- 将对象**存储到缓存数据库**时候需要序列化，将对象从缓存数据库中读取出来的时候需要反序列化

## 序列化协议对应于TCP/IP 4层模型中的那一层？

OSI七层协议都有：

>应用层：为计算机用户提供服务
>
>表示层：数据处理
>
>会话层：管理应用程序之间的会话
>
>传输层：为两台主机进程之间的通信提供通用的数据传输服务
>
>网络层：路由和寻址
>
>数据链路层：管理相邻节点之间的数据通信
>
>物理层

OSI七层协议模型中，表示层所做的事情就是对应用层的用户数据进行处理，转化成二进制流。反过来，就是将二进制流转化成应用层的用户数据。

OSI七层协议模型中的应用层、表示层和会话层都对应于TCP/IP四层模型中的应用层，所以序列化协议属于TCP/IP协议应用层的一部分。

## 常见序列化协议

#### JDK自带的序列化方式

序列化效率低，部分版本有安全漏洞

- 简单介绍
- 如何使用
- 缺点

#### kryo

如何使用kryo完成序列化的操作呢？

```java
class Person implements Serializable
{
    String name;
}
public class Test01 {
    public static void main(String[] args) throws Exception {
        Kryo kryo = new Kryo();
        kryo.register(Person.class);
        Person person = new Person();
        person.name = "qaq";

        Output output = new Output(new FileOutputStream("file.bin"));
        kryo.writeObject(output,person);
        output.close();

        Input  input = new Input(new FileInputStream("file.bin"));
        Person p = kryo.readObject(input,Person.class);
        input.close();
        assert "qaq".equals(p.name);
    }
}
```

- 如何确保kyro是线程安全的

  kryo不是线程安全的，针对多线程的情况，要对kryo做封装。可以每当需要序列化和反序列化的时候都实例化一次，或者是借助ThreadLocal来维护。

我们的RPC项目里面是如何使用到序列化和反序列化呢？

```java
/**
 * Kryo序列化类，Kryo的序列化效率很高，但是只与java语言兼容
 */

@Slf4j
public class KryoSerializer implements Serializer {

    /**
     * kryo不是线程安全的，针对多线程的情况，要对kryo做封装。可以每当需要序列化和反序列化的时候都实例化一次，或者是借助ThreadLocal来维护。
     * 另外：ThreadLocal.withInitial为ThreadLocal的Lambda构造方式
     */
    private final ThreadLocal<Kryo> kryoThreadLocal = ThreadLocal.withInitial(() -> {
        Kryo kryo = new Kryo();
        kryo.register(RpcResponse.class);//在使用Kryo序列化之前需要将序列化的类通过register()方法注册到其中去
        kryo.register(RpcRequest.class);//同上
        /*
        表示对循环引用的支持，此属性默认是开启的，如果能够确定不会有循环引用的发生，可以设置为false，设置成false的话，就是关闭了循环引用的检测，效率和性能都会提高.
        但是如果遇到循环应用，就会报"栈内存溢出"的错误
         */
        kryo.setReferences(true);
        /*
        默认值为false，表示是否关闭注册行为，false就是不关闭，推荐不关闭，关闭后可能存在序列化问题
        如果设置为true，会在遇到任何未注册的类的时候抛出异常。
         */
        kryo.setRegistrationRequired(false);
        return kryo;
    });

    /**
     * 对传入的对象进行序列化
     * @param obj 要序列化的对象
     * @return
     */
    @Override
    public byte[] serialize(Object obj) {
        try (ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
                Output output = new Output(byteArrayOutputStream)) {
            Kryo kryo = kryoThreadLocal.get();//获取Kryo对象
            kryo.writeObject(output, obj);// Object->byte:将对象序列化为byte数组
            kryoThreadLocal.remove();
            return output.toBytes(); //返回序列化完毕的对象
        } catch (Exception e) {
            throw new SerializeException("Serialization failed");
        }
    }

    /**
     * 反序列化
     * @param bytes 序列化后的字节数组
     * @param clazz 目标类
     * @param <T>
     * @return
     */
    @Override
    public <T> T deserialize(byte[] bytes, Class<T> clazz) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
                Input input = new Input(byteArrayInputStream)) {
            Kryo kryo = kryoThreadLocal.get();
            // byte->Object:从byte数组中反序列化出对对象
            Object o = kryo.readObject(input, clazz); //byte-class：将字节数组转化为对象
            kryoThreadLocal.remove();
            return clazz.cast(o);
        } catch (Exception e) {
            throw new SerializeException("Deserialization failed");
        }
    }

}
```

#### protobuf

#### protostuff

#### hession

## 参考

https://www.cnkirito.moe/rpc-serialize-1/

#### 