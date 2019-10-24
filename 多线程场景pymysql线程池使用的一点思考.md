最近在做的项目中有用到pymysql线程池的使用，并且是在多线程场景下使用，使用过程中实际跑多线程遇到了一些错误，所以借此机会简单分析了一下。

先看一下下面的代码多线程场景下跑是否会有问题？

dao/dbcon.py

``` python
# -*-coding:utf-8-*-
import pymysql
from DBUtils.PooledDB import PooledDB

from conf.config import ConfParser
from logger.logger import *

conf = ConfParser()


class DB(object):
    """docstring for DbConnection"""
    __pool = None

    def __init__(self):
        self.pool = DB.__get_conn()
        self.conn = None
        self.cursor = None

    @staticmethod
    def __get_conn():
        if DB.__pool is None:
            try:
                DB.__pool = PooledDB(creator=pymysql, host=conf.get("DB", "host"), port=int(conf.get("DB", "port")), user=conf.get("DB", "user"), passwd=conf.get("DB", "passwd"), db=conf.get("DB", "db"), charset=conf.get("DB", "charset"))
            except Exception as e:
                logging.error("%s : %s" % (Exception, e))
        return DB.__pool

    def connect(self, cursor=pymysql.cursors.DictCursor):
        self.conn = self.pool.connection()
        self.cursor = self.conn.cursor(cursor=cursor)
        return self.cursor

    def close(self):
        self.cursor.close()
        self.conn.close()

    def query_sql(self, sql, params=None):
        self.connect()
        self.cursor.execute(sql, params)
        result = self.cursor.fetchall()
        self.close()
        return result

    def execute_sql(self, sql, params=None):
        self.connect()
        try:
            self.cursor.execute(sql, params)
            result = self.cursor.lastrowid
            self.conn.commit()
            self.close()
        except Exception as e:
            self.conn.rollback()
            self.close()
            logging.error(str(e))
            raise Exception("database commit error")
        return result

    def update_sql(self, sql, params=None):
        self.connect()
        try:
            result = self.cursor.execute(sql, params)
            self.conn.commit()
            self.close()
        except Exception as e:
            self.conn.rollback()
            self.close()
            logging.error(str(e))
            raise Exception("database commit error")
        return result
        
```

dao/querydb.py

``` python
# -*-coding:utf-8-*-

from dao.dbcon import DB
from conf.config import ConfParser
db = DB()
conf = ConfParser()
local_ip = conf.get("EMU", "ServerLocalIP")


class QueryDB(object):

    def __init__(self):
        self.local_ip = local_ip

    def getBaseNodeDiskImgFile(self, node_id):
        sql = "SELECT diskimg_file From `tb_node` WHERE node_id = %s;"
        res = db.query_sql(sql, (node_id,))
        return res[0]['diskimg_file']

    def GetNodeBName(self, nodeid):
        sql = "SELECT node_b_name from ``tb_node``  where node_id=%s"
        res = db.query_sql(sql, (nodeid,))
        return res

    def get_iso_path(self, iso_id):
        sql = "SELECT * FROM `tb_vm_iso` WHERE id = %s;"
        result = db.query_sql(sql, (iso_id,))[0]
        return result["file_path"]

    def get_case(self, case_id):
        sql = "SELECT * FROM `tb_case` WHERE case_id = %s;"
        result = db.query_sql(sql, (int(case_id),))
        return result

    def get_template(self, template_id):
        sql = "SELECT * FROM `tb_vm_template` WHERE id = %s;"
        result = db.query_sql(sql, (template_id,))
        return result
 
```

storage/storage.py

``` python
# -*- coding: utf-8 -*-

from dao.querydb import QueryDB

db = QueryDB()


class CentralizedStorage(object):

    def __init__(self):
        pass

    def test1(self, thread_name):
        res = db.getBaseNodeDiskImgFile(121212)
        print(thread_name, "getBaseNodeDiskImgFile", res)

    def test2(self, thread_name):
        db2 = QueryDB()
        res = db2.GetNodeBName(121212)
        print(thread_name, "GetNodeBName", res)

    def test3(self, thread_name):
        db3 = QueryDB()
        res = db3.get_iso_path(114)
        print(thread_name, "get_iso_path", res)

    def test4(self, thread_name):
        db4 = QueryDB()
        res = db4.get_case(20160802145038043410)
        print(thread_name, "get_case", res)
```

main.py

``` python
# -*- coding: utf-8 -*-

import threading
from storage.storage import CentralizedStorage
from dao.querydb import QueryDB


def db_test(thread_name):
    storage = CentralizedStorage()
    storage.test1(thread_name)
    storage.test2(thread_name)
    storage.test3(thread_name)
    storage.test4(thread_name)
    db_object = QueryDB()
    res = db_object.get_template(1615)
    print(thread_name, "get_template:", res)


if __name__ == "__main__":
    for i in range(10):
        thread = threading.Thread(target=db_test, args=(("Thread" + str(i) + ":"),))
        thread.setDaemon(False)
        thread.start()
```

上面代码单线程执行时一点问题也没有，但是多线程执行就会报如下错误：

![图片](https://uploader.shimo.im/f/BfjB2ZIUHyAPzbr7.png!thumbnail)

我们可以分析一下上面的代码，注意看dao/querydb.py这个文件第5行db = DB()，这个文件中所有的查询语句都是共用这一个DB实例的，再看dao/dbcon.py这个文件，DB这个类创建了一个连接池，供所有sql执行来用，还封装了几个底层真正执行sql语句的函数，重点在于这几个sql执行语句。

我们来看一下，它们基本流程都是从线程池取出一个connect赋给该实例成员变量self.conn，然后获得游标赋给该实例成员变量self.cursor，然后通过游标执行sql语句，最后close掉。问题就在于这里，前面我们看到过，DB只进行了一次实例化，出来的实例大家共用，如果多线程场景下，线程1获得了conn和cursor并赋值给DB实例的成员变量后还没完成后续操作，线程2就进入了，重新获取了conn和cursor并重新给DB实例的成员变量赋值，因为共用一个DB实例，这样一定会覆盖掉线程1时候赋的值，线程1再继续执行就可能会出错。

分析出了出错原因，那么如何修改呢？通过分析得出本质上就是让conn和cursor不共用，那就可以对应两种思路，一是DB实例不共用，或者是DB实例共用但conn和cursor不共用。

* DB实例不共用

这种方式其他文件不用动，就是在dao/querydb.py文件中每一个函数下都进行一次DB的实例化，公共的DB实例可以删掉，修改后的代码如下：

``` python
# -*-coding:utf-8-*-

from dao.dbcon_bac import DB
from conf.config import ConfParser
conf = ConfParser()
local_ip = conf.get("EMU", "ServerLocalIP")


class QueryDB(object):

    def __init__(self):
        self.local_ip = local_ip

    def getBaseNodeDiskImgFile(self, node_id):
        db = DB()
        sql = "SELECT diskimg_file From `tb_node` WHERE node_id = %s;"
        res = db.query_sql(sql, (node_id,))
        return res[0]['diskimg_file']

    def GetNodeBName(self, nodeid):
        db = DB()
        sql = "SELECT node_b_name from `tb_node ` where node_id=%s"
        res = db.query_sql(sql, (nodeid,))
        return res

    def get_iso_path(self, iso_id):
        db = DB()
        sql = "SELECT * FROM `tb_vm_iso` WHERE id = %s;"
        result = db.query_sql(sql, (iso_id,))[0]
        return result["file_path"]

    def get_case(self, case_id):
        db = DB()
        sql = "SELECT * FROM `tb_case` WHERE case_id = %s;"
        result = db.query_sql(sql, (int(case_id),))
        return result

    def get_template(self, template_id):
        db = DB()
        sql = "SELECT * FROM `tb_vm_template` WHERE id = %s;"
        result = db.query_sql(sql, (template_id,))
        return result
        
```

因为连接池我们需要共用的，也就是只能有一个连接池，那这种方式呢我们看一下连接池会不会也变得不共用，每次实例都会创建一个新的连接池呢？
我们注意看dbcon.py里的实现，__pool是一个类的成员变量而非实例成员变量，在第一次实例化时就会生成一个连接池赋值给这个类成员变量，以后的实例化会先判断这个类成员变量DB.__pool是否为空，如果不为空就不再重新生成连接池，这样就保证了每一个实例实际上都是共用了类的连接池，而且不会重复生成。如果不先进行判空，则会有问题，如果判了空，就是ok的。

* DB实例共用但conn和cursor不共用

那这一种方式呢，其他文件不动，修改dbcon.py函数，每一个sql实际执行函数里都重新从连接池里取得一个conn并获取cursor，这两个变量不再赋值给实例成员变量，而是作为局部变量使用，这样不同线程里调用sql执行函数都会用自己的conn而不会共用，这样就跟DB被实例化几次没什么关系了，代码如下：

``` python
# -*-coding:utf-8-*-
import pymysql
from DBUtils.PooledDB import PooledDB

from conf.config import ConfParser
from logger.logger import *

conf = ConfParser()


class DB(object):
    """docstring for DbConnection"""
    __pool = None

    def __init__(self):
        self.pool = DB.__get_conn_pool()

    @staticmethod
    def __get_conn_pool():
        if DB.__pool is None:
            try:
                DB.__pool = PooledDB(creator=pymysql, host=conf.get("DB", "host"), port=int(conf.get("DB", "port")),
                                     user=conf.get("DB", "user"), passwd=conf.get("DB", "passwd"),
                                     db=conf.get("DB", "db"), charset=conf.get("DB", "charset"))
            except Exception as e:
                logging.error("%s : %s" % (Exception, e))
        return DB.__pool

    def _get_connection(self):
        conn = self.pool.connection()
        cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
        return conn, cursor

    def _close_connection(self, conn, cursor):
        if cursor:
            cursor.close()
        if conn:
            conn.close()

    def query_sql(self, sql, params=None):
        conn, cursor = self._get_connection()
        try:
            cursor.execute(sql, params)
            result = cursor.fetchall()
            self._close_connection(conn, cursor)
        except Exception as e:
            self._close_connection(conn, cursor)
            logging.error(str(e))
            raise Exception("database execute error")
        return result

    def execute_sql(self, sql, params=None):
        conn, cursor = self._get_connection()
        try:
            cursor.execute(sql, params)
            result = cursor.lastrowid
            conn.commit()
            self._close_connection(conn, cursor)
        except Exception as e:
            conn.rollback()
            self._close_connection(conn, cursor)
            logging.error(str(e))
            raise Exception("database commit error")
        return result

    def update_sql(self, sql, params=None):
        conn, cursor = self._get_connection()
        try:
            result = cursor.execute(sql, params)
            conn.commit()
            self._close_connection(conn, cursor)
        except Exception as e:
            conn.rollback()
            self._close_connection(conn, cursor)
            logging.error(str(e))
            raise Exception("database commit error")
        return result
        
```


