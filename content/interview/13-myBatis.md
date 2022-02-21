## 什么是 Mybatis？

1. Mybatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。程序员直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高。
2. Mybatis 可以使用 XML 或注解来配置和映射原生信息，将 POJO 映射成数据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
3. 通过 XML 文件或注解的方式将要执行的各种 statement 配置起来，并通过 Java 对象和 statement 中 SQL 的动态参数进行映射生成最终执行的 SQL 语句，最后由 Mybatis 框架执行 SQL 并将结果映射为 Java 对象并返回。（从执行 SQL 到返回 result 的过程）



## Mybatis 的优点

1. 基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL 写在 XML 里，解除 SQL 与程序代码的耦合，便于统一管理，提供 XML 标签，支持编写动态 SQL 语句，并可重用。
2. 与 JDBC 相比，减少了 50% 以上的代码量，消除了 JDBC 大量冗余的代码，不需要手动开关连接。
3. 很好的与各种数据库兼容（因为 Mybatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 Mybatis 都支持）
4. 能够与 Spring 很好的集成。
5. 提供映射标签，支持对象与数据库的 ORM 字段关系映射；提供对象关系映射标签，支持对象关系组件维护。



## Mybatis 的缺点

1. SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求。
2. SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。



## Mybatis 框架适用场合

1. Mybatis 专注于 SQL 本身，是一个足够灵活的 DAO 层解决方案。
2. 对性能的要求很高，或者需求变化较多的项目，如互联网项目，Mybatis 将是不错的选择。



## `#{}` 和 `${}` 的区别是什么？

`#{}` 是预编译处理，`${}` 是字符串替换。

在使用 `#{}` 时，MyBatis 会将 SQL 中 `#{}` 替换成 "?"，配合 PreparedStatement 的 set 方法赋值。

Mybatis 在处理 `${}` 时，就是把 `${}` 替换成变量的值。

使用 `#{}` 可以有效的防止 SQL 注入，提高系统安全性。



## 当实体类中的属性名和表中的字段名不一样，怎么办？

1. 通过在查询的 SQL 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致

   ```xml
   <select id="findDirectoryDetailById" parameterType="java.lang.Long"
               resultType="com.allen.model.DemoVO">
           SELECT
           t.name as tableName
           FROM
           demo_table t
           WHERE t.id = #{id}
   </select>
   ```

2. 通过 `<resultMap>` 映射字段名和实体类属性名的一一对应的关系。

   ```xml
   <select id="getOrder" parameterType="int"
   resultMap="orderresultmap">
   select * from orders where order_id=#{id}
   </select>
   <resultMap type=”me.gacl.domain.order” id=”orderresultmap”>
   <!–用id 属性来映射主键字段–>
   <id property=”id” column=”order_id”>
   <!–用result 属性来映射非主键字段，property 为实体类属性名，column
   为数据表中的属性–>
   <result property = “orderno” column =”order_no”/>
   <result property=”price” column=”order_price” />
   </reslutMap>
   ```



## 模糊查询 like 语句该怎么写？

1. 在 Java 代码中添加 sql 通配符

   ```java
   String wildcardname = “%smi%”;
   List<Name> names = mapper.selectlike(wildcardname);
   ```

   ```xml
   <select id=”selectlike”>
   select * from foo where bar like #{value}
   </select>
   ```

2. 在 sql 语句中拼接通配符，会引起 sql 注入

   ```java
   String wildcardname = “smi”;
   List<name> names = mapper.selectlike(wildcardname);
   ```

   ```xml
   <select id=”selectlike”>
   select * from foo where bar like "%"#{value}"%"
   </select>
   ```



## 通过 XML 映射文件，都会写一个 Dao 接口与之对应，这个 Dao接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

Dao 接口即 Mapper 接口。接口的全限名，就是映射文件中的 namespace 的值；接口的方法名，就是映射文件中 Mapper 的 Statement 的 id 值；接口方法内的参数，就是传递给 sql 的参数。

Mapper 接口是没有实现类的，当调用接口方法时，接口全限名 + 方法名拼接字符串作为 key 值，可唯一定位一个 MapperStatement。在 Mybatis 中，每一个 `<select>`、`<insert>`、`<update>`、`<delete>` 标签，都会被解析为一个 MapperStatement 对象。

举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到 namespace 为 com.mybatis3.mappers.StudentDao 下面 id 为 findStudentById 的 MapperStatement。

Mapper 接口里的方法，是不能重载的，因为是使用全限名+方法名的保存和寻找策略。Mapper 接口的工作原理是JDK 动态代理，Mybatis 运行时会使用JDK动态代理为Mapper 接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement 所代表的sql，然后将sql 执行结果返回。



## MyBatis 有几种分页方式？

**分页方式**：逻辑分页和物理分页。

**逻辑分页**： 使用 MyBatis 自带的 RowBounds 进行分页，它是一次性查询很多数据，然后在数据中再进行检索。

**物理分页**： 自己手写 SQL 分页或使用分页插件 PageHelper，去数据库查询指定条数的分页数据的形式。



## RowBounds 是一次性查询全部结果吗？为什么？

RowBounds 表面是在“所有”数据中检索数据，其实并非是一次性查询出所有数据，因为 MyBatis 是对 jdbc 的封装，在 jdbc 驱动中有一个 Fetch Size 的配置，它规定了每次最多从数据库查询多少条数据，假如你要查询更的数据，它会在你执行 next()的时候，去查询更多的数据。就好比你去自动取款机取 10000 元，但取款机每次最多能取 2500 元，所以你要取 4 次才能把钱取完。只是对于 jdbc 来说，当你调用 next()的时候会自动帮你完成查询工作。这样做的好处可以有效的防止内存溢出。



## MyBatis 逻辑分页和物理分页的区别是什么？

逻辑分页是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。

物理分页是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。



## Mybatis 是如何将sql 执行结果封装为目标对象并返回的？都有哪些映射形式？

1. 使用 `<resultMap>` 标签，逐一定义数据库列名和对象属性名之间的映射关系。
2. 使用 SQL 列的别名功能，将列的别名书写为对象属性名。

有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。



## 如何执行批量插入？

先创建一个 insert 语句：

```xml
<insert id="insertName">
insert into names (name) values (#{value})
</insert>
```

Java 批量处理插入：

```java
List<String> names = new ArrayList();
names.add(“fred”);
names.add(“barney”);
names.add(“betty”);
names.add(“wilma”);
// 注意这里executortype.batch
Sqlsession sqlsession = Sqlsessionfactory.openSession(Executortype.batch);
try {
	NameMapper mapper = sqlsession.getMapper(NameMapper.class);
	for (String name: names) {
		mapper.insertName(name);
	}
	sqlsession.commit();
} catch (Exception e) {
	e.printStackTrace();
    sqlSession.rollback();
    throw e;
}finally {
	sqlsession.close();
}
```



## 如何获取自动生成的（主）键值

insert 方法总是返回一个 int 值，这个值代表的是插入的行数。

如果采用自增长策略，自动生成的键值在 insert 方法执行完后可以被设置到传入的参数对象中。

示例：

```xml
<insert id=”insertname” usegeneratedkeys=”true” keyproperty=”
id”>
insert into names (name) values (#{name})
</insert>
```

```java
name name = new name();
name.setname(“fred”);
int rows = mapper.insertname(name);
// 完成后,id 已经被设置到对象中
system.out.println(“rows inserted = ” + rows);
system.out.println(“generated key value = ” + name.getid());
```



## 在 mapper 中如何传递多个参数？

- DAO 层的函数：

```java
public UserselectUser(String name, String area);
```

```xml
<!-对应的xml,#{0}代表接收的是dao 层中的第一个参数，#{1}代表dao 层中第二参数，更多参数一致往后加即可。->
<select id="selectUser"resultMap="BaseResultMap">
select * fromuser_user_t whereuser_name = #{0} anduser_area = #{1}
</select>
```

- 使用 `@param` 注解：

```java
public interface usermapper {
	user selectuser(@param(“username”) string username, @param(“hashedpassword”) string hashedpassword);
}
```

然后，就可以在xml像下面这样使用（推荐封装为一个map，作为单个参数传递给mapper）:

```xml
<select id=”selectuser” resulttype=”user”>
select id, username, hashedpassword
from some_table
where username = #{username}
and hashedpassword = #{hashedpassword}
</select>
```

- 多个参数封装成 map

```java
try {
	//映射文件的命名空间.SQL 片段的ID，就可以调用对应的映射文件中的SQL
	//由于我们的参数超过了两个，而方法中只有一个Object 参数收集，因此我们使用Map 集合来装载我们的参数
	Map < String, Object > map = new HashMap();
	map.put("start", start);
	map.put("end", end);
	return sqlSession.selectList("StudentID.pagination", map);
} catch (Exception e) {
	e.printStackTrace();
	sqlSession.rollback();
	throw e;
} finally {
	MybatisUtil.closeSqlSession();
}
```



## Mybatis 动态sql 有什么用？执行原理？有哪些动态sql？

Mybatis 动态 sql 可以在 Xml 映射文件内，以标签的形式编写动态 sql，执行原理是根据表达式的值完成逻辑判断并动态拼接 sql 的功能。

Mybatis 提供了9 种动态sql 标签：trim | where | set | foreach | if | choose | when | otherwise | bind。



## Xml映射文件中，除了常见的 select | insert | updae | delete 标签之外，还有哪些标签？

`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，加上动态sql 的9 个标签， 其中 `<sql>` 为 sql 片段标签，通过 `<include>` 标签引入 sql 片段，`<selectKey>` 为不支持自增的主键生成策略标签。



## Mybatis的Xml映射文件中， 不同的Xml映射文件， id 是否可以重复？

不同的Xml映射文件，如果配置了namespace，那么id 可以重复；如果没有配置namespace，那么id 不能重复；

原因就是 namespace + id 是作为 Map<String, MapperStatement> 的 key 使用的， 如果没有 namespace，就剩下 id，那么， id  重复会导致数据互相覆盖。有了 namespace，自然id 就可以重复，namespace 不同，namespace + id 自然也就不同。



## 为什么说Mybatis 是半自动ORM 映射工具？它与全自动的区别在哪里？

Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 Mybatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。



## 一对一、一对多的关联查询？

```xml
<mapper namespace="com.lcb.mapping.userMapper">
	<!--association 一对一关联查询-->
    <select id="getClass" parameterType="int" resultMap="ClassesResultMap">
		select * from class c,teacher t where c.teacher_id=t.t_id and
		c.c_id=#{id}
	</select>
	<resultMap type="com.lcb.user.Classes" id="ClassesResultMap">
		<!-- 实体类的字段名和数据表的字段名映射-->
		<id property="id" column="c_id"/>
		<result property="name" column="c_name"/>
		<association property="teacher" javaType="com.lcb.user.Teacher">
			<id property="id" column="t_id"/>
			<result property="name" column="t_name"/>
		</association>
	</resultMap>
    
	<!--collection 一对多关联查询-->
	<select id="getClass2" parameterType="int" resultMap="ClassesResultMap2">
		select * from class c,teacher t,student s where c.teacher_id=t.t_id
		and c.c_id=s.class_id and c.c_id=#{id}
	</select>
	<resultMap type="com.lcb.user.Classes" id="ClassesResultMap2">
		<id property="id" column="c_id"/>
		<result property="name" column="c_name"/>
		<association property="teacher" javaType="com.lcb.user.Teacher">
			<id property="id" column="t_id"/>
        	<result property="name" column="t_name"/>
		</association>
		<collection property="student" ofType="com.lcb.user.Student">
			<id property="id" column="s_id"/>
			<result property="name" column="s_name"/>
		</collection>
	</resultMap>
</mapper>
```



## MyBatis 实现一对一有几种方式？具体怎么操作的？

有联合查询和嵌套查询，联合查询是几个表联合查询，只查询一次，通过在 resultMap 里面配置 association 节点配置一对一的类就可以完成；

嵌套查询是先查一个表，根据这个表里面的结果的外键 id，去再另外一个表里面查询数据，也是通过 association 配置，但另外一个表的查询通过 select 属性配置。



## MyBatis 实现一对多有几种方式，怎么操作的？

有联合查询和嵌套查询。

联合查询是几个表联合查询，只查询一次,通过在 resultMap 里面的 collection 节点配置一对多的类就可以完成；

嵌套查询是先查一个表，根据这个表里面的结果的外键 id，去再另外一个表里面查询数据，也是通过配置collection，但另外一个表的查询通过select 节点配置。



## MyBatis 是否支持延迟加载？延迟加载的原理是什么？

Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询。

MyBatis 支持延迟加载，可以配置是否启用延迟加载 lazyLoadingEnabled=true | false。

延迟加载的原理的是调用的时候触发加载，而不是在初始化的时候就加载信息。比如调用 a. getB(). getName()，这个时候发现 a. getB() 的值为 null，此时会单独触发事先保存好的关联 B 对象的 SQL，先查询出来 B，然后再调用 a. setB(b)，而这时候再调用 a. getB(). getName() 就有值了，这就是延迟加载的基本原理。

它的原理是，使用 CGLIB 创建目标对象的代理对象， 当调用目标方法时，进入拦截器方法，比如调用 a.getB().getName()，拦截器 invoke() 方法发现 a.getB() 是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName() 方法的调用。这就是延迟加载的基本原理。

当然了， 不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。



## 说一下 MyBatis 的一级缓存和二级缓存？

**一级缓存**：基于 PerpetualCache 的 HashMap 本地缓存，它的声明周期是和 SQLSession 一致的，有多个 SQLSession 或者分布式的环境中数据库操作，可能会出现脏数据。当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认一级缓存是开启的。

**二级缓存**：也是基于 PerpetualCache 的 HashMap 本地缓存，不同在于其存储作用域为 Mapper 级别的，如果多个 SQLSession 之间需要共享缓存，则需要使用到二级缓存，并且二级缓存可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态)。

**开启二级缓存数据查询流程**：二级缓存 -> 一级缓存 -> 数据库。

**缓存更新机制**：当某一个作用域(一级缓存 Session/二级缓存 Mapper)进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。



## 什么是 MyBatis 的接口绑定？有哪些实现方式？

接口绑定，就是在 MyBatis 中任意定义接口，然后把接口里面的方法和 SQL 语句绑定，我们直接调用接口方法就可以，这样比起原来了 SqlSession 提供的方法我们可以有更加灵活的选择和设置。

接口绑定有两种实现方式：

一种是通过注解绑定， 就是在接口的方法上面加上 @Select、@Update 等注解，里面包含 Sql 语句来绑定； 

另外一种就是通过 xml 里面写 SQL 来绑定，在这种情况下，要指定 xml 映射文件里面的 namespace 必须为接口的全路径名。当Sql 语句比较简单时候，用注解绑定，当 SQL 语句比较复杂时候，用 xml 绑定，一般用xml 绑定的比较多。



## 使用 MyBatis 的 mapper 接口调用时有哪些要求？

1. Mapper 接口方法名和 mapper.xml 中定义的每个 SQL 的 id 相同；
2. Mapper 接口方法的输入参数类型和 mapper.xml 中定义的每个 sql 的 parameterType 的类型相同；
3. Mapper 接口方法的输出参数类型和 mapper.xml 中定义的每个 sql 的 resultType 的类型相同；
4. Mapper.xml 文件中的 namespace 即是 mapper 接口的类路径。



## Mapper 编写有哪几种方式？

第一种： 接口实现类继承 SqlSessionDaoSupport：使用此种方法需要编写 mapper 接口， mapper 接口实现类、mapper.xml 文件。

1、在 sqlMapConfig.xml 中配置 mapper.xml 的位置

```xml
<mappers>
	<mapper resource="mapper.xml 文件的地址" />
	<mapper resource="mapper.xml 文件的地址" />
</mappers>
```

2、定义 mapper 接口

3、实现类集成 SqlSessionDaoSupport mapper 方法中可以 this.getSqlSession() 进行数据增删改查。

4、spring 配置

```xml
<bean id=" " class="mapper 接口的实现">
	<property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
</bean>
```

第二种： 使用 org.mybatis.spring.mapper.MapperFactoryBean：

1、在 sqlMapConfig.xml 中配置 mapper.xml 的位置，如果 mapper.xml 和 mappre 接口的名称相同且在同一个目录，这里可以不用配置

```xml
<mappers>
	<mapper resource="mapper.xml 文件的地址" />
	<mapper resource="mapper.xml 文件的地址" />
</mappers>
```

2、定义 mapper 接口：

- mapper.xml 中的 namespace 为 mapper 接口的地址
- mapper 接口中的方法名和 mapper.xml 中的定义的 statement 的 id 保持一致
- Spring 中定义

```xml
<bean id="" class="org.mybatis.spring.mapper.MapperFactoryBean">
	<property name="mapperInterface" value="mapper 接口地址" />
	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

第三种： 使用mapper 扫描器：

1、mapper.xml 文件编写：

mapper.xml 中的 namespace 为mapper 接口的地址；

mapper 接口中的方法名和 mapper.xml 中的定义的 statement 的 id 保持一致；

如果将 mapper.xml 和 mapper 接口的名称保持一致则不用在 sqlMapConfig.xml 中进行配置。

2、定义mapper 接口：

注意 mapper.xml 的文件名和 mapper 的接口名称保持一致， 且放在同一个目录

3、配置mapper 扫描器：

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="mapper 接口包地址"></property>
	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

4、使用扫描器后从spring 容器中获取mapper 的实现对象。



## MyBatis 和 hibernate 的区别有哪些？

**灵活性**：MyBatis 更加灵活，自己可以写 SQL 语句，使用起来比较方便。

**可移植性**：MyBatis 有很多自己写的 SQL，因为每个数据库的 SQL 可以不相同，所以可移植性比较差。

**学习和使用门槛**：MyBatis 入门比较简单，使用门槛也更低。

**二级缓存**：hibernate 拥有更好的二级缓存，它的二级缓存可以自行更换为第三方的二级缓存。



## MyBatis 有哪些执行器（Executor）？

MyBatis 有三种基本的 Executor 执行器：

**SimpleExecutor**：每执行一次 update 或 select 就开启一个 Statement 对象，用完立刻关闭 Statement 对象；

**ReuseExecutor**：执行 update 或 select，以 SQL 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后不关闭 Statement 对象，而是放置于 Map 内供下一次使用。简言之，就是重复使用 Statement 对象；

**BatchExecutor**：执行 update（没有 select，jdbc 批处理不支持 select），将所有 SQL 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理，与 jdbc 批处理相同。



## MyBatis 分页插件的实现原理是什么？

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 SQL，然后重写 SQL，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。



## MyBatis 如何编写一个自定义插件？

自定义插件实现原理

MyBatis 自定义插件针对 MyBatis 四大对象（Executor、StatementHandler、ParameterHandler、ResultSetHandler）进行拦截：

**Executor**：拦截内部执行器，它负责调用 StatementHandler 操作数据库，并把结果集通过 ResultSetHandler 进行自动映射，另外它还处理了二级缓存的操作；

**StatementHandler**：拦截 SQL 语法构建的处理，它是 MyBatis 直接和数据库执行 SQL 脚本的对象，另外它也实现了 MyBatis 的一级缓存；

**ParameterHandler**：拦截参数的处理；

**ResultSetHandler**：拦截结果集的处理。

**自定义插件实现关键**

MyBatis 插件要实现 Interceptor 接口，接口包含的方法，如下：

```java
public interface Interceptor {
  Object intercept(Invocation invocation) throws Throwable;
  Object plugin(Object target);
  void setProperties(Properties properties);
}
```

setProperties 方法是在 MyBatis 进行配置插件的时候可以配置自定义相关属性，即：接口实现对象的参数配置；

plugin 方法是插件用于封装目标对象的，通过该方法我们可以返回目标对象本身，也可以返回一个它的代理，可以决定是否要进行拦截进而决定要返回一个什么样的目标对象，官方提供了示例：return Plugin. wrap(target, this)；

intercept 方法就是要进行拦截的时候要执行的方法。

自定义插件实现示例

官方插件实现：

```java
@Intercepts({@Signature(type = Executor. class, method = “query”, args = {MappedStatement. class, Object. class, RowBounds. class, ResultHandler. class})})
public class TestInterceptor implements Interceptor {
  public Object intercept(Invocation invocation) throws Throwable {
    Object target = invocation. getTarget(); //被代理对象
    Method method = invocation. getMethod(); //代理方法
    Object[] args = invocation. getArgs(); //方法参数
    // do something . . . . . . 方法拦截前执行代码块
    Object result = invocation. proceed();
    // do something . . . . . . . 方法拦截后执行代码块
    return result;
  }
  public Object plugin(Object target) {
    return Plugin. wrap(target, this);
  }
}
```



## 简述Mybatis 的插件运行原理，以及如何编写一个插件

Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、Executor 这4 种接口的插件，Mybatis 使用 JDK 的动态代理， 为需要拦截的接口生成代理对象以实现接口方法拦截功能， 每当执行这4 种接口对象的方法时，就会进入拦截方法，具体就是 InvocationHandler 的 invoke() 方法，当然，只会拦截那些你指定需要拦截的方法。

编写插件：实现Mybatis 的 Interceptor 接口并复写 intercept() 方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住， 别忘了在配置文件中配置你编写的插件。