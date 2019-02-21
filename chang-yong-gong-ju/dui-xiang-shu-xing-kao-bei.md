# 对象属性拷贝

`FastBeanCopier`类. 提供高效的bean复制.支持复杂结构,类型转换,集合泛型,支持bean到map,map到bean的复制.

原理: 使用工具类`Proxy`,通过`javassist`去动态构造一个类,通过原生的方式调用get set方法.而不是通过低效的反射.

```java
 //将source对象中的属性复制到target中.
 FastBeanCopier.copy(source,target);

 //将source对象中的属性复制到target中.不复制id字段
 FastBeanCopier.copy(source,target,"id");
  //只复制id属性
 FastBeanCopier.copy(source,target,FastBeanCopier.include("id"));
 
```

约定: 如果属性类实现了`Cloneable`接口,在复制的时候将调用`clone`方法.所以如果你实现了`Cloneable`接口,就必须重写`clone`方法并且为`public`修饰的.

