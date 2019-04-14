创建  新用户 -  townhouseadmin   密码 - secret  权限- root 全部权限 【 可以直接给root使用 】

修改名称 config/database.php:16 mysql 改为  system

添加数据库连接 config/database.php:34

```
'system' => [
    'driver' => 'mysql',
    'host' => env('TENANCY_HOST', '127.0.0.1'),
    'port' => env('TENANCY_PORT', '3306'),
    'database' => env('TENANCY_DATABASE', 'townhousedb'),
    'username' => env('TENANCY_USERNAME', 'root'),
    'password' => env('TENANCY_PASSWORD', '123456'),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'strict' => true,
    'engine' => null,
],
```

_.env  添加数据库基本连接_

```

DB_CONNECTION=system
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=townhousedb
DB_USERNAME=root
DB_PASSWORD=123456
//将多租户包生成的UUID长度限制在32个字符以下
LIMIT_UUID_LENGTH_32=true
```







## 题外话

MacOS 数据库管理工具  [https://www.sequelpro.com/](https://www.sequelpro.com/)

window 数据库管理工具 [https://www.navicat.com/en/](https://www.navicat.com/en/)

