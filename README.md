# 1009-
1009模块三作业：

环境简介

| 序号 | 主机名 | IP             | 备注                     |
| ---- | ------ | -------------- | ------------------------ |
| 1    | node1  | 192.168.66.122 | Harbor（可访问互联网）   |
| 2    | vms40  | 192.168.31.40  | client（内网不通互联网） |



- 构建本地镜像（node1）。

  - GO源码:  由于本地有nginx占用`80` 端口 所以httpserver改为`8088`

    ```shell
    [root@node1 httpserver]# pwd
    /root/goprojects/src/github.com/cncamp/golang/httpserver
    [root@node1 httpserver]#vim main.go
    ```

    ```go
    package main
    
    import (
            "flag"
            "fmt"
            "io"
            "log"
            "net/http"
    
            _ "net/http/pprof"
    
            "github.com/golang/glog"
    )
    
    func main() {
            flag.Set("v", "4")
            glog.V(2).Info("Starting http server...")
            http.HandleFunc("/", rootHandler)
            err := http.ListenAndServe(":8088", nil)
            // mux := http.NewServeMux()
            // mux.HandleFunc("/", rootHandler)
            // mux.HandleFunc("/healthz", healthz)
            // mux.HandleFunc("/debug/pprof/", pprof.Index)
            // mux.HandleFunc("/debug/pprof/profile", pprof.Profile)
            // mux.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
            // mux.HandleFunc("/debug/pprof/trace", pprof.Trace)
            // err := http.ListenAndServe(":80", mux)
            if err != nil {
                    log.Fatal(err)
            }
    }
    
    func healthz(w http.ResponseWriter, r *http.Request) {
            io.WriteString(w, "ok\n")
    }
    
    func rootHandler(w http.ResponseWriter, r *http.Request) {
            fmt.Println("entering root handler")
            user := r.URL.Query().Get("user")
            if user != "" {
                    io.WriteString(w, fmt.Sprintf("hello [%s]\n", user))
            } else {
                    io.WriteString(w, "hello [stranger]\n")
            }
            io.WriteString(w, "===================Details of the http request header:============\n")
            for k, v := range r.Header {
                    io.WriteString(w, fmt.Sprintf("%s=%s\n", k, v))
            }
    }
    ```

  - 编译GO源码为二进制可执行文件

    ```shell
    [root@node1 httpserver]# cd $GOPATH/src/github.com/cncamp/golang/httpserver
    [root@node1 httpserver]# go build main.go
    ```

  - Dockerfile: 同样dockerfile也将80端口改为8088

    ```shell
    [root@node1 httpserver]# cd $GOPATH/src/github.com/cncamp/golang/httpserver
    [root@node1 httpserver]# vim Dockerfile
    FROM ubuntu
    ENV MY_SERVICE_PORT=8088
    ENV MY_SERVICE_PORT1=8088
    ENV MY_SERVICE_PORT2=8088
    ENV MY_SERVICE_PORT3=8088
    LABEL multi.label1="value1" multi.label2="value2" other="value3"
    ADD main /httpserver
    EXPOSE 8088
    ENTRYPOINT /httpserver
    ```

- 编写 Dockerfile 将练习 2.2 编写的 httpserver 容器化（请思考有哪些最佳实践可以引入到 Dockerfile 中来）。node1

  ```shell
  [root@node1 httpserver]# docker build -t httpsever:v1 .
  Sending build context to Docker daemon  14.47MB
  Step 1/9 : FROM ubuntu
   ---> 597ce1600cf4
  Step 2/9 : ENV MY_SERVICE_PORT=8088
   ---> Running in ba66f2edfd7c
  Removing intermediate container ba66f2edfd7c
   ---> 4246a7b9cec4
  Step 3/9 : ENV MY_SERVICE_PORT1=8088
   ---> Running in a6a88d444790
  Removing intermediate container a6a88d444790
   ---> 7755c34b7046
  Step 4/9 : ENV MY_SERVICE_PORT2=8088
   ---> Running in d3ec36f74250
  Removing intermediate container d3ec36f74250
   ---> 6f7a377d424a
  Step 5/9 : ENV MY_SERVICE_PORT3=8088
   ---> Running in 509c7af730e3
  Removing intermediate container 509c7af730e3
   ---> 06962ff37461
  Step 6/9 : LABEL multi.label1="value1" multi.label2="value2" other="value3"
   ---> Running in ff9d5ffd4858
  Removing intermediate container ff9d5ffd4858
   ---> 9eaf7ebb0504
  Step 7/9 : ADD main /httpserver
   ---> fc83f0e72ae5
  Step 8/9 : EXPOSE 8088
   ---> Running in a918869413cd
  Removing intermediate container a918869413cd
   ---> 4bf32629158e
  Step 9/9 : ENTRYPOINT /httpserver
   ---> Running in 47e75d7d7c2d
  Removing intermediate container 47e75d7d7c2d
   ---> 3b738f90739d
  Successfully built 3b738f90739d
  Successfully tagged httpsever:v1
  [root@node1 httpserver]#
  [root@node1 httpserver]# docker images
  REPOSITORY                          TAG       IMAGE ID       CREATED          SIZE
  httpsever                           v1        3b738f90739d   35 seconds ago   80MB
  ubuntu                              latest    597ce1600cf4   12 days ago      72.8MB
  192.168.66.122:5000/library/nginx   latest    822b7ec2aaf2   5 weeks ago      133MB
  nginx                               latest    822b7ec2aaf2   5 weeks ago      133MB
  ```

  

- 将镜像推送至 Docker 官方镜像仓库。

  - 先在node1上将镜像tag成harbor的命名格式

  ```shell
  [root@node1 httpserver]#[root@node1 httpserver]# docker tag httpsever:v1 192.168.66.122:5000/library/httpserver:v1
  [root@node1 httpserver]# docker images
  REPOSITORY                               TAG       IMAGE ID       CREATED         SIZE
  httpsever                                v1        3b738f90739d   2 minutes ago   80MB
  192.168.66.122:5000/library/httpserver   v1        3b738f90739d   2 minutes ago   80MB
  ubuntu                                   latest    597ce1600cf4   12 days ago     72.8MB
  nginx                                    latest    822b7ec2aaf2   5 weeks ago     133MB
  192.168.66.122:5000/library/nginx        latest    822b7ec2aaf2   5 weeks ago     133MB
  mysql                                    5.7       8cf625070931   2 months ago    448MB
  goharbor/redis-photon                    v2.3.1    4a0d49a4ece0   2 months ago    191MB
  goharbor/harbor-registryctl              v2.3.1    91e798004920   2 months ago    132MB
  goharbor/registry-photon                 v2.3.1    972ce19b1882   2 months ago    81.2MB
  goharbor/nginx-photon                    v2.3.1    3b3ede1db494   2 months ago    44.3MB
  goharbor/harbor-log                      v2.3.1    40a54594fe22   2 months ago    194MB
  goharbor/harbor-jobservice               v2.3.1    d6e174ae0a00   2 months ago    171MB
  goharbor/harbor-core                     v2.3.1    f05acc3947d6   2 months ago    158MB
  goharbor/harbor-portal                   v2.3.1    4a15c5622fda   2 months ago    57.6MB
  goharbor/harbor-db                       v2.3.1    b16a9c81ef03   2 months ago    263MB
  goharbor/prepare                         v2.3.1    4ce629d59c20   2 months ago    288MB
  nacos/nacos-server                       1.3.2     f3549150bc00   14 months ago   914MB
  redis                                    3.0.7     c44fa74ead88   4 years ago     91.6MB
  java                                     8         d23bdf5b1b1b   4 years ago     643MB
  [root@node1 httpserver]# 
  ```

  - 由于node1本身就是harbor，并且/etc/docker/daemon.json设置为阿里云镜像加速地址，所有将镜像保存至另外服务器上，再推入Harbor。node1

  ```sh
  [root@node1 httpserver]# docker save -o httpserver.tar 192.168.66.122:5000/library/httpserver:v1
  [root@node1 httpserver]# ll
  total 94604
  -rw-r--r--. 1 root root      236 Oct 14 00:04 Dockerfile
  -rwxr-xr-x. 1 root root  7242768 Aug 12 15:30 httpserver
  -rw-------. 1 root root 82392576 Oct 14 01:00 httpserver.tar
  -rwxr-xr-x. 1 root root  7220813 Oct 13 23:53 main
  -rw-r--r--. 1 root root     1195 Oct 13 23:53 main.go
  -rw-r--r--. 1 root root      377 Aug 12 15:30 Makefile
  [root@node1 httpserver]# scp httpserver.tar  root@192.168.31.40:/home
  httpserver.tar                          100%   79MB  39.3MB/s   00:02
  ```

  在另外一台VMS40上先load 再push。vms40

  ```shell
  [root@vms40 home]# docker load -i httpserver.tar
  da55b45d310b: Loading layer [==================================================>]  75.16MB/75.16MB
  a68ddb7aad44: Loading layer [==================================================>]  7.223MB/7.223MB
  Loaded image: 192.168.66.122:5000/library/httpserver:v1
  [root@vms40 home]# docker login 192.168.66.122:5000
  Authenticating with existing credentials...
  WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
  Configure a credential helper to remove this warning. See
  https://docs.docker.com/engine/reference/commandline/login/#credentials-store
  
  Login Succeeded
  [root@vms40 home]# docker push 192.168.66.122:5000/library/httpserver:v1
  The push refers to repository [192.168.66.122:5000/library/httpserver]
  a68ddb7aad44: Pushed
  da55b45d310b: Pushed
  v1: digest: sha256:47566441cd6c79252ee98d10bf40a5426d728b75a1c7ca07acd345ccbfc46304 size: 740
  [root@vms40 home]#
  ```

- 通过 Docker 命令本地启动 httpserver。

  在VMS40上以8888端口启动httpserver,

  ```shell
  [root@vms40 home]# docker images
  REPOSITORY                                                      TAG        IMAGE ID       CREATED          SIZE
  192.168.66.122:5000/library/httpserver                          v1         3b738f90739d   36 minutes ago   80MB
  [root@vms40 home]# docker run -p 8888:8088 3b738f90739d
  ERROR: logging before flag.Parse: I1013 17:09:15.684796       7 main.go:17] Starting http server...
  ```

  ![2021-10-14_011248.png](https://i.loli.net/2021/10/14/2Gc4wQlPbx7AqXD.png)

- 通过 nsenter 进入容器查看 IP 配置。

  在VMS40上下载并源码按照nsenter

  ```shell
  [root@vms40 ~]# docker ps
  CONTAINER ID   IMAGE                                               COMMAND                  CREATED          STATUS          PORTS                                       NAMES
  0a7c5285b1f7   3b738f90739d                                        "/bin/sh -c /httpser…"   12 minutes ago   Up 11 minutes   0.0.0.0:8888->8088/tcp, :::8888->8088/tcp   condescending_raman
  [root@vms40 ~]# docker inspect -f {{.State.Pid}} 0a7c5285b1f7
  10802
  [root@vms40 ~]# nsenter --target 10802 --mount --uts --ipc --net --pid
  root@0a7c5285b1f7:/# ls
  bin  boot  dev  etc  home  httpserver  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
  ```

作业需编写并提交 Dockerfile 及源代码。
提交链接：https://jinshuju.net/f/rxeJhn
截止日期：10月17日晚23:59之前

