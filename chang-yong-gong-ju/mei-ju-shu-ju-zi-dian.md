# 枚举数据字典

#### 数据字典

可通过枚举来定义数据字典,定义一个枚举,并实现`EnumDict`接口:

```java
@AllArgsConstructor
@Getter
@Dict(id="data-status") //定义一个id,默认为 DataStatusEnum.class.getSimpleName();
//支持反序列化
@JSONType(deserializer = EnumDict.EnumDictJSONDeserializer.class)
public enum DataStatusEnum implements EnumDict<Byte> {
    ENABLED((byte) 1, "正常"),
    DISABLED((byte) 0, "禁用"),
    LOCK((byte) -1, "锁定"),
    DELETED((byte) -10, "删除");

    private Byte value;

    private String text;
    
     @Override
    public boolean isWriteJSONObjectEnabled() {
        //在序列化json的时候，是否将枚举序列化为对象。默认为true。
        //可通过环境变量hsweb.enum.dict.disableWriteJSONObject修改全局默认值
        return true;
    }
}
```

在实体类中使用:

```java
@Data
public class User  {
    private String id;

    //单选
    private DataStatusEnum status;

    //多选
    private DataStatusEnum[] statusArr;
}
```

作用: 

1. 当值为单选,在持久化到数据库时,将自动存储字典的value值. 因此数据库字段的类型应该与value字段的类型一致. 

2. 当值为多选,并且枚举选项数量小于`64`个,则会将值进行位运算\(`EnumDict.toBit`\)后存储.在查询的时候也使用位运算进行查询. 因此数据库字段的类型应该为数字类型。 如: `where().in("statusArr",0,-1);` 则将生成sql : `where status_arr & {bit} != {bit}` 。 在java中可以通过`EnumDict`中的静态方法进行判断,如 `in` 和 `anyIn`. 

3. 当枚举选项数量大于等于`64`个的时候,需要自行实现存储和查询逻辑,可以使用中间表的方式,也可以使用hsweb自带的实现,模块:`hsweb-system/hsweb-system-dictionary`。

注意: 1,2的功能由`hsweb-commons-dao`模块去实现,如果你不没有使用hsweb自带的dao实现,可能无法使用此功能.

所有的字典都会注册到:`DictDefineRepository`,可通过此类去获取字典,以提供给前端或者其他地方使用.

在前端可通过接口:  

`dictionary/define/{id}` 获取数据字段信息。

`dictionary/define/{id}/items` 获取字典选项信息。

