### Spring
1. 介绍一下Spring  
Spring是一个轻量级的IOC和AOP框架，可以一站式构建企业级应用，解决企业应用开发的复杂性  
2. Spring中bean的作用域  
singleton，prototype，request，session，globalSession  
3. Spring中bean的生命周期  
![bean的生命周期](https://uploadfiles.nowcoder.com/images/20180926/308572_1537967995043_4D7CF33471A392D943F00167D1C86C10)
- 根据配置文件获取bean的定义，解析成BeanDefinition注册到Spring容器中；  
- 使用getBean获取bean并根据配置对bean进行初始化；  
- 检查Aware接口，如果bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID，如果bean实现了BeanFactoryAware接口，工厂调用Bean的setBeanFactory()方法传入工厂自身；  
- 将Bean实例传递给bean的前置处理器的postProcessBeforeInitialization(Object bean，String beanname)方法；  
- 调用bean的初始化方法init；  
- 将Bean实例传递给bean的后置处理器的postProcessAfterInitialization(Object bean，String beanname)方法；  
- 使用Bean；  
- 容器关闭之前，调用bean的销毁方法  
### AOP
1. AOP的原理  
面向切面编程，作为面向对象的一种补充，用于处理系统中分布于各个模块的横切关注点，比如事务管理、日志、缓存等。实现的关键在于AOP框架自动创建的**AOP代理**，AOP代理主要分为静态代理和动态代理，静态代理(在编译阶段生成AOP代理类，即编译时增强)的代表是AspectJ，动态代理代表是SpringAOP，实现方式为JDK动态代理和CDLib动态代理。JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口，核心是invocation接口和proxy类。如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。


### Mybatis
1. 动态SQL  
对于一些复杂的查询，我们可能会指定**多个查询条件**，但是这些条件可能存在也可能不存在，需要根据用户指定的条件动态生成SQL语句。如果不使用持久层框架我们可能需要自己拼装SQL语句，还好MyBatis提供了动态SQL的功能来解决这个问题。  
![用于实现动态SQL的元素](https://images2017.cnblogs.com/blog/1240732/201711/1240732-20171101153219685-1084206827.png)  
使用示例：在配置文件中，使用动态sql元素配置可能的sql语句  
实体类：Customer
```
public class Customer {
    private Integer id;             //主键
    private String username;        //客户名称
    private String jobs;            //职业
    private String phone;           //电话
    //省略setter和getter方法
}
```
配置文件CustomerMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

        <!--namespace表示命名空间-->
<mapper namespace="com.ma.mapper.CustomerMapper">

    <!--if元素使用-->
    <select id="findCustomerByNameAndJobs" parameterType="com.ma.po.Customer" resultType="com.ma.po.Customer">
        select * from t_customer where 1=1
        <if test="username != null and username != ''">
            and username like '%${username}%'
        </if>
        <if test="jobs != null and jobs !=''">
            and jobs = #{jobs}
        </if>
    </select>
    <!--choose when otherwise使用-->
    <select id="findCustomerByNameOrJobs" parameterType="com.ma.po.Customer" resultType="com.ma.po.Customer">
        select * from t_customer where 1=1
        <choose>
            <when test="username != null and username !=''">
                and username like '%${username}%'
            </when>
            <when test="jobs != null and jobs != ''">
                and jobs = #{jobs}
            </when>
            <otherwise>
                and phone is not null
            </otherwise>
        </choose>
    </select>

    <!--where trim 使用-->
    <select id="findCustomerByNameAndJobs1" parameterType="com.ma.po.Customer" resultType="com.ma.po.Customer">
        select * from t_customer
        <where>
            <if test="username != null and username != ''">
                and username like '%${username}%'
            </if>
            <if test="jobs != null and jobs !=''">
                and jobs = #{jobs}
            </if>
        </where>
    </select>
    <!--where trim 使用-->
    <select id="findCustomerByNameAndJobs2" parameterType="com.ma.po.Customer" resultType="com.ma.po.Customer">
        select * from t_customer
        <trim prefix="where" prefixOverrides="and">
            <if test="username != null and username != ''">
                and username like '%${username}%'
            </if>
            <if test="jobs != null and jobs !=''">
                and jobs = #{jobs}
            </if>
        </trim>
        <if test="username != null and username != ''">
            and username like '%${username}%'
        </if>
        <if test="jobs != null and jobs !=''">
            and jobs = #{jobs}
        </if>
    </select>

    <!--set使用-->
    <update id="updateCustomer" parameterType="com.ma.po.Customer">
        update t_customer
        <set>
            <if test="username != null and username != ''">
                username = #{username}
            </if>
            <if test="jobs != null and jobs != ''">
                jobs = #{jobs}
            </if>
            <if test="phone != null and phone != ''">
                phone = #{phone},
            </if>
        </set>
        where id = #{id}
    </update>

    <!--foreach元素使用-->
    <select id="findCustomerByIds" parameterType="List" resultType="com.ma.po.Customer">
        select * from t_customer where id in
        <foreach item="id" collection="list" open="(" separator="," close=")">
            #{id}
        </foreach>
    </select>
    <!--bind元素使用,根据客户名模糊查询信息-->
    <select id="findCustomerByName" parameterType="com.ma.po.Customer" resultType="com.ma.po.Customer">
        <!--_parameter.getUsername()也可以直接写成传入的字段属性名，即username-->
        <bind name="pattern_username" value="'%'+_parameter.getUsername()+'%'"/>
        select * from t_customer
        where
        username like #{pattern_username}
    </select>
</mapper>
```
测试使用MyBatisTest  
```
import com.ma.po.Customer;
import com.ma.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * @author mz
 * @version V1.0
 * @Description: Mybatis测试
 * @create 2017-11-01 15:20
 */
public class MybatisTest {


    /**
     * 根据姓名和职业查询客户if
     */
    @Test
    public void findCustomerByNameAndJobsTest() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建Customer对象，封装需要组合查询的条件
        Customer customer = new Customer();
        customer.setUsername("jack");
        customer.setJobs("teacher");
        //执行SqlSession的查询方法，返回结果集
        List<Customer>customers = sqlSession.selectList("com.ma.mapper.CustomerMapper.findCustomerByNameAndJobs", customer);
        //输出结果
        for (Customer customer1 : customers) {
            System.out.println(customer1);
        }
        //关闭SqlSession
        sqlSession.close();
    }

    /**
     * 根据客户名或职业查询客户信息choose when otherwise
     */
    @Test
    public void findCustomerByNameOrJobsTest() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建Customer对象，封装需要组合查询的条件
        Customer customer = new Customer();
        customer.setUsername("jack");
        customer.setJobs("teacher");
        //执行SqlSession的查询方法，返回结果集
        List<Customer> list = sqlSession.selectList("com.ma.mapper.CustomerMapper.findCustomerByNameOrJobs", customer);
        for (Customer customer1 : list) {
            System.out.println(customer1);
        }
        //关闭SqlSession
        sqlSession.close();
    }
    /**
     * 根据姓名和职业查询客户where
     */
    @Test
    public void findCustomerByNameAndJobs1Test() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建Customer对象，封装需要组合查询的条件
        Customer customer = new Customer();
        customer.setUsername("jack");
        customer.setJobs("teacher");
        //执行SqlSession的查询方法，返回结果集
        List<Customer>customers = sqlSession.selectList("com.ma.mapper.CustomerMapper.findCustomerByNameAndJobs1", customer);
        //输出结果
        for (Customer customer1 : customers) {
            System.out.println(customer1);
        }
        //关闭SqlSession
        sqlSession.close();
    }
    /**
     * 根据姓名和职业查询客户trim
     */
    @Test
    public void findCustomerByNameAndJobs2Test() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建Customer对象，封装需要组合查询的条件
        Customer customer = new Customer();
        customer.setUsername("jack");
        customer.setJobs("teacher");
        //执行SqlSession的查询方法，返回结果集
        List<Customer>customers = sqlSession.selectList("com.ma.mapper.CustomerMapper.findCustomerByNameAndJobs2", customer);
        //输出结果
        for (Customer customer1 : customers) {
            System.out.println(customer1);
        }
        //关闭SqlSession
        sqlSession.close();
    }

    /**
     * 更新数据，set测试
     */
    @Test
    public void updateCustomerTest() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建Customer对象，封装需要组合查询的条件
        Customer customer = new Customer();
        customer.setId(3);
        customer.setPhone("1254587855");
        //执行SqlSession的查询方法，返回影响行数
        int rows = sqlSession.update("com.ma.mapper.CustomerMapper.updateCustomer", customer);

        if (rows > 0) {
            System.out.println("修改了"+ rows +"条数据!");
        } else {
            System.out.println("failed");
        }
        //提交事务
        sqlSession.commit();
        //关闭SqlSession
        sqlSession.close();
    }

    /**
     * foreach测试
     */
    @Test
    public void findCustomerByIdsTest() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建List集合，封装查询id
        List<Integer> ids = new ArrayList<>();
        ids.add(1);
        ids.add(2);
        //执行SqlSession的查询方法，返回结果集
        List<Customer> customers = sqlSession.selectList("com.ma.mapper.CustomerMapper.findCustomerByIds", ids);

        for (Customer customer : customers) {
            System.out.println(customer);
        }
        //关闭SqlSession
        sqlSession.close();
    }

    /**
     * 模糊查询bind
     */
    @Test
    public void findCustomerByNameTest() {
        //获取SqlSession
        SqlSession sqlSession = MybatisUtils.getSession();
        //创建Customer对象，封装需要组合查询的条件
        Customer customer = new Customer();
        customer.setUsername("j");
        //执行SqlSession的查询方法，返回结果集
        List<Customer> customers = sqlSession.selectList("com.ma.mapper.CustomerMapper.findCustomerByName", customer);
        for (Customer customer1 : customers) {
            System.out.println(customer1);
        }
        //关闭SqlSession
        sqlSession.close();
    }
}
```
2. Mybatis是如何防止注入的  
SQL注入解释：恶意的SQL语句被插入到实体字段中，攻击参数检验不足的程序，比如(or ‘1’=‘1’)，在安全性要求很高的应用中，经常使用将SQL语句替换为存储过程的方式防止SQL注入，实际中，可能不需要这么死板的方式
