# 拓展实体类

## 说明

hsweb提供的业务功能只提供了基础的，必要的属性。在某些场景下，可能需要拓展某个实体的字段，比如要给组织架构功能添加更多的属性信息。

## 添加字段

编写实体类，继承需要拓展的实体。

{% code-tabs %}
{% code-tabs-item title="com.myproject.entity.CustomOrganizationalEntity" %}
```java
package com.myproject.entity;

import org.hswebframework.web.entity.organizational.SimpleOrganizationalEntity;
@Table //加上jpa注解,无需再修改 mybatis mapper.xml
public class CustomOrganizationalEntity extends SimpleOrganizationalEntity {
    
    /**********新增加的字段**********/
    
    @Column(name="leader")
    private String leader;

    @Column(name="name_en")
    private String nameEn;

    @Column(name="other_property")
    private String otherProperty;

    public String getLeader() {
        return leader;
    }

    public void setLeader(String leader) {
        this.leader = leader;
    }

    public String getNameEn() {
        return nameEn;
    }

    public void setNameEn(String nameEn) {
        this.nameEn = nameEn;
    }

    public String getOtherProperty() {
        return otherProperty;
    }

    public void setOtherProperty(String otherProperty) {
        this.otherProperty = otherProperty;
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 覆盖默认实体

在创建了新的实体类后,需要告诉hsweb,不再使用默认的实体类。

{% hint style="info" %}
目前提供了3种覆盖方式\([Java自带ServiceLoader](tuo-zhan-shi-ti-lei.md#serviceloader-fang-shi),[application.yml配置](tuo-zhan-shi-ti-lei.md#applicationyml-fang-shi),[Java类配置](tuo-zhan-shi-ti-lei.md#java-lei-pei-zhi-fang-shi)\)，任选其一即可
{% endhint %}

### ServiceLoader方式

在`resources`下创建文件：

`META-INF/services/org.hswebframework.web.entity.organizational.OrganizationalEntity`

内容为：`com.myproject.entity.CustomOrganizationalEntity`

### application.yml方式

{% code-tabs %}
{% code-tabs-item title="application.yml" %}
```yaml
hsweb: 
   entity:
     mappings:
         -  source-base-package: org.hswebframework.web.entity.organizational
            target-base-package: com.myproject.entity
            mapping:
                OrganizationalEntity: CustomOrganizationalEntity
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Java类配置方式

{% code-tabs %}
{% code-tabs-item title="CustomEntityMappingCustomizer" %}
```java
 @Component
    public class CustomEntityMappingCustomizer implements EntityMappingCustomizer {
        @Override
        public void customize(MapperEntityFactory entityFactory) {
            //OrganizationalEntity使用CustomOrganizationalEntity实现
            entityFactory.addMapping(OrganizationalEntity.class,
                    MapperEntityFactory.defaultMapper(CustomOrganizationalEntity.class));
        }
    }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 修改Mybatis配置

{% hint style="warning" %}
此步骤仅在未使用JPA注解或者需要修改mapper.xml时进行
{% endhint %}

### 使用新的mapper.xml

创建新的mapper.xml

{% code-tabs %}
{% code-tabs-item title="com/myproject/mappers/OrganizationalMapper.xml" %}
```markup
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://www.mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.hswebframework.web.dao.organizational.OrganizationalDao">
    <resultMap id="OrganizationalResultMap" type="org.hswebframework.web.entity.organizational.OrganizationalEntity">
        <!--默认的属性-->
        <id property="id" column="u_id" javaType="string" jdbcType="VARCHAR"/>
        <result property="name" column="name" javaType="String" jdbcType="VARCHAR"/>
        <result property="fullName" column="full_name" javaType="String" jdbcType="VARCHAR"/>
        <result property="code" column="code" javaType="String" jdbcType="VARCHAR"/>
        <result property="optionalRoles" column="optional_roles" javaType="java.util.List" jdbcType="CLOB"/>
        <result property="parentId" column="parent_id" javaType="String" jdbcType="VARCHAR"/>
        <result property="path" column="path" javaType="String" jdbcType="VARCHAR"/>
        <result property="sortIndex" column="sort_index" javaType="Long" jdbcType="DECIMAL"/>
        <result property="status" column="status" javaType="Byte" jdbcType="DECIMAL"/>
        <result property="level" column="level" javaType="Integer" jdbcType="DECIMAL"/>
        <!--重点: 拓展的属性-->
        <result property="nameEn" column="name_en" javaType="String" jdbcType="VARCHAR"/>
        <result property="leader" column="leader" javaType="String" jdbcType="VARCHAR"/>
        <result property="otherProperty" column="other_property" javaType="String" jdbcType="VARCHAR"/>

    </resultMap>

    <!--用于动态生成sql所需的配置-->
    <sql id="config">
        <bind name="resultMapId" value="'OrganizationalResultMap'"/>
        <bind name="tableName" value="'s_organization'"/>
    </sql>
    <!--修改parameterType为新的实体类型-->
    <insert id="insert" parameterType="com.myproject.entity.CustomOrganizationalEntity">
        <include refid="config"/>
        <include refid="BasicMapper.buildInsertSql"/>
    </insert>

    <delete id="deleteByPk" parameterType="String">
        delete from s_organization where u_id =#{id}
    </delete>

    <delete id="delete" parameterType="org.hswebframework.web.commons.entity.Entity">
        <include refid="config"/>
        <include refid="BasicMapper.buildDeleteSql"/>
    </delete>

    <update id="update" parameterType="org.hswebframework.web.commons.entity.Entity">
        <include refid="config"/>
        <include refid="BasicMapper.buildUpdateSql"/>
    </update>

    <select id="query" parameterType="org.hswebframework.web.commons.entity.Entity" resultMap="OrganizationalResultMap">
        <include refid="config"/>
        <include refid="BasicMapper.buildSelectSql"/>
    </select>

    <select id="count" parameterType="org.hswebframework.web.commons.entity.Entity" resultType="int">
        <include refid="config"/>
        <include refid="BasicMapper.buildTotalSql"/>
    </select>
</mapper>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 覆盖默认的mapper.xml

#### java类配置方式

{% code-tabs %}
{% code-tabs-item title="CustomMybatisMapperCustomizer" %}
```java
@Component //提供给spring才会生效
public class CustomMybatisMapperCustomizer implements MybatisMapperCustomizer {
    @Override
    public String[] getExcludes() {
    //不加载默认的配置
        return new String[]{
                "classpath*:org/hswebframework/**/OrganizationalMapper.xml"
        };
    }

    @Override
    //追加配置
    public String[] getIncludes() {
        return new String[]{
                "classpath*:com/myproject/mappers/OrganizationalMapper.xml"
        };
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### application.yml方式

{% code-tabs %}
{% code-tabs-item title="application.yml" %}
```yaml
mybatis:
  mapper-location-excludes: classpath*:org/hswebframework/**/OrganizationalMapper.xml #不加载的xml
  mapper-locations: classpath*:com/myproject/mappers/OrganizationalMapper.xml
```
{% endcode-tabs-item %}
{% endcode-tabs %}

