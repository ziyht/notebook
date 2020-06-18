# docker-compose

Compose 项目是Docker官方的开源项目，负责实现Docker容器集群的快速编排，开源代码在https://github.com/docker/compose 上
	
我们知道使用Dockerfile模板文件可以让用户很方便的定义一个单独的应用容器，其实在工作中，经常会碰到需要多个容器相互配合来完成的某项任务情况，例如工作中的web服务容器本身，往往会在后端加上数据库容器，甚至会有负责均衡器，比如LNMP服务

Compose 就是来做这个事情的，它允许用户通过一个单独的`docker-compose.yml`模板文件(YAML格式)来定义一组相关联的应用容器为一个项目(project)	

Compose 中有两个重要的概念：

* 服务(`service`):一个应用的容器，实际上可以包括若干运行相同镜像的容器实例
* 项目(`project`):由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml中定义

Compose 项目是由Python编写的，实际上就是调用了Docker服务提供的API来对容器进行管理，因此，只要所在的操作系统的平台支持Docker API，就可以在其上利用Compose来进行编排管理.