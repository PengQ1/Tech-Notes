
Mysql Document的官网：https://dev.mysql.com/doc/

官网里的Demo程序：
```
/* Standard C++ includes */
#include <stdlib.h>
#include <iostream>

/*
  Include directly the different
  headers from cppconn/ and mysql_driver.h + mysql_util.h
  (and mysql_connection.h). This will reduce your build time!
*/
#include "mysql_connection.h"

#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/statement.h>

using namespace std;

int main(void)
{
cout << endl;
cout << "Running 'SELECT 'Hello World!' »
   AS _message'..." << endl;

try {
  sql::Driver *driver;
  sql::Connection *con;
  sql::Statement *stmt;
  sql::ResultSet *res;

  /* Create a connection */
  driver = get_driver_instance();
  con = driver->connect("tcp://127.0.0.1:3306", "root", "root");
  /* Connect to the MySQL test database */
  con->setSchema("test");

  stmt = con->createStatement();
  res = stmt->executeQuery("SELECT 'Hello World!' AS _message"); // replace with your statement
  while (res->next()) {
    cout << "\t... MySQL replies: ";
    /* Access column data by alias or column name */
    cout << res->getString("_message") << endl;
    cout << "\t... MySQL says it again: ";
    /* Access column fata by numeric offset, 1 is the first column */
    cout << res->getString(1) << endl;
  }
  delete res;
  delete stmt;
  delete con;

} catch (sql::SQLException &e) {
  cout << "# ERR: SQLException in " << __FILE__;
  cout << "(" << __FUNCTION__ << ") on line " »
     << __LINE__ << endl;
  cout << "# ERR: " << e.what();
  cout << " (MySQL error code: " << e.getErrorCode();
  cout << ", SQLState: " << e.getSQLState() << " )" << endl;
}

cout << endl;

return EXIT_SUCCESS;
}
```

使用命令build一下：
```
g++ DataOps.cpp -o DataOps
```
会报错：
```
DataOps.cpp:10:10: fatal error: 'mysql_connection.h' file not found
#include "mysql_connection.h"
         ^~~~~~~~~~~~~~~~~~~~
1 error generated.
```

很明显，这里面是需要一些libaray, 而你本地没有。因此需要下载。

下载Mysql Connector/C++ 8.0:
https://dev.mysql.com/downloads/connector/cpp/
根据自己的系统选择，我是Mac系统，因此选择了MacOS。
下载完，解压，然后把它们move到一个你放lib的地方，我选择在/usr/lib/下新建了一个mysql的目录，并把里面的内容都移进去了。
```
mysql $ ls
INFO_BIN	LICENSE		include
INFO_SRC	README		lib64
```
然后使用命令进行build:
```
g++ DataOps.cpp -o DataOps -I /usr/lib/mysql/include/jdbc/ -L /usr/lib/mysql/lib64/
In file included from DataOps.cpp:10:
/usr/lib/mysql/include/jdbc/mysql_connection.h:37:10: fatal error:
      'boost/shared_ptr.hpp' file not found
#include <boost/shared_ptr.hpp>
         ^~~~~~~~~~~~~~~~~~~~~~
1 error generated.
```
提示还缺一个boost的库，于是需要去boost官网上下载：
https://dl.bintray.com/boostorg/release/1.72.0/source/
之后还是一个套路，建议挪到/usr/lib/目录下，然后在自己的编译选项里面加上这个路径，


最终编译选项：
```
g++ DataOps.cpp -o DataOps -I /usr/lib/mysql/include/jdbc/  -I /usr/lib/boost/  -L /usr/lib/mysql/lib64/ -lmysqlcppconn
```

因为用到很多动态链接库，请务必执行以下命令，export一个环境变量，我因为这个问题调试了快一个下午，总是提示我动态库找不到:
```
export DYLD_LIBRARY_PATH=/usr/lib/mysql/lib64
```
测试代码：
```
/*
// Create a table 

CREATE TABLE IF NOT EXISTS `iwen_user_info`(
   `uid` INT UNSIGNED AUTO_INCREMENT,
   `username` VARCHAR(100) NOT NULL,
   `password` VARCHAR(40) NOT NULL,
   `create_date` DATE,
   PRIMARY KEY ( `uid` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE IF NOT EXISTS `chat_record`(
   `message_id` INT UNSIGNED AUTO_INCREMENT,
   `username` VARCHAR(100) NOT NULL,
   `message` VARCHAR(300) NOT NULL,
   `time` DATETIME,
   PRIMARY KEY ( `message_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;


// Insert data

INSERT INTO iwen_user_info 
(username, password, create_date) 
VALUES
("pengqi", "ca$hc0w", NOW());

INSERT INTO chat_record 
(username, message, time) 
VALUES
("pengqi", "Today is a good day!", NOW());

INSERT INTO chat_record 
(username, message, time) 
VALUES
("jingwen", "Tomorrow is another day!", NOW());

*/

/*

Compile command:
g++ DataOps.cpp -o DataOps -I /usr/lib/mysql/include/jdbc/  -I /usr/lib/boost/  -L /usr/lib/mysql/lib64/ -lmysqlcppconn

Remember:
Must export a environment variable!!! It takes abount half day to debug on this.
export DYLD_LIBRARY_PATH=/usr/lib/mysql/lib64

*/

/* Standard C++ includes */
#include <stdlib.h>
#include <iostream>

/*
  Include directly the different
  headers from cppconn/ and mysql_driver.h + mysql_util.h
  (and mysql_connection.h). This will reduce your build time!
*/
#include "mysql_connection.h"

#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/statement.h>

using namespace std;

int main(void)
{

try {
  sql::Driver *driver;
  sql::Connection *con;
  sql::Statement *stmt;
  sql::ResultSet *res;

  /* Create a connection */
  driver = get_driver_instance();
  con = driver->connect("tcp://127.0.0.1:3306", "root", "123456");

  /* Connect to the MySQL test database */
  con->setSchema("iWen");

  stmt = con->createStatement();
  res = stmt->executeQuery("SELECT * FROM iwen_user_info");

  while (res->next()) {
    cout << "\t... MySQL replies: ";
    /* Access column data by alias or column name */
    cout << res->getString("username") << endl;
    cout << "\t... MySQL says it again: ";
    /* Access column data by numeric offset, 1 is the first column */
    cout << res->getString(1) << endl;
  }
  delete res;
  delete stmt;
  delete con;

} catch (sql::SQLException &e) {
  cout << "# ERR: SQLException in " << __FILE__;
  cout << "(\" << __FUNCTION__ << \") on line " << __LINE__ << endl;
  cout << "# ERR: " << e.what();
  cout << " (MySQL error code: " << e.getErrorCode();
  cout << ", SQLState: " << e.getSQLState() << " )" << endl;
}

cout << endl;

return EXIT_SUCCESS;
}
```

以下是我的执行结果：
```
./DataOps
	... MySQL replies: pengqi
	... MySQL says it again: 1
	... MySQL replies: jingwen
	... MySQL says it again: 2
	... MySQL replies: pyin
	... MySQL says it again: 3
```

当然，这需要首先在电脑上安装一个Mysql,然后按照代码注释里面的语句进行创建table, insert数据，这样才能query成功。