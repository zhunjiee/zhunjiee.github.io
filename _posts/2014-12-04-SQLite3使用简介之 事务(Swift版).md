---
layout: post
title: "SQLite3使用简介之 事务(Swift版)"
date: 2014-12-04
comments: false
categories: 实用技术
---

##什么是事务?
举一个最简单的例子:

银行账户中有1000元钱,同时对该账户进行取钱和存钱的操作,注意是同一时间,所以拿到的账户余额应该都是1000元.假设我们先取出500块,按理说账户中应该还剩500块,但同时进行存钱操作,存钱时拿到了账户里原来的存款是1000元,再存1000元就变成了2000元,至此,整个过程结束,你的账户里剩余存款2000元.是不是很高兴?但这种事情在现实中是不会出现的,别想太多


- 事务的作用:

1. 事务能够保证多条SQL语句, 要么一起成功, 要么一起失败

2. 想要保证多条SQL语句一起成功或者一起失败, 就必须在执行多条SQL语句之前开启事物, 只有遇到了提交事务, 数据库才会修改数据

3. 如果中途遇到了意外, 我们可以通过回滚事物来还原以前的数据

4. 提高性能




---

定义方法
   

```

    // 开启事物

    func beginTransaction() -> Bool

    {

        return execSQL("BEGIN TRANSACTION;")

    }

    

    // 提交事物

    func commit() -> Bool

    {

        return execSQL("COMMIT TRANSACTION;")

    }

    

    // 回滚事物

    func rollback() -> Bool

    {

        return execSQL("ROLLBACK TRANSACTION;")

    }

```



使用方法



```

    func test2()
    {
        // 1.开启事物
        SQLiteManager.shareInstance.beginTransaction()
        for i in 0..<100
        {
            let p = Person(name: "zs+ \(i)", age: 500 + i)
            p.insertPerson()
            if i == 5
            {
                // 只要执行到rollback, 系统就会利用副本覆盖已经修改的数据库
                SQLiteManager.shareInstance.rollback()
                break
            }
        }
        // 2.提交事物
        SQLiteManager.shareInstance.commit()

    }


```



##事务性能优化

当向数据库一次性写入很多条数据时,可能会存在一些性能问题.



比如在主线程写入大量数据,会消耗一定的时间,卡住主线程,这对用户体验是非常不好的;



sqlite3_exec该方法内部系统会**自动开启和提交事物**, 默认情况下每执行一条SQL语句都会开启和提交一次事物



重复的开启和提交是非常非常**消耗性能**的, 所以为了避免这种问题, 我们可以**自己开启事务**, 只要我们自己开了事务 ,系统就不会自动帮我们开启事务了



开发中我们自己手动开启事务, 并且利用预编译绑定值 (sqlite3_prepare_v2 和 sqlite3_bind_int64/double/text)



```

    func test3()

    {

        let start = CFAbsoluteTimeGetCurrent()

        



        // 系统默认开启事务, 10000条语句耗时: 18.5379449725151

		// 我们自己手动开启事务, 100000条语句耗时: 0.922056019306183

		// 我们自己手动开启事务, 并且利用预编译绑定值, 100000条数据耗时: 0.08348896503448



	SQLiteManager.shareInstance.beginTransaction()

        

        for i in 0..<100000

        {

            let p = Person(name: "zs+ \(i)", age: 500 + i)

//            p.insertPerson() // 系统默认开启事务方法,写入一条数据开启一次,十分消耗性能

            p.insertBindPerson() // 我们自己手动开启事务的方法,并利用预编译绑定值

        }

        SQLiteManager.shareInstance.commit()

        

        let end = CFAbsoluteTimeGetCurrent()

        print(end - start)

    }

```







我们自己手动开启事务, 并且利用预编译绑定值





```

    func insertBindPerson() -> Bool

    {

        // 1.定义SQL语句

        let sql = "INSERT INTO T_Person \n" +

            "(name, age) \n" +

            "VALUES \n" +

        "( ? , ?);"

        

        // 2.执行SQL语句

        return SQLiteManager.shareInstance.insert(sql, args: name!, age)

    }







    private let SQLITE_TRANSIENT = unsafeBitCast(-1, sqlite3_destructor_type.self)

    

    /// 动态绑定之后再插入

    func insert(sql: String, args: CVarArgType...) -> Bool

    {



        var stmt: COpaquePointer = nil

        // 1.预编译SQL语句

        if sqlite3_prepare_v2(db, sql.cStringUsingEncoding(NSUTF8StringEncoding)!, -1, &stmt, nil) != SQLITE_OK

        {

            print("预编译失败")

            return false

        }

        // 2.绑定参数

        var index: Int32 = 1

        for objc in args

        {

            if objc is Int

            {

                /*

                第一个参数: stmt对象

                第二个参数: 需要绑定的字段的索引

                注意: 字段索引从1开始

                第三个参数: 需要绑定到SQL上的值

                */

                sqlite3_bind_int64(stmt, index, sqlite3_int64(objc as! Int))

                

            }else if objc is Double

            {

                sqlite3_bind_double(stmt, index, objc as! Double)

            }else if objc is String

            {

                /*

                第一个参数: stmt对象

                第二个参数: 需要绑定的字段的索引

                第三个参数: 需要绑定到SQL上的值

                第四个参数: 没用过

                第五个参数: bind方法中如何处理传入的字符串

                处理方式有两种: 1.方法内部不管传入的字符串, 不会对传入的字符串进行retain(copy)操作, 所以在使用该字符串时字符串可能已经释放了, 此时系统就会随便设置一个值

                2.方法内部会管理传入的字符串, 所以在使用字符串时字符串不会被释放, 使用完毕之后bind方法内部会自动释放该字符串

                #define SQLITE_STATIC      ((sqlite3_destructor_type)0)  上述1

                #define SQLITE_TRANSIENT   ((sqlite3_destructor_type)-1)  上述2



                */

                let text = objc as! String

                let cText = text.cStringUsingEncoding(NSUTF8StringEncoding)!

                sqlite3_bind_text(stmt, index, cText, -1, SQLITE_TRANSIENT)

            }

            

            index++

        }

    

        // 3.执行SQL语句

        if sqlite3_step(stmt) != SQLITE_DONE

        {

            print("执行SQL语句失败")

            return false

        }

        // 4.重置STMT

        sqlite3_reset(stmt)

        // 5.关闭STMT

        sqlite3_finalize(stmt)

        

        // 6.返回结果

        return true

    }

    

```



系统默认开启事务插入数据的方法



```

    func insertPerson() -> Bool

    {

        // 1.定义SQL语句

        let sql = "INSERT INTO T_Person \n" +

        "(name, age) \n" +

        "VALUES \n" +

        "('\(name!)', \(age));"

        

        // 2.执行SQL语句

        return SQLiteManager.shareInstance.execSQL(sql)

    }

    

    



    /// 执行SQL语句

    func execSQL(sql: String) -> Bool

    {

		// sqlite3_exec方法默认执行一次操作就会开启一次事务,当写入大量数据时十分消耗性能

       if sqlite3_exec(db, sql.cStringUsingEncoding(NSUTF8StringEncoding)!, nil, nil, nil) != SQLITE_OK

        {

            return false

        }

        

        return true

    }

```

##SQL多线程

```

/// 执行SQL语句

    func execSQL(sql: String) -> Bool

    {

        // 2.执行SQL语句

        /*

        第1个参数: 一个已经打开的数据库

        第2个参数: 需要执行的SQL语句

        第3个参数: 执行SQL之后的回调

        第4个参数: 第三个参数的第一个参数

        第5个参数: 错误信息的指针, 如果执行过程中发生错误就会给传入的指针赋值

        */

        if sqlite3_exec(db, sql.cStringUsingEncoding(NSUTF8StringEncoding)!, nil, nil, nil) != SQLITE_OK

        {

            return false

        }

        

        return true

    }

    

    // 创建一个串行队列

    let queue = dispatch_queue_create("com.zhunjiee", DISPATCH_QUEUE_SERIAL)

    

    /// 异步执行SQL语句

    func execSQLQueue(action: (db: COpaquePointer)->())

    {

        dispatch_async(queue) { () -> Void in

            print(NSThread.currentThread())

            action(db: self.db)

        }

    }

```



```

override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {

        

        SQLiteManager.shareInstance.execSQLQueue { (db) -> () in

            

            let sql = "INSERT INTO T_Person \n" +

                "(name, age) \n" +

                "VALUES \n" +

            "('xxxxx', 9999);"

            

            print(NSThread.currentThread())

            sqlite3_exec(db, sql.cStringUsingEncoding(NSUTF8StringEncoding)!, nil, nil, nil)

}

```

