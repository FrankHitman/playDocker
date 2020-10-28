
#### docker commit
what the influence of running commit in docker image
当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。
而 Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。
换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。

例如：
```
z0042sjt@md2btk6c MINGW64 /d/work/playDocker (master)
$ docker commit \
> --author "dongyue <frank@gmail.com>" \
> --message "修改默认主页" \
> webserver \
> nginx:v2
sha256:c83055333a8c0b986c27891c90be38dae22ee991b279f0e2f82b6528216a78f5
```
webserver 是容器名称，这个容器中的nginx首页被修改过； nginx:v2是新的镜像与tag名称

#### docker diff
我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动。
```
z0042sjt@md2btk6c MINGW64 /d/work/playDocker (master)
$ docker diff webserver
C /run
A /run/nginx.pid
C /usr
C /usr/share
C /usr/share/nginx
C /usr/share/nginx/html
C /usr/share/nginx/html/index.html
C /root
A /root/.bash_history
C /var
C /var/cache
C /var/cache/nginx
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp
```

#### docker history
查看镜像内的历史纪录
```
$ docker history nginx:v2
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
c83055333a8c        45 seconds ago      nginx -g daemon off;                            215B                修改默认主页
719cd2e3ed04        2 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>           2 weeks ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B
<missing>           2 weeks ago         /bin/sh -c set -x     && addgroup --system -…   54MB
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV PKG_RELEASE=1~stretch    0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV NJS_VERSION=0.3.2        0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.17.0     0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:5ffb798d64089418e…   55.3MB
```
#### 慎用 docker commit
使用 docker commit 命令虽然可以比较直观的帮助理解镜像分层存储的概念，但是实际环境中并不会这样使用。

首先，如果仔细观察之前的 docker diff webserver 的结果，你会发现除了真正想要修改的 /usr/share/nginx/html/index.html 文件外，由于命令的执行，还有很多文件被改动或添加了。这还仅仅是最简单的操作，如果是安装软件包、编译构建，那会有大量的无关内容被添加进来，如果不小心清理，将会导致镜像极为臃肿。

此外，使用 docker commit 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 黑箱镜像，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间后也无法记清具体在操作的。虽然 docker diff 或许可以告诉得到一些线索，但是远远不到可以确保生成一致镜像的地步。这种黑箱镜像的维护工作是非常痛苦的。

而且，回顾之前提及的镜像所使用的分层存储的概念，除当前层外，之前的每一层都是不会发生改变的，换句话说，任何修改的结果仅仅是在当前层进行标记、添加、修改，而不会改动上一层。如果使用 docker commit 制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。这会让镜像更加臃肿。