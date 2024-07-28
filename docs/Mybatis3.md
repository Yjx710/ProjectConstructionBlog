# `Mybatis3`

## 多表映射

###  实体类设计方案

   - 对一关系：属性中包含对方对象

```java
@Data
public class Order {
  private Integer orderId;
  private String orderName;
  private Integer customerId;
  private Customer customer; // 对一，属性中包含对方对象
}
```

- 对多关系：属性中包含对方对象集合

    ```java
    @Data
    public class Customer {
      private Integer customerId;
      private String customerName;
      private List<Order> orderList; // 对多，属性中包含对方对象集合
    }
    ```

    > **注意**：只有真实发生多表查询时，才需要设计和修改实体类，否则不提前设计和修改实体类！
    > 无论多少张表联查，实体类设计都是两两考虑!
    > 在查询映射的时候，只需要关注本次查询相关的属性！例如：查询订单和对应的客户，就不要关注客户中的订单集合！

### 对一关系映射配置

- 使用`<association>`标签：常用属性如下
  - `property`属性：对方对象的属性名
  - `javaType`属性：对方对象类的全类名

> `OrderMapper.java`

```java
/**
   * 根据ID查询订单，以及订单关联的用户的信息！
   *
   * @param orderId 需要查询的订单id
   * @return 返回订单和关联的用户信息
   */
Order selectOrderWithCustomer(Integer orderId);
```

> `OrderMapper.xml`

```xml
<mapper namespace="com.yjx.mapper.OrderMapper">
  <!-- 创建resultMap实现“对一”关联关系映射 -->
  <!-- id属性：通常设置为这个resultMap所服务的那条SQL语句的id加上“ResultMap” -->
  <!-- type属性：要设置为这个resultMap所服务的那条SQL语句最终要返回的类型 -->
  <resultMap id="selectOrderWithCustomerResultMap" type="order">
    <!-- 先配置映射Order本身的属性 -->
    <id property="orderId" column="order_id"/>
    <result property="orderName" column="order_name"/>
    <result property="customerId" column="customer_id"/>
    <!-- “对一”关联关系: 使用association标签配置 -->
    <!-- property属性：在Order类中 对一 的一端进行引用时使用的属性名 -->
    <!-- javaType属性：一的一端类 的全类名 -->
    <association property="customer" javaType="customer">
      <id property="customerId" column="customer_id"/>
      <result property="customerName" column="customer_name" />
    </association>
  </resultMap>
  <select id="selectOrderWithCustomer" resultMap="selectOrderWithCustomerResultMap">
    select
      *
    from
      `mybatis-example`.t_order as o
    left join
      `mybatis-example`.t_customer as c
    on
      o.customer_id = c.customer_id
    where
      order_id = #{orderId}
  </select>
</mapper>
```

### 对多关系映射配置

* 使用`<collection>`标签：常用属性如下
  * `property`属性：对方对象集合的属性名
  * `ofType`属性：集合属性中元素的类型

> `CustomerMapper.java`

```java
/**
   * 根据ID查询客户，以及客户关联的订单信息！
   *
   * @param customerId 需要查询的客户id
   * @return 返回客户信息和对应的订单信息
   */
Customer selectCustomerWithOrder(Integer customerId);
```

> `CustomerMapper.xml`

```xml
<mapper namespace="com.yjx.mapper.CustomerMapper">
    <!-- 配置resultMap实现从Customer到OrderList的“对多”关联关系 -->
    <resultMap id="selectCustomerWithOrderResultMap" type="customer">
    <!-- 先配置映射Customer本身的属性 -->
    <id column="customer_id" property="customerId"/>

      <!-- 开启自动映射,并且开启驼峰式支持!可以省略 result!-->
<!--    <result column="customer_name" property="customerName"/>-->

    <!-- “对多”关联关系: 使用collection标签配置 -->
    <!-- property属性：在Customer类中，关联“多”的一端的属性名 -->
    <!-- ofType属性：集合属性中元素的类型，记得加ofType，否则 Cause: java.lang.NullPointerException -->
    <collection property="orderList" ofType="order">
      <!-- 映射Order的属性 -->
      <id column="order_id" property="orderId"/>
<!--      <result column="order_name" property="orderName"/>-->
    </collection>
  </resultMap>
  <select id="selectCustomerWithOrder" resultMap="selectCustomerWithOrderResultMap">
    select
        c.customer_id, c.customer_name, o.order_id, o.order_name
    from
        `mybatis-example`.t_customer as c
    left join
        `mybatis-example`.t_order as o
    on
        c.customer_id = o.customer_id
    where
        c.customer_id = #{customerId}
  </select>
</mapper>
```

### 多表映射优化

>  `mybatis-config.xml `-> `setting`属性 -> `autoMappingBehavior`

```xml
<!--开启resultMap自动映射 -->
<setting name="autoMappingBehavior" value="FULL"/>
<!--配置自动识别驼峰式命名规则 -->
<setting name="mapUnderscoreToCamelCase" value="true"/>
```

* 开启自动映射,并且开启驼峰式支持!可以省略`mapper.xml` `result`标签的编写!



## 动态语句

### if 和 where 标签

> `EmployeeMapper.java`

```java
/**
    * 根据条件筛选员工
    *
    * @param employee 筛选条件：员工信息
    * @return 返回符合条件的员工信息
    */
List<Employee> selectEmployeeByCondition(Employee employee);
```

> `EmployeeMapper.xml`

```xml
<select id="selectEmployeeByCondition" resultType="employee">
  select emp_id, emp_name, emp_salary from `mybatis-example`.t_emp
  <!--
    这里我们给每条语句前面加个or关键字 防止多个条件生效 语句拼接不起来导致报错等情况
    注意：where标签会自动去掉“标签体内前面多余的and/or”
   -->
  <where>
    <!-- if标签让我们可以有选择的加入SQL语句的片段。这个SQL语句片段是否要加入整个SQL语句，就看if标签判断的结果是否为true -->
 	<!-- 在if标签的test属性中，可以访问实体类的属性，不可以访问数据库表的字段 -->
    <if test="empName != null">
      or emp_name = #{empName}
    </if>
    <if test="empSalary &gt; 500">
      or emp_salary &gt; #{empSalary}
    </if>
  </where>
</select>

<!-- &gt; 代表的是大于号。 因为 > 不是很好的能被Mybatis解析 -->
```

### set标签

> `EmployeeMapper.java`

```java
/**
   * 根据员工id更新对应员工信息
   *
   * @param employee 员工信息
   * @return 受影响的行数，大于0更新成功，反之失败
   */
int updateEmployeeDynamic(Employee employee);
```

> `EmployeeMapper.xml`

```xml
<update id="updateEmployeeDynamic">
    update `mybatis-example`.t_emp
    <!--
         这里我们给每条语句后面拼接个, 防止多个条件生效 语句拼接不起来导致报错等情况
         注意：使用set标签动态管理set子句，并且动态去掉两端多余的逗号
       -->
    <set>
        <if test="empName != null">
            emp_name = #{empName},
        </if>
        <if test="empSalary != null">
            emp_salary = #{empSalary},
        </if>
    </set>
    where
    emp_id = #{empId}
</update>

<!-- 
第一种情况：所有条件都满足 SET emp_name = ?, emp_salary = ?
第二种情况：部分条件满足 SET emp_salary = ?
第三种情况：所有条件都不满足 update t_emp where emp_id = ?
	注意: 没有set子句的update语句会导致SQL语法错误 
-->
```

### trim标签

> `EmployeeMapper.java`

```java
/**
   * 根据选择条件查询员工信息。
   *
   * @param employee 一个Employee对象，包含用于查询条件的员工属性值。可以是一个空对象，具体查询条件取决于实现逻辑。
   * @return 返回一个Employee列表，符合查询条件的员工将被包含在列表中。
   */
List<Employee> selectEmployeeByConditionByChoose(Employee employee);


/**
   * 根据员工id更新对应员工信息 - 测试trim标签 suffix属性、suffixOverrides属性
   *
   * @param employee 员工信息
   * @return 受影响的行数，大于0更新成功，反之失败
   */
int updateEmployeeDynamic2(Employee employee);
```

> `EmployeeMapper.xml`

```xml
<!--
  trim标签
    - prefix属性：指定要动态添加的前缀
    - suffix属性：指定要动态添加的后缀
    - prefixOverrides属性：指定要动态去掉的前缀，使用“|”分隔有可能的多个值
    - suffixOverrides属性：指定要动态去掉的后缀，使用“|”分隔有可能的多个值
-->


<!--
  prefix属性和prefixOverrides属性用法
    通过名字和薪水查找某个员工，
    prefix=where：代替了之前 select emp_id, emp_name, emp_salary where  这就是prefix的用法
    prefixOverrides="and"：
      - 场景：当empName为null时，sql语句将会变成 select emp_id, emp_name, emp_salary from `mybatis-example`.t_emp where and emp_salary = ?，可以看到where 后面有个and，此时的sql是错误的。
      - 解决方案：prefixOverrides="and"，这里指定了and，意思就是动态去掉and前缀。此时sql语句正常：select emp_id, emp_name, emp_salary from `mybatis-example`.t_emp where emp_salary = ?
-->
<select id="findByNameAndSalary" resultType="employee">
    select
    emp_id, emp_name, emp_salary
    from
    `mybatis-example`.t_emp
    <trim prefix="where" prefixOverrides="and">
        <if test="empName != null">
            emp_name = #{empName}
        </if>
        <if test="empSalary != null">
            and emp_salary = #{empSalary}
        </if>
    </trim>
</select>



<!--
    suffix属性和suffixOverrides属性用法
      根据id更新对应员工的姓名和薪水，
      suffix="where emp_id = #{empId}"：代替了之前 select emp_id, emp_name, emp_salary where emp_id = #{empId} 这就是suffix的用法
      suffixOverrides=","：
        - 场景：当empSalary为null时，sql语句将会变成 update `mybatis-example`.t_emp set emp_name = ?, where emp_id = ? 可以看到where 前面有个，此时的sql是错误的。
        - 解决方案：suffixOverrides=","，这里指定了, 意思就是动态去掉,后缀。此时sql语句正常：update `mybatis-example`.t_emp set emp_name = ? where emp_id = ?
  -->
<update id="updateEmployeeDynamic2">
    <trim suffix="where emp_id = #{empId}" suffixOverrides=",">
        update  `mybatis-example`.t_emp
        set
        <if test="empName != null">
            emp_name = #{empName},
        </if>
        <if test="empSalary != null">
            emp_salary = #{empSalary},
        </if>
    </trim>
</update>
```

### choose/when/otherwise标签

> `EmployeeMapper.java`

```java
/**
   * 根据选择条件查询员工信息。
   *
   * @param employee 一个Employee对象，包含用于查询条件的员工属性值。可以是一个空对象，具体查询条件取决于实现逻辑。
   * @return 返回一个Employee列表，符合查询条件的员工将被包含在列表中。
   */
List<Employee> selectEmployeeByConditionByChoose(Employee employee);
```

> `EmployeeMapper.xml`

```xml
<!--
  choose/when/otherwise标签：多个分支条件中，仅执行一个
    - 从上到下依次执行条件判断
    - 第一个满足条件的分支会被采纳
    - 被采纳分支后面的分支都将不被考虑
    - 如果所有的when分支都不满足，那么就执行otherwise分支

    有点像if - else if - else
-->
  <select id="selectEmployeeByConditionByChoose" resultType="employee">
    select
        *
    from
        `mybatis-example`.t_emp
    where
    <choose>
      <when test="empName != null">
        emp_name = #{empName}
      </when>
      <when test="empSalary &lt; 3000">
        emp_salary &lt; 3000
      </when>
      <otherwise>
        1 = 1
      </otherwise>
    </choose>
  </select>
```

### `foreach`标签

> `EmployeeMapper.java`

```java
/**
   * 批量插入员工信息到数据库。
   *
   * @param employees 员工信息列表，是一个不可为空的List对象，其中包含了待插入的员工信息。
   * @return 返回插入的员工记录数。若全部插入成功，返回员工列表的大小；若有插入失败，返回实际插入成功的记录数。
   */
int batchInsertionEmployee(@Param("employees") List<Employee> employees);

/**
   * 批量更新员工信息。
   *
   * @param employees 要更新的员工信息列表，列表中的每个员工对象都包含了员工的详细信息。
   * @return 更新成功的员工数量。如果所有员工都成功更新，则返回员工列表的大小；如果有员工更新失败，则返回成功更新的员工数量。
   */
int massUpdateEmployee(@Param("employees") List<Employee> employees);
```

> `EmployeeMapper.xml`

```xml
<!--
    foreach标签
      collection属性：要遍历的集合，指定batchInsertionEmployee方法的参数名称(如果没有给接口中List类型的参数使用@Param注解指定一个具体的名字，那么在collection属性中默认可以使用collection或list来引用这个list集合)
      item属性：遍历出来的每一个对象。名字随意
      separator属性：各个标签体字符串之间的分隔符
      open属性：字符串整体的前面要添加的字符串
      close属性：字符串整体的后面要添加的字符串
      index属性：这里起一个名字，便于后面引用
          遍历List集合，这里能够得到List集合的索引值
          遍历Map集合，这里能够得到Map集合的key
 -->
<!--  批量插入-->
  <insert id="batchInsertionEmployee">
    insert into `mybatis-example`.t_emp (emp_name, emp_salary)
      <foreach collection="employees" item="emp" separator="," open="values">
        (#{emp.empName}, #{emp.empSalary})
      </foreach>
  </insert>


<!--  批量更新
      注意：上面批量插入本质上是一条SQL语句，而实现批量更新则需要多条SQL语句拼起来，用分号分开。也就是一次性发送多条SQL语句让数据库执行
           需要在数据库连接信息的URL地址加入：?allowMultiQueries=true
           例子：jdbc.url=jdbc:mysql://localhost:3306/mybatis-example?allowMultiQueries=true
-->
 <update id="massUpdateEmployee">
    <foreach collection="employees" item="emp" separator=";">
        update `mybatis-example`.t_emp set emp_name = #{emp.empName}, emp_salary = #{emp.empSalary} where emp_id = #{emp.empId}
    </foreach>
 </update>
```

### 抽取重复的`sql`

> `EmployeeMapper.xml`

```xml
<!-- 使用sql标签抽取重复出现的SQL片段 -->
<sql id="mySelectSql">
    select
    emp_id, emp_name, emp_salary
    from
    `mybatis-example`.t_emp
</sql>


<select id="selectEmployeeByCondition" resultType="employee">
    <!-- 引用已抽取的SQL片段：使用include标签引用声明的SQL片段-->
    <include refid="mySelectSql" />
    <where>
        <if test="empName != null">
            or emp_name = #{empName}
        </if>
        <if test="empSalary &gt; 500">
            or emp_salary &gt; #{empSalary}
        </if>
    </where>
</select>
```



## 高级扩展

### Mapper批量映射优化

* 场景：Mapper 配置文件很多时，在全局配置文件中一个一个注册太麻烦

  ![image-20240316152437133](/images/mybatis/image-20240316152437133.png)

* 解决：指定其所在的包，此时这个包下的所有 Mapper 配置文件将被自动加载、注册

![image-20240316152606252](/images/mybatis/image-20240316152606252.png)

> 注意：需要满足资源创建要求
>
>  - Mapper 接口和 Mapper 配置文件名称一致
>       - Mapper 接口：`EmployeeMapper.java`
>       - Mapper 配置文件：`EmployeeMapper.xml`
> - Mapper 配置文件放在 Mapper 接口所在的包内
>   1. resources目录下创建和mapper接口包一致的文件夹存放对应的`xml`文件
>   2. `com.yjx.mapper` 点的方式不行识别不到(`com.yjx.mapper` 三个连在一起)。需 `com/yjx/mapper`建立文件夹(打包后的路径是`com `下级 `yjx` 下级 mapper)。

![image-20240316152840333](/images/mybatis/image-20240316152840333.png)

![image-20240316153114220](/images/mybatis/image-20240316153114220.png)

![image-20240316153422147](/images/mybatis/image-20240316153422147.png)



### 分页插件`PageHelper`

**插件机制和`PageHelper`插件介绍**

* 插件可以在用于语句执行过程中进行拦截，并允许通过自定义处理程序来拦截和修改 `SQL` 语句、映射语句的结果等
* `Mybatis`的插件机制包括以下三个组件：
  * `Interceptor`（拦截器）：定义一个拦截方法 `intercept`，该方法在执行 `SQL` 语句、执行查询、查询结果的映射时会被调用。
  2. `Invocation`（调用）：实际上是对被拦截的方法的封装，封装了 `Object target`、`Method method` 和 `Object[] args` 这三个字段。
  * `InterceptorChain`（拦截器链）：对所有的拦截器进行管理，包括将所有的 Interceptor 链接成一条链，并在执行 `SQL` 语句时按顺序调用。

 **`PageHelper`插件使用**

1. maven工程导入依赖

   ```xml
   <dependency>
       <groupId>com.github.pagehelper</groupId>
       <artifactId>pagehelper</artifactId>
       <version>5.1.11</version>
   </dependency>
   ```

2. `mybatis-config.xml`配置分页插件

   ```xml
   <!--    com.github.pagehelper.PageInterceptor 是 PageHelper 插件的名称-->
   <!--    dialect 属性用于指定数据库类型（支持多种数据库）-->
   <plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="helperDialect" value="mysql"/>
    </plugin>
   </plugins>
   ```

3. 在查询方法中使用分页

> `EmployeeMapper.xml`

```xml
<mapper namespace="com.yjx.mapper.EmployeeMapper">
  <select id="findAll" resultType="employee">
    select * from `mybatis-example`.t_emp
  </select>
</mapper>
```

> `Test.java`

```java
@Test
public void testPageHelper() {
    EmployeeMapper employeeMapper = SqlSessionUtil.getSqlSession().getMapper(EmployeeMapper.class);
    // 设置分页数据（当前是第几页，每页显示多少条数据）
    PageHelper.startPage(1, 2);
    List<Employee> employeeList = employeeMapper.findAll();
    // 将查询的数据封装到一个PageInfo的分页实体类（一共多少页，一共多少条等）
    PageInfo<Employee> pageInfo = new PageInfo<>(employeeList);
    System.out.println(pageInfo);
    System.out.println("总记录数：" + pageInfo.getTotal());
    System.out.println("总页数：" + pageInfo.getPages());
    System.out.println("获取当前页码：" + pageInfo.getPageNum());
    System.out.println("获取每页显示记录数：" + pageInfo.getPageSize());
    System.out.println("获取查询页的数据集合：" + pageInfo.getList());
}
```



### `ORM`思维

`ORM`（Object-Relational Mapping，对象-关系映射）是一种将数据库和面向对象编程语言中的对象之间进行转换的技术。它将对象和关系数据库的概念进行映射，最后就可以通过方法调用进行数据库操作!! 最终: **让我们可以使用面向对象思维进行数据库操作**！！！

> `ORM` 框架有两种方式：半自动和全自动

- 半自动 `ORM` 通常需要程序员手动编写 `SQL` 语句或者配置文件，将实体类和数据表进行映射，还需要手动将查询的结果集转换成实体对象。
- 全自动 `ORM` 则是将实体类和数据表进行自动映射，使用 `API` 进行数据库操作时，`ORM` 框架会自动执行` SQL` 语句并将查询结果转换成实体对象，程序员无需再手动编写 `SQL` 语句和转换代码。

> 区别

1. 映射方式：半自动 `ORM` 框架需要程序员手动指定实体类和数据表之间的映射关系，通常使用 XML 文件或注解方式来指定；全自动` ORM `框架则可以自动进行实体类和数据表的映射，无需手动干预。
2. 查询方式：半自动 `ORM` 框架通常需要程序员手动编写 `SQL` 语句并将查询结果集转换成实体对象；全自动 `ORM` 框架可以自动组装 `SQL `语句、执行查询操作，并将查询结果转换成实体对象。
3. 性能：由于半自动 `ORM` 框架需要手动编写 `SQL` 语句，因此程序员必须对 `SQL` 语句和数据库的底层知识有一定的了解，才能编写高效的 `SQL `语句；而全自动 `ORM` 框架通过自动优化生成的 `SQL` 语句来提高性能，程序员无需进行优化。
4. 学习成本：半自动 `ORM` 框架需要程序员手动编写 `SQL` 语句和映射配置，要求程序员具备较高的数据库和` SQL `知识；全自动 `ORM `框架可以自动生成 `SQL` 语句和映射配置，程序员无需了解过多的数据库和 `SQL` 知识。

> 常见的半自动 `ORM` 框架

`MyBatis` 等

> 常见的全自动 `ORM` 框架

`Hibernate`、`Spring Data JPA`、`MyBatis-Plus` 等



### 逆向工程

>     MyBatis 的逆向工程是一种自动化生成持久层代码和映射文件的工具，它可以根据数据库表结构和设置的参数生成对应的实体类、Mapper.xml 文件、Mapper 接口等代码文件，简化了开发者手动生成的过程。逆向工程使开发者可以快速地构建起 DAO 层，并快速上手进行业务开发。
>    `MyBatis` 的逆向工程有两种方式：
>
>  1. `MyBatis Generator` 插件实现
>
>  2.  `Maven`插件实现
>
>     逆向工程一般需要指定一些配置参数，例如数据库连接 URL、用户名、密码、要生成的表名、生成的文件路径等
>
> **好处**
>
> -  `MyBatis` 的逆向工程为程序员提供了一种方便快捷的方式，能够快速地生成持久层代码和映射文件，是半自动 `ORM` 思维像全自动发展的过程，提高程序员的开发效率。
>
> **注意：逆向工程只能生成单表crud的操作，多表查询依然需要我们自己编写！**



### 逆向工程插件`MyBatisX`

> `MyBatisX` 是一个 `MyBatis` 的**代码生成插件**，可以**通过简单的配置和操作快速生成** `MyBatis Mapper`、`pojo` 类和 `Mapper.xml` 文件

1. **安装插件**

   ![image-20240316214918584](/images/mybatis/image-20240316214918584.png)

2. **连接数据库**

   ![image-20240316223128232](/images/mybatis/image-20240316223128232.png)

   * 填写数据库信息

     ![image-20240316223312460](/images/mybatis/image-20240316223312460.png)

   * 展示库表

     ![image-20240316223421334](/images/mybatis/image-20240316223421334.png)

   * 逆向工程使用

     ![image-20240316223600801](/images/mybatis/image-20240316223600801.png)

     ![image-20240316223747163](/images/mybatis/image-20240316223747163.png)

     ![image-20240316223958567](/images/mybatis/image-20240316223958567.png)

   * 然后**点击Finish**即可

   * 查看生成结果

     ![image-20240316224058589](/images/mybatis/image-20240316224058589.png)