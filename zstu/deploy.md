# ubuntu下部署springboot + vue


## 检查各种依赖

### mysql

| account | password | port |
| ------- | -------- | ---- |
| root    | jerry    | 3306 |

### redis

port : 6379

### node.js

安装node.js 

```
curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
 
sudo apt-get install -y nodejs
```

### jdk

```java -version``` 

```
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

### mvn

```mvn -version```

```
Apache Maven 3.8.1 (05c21c65bdfed0f71a2f2ada8b84da59348c4c5d)
Maven home: /opt/apache-maven-3.8.1
Java version: 1.8.0_221, vendor: Oracle Corporation, runtime: /opt/jdk1.8/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.15.0-54-generic", arch: "amd64", family: "unix"
```


## 配置 nginx

[官方文档](http://nginx.org/en/docs/)

### 命令

```
# 开机配置
systemctl enable nginx # 开机自动启动
systemctl disable nginx # 关闭开机自动启动

# 启动 Nginx
systemctl start nginx # 启动Nginx成功后，可以直接访问主机IP，此时会展示Nginx默认页面

# 停止 Nginx
systemctl stop nginx

# 重启 Nginx
systemctl restart nginx

# 重新加载 Nginx
systemctl reload nginx

 # 查看 Nginx 运行状态
systemctl status nginx

# 查看 Nginx 进程
ps -ef | grep nginx

# 杀死 Nginx 进程
kill -9 pid # 根据上面查看到的 Nginx 进程号，杀死 Nginx 进程，-9 表示强制结束进程
```

服务器里已经安装好nginx了 直接用就好了

### 配置

用nginx监听80端口，进行反向代理,根据路由转发到对映的端口进行处理。



```conf
server {
	listen		 80;
        server_name 	 jerry916.co;

	location / {
		root   /home/ubuntu/app/partybuilding/dist;
		try_files $uri $uri/ /index.html;
        	index  index.html index.htm;
        }

	location /prod-api/{
		proxy_set_header Host $http_host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header REMOTE-HOST $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass http://localhost:8080/;
	}

	location /lianzheng/{
		proxy_set_header Host $http_host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header REMOTE-HOST $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass http://localhost:8080;
	}
       
	location /wx/{
		proxy_set_header Host $http_host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header REMOTE-HOST $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass http://localhost:8080;
	}
	error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
 	}

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/jerry916.co/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/jerry916.co/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = jerry916.co) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen       80;
        server_name  jerry916.co;
    return 404; # managed by Certbot

}

```
### websocket

nginx 反向代理 websocket 

ws和http 都是建立在tcp上的 使用的是同一个端口  在转发时修改下首部就好了

```
location /wss/ {  
   proxy_pass http://localhost:8989/;  
   proxy_http_version 1.1;  
   proxy_set_header Upgrade $http_upgrade;  
   proxy_set_header Connection "Upgrade";  
}

```

### 路径

```
.
├── conf.d
│   └── partybuilding.conf
├── fastcgi_params
├── koi-utf
├── koi-win
├── mime.types
├── modules -> /usr/lib/nginx/modules
├── nginx.conf
├── scgi_params
├── uwsgi_params
└── win-utf

```


## 配置 supervisor

### 安装

``` sudo apt-get install supervisor ```刚开始打成了supervisord 一直是找不到package 

默认的安装路径是/etc/spervisor/ 在这个目录下新键路径var存一些文件

### 命令

```

#更新新的配置到supervisord
supervisorctl update

#重新启动配置中的所有程序
supervisorctl reload

#启动某个进程(program_name=你配置中写的程序名称)
supervisorctl start program_name

#查看正在守候的进程
supervisorctl

#停止某一进程 (program_name=你配置中写的程序名称)
pervisorctl stop program_name

#重启某一进程 (program_name=你配置中写的程序名称)
supervisorctl restart program_name

#停止全部进程
supervisorctl stop all

```


### 配置

生成supervisor.conf文件  ```echo_supervisord_conf > supervisord.conf```

修改里面生成的socket 和 log文件的路径 把他搞到自己建的var里

本来放在了```/etc```下但是各种东西都没有权限真的太麻烦了，就把他改到```/home/ubuntu/etc/```下了 

修改好后的文件内容 
supervisor.conf
```conf
; Sample supervisor config file.
;
; For more information on the config file, please see:
; http://supervisord.org/configuration.html
;
; Notes:
;  - Shell expansion ("~" or "$HOME") is not supported.  Environment
;    variables can be expanded using this syntax: "%(ENV_HOME)s".
;  - Comments must have a leading space: "a=b ;comment" not "a=b;comment".

[unix_http_server]
file=/home/ubuntu/etc/supervisor/var/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/home/ubuntu/etc/supervisor/var/log/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/home/ubuntu/etc/supervisor/var/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)
;umask=022                   ; (process file creation umask;default 022)
user=ubuntu                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY="value"     ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///home/ubuntu/etc/supervisor/var/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; when to restart if exited after running (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; autorestart if exited after running (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=false         ; redirect_stderr=true is not allowed for eventlisteners
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /home/ubuntu/etc/supervisor/conf.d/*.ini

```

配置项目的管理文件

partybuilding.ini
```ini
[program:partybuilding]
command=java -jar ruoyi-admin.jar
directory=/home/ubuntu/app/partybuilding
autostart=true
autorestart=unexpected
user=ubuntu
stdout_logfile=/home/ubuntu/etc/supervisor/var/log/partybuilding-stdout.log
stderr_logfile=/home/ubuntu/etc/supervisor/var/log/partybuilding-stderr.log


```

文件路径
``` 
.
├── conf.d
│   └── partybuilding.ini
├── supervisord.conf
└── var
    └── log
```

## https

使用Certbot向Let's Encrypt免费申请HTTPS证书 ，安装插件 ``` sudo apt-get install python-certbot-nginx ```
在调用命令``` sudo certbot --nginx ```

### TODO
设置定时任务 申请证书 这个证书有效期3个月


## frp

> 将 frps 及 frps.ini 放到具有公网 IP 的机器上。
> 将 frpc 及 frpc.ini 放到处于内网环境的机器上。

### frpc
```ini
[common]
server_addr =  115.159.68.217
server_port = 7000
token = xinxipartybuilding
 
[http]
type = https
local_ip =  10.11.246.100
local_port = 443
custom_domains = jerry916.co
```

### frps
```ini
[common]
bind_port = 7000
vhost_https_port = 6000
token = xinxipartybuilding 
```
刚开始没有用```https```用的是```http```一直不行，应该是```gitlab```上的要求

通过```jerry916.co:6000``` 映射到  ```10.11.246.100:443```

## 脚本

### jar

打包```mvn clean package -Dmaven.test.skip=true```

### ruoyi-ui
```npm install ``` 错误 ```unable to resolve dependency tree```好像是npm版本比较高的原因

解决方法```npm i --legacy-peer-deps```

```ruoyi-ui```打包```npm run build:prod```

### posh-ssh
**这个实在```windows powershell```下使用的不是在```powershell core```的**
```get-command -Module posh-ssh ``` 查询命令的命令
```New-SSHSession -ComputerName "xxx.xxx.xxx.xxx" -Credential (Get-Credential root)```建立会话
```Get-SSHSession```获取会话
```Remove-SSHSession -Index 0 -Verbose```删除会话

### ssh的登录问题
执行到```java```和```mvn```的命令时报错```command not found```简直让我匪夷所思，就去各种百度，百度到的各种东西都差不多的没用然后再```stack overflow```上看到了一个人的回答突然就有方向了。
>It is not about java. It is mostly about SSH. When you run command using SSH you actually connect to remote machine using specific environment. In your case using user ravi. I believe that this user does not have java in his PATH variable defined in his profile script (e.g. .bachrc). 
>Try to run ssh ravi@192.168.3.90 "echo $PATH" and see that java is not there.
>Now the question is what do you want. If you want just to run java use absolute path. If you want to be able to run java using your command line add java to PATH for user account you are using.  
原来是我通过这种方式登录没有加载我要的环境变量

在man page的INVOCATION一节讲述了bash的四种模式，bash会依据这四种模式而选择加载不同的配置文件，然后这些加载配置文件的顺序就导致了这些命令能不能执行。

![avatar](./shell.png)

使用```$ echo $0```命令判断 ```login```显示```-bash``` ```non-login```显示```bash``` 
> -l Make bash act as if it had been invoked as a login shell

#### login shell>
>Anytime you SSH, you are using a login shell.
```login shell```是指用户以非图形化界面或者以ssh登陆到机器上时获得的第一个```shell```,差不多用什么方式登录第一个都是```login shell```。

#### non-login shell
```non-login shell```是指通过```login shell```开启的``shell```。

#### interactive shell
```interactive```是交互的，```interactive shell```会有一个输入提示符，并且它的标准输入、输出和错误输出都会显示在控制台上。所以一般来说只要是需要用户交互的，即一个命令一个命令的输入的shell都是interactive shell。

>A login shell is one whose first character of argument zero is a-, or one started with the --login option.An interactive shell is one started without non-option arguments(unless -s is specified) and without the -c option whose standardinput and error are both connected to terminals (as determined byisatty(3)), or one started with the -i option.  PS1 is set and $-includes i if bash is interactive, allowing a shell script or astartup file to test this state.

#### non-interactive shell
无需用户交互,通常来说如```bash script.sh```此类执行脚本的命令就会启动一个```non-interactive shell```，它不需要与用户进行交互，执行完后它便会退出创建的```shell```。


>When bash is invoked as an interactive login shell, or as a non-interactive shell with the --login option, it first reads and executes commands from the file /etc/profile, if that file exists.  After reading that file, it looks for ~/.bash_profile, ~/.bash_login, and ~/.profile, in that order, and reads and executes commands from the first one that exists and is readable.The --noprofile option may be used when the shell is started toinhibit this behavior.

#### 解决
##### sshd_config
修改```env```,在```sudo vim /etc/ssh/sshd_config``` 修改内容```PermitUserEnvironment yes``` 

重启一下```sshd```服务```sudo service sshd restart```

通过```env | grep PATH```查找出当前的环境变量，写到```~/.ssh/environment```

##### ~/.xxx
加载配置文件顺序
1. ```~/.bash_profile```
2. ``` ~/.bash_login```
3. ```~/.profile```
   
在没有前面的文件挡着的情况下当前文件会被加载，在被加载文件里里面写入```PATH```应该可以。(没有去试过)


### deploy.ps1

```ps1
$username = "ubuntu" 
$password = "jerry" 
$secure = $password | ConvertTo-SecureString -AsPlainText -Force 
$cred = New-Object System.Management.Automation.PSCredential($username, $secure) 
New-SSHSession -ComputerName 115.159.68.217 -Credential $cred -AcceptKey 
Invoke-SSHCommand -SessionId 0 -Command "cd /home/ubuntu/etc/supervisor/;supervisorctl stop partybuilding;" # 停止进程
Invoke-SSHCommand -SessionId 0 -Command "cd /home/ubuntu/apps/partybuilding/;git pull;"   #pull新代码
Invoke-SSHCommand -SessionId 0 -Command "cd /home/ubuntu/apps/partybuilding/;mvn clean package  '-Dmaven.test.skip=true';"   #更新jar包
Invoke-SSHCommand -SessionId 0 -Command "cd /home/ubuntu/apps/partybuilding/ruoyi-ui/;npm run build:prod"  #更新前端页面
Invoke-SSHCommand -SessionId 0 -Command "cd /home/ubuntu/etc/supervisor/;supervisorctl start partybuilding;"  #重新启动进程
Remove-SSHSession -Index 0 -Verbose
```