### Spring事务

#### 事务隔离级别

#### 事务的传播行为

一个事务方法被另一个事务方法调用时，该事务方法应该如何执行 

###### **REQUIRED**

```Java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)
```

未加事务

```java
 @Override
    //@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
    public int addTest() throws Exception {
        Test test = Test.builder()
                .name("test"+new Random().nextInt())
                .build();
        log.info("name:"+test.getName());
        int count = transactionMapper.addTest(test);
        if (count<1){
            throw new Exception("add test error");
        }
        int i = 1/0;
        return count;
    }



try{
            
        }
        catch(Exception e){
            log.error("#####"+e.getMessage());
            throw e;
        }
```



执行添加操作前，数据库Test表

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\required.png)

执行addTest()方法，控制台信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\required_console.png)

数据库Test表

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\required_database.png)

**未添加事务抛出异常后，数据依然添加到Test表**



添加事务注解，执行同样操作

```java
    @Override
    @Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
    public int addTest() throws Exception {
        Test test = Test.builder()
                .name("test"+new Random().nextInt())
                .build();
        log.info("name:"+test.getName());
        int count = transactionMapper.addTest(test);
        if (count<1){
            throw new Exception("add test error");
        }
        int i = 1/0;
        return count;
    }
```

控制台信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\required_database1.png)

数据库没有新增数据，说明数据回滚了

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\required_database.png)

**添加默认事务，抛出异常数据回滚**
###### SUPPORTS
```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
如果当前环境有事务，就加入到当前事务；如果没有事务，就以非事务的方式执行
```

```java
	@Override
    @Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
    public int addTest() throws Exception {
        Test test = Test.builder()
                .name("test"+new Random().nextInt())
                .build();
        log.info("name:"+test.getName());
        int count = transactionMapper.addTest(test);
        if (count<1){
            throw new Exception("add test error");
        }
        int i = 1/0;
        return count;
    }
```

该方法当前没有事务，打印控制台信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support.png)

Test表信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support_database.png)

抛出异常，但数据没有回滚，说明当前方法以非事务的方式执行



当前方法存在与事务中

```java
@Override
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
public void test() throws Exception {
    addTest();
    int i = 1/0;
}

@Override
@Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
public int addTest() throws Exception {
    Test test = Test.builder()
        .name("test"+new Random().nextInt())
        .build();
    log.info("name:"+test.getName());
    int count = transactionMapper.addTest(test);
    if (count<1){
        throw new Exception("add test error");
    }
    //int i = 1/0;
    return count;
}
```

打印控制台信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support_console.png)

Test表信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support_database1.png)

抛出异常后数据回滚，说明当前环境有事务，就加入当前事务

另一种情况

```java
@Override
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRED)
public void test() throws Exception {
    addTest();
    //int i = 1/0;
}

@Override
@Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
public int addTest() throws Exception {
    Test test = Test.builder()
        .name("test"+new Random().nextInt())
        .build();
    log.info("name:"+test.getName());
    int count = transactionMapper.addTest(test);
    if (count<1){
        throw new Exception("add test error");
    }
    int i = 1/0;
    return count;
}
```

控制台信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\supports_console1.png)

Test表信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support_database1.png)

手动方式开启事务

```java
@Override
public void test() throws Exception {
    // 手动开启事务  start
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
    TransactionStatus status = txManager.getTransaction(def);
    addTest();
    int i = 1/0;
    // 手动提交事务
    txManager.commit(status);
}
@Override
@Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
public int addTest() throws Exception {
    Test test = Test.builder()
        .name("test"+new Random().nextInt())
        .build();
    log.info("name:"+test.getName());
    int count = transactionMapper.addTest(test);
    if (count<1){
        throw new Exception("add test error");
    }
    int i = 1/0;
    return count;
}
```

控制台信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support_console2.png)

Test表信息

![](C:\Users\Admin\Desktop\工作\uds\学习\事务\support_database1.png)

抛出异常后数据回滚，手动开启事务也可以回滚

**注意**

其实只要父级方法开启事务，调用子方法addTest(),相当于this.addTest()，就是直接在test()里累加代码，也是test()方法的一部分，子方法不加SUPPORTS事务，数据也会回滚

###### REQUIRES_NEW

父级异常，子方法可以正常提交

方法A存在事务，方法A（将方法A称为父级方法）调用方法B，方法B为REQUIRES_NEW时，方法B会创建一个新的事务，而不用方法A的事务，这样如果方法B没有异常，方法A调用完方法B，方法A又报了异常，只会导致方法A回滚，而方法B可以正常提交

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void methodA() throws Exception {
    Test2 test2 = Test2.builder()
        .salary(new Random().nextInt()*1000)
        .build();
    int count = transactionMapper.addTest2(test2);
    if (count<1){
        throw new Exception("add test2 error");
    }
    methdoB();
    int i =1/0;

}

@Override
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRES_NEW)
public void methdoB() throws Exception {
    Test test = Test.builder()
        .name("test"+new Random().nextInt())
        .build();
    log.info("name:"+test.getName());
    int count = transactionMapper.addTest(test);
    if (count<1){
        throw new Exception("add test error");
    }
}
```

实验结果应该是：Test2表没有数据，而Test表有数据，说明A方法回滚，B方法没有回滚正常提交

但实际结果是：Test2表和Test表都没有数据？

使用exposeProxy属性的作用 --- 同一个对象里方法间调用事务传播行为生效的方法

###### NESTED

父级异常，它必然回滚，它异常，父级可以不回滚



