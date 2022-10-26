#### 安装mysql

- 给mysql配置环境变量，D:\mysql\bin：

- 在根目录下创建data文件夹，一定要是空文件：

- 创建mysql 配置文件(my.ini)，在mysql根目录下创建：

  ``` js
  [Client]
  port = 3306
  [mysqld]
  port = 3306
  basedir=D:\mysql
  datadir=D:\mysql\data
  max_connections=200
  character-set-server=utf8
  default-storage-engine=INNODB
  [mysql]
  default-character-set=utf8
  ```

- 初始化data目录及root用户密码**(以管理员身份运行)**：

  ``` cmd
  mysqld --initialize --user=mysql --console
  ```

  

  ![Snipaste_2022-08-05_13-15-21](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-08-05_13-15-21.png)

- 安装mysql**(以管理员身份运行)**：

  ``` js
  mysqld --install
  # 如果之前安装过,要先卸载
  mysqld --remove
  ```

- 启动和停止mysql**(以管理员身份运行)**：

  ``` cmd
  net start mysql
  ```

- 连接mysql：

  ``` js
  mysql -u root -p
  // 密码就是初始化时显示的密码
  ```

- 设置密码：

  ``` js
  ALTER USER 'root'@'localhost' identified by '密码';
  ```

  

