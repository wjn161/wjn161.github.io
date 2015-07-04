## 目录

<!-- toc -->

-----
## 下载并在Windows上安装Docker

[Docker](http://www.docker.com/)是一个虚拟化的容器，简单的可以理解为我们常用的虚拟机，但是比起VM来说，要轻量级一些，方便使用和部署，Docker目前在技术运维界已经非常火了，关于Docker的讨论也非常多，这里不展开了，我们只是简单的使用它来快速搭建一个Redis的Linux开发环境，**并不能用于生产环境**

* 第一步，到[这里](https://docs.docker.com/installation/windows/)查找Windows上的Docker安装教程，后来被引导到了GitHub上的一个项目，叫[boot2docker](https://github.com/boot2docker)，这里你可以找到各种操作系统下的Docker安装教程，我们找到Windows的下载并安装。安装的过程基本都是直接点下一步，下一步就能安装成功了，如果遇到问题，可以先到[这里](https://docs.docker.com/installation/windows/)看看，不行就自己Google吧，(这些问题用百度找不太方便)

* 第二步，因为大部分资源都在国外的服务器上，这时候你可能需要一个代理，如果提示由于网络原因无法下载镜像文件，请在cmd中使用代理，这里推荐一个**只对程序员有效**的免费代理，目前主流的几个程序站都有翻墙和加速效果，地址是 http://hx.gy:1080 ，如果需要浏览器代理，请看这篇[文章](http://wujn.me/articles/honxin-proxy.html),代理设置方式为在cmd中输入 `set http_proxy=http://hx.gy:1080`

* 第三步，安装完成后,需要执行两个命令`boot2docker init`和`boot2docker start`，效果如图

![init](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/001.png)

![start](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/002.png)

* 第四步，按照第二步第二个命令中系统的提示，在cmd中复制并输入环境变量，具体看提示即可。完成环境变量设置后输入`docker version` 查看输出是否正常，如下图：

![start](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/003.png)

* 第五步，如果第三步输入`docker version`报错且提示

```bash
FATA[0000] Get http:///var/run/docker.sock/v1.18/version: dial unix /var/run/doc
ker.sock: An address incompatible with the requested protocol was used.. Are you
 trying to connect to a TLS-enabled daemon without TLS?
```
那么可以尝试重启一下boot2docker的服务，在cmd中运行命令`boot2docker restart`如果还是不行，那么可以尝试重新下载boot2docker中docker的镜像
```bash
boot2docker stop
boot2docker download
boot2docker start
```

## 使用Docker创建Redis服务

### 1.创建Redis服务端的Docker Container

* 首先，我们需要创建一个DockerFile命名为DockFile,(没有后缀名,文件夹需要读写权限)里面代码如下

```bash
FROM        ubuntu:14.04
RUN         apt-get update && apt-get install -y redis-server
EXPOSE      6379
ENTRYPOINT  ["/usr/bin/redis-server"]
```

* 其次，运行命令 `docker build -t <your name>/redis - < DockerFile`，把<your name>替换为你自己的名字，于是运行的命令是`docker build -t wjn161/redis - < DockerFile` build的过程很漫长，需要下载很多文件，可以暂时干点别的事情。

* 最后，运行Redis服务，-d参数表示把redis作为一个后台服务运行，命令为 `docker run --name redis -d -p 6379:6379 wjn161/redis` 

> 参数说明：--name是给container取一个别名，以后再stop和attach的时候就可以使用这个别名，-d 是作为一个后台服务运行，-p是端口映射，把container的6379映射到host的6379端口，这样就可以在host机上使用192.168.59.103:6379来连接container中的redis服务了。


### 2. 创建另外一个Application容器(Redis客户端，用于运行redis-cli)

* 首先，在cmd中输入命令`docker run -link redis:db -i -t ubuntu:14.04 /bin/bash` 然后cmd会自动进入到新container中的bash界面。
* 其次，在bash中安装redis-cli，输入命令

```bash
apt-get update
apt-get -y install redis-server
service redis-server stop
```
* 由于在run的命令中加入了-link参数，所以在这个container中得到了别名db开头的一些环境变量，在bash中输入`env | grep DB_` 会得到以下的一些数据

```bash
DB_PORT_6379_TCP_PORT=6379
DB_PORT=tcp://172.17.0.10:6379
DB_PORT_6379_TCP=tcp://172.17.0.10:6379
DB_PORT_6379_TCP_ADDR=172.17.0.10
DB_PORT_6379_TCP_PROTO=tcp
```
* 最后我们就可以在测试容器中通过这些变量连接到Redis服务了

### 3. 测试和使用Redis服务

简单写一个程序测试Redis
* 第一步，新建一个C#的ConsoleApplication
* 第二步，安装Redis的驱动，这里使用[ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis),直接在NuGet中获取，命令为`Install-Package ServiceStack.Redis`，安装成功后编写测试代码

```C#
using System;
using ServiceStack.Redis;

namespace RedisClientTest
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var client = new RedisClient("192.168.59.103", 6379))
            {
                client.Set("Hello", "Docker");
                var result = client.Get<String>("Hello");
                Console.WriteLine(result);
            }
            Console.ReadKey();
        }
    }
}
```
运行后输出的结果：

![HelloDocker](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/004.png)

## 使用别人写好的DockerFile来创建mysql container

在GitHub上有很多别人写好的DockerFile来帮助你创建各种优化过配置的Docker Container 这里以mysql为例，我们找到这个项目[tutum-docker-mysql](https://github.com/tutumcloud/tutum-docker-mysql)，按照项目的说明，我们在cmd中输入`docker run -d -p 3306:3306 tutum/mysql` 经过漫长的下载和安装过程之后(*尼玛居然下载了1个小时*)，然后使用`docker ps`命令查看一下是否正常运行，

![ps](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/006.png)

截图说明运行正常，之后使用log命令查看日志中的随机密码，即可链接到服务器了

![ps](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/007.png)

然后继续阅读项目中的说明就可以完成各种mysql的工作了，同样的，最后我们还是会写一段代码测试一下mysql是否正常工作。

* 第一步 新建一个C#的ConsoleApplication
* 第二步 安装mysql的C#驱动，这里使用Mysql.Data,直接在Nuget中安装，输入`Install-Package MySql.Data` 
* 第三步 编写测试代码

```C#

using System;
using MySql.Data.MySqlClient;

namespace MySqlClientTest
{
    class Program
    {
        static void Main(string[] args)
        {
            const string connectionString = "server=192.168.59.103;port=3306;database=mysql;uid=admin;pwd=338CCGvO5mE2";
            const string sqlstr = "SELECT * FROM user";
            using (var connection = new MySqlConnection(connectionString))
            {
                connection.Open();
                var cmd = new MySqlCommand(sqlstr, connection);
                var reader = cmd.ExecuteReader();
                Console.Write("Host");
                Console.Write("\t\t");
                Console.Write("User");
                Console.WriteLine();
                while (reader.Read())
                {
                    Console.Write((string)reader["Host"]);
                    Console.Write("\t\t");
                    Console.Write((string)reader["User"]);
                    Console.WriteLine();
                }
                if (!reader.IsClosed)
                {
                    reader.Close();
                }
            }
        }
    }
}

```
最后输出为：

![mysql](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/005.png)

客户端的结果：

![mysqlclient](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/008.png)

## 最后的工作

* container中的内容在在container关闭之后都会丢失，所以当所有工作都做完之后，需要通过commit命令来提交修改，这样才能保存所有的内容。命令为：`docker commit [CONTAINER-ID] [STATUS]`，我们可以先用`docker ps`来得到正在运行的container的信息，然后输入命令`docker commit c2982423f0c6 tutum/mysql:latest`来提交最新的内容，保存到镜像中，最后通过`ps images`查看镜像

![dockerimages](http://7tsy9a.com1.z0.glb.clouddn.com/mewujndocker/009.png)

## 总结

* 以上提到的只是Docker的一些初级的应用，其中还有很多命令以及最重要的网络设置等内容，还需要进一步深入研究，最后，请勿在没有弄清楚原理的情况下在生产环境中尝试Docker。

* 另外，在进入container的bash后，需要返回到host机的cmd中时，如果输入exit，则container会退出，这样所有的东西都不能保存了，非常恶心，所以退出的时候需要通过快捷键`ctrl+p+ctrl+q`来完成（相当于detach了），这样再次进入的时候就可以通过命令`docker attach [CONTIAINER ID]`进入之前退出的container了。

## 参考

* [https://github.com/tutumcloud/tutum-docker-mysql](https://github.com/tutumcloud/tutum-docker-mysql)
* [http://stackoverflow.com/questions/19688314/how-do-you-attach-and-detach-from-dockers-process](http://stackoverflow.com/questions/19688314/how-do-you-attach-and-detach-from-dockers-process)
* [https://docs.docker.com/examples/running_redis_service/](https://docs.docker.com/examples/running_redis_service/)
* [http://blog.jackriver.im/building-a-redis-docker-container/](http://blog.jackriver.im/building-a-redis-docker-container/)
* [https://github.com/boot2docker/boot2docker](https://github.com/boot2docker/boot2docker)