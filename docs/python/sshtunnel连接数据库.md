# python SSH连接数据库

假设我们已经在远程服务器上安装了一个postgresql数据库，可以通过以下参数将其连接到该数据库。



SSH参数：

- 服务器的IP：10.0.0.101
- SSH端口：22(SSH的默认端口)
- 用户名：my_username
- 密码：my_password

数据库参数：

- 端口：5432(PostgreSQL默认端口)
- 数据库名称：db
- 数据库用户：postgres_user(默认用户名是postgres)
- 数据库密码：postgres_pswd(默认密码为空字符串)
- 包含我们的数据的表：MY_TABLE

```python
from sshtunnel import SSHTunnelForwarder
from sqlalchemy import create_engine
import pandas as pd

server = SSHTunnelForwarder(
    ('10.0.0.101', 22),
    ssh_username="my_username",
    ssh_password="my_password",
    remote_bind_address=('127.0.0.1', 5432)
    )

server.start()
local_port = str(server.local_bind_port)
engine = create_engine('postgresql://{}:{}@{}:{}/{}'.format("postgres_user","postgres_pswd","127.0.0.1", local_port,"db"))

dataDF = pd.read_sql("SELECT * FROM \"{}\";".format("MY_TABLE"), engine)

server.stop()
```





我要使用非PostgreSQL数据库-MySQL并使用sshtunnel,
此代码将获取一个表并将其转换为excel文件。

```python
import sshtunnel
import sqlalchemy
import pymysql
import pandas as pd
from pandas import ExcelWriter
import datetime as dt
from sshtunnel import SSHTunnelForwarder

server = SSHTunnelForwarder(
    ('ssh.pythonanywhere.com'),
    ssh_username='username',
    ssh_password='password',
    remote_bind_address=('username.mysql.pythonanywhere-services.com', 3306) )

server.start()
local_port = str(server.local_bind_port)
db = 'username$database'
engine = sqlalchemy.create_engine(f'mysql+pymysql://username:password@127.0.0.1:{local_port}/{db}')

print('Engine Created')

df_read = pd.read_sql_table('tablename',engine)
print('Grabbed Table')
writer = ExcelWriter('excelfile.xlsx')
print('writer created')
df_read.to_excel(writer,'8==D') # '8==D' specifies sheet
print('df to excel')
writer.save()
print('saved')
server.stop()
```

