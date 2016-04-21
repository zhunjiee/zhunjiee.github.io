---
layout: post
title: "SQLite3使用简介(Swift版)"
date: 2014-12-03
comments: false
categories: 实用技术
---

##SQLite3使用步骤

###在OC中的使用步骤

1. 添加SQLite动态库：libsqlite3.dylib

2. 导入主头文件：`#import <sqlite3.h>`

3. 利用C语言函数创建\打开数据库，编写SQL语句



###在Swift中的使用步骤:

>注:



- 由于SQLite3.0是纯C语言的库,在swift中使用时需要建立桥接文件才可以使用

	- 使用SQLite3.0必须先导入libsqlite3.0.tbd库

	- 新建 SQLite+Bridge.h 桥接文件

	- build settings 中搜索 bridge ,code generation中的Bridge Header 中写入桥接文件的全目录(从项目根目录下一级开始写)

	- SQLite+Bridge.h 中 `#import "SQLite3.h"`



##SQLite3常用函数



1.打开数据库 sqlite3_open



```

int sqlite3_open(

    const char *filename,   // 数据库的文件路径

    sqlite3 **ppDb          // 数据库实例

);

```



2.执行任何SQL语句 sqlite3_exec



```

int sqlite3_exec(

    sqlite3*,                                  // 一个已经打开的数据库实例

    const char *sql,                           // 需要执行的SQL语句

    int (*callback)(void*,int,char**,char**),  // SQL语句执行完毕后的回调

    void *,                                    // 回调函数的第1个参数

    char **errmsg                              // 错误信息

);

```



3.检查SQL语句的合法性（查询前的准备）sqlite3\_prepare_v2



```

int sqlite3_prepare_v2(

    sqlite3 *db,            // 数据库实例

    const char *zSql,       // 需要检查的SQL语句

    int nByte,              // SQL语句的最大字节长度

    sqlite3_stmt **ppStmt,  // sqlite3_stmt实例，用来获得数据库数据

    const char **pzTail

);

```



4.查询一行数据 sqlite3_step



```

int sqlite3_step(sqlite3_stmt*); // 如果查询到一行数据，就会返回SQLITE_ROW

```



5.利用stmt获得某一字段的值（字段的下标从0开始）



```

double sqlite3_column_double(sqlite3_stmt*, int iCol);  // 浮点数据

int sqlite3_column_int(sqlite3_stmt*, int iCol); // 整型数据

sqlite3_int64 sqlite3_column_int64(sqlite3_stmt*, int iCol); // 长整型数据

const void *sqlite3_column_blob(sqlite3_stmt*, int iCol); // 二进制文本数据

const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);  // 字符串数据

```





##开发中的应用



开发中我们需要创建SQLiteManager单例,在此单例方法中我们可以定义打开数据库/执行SQL语句/执行query查询...等方法



```

class SQLiteManager: NSObject {

    

    // 单例

    static let shareInstance: SQLiteManager = SQLiteManager()

    

    

    var db:COpaquePointer = nil

    /// 打开数据库,如果不存在就新建数据库

    func openDB(str: String) -> Bool {

        

        // 1. 生成SQLite文件路径

        let path = str.cacheDir()

        guard let cPath = path.cStringUsingEncoding(NSUTF8StringEncoding) else {

            return false

        }

        

        // 2. 创建数据库文件

        // 第一个参数: 数据库文件的路径

        // 第二个参数: 数据库的指针(句柄), 只要打开成功系统就会将打开的数据库赋值给该指针

        // 后续所有关于数据库的操作, 都依赖于该指针

        // sqlite3_open方法会返回一个整型的值, 告诉我们是否打开成功

        if sqlite3_open(cPath, &db) != SQLITE_OK {

            return false

        }

        

        // 3. 新建表

        if !createTable() {

            return false

        }

        

        return true

    }

    

    

    /// 创建表

    private func createTable() -> Bool {

        // 1.定义SQL语句

        let sqlStr = "CREATE TABLE IF NOT EXISTS T_Person( \n " +

            "id INTEGER PRIMARY KEY AUTOINCREMENT, \n " +

            "name TEXT , \n " +

            "age INTEGER \n " +

        ");"

        

        // 2. 执行并返回SQL语句

        return execSQL(sqlStr)

    }

    

    

    

    /// 执行SQL语句

    func execSQL(sql: String) -> Bool {

        /*

        第1个参数: 一个已经打开的数据库

        第2个参数: 需要执行的SQL语句

        第3个参数: 执行SQL之后的回调

        第4个参数: 第三个参数的第一个参数

        第5个参数: 错误信息的指针, 如果执行过程中发生错误就会给传入的指针赋值

        */

        if sqlite3_exec(db, sql.cStringUsingEncoding(NSUTF8StringEncoding)!, nil, nil, nil) != SQLITE_OK {

            return false

        }

        

        return true

    }

    

    

    

    /// 执行查询语句 -> 返回一个字典数组

    func querySQL(str: String) -> [[String : AnyObject]]? {

        

        // 1.预编译

        /*

        第一个参数: 一个已经打开的数据库

        第二个参数: 需要执行的SQL语句

        第三个参数: 需要执行的SQL语句的长度, 传入-1系统会自动计算

        第四个参数: 结果集指针(以后所有获取数据的操作, 都依赖于该指针)

        第五个参数: 没用过

        */

        var stmt: COpaquePointer = nil

        if sqlite3_prepare_v2(db, str, -1, &stmt, nil) != SQLITE_OK {

            return nil

        }

        

        // 2. 利用结果集取出结果

        var array = [[String : AnyObject]]()

        

        while sqlite3_step(stmt) == SQLITE_ROW {

            // 1. 获取当前行一共有多少列

            let count = sqlite3_column_count(stmt)

            

            var dict = [String : AnyObject]()

            

            for i in 0..<count {

                // 2. 获取每一列的名称

                let cName = sqlite3_column_name(stmt, i)

                let name = String(CString: cName, encoding: NSUTF8StringEncoding)!

                

                // 3. 获取每一列取值的数据类型

                let type = sqlite3_column_type(stmt , i)

                

                switch type {

                case SQLITE_INTEGER:

                    let intValue = sqlite3_column_int(stmt, i)

                    dict[name] = Int(intValue)

                case SQLITE_FLOAT:

                    let floatValue = sqlite3_column_double(stmt, i)

                    dict[name] = floatValue

                case SQLITE_BLOB:

                    // 暂时不做: 因为在开发中不建议将二进制存储到数据库中

                    print("blob")

                case SQLITE_NULL:

                    dict[name] = NSNull()

                case SQLITE3_TEXT:

                    let cTextValue = UnsafePointer<Int8>(sqlite3_column_text(stmt, 1))

                    let textValue = String(CString: cTextValue, encoding: NSUTF8StringEncoding)

                    dict[name] = textValue

                    print(textValue)

                default:

                    print("other")

                }

            }

            array.append(dict)

        }

        return array

    }

}

```



>开发技巧:



>不同于服务端,一般iOS应用在APPDelegate中就打开数据库,这样能提高效率



```

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {

        

        // 1.打开数据库

        SQLiteManager.shareInstance.openDB("demo.sqlite")

        

        return true

    }

```



Person模型实例



```

class Person: NSObject {

    /// ID

    var id: Int = 0

    /// 姓名

    var name: String?

    /// 年龄

    var age: Int = 0

    

    init(name: String, age: Int)

    {

        self.name = name

        self.age = age

    }

    

    init(dict: [String: AnyObject])

    {

        super.init()

        setValuesForKeysWithDictionary(dict)

    }

    

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

    

    /// 查询所有用户信息

    class func loadPersons() -> [Person]? {

        

        // 1.定义查询语句

        let sql = "SELECT id, name, age FROM T_Person;"

        

        // 2.执行SQL语句拿到查询结果

        guard let arrayDict = SQLiteManager.shareInstance.querySQL(sql) else

        {

            return nil

        }

        

        print(arrayDict)

        

        // 3.将查询结果转换为模型

        var models = [Person]()

        for dict in arrayDict as [[String: AnyObject]]

        {

            models.append(Person(dict: dict))

        }

        // 4.返回结果

        return models

    }

}

```





