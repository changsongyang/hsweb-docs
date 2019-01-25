# 动态SQL条件

## 说明

hsweb定义了一种支持动态条件的数据结构:

```javascript
{  //where name like 张%
    terms:[
        {
            column:"name", //属性名或者列名
            value:"张", //值,为null或者为""时,此条件将被忽略
            type:"and", // 连接类型,and 或者 or
            termType:"like", // like
            options:["startWith"], //可选项,不同的termType支持不同.
            terms:[
                //嵌套,结构同上
                
            ]
        }
    ]
}
```

对应的Java类有: `QueryParamEntity`，`UpdateParamEntity`，`DeleteParamEntity` 

## DSL

hsweb提供了通过DSL的方式来构造上述结构，用例:

如果继承了GenericEntityService的类可以直接使用DSL进行查询,修改,删除操作

```java
//select * from table where name = ? limit 0,20
createQuery().where("name",name).list()
// select * from table where name like ?||'%';
// 构造并调用其他的dao
DefaultDSLQueryService.createQuery(userDao)
                .where("name$like$startWith",name)
                .listNoPaging();
```

{% hint style="info" %}
 上述例子中 name$like$startWith的含义: name\(列\), like\(条件\),startWith\(条件选项\)
{% endhint %}

## 默认支持的SQL条件

<table>
  <thead>
    <tr>
      <th style="text-align:left">termType</th>
      <th style="text-align:left">SQL</th>
      <th style="text-align:left">DSL</th>
      <th style="text-align:left">说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">is 或者 eq</td>
      <td style="text-align:left">=</td>
      <td style="text-align:left">.is()</td>
      <td style="text-align:left">等于</td>
    </tr>
    <tr>
      <td style="text-align:left">not</td>
      <td style="text-align:left">!=</td>
      <td style="text-align:left">.not()</td>
      <td style="text-align:left">不等于</td>
    </tr>
    <tr>
      <td style="text-align:left">gt</td>
      <td style="text-align:left">></td>
      <td style="text-align:left">.gt()</td>
      <td style="text-align:left">大于</td>
    </tr>
    <tr>
      <td style="text-align:left">lt</td>
      <td style="text-align:left">
        <</td>
          <td style="text-align:left">.lt()</td>
          <td style="text-align:left">小于</td>
    </tr>
    <tr>
      <td style="text-align:left">gte</td>
      <td style="text-align:left">>=</td>
      <td style="text-align:left">.gte()</td>
      <td style="text-align:left">大于等于</td>
    </tr>
    <tr>
      <td style="text-align:left">lte</td>
      <td style="text-align:left">
        <=</td>
          <td style="text-align:left">.lte()</td>
          <td style="text-align:left">小于等于</td>
    </tr>
    <tr>
      <td style="text-align:left">like</td>
      <td style="text-align:left">like</td>
      <td style="text-align:left">.like()</td>
      <td style="text-align:left">
        <p>模糊匹配.支持options</p>
        <ol>
          <li>reverse 反转，如: ? like name</li>
          <li>startWith: like ?||%</li>
          <li>endWith: like %||?</li>
        </ol>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">nlike</td>
      <td style="text-align:left">not like</td>
      <td style="text-align:left">.notLike()</td>
      <td style="text-align:left">同like</td>
    </tr>
    <tr>
      <td style="text-align:left">in</td>
      <td style="text-align:left">in</td>
      <td style="text-align:left">.in()</td>
      <td style="text-align:left">
        <p>值可以为以下格式:</p>
        <ol>
          <li>1,2,3,4 字符串以半角逗号分割.</li>
          <li>[1,2,3,4] 集合.</li>
        </ol>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">nin</td>
      <td style="text-align:left">not in</td>
      <td style="text-align:left">.notIn()</td>
      <td style="text-align:left">同in</td>
    </tr>
    <tr>
      <td style="text-align:left">isnull</td>
      <td style="text-align:left">is null</td>
      <td style="text-align:left">.isNull()</td>
      <td style="text-align:left">为null</td>
    </tr>
    <tr>
      <td style="text-align:left">notnull</td>
      <td style="text-align:left">not null</td>
      <td style="text-align:left">.notNull()</td>
      <td style="text-align:left">不为null</td>
    </tr>
    <tr>
      <td style="text-align:left">empty</td>
      <td style="text-align:left">= ''</td>
      <td style="text-align:left">.isEmpty()</td>
      <td style="text-align:left">为空字符</td>
    </tr>
    <tr>
      <td style="text-align:left">nempty</td>
      <td style="text-align:left">!=''</td>
      <td style="text-align:left">.notEmpty()</td>
      <td style="text-align:left">不为字符</td>
    </tr>
    <tr>
      <td style="text-align:left">bwt</td>
      <td style="text-align:left">between</td>
      <td style="text-align:left">.between()</td>
      <td style="text-align:left">在之间</td>
    </tr>
    <tr>
      <td style="text-align:left">nbwt</td>
      <td style="text-align:left">not between</td>
      <td style="text-align:left">.notBetween</td>
      <td style="text-align:left">不在之间</td>
    </tr>
  </tbody>
</table>## 自定义SQL条件

上述默认支持的SQL条件其实就是用来定义列与条件的拼接方式。在某些场景，我们可能需要定义一些通用的查询条件。

例如： 根据部门ID 查询所有此部门下所有人员的数据，而且有很多功能都要使用此条件。

{% hint style="warning" %}
按照以前的方式，需要在各个需要使用此查询条件的功能里 添加Dao方法或者修改查询配置
{% endhint %}

{% hint style="success" %}
hsweb提供了SQL条件的拓展功能，用于添加特殊的查询条件。让自定义的条件用起来就像 like ,&gt; , &lt;一样方便。
{% endhint %}

### 创建拓展类

```java
//AbstractSqlTermCustomer提供了一些便利的方法
@org.springframework.stereotype.Component //注入到spring自动开启支持此条件
public class UserInDepartmentTerm extends AbstractSqlTermCustomizer{

    @Override
    public String getTermType() {
        //对应Term参数中的属性termType
        return "user-in-dept";
    }

    @Override
    public Dialect[] forDialect() {
        //对特定对数据库类型生效,返回null时对全部支持对数据库类型生效
        return null;
    }

    @Override
    public SqlAppender accept(String wherePrefix, Term term, RDBColumnMetaData column, String tableAlias) {
        //当传入了my-term条件对时候,会调用此方法进行拼接
        //对传入对参数进行转换,此步骤为必须的
       ChangedTermValue termValue =createChangedValue(term);
        
       //转换参数,将参数转为集合,以支持in查询. 
       List<Object> idList = BoostTermTypeMapper.convertList(column, termValue.getOld());
        
       SqlAppender appender=  new SqlAppender();
      
       appender.add(createColumnName(column,tableAlias),"in (select user_id from user_department where t_id");
        
        //根据参数的数量,构造对应的=或者in条件
       Object newValue= appendCondition(idList,wherePrefix,appender);
        
       appender.add(")");
       //设置新的值到条件中
       termValue.setValue(newValue);
        
        return appender;
    }
}
```

使用:

```java
// where user_id in(select user_id from user_department where t_id = ? )
createQuery().where("userId$user-in-dept","test")
```

{% hint style="success" %}
这样，无需修改其他任何配置，即可支持查询任何表里指定部门下用户的数据了。
{% endhint %}

在[组织架构](../ye-wu-gong-neng/zu-zhi-jia-gou.md#sql-tiao-jian)功能中,提供了一些已经拓展好的SQL条件。

