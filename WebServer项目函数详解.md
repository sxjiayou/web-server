## epoll服务器工作流程

## 1、切换到工作目录

**功能：**方便查看浏览器请求数据的文件路径

```c
        //1、创建一个字符数组
	char pwd_path[256]="";
         //2、getenv()获取当前目录的工作路径
	char * path = getenv("PWD");
	///得到：home/itheima/share/bjc++34/07day/web-http
        //3、strcpy()将path复制到pwd_path
	strcpy(pwd_path,path);
	//4、strcat()实现字符串拼接,即pwd_path在pwd_path加上/web-http
	strcat(pwd_path,"/web-http");
         //5、chdir()改变当前目录
	chdir(pwd_path);
```



## 2、创建、绑定套接字

> tcp4bind(PORT,NULL)；
>
> 参数：
>
> PORT：端口号  #define PORT 8889
>
> NULL：IP地址，设置为空，则任何IP地址都可以连接

```c
int tcp4bind(short port,const char *IP)
{
    //1、创建一个IPV4套接字结构体
    struct sockaddr_in serv_addr;
    //2、创建套接字API
    int lfd = Socket(AF_INET,SOCK_STREAM,0);
    //3、初始化套接字结构体
    //bzero函数置字节字符串serv_addr的前n个字节为零且包括‘\0’。
    bzero(&serv_addr,sizeof(serv_addr));
    //4、给套接字结构体赋值
    if(IP == NULL){
        //如果这样使用 0.0.0.0,任意ip将可以连接
        //绑定的是通配地址INADDR_ANY
        serv_addr.sin_addr.s_addr = INADDR_ANY;
    }else{
        //inet_pton函数将点分十进制数据转换为大端数据
        if(inet_pton(AF_INET,IP,&serv_addr.sin_addr.s_addr) <= 0){
            perror(IP);//转换失败
            exit(1);
        }
    }
    //AF_INET代表IPV4
    serv_addr.sin_family = AF_INET;
    //将主机字节序转换为网络字节序，即将端口数据转换为大端数据
    serv_addr.sin_port   = htons(port);
    //5、设置端口复用
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    //6、绑定套接字
    Bind(lfd,(struct sockaddr *)&serv_addr,sizeof(serv_addr));
    return lfd;
}
```

### API详解

```c
1、IPV4套接字结构体
struct sockaddr_in {
               sa_family_t    sin_family; /* address family: AF_INET */
               in_port_t      sin_port;   /* port in network byte order */
               struct in_addr sin_addr;   /* internet address */
           }
/* Internet address. */          
struct in_addr {
               uint32_t       s_addr;     /* address in network byte order */
           };
参数：
    sin_family: 协议  AF_INET
    sin_portL端口
    sin_addr  ip地址
        
2、创建套接字API
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
参数:
	domain:AF_INET（IPV4）；AF_INET6（IPV6)
	type: SOCK_STREAM 流式套接字；用于tcp通信
	protocol: 0；协议自动填充
返回值：成功返回文件描述符,失败返回-1

3、设置端口复用API
//在server代码的socket()和bind()调用之间插入如下代码：
int opt = 1;
//设置端口复用	
setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    参数：
        lfd：套接字
        SOL_SOCKET：选项所在的协议层
        SO_REUSEADDR：需要被访问的选项名，其值代表保持连接
        &opt：保持连接
        sizeof(opt)：现选项值的长度
      
4、绑定套接字API
int Bind(int fd, const struct sockaddr *sa, socklen_t salen)
    功能：给套接字绑定固定的端口和IP
    参数：
	sockfd: 套接字
	addr: ipv4套接字结构体地址
	addrlen: ipv4套接字结构体的大小
	返回值：成功返回0 失败返回;-1
```

## 2、监听套接字

> Listen(lfd,128);
>
> 参数：
>
> lfd：tcp4bind()函数返回的创建绑定的套接字
>
> 128：已连接队列和未连接队列的总数

```c
int Listen(int fd, int backlog)
{
        int n;
	if ((n = listen(fd, backlog)) < 0)
		perr_exit("listen error");
    return n;
}

```

### API详解

```c
//监听套接字API
int listen(int sockfd, int backlog);
参数:
    sockfd : 套接字
    backlog :  已完成连接队列和未完成连接队列数之和的最大值  128
```

## 3、创建树

>  epoll_create(1);
>

```c
创建红黑树
#include <sys/epoll.h>
int epoll_create(int size);
    参数:
    size :  监听的文件描述符的上限,  2.6版本之后写1即可,
    返回:  返回树的句柄
```

## 4、lfd上树

> ​	epoll_ctl(epfd,EPOLL_CTL_ADD,lfd,&ev);
>

```c
1、上树API
#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
参数:
    epfd : 树的句柄
    op : EPOLL_CTL_ADD 上树   EPOLL_CTL_DEL 下树 EPOLL_CTL_MOD 修改
    fd : 上树,下树的文件描述符
    event :   上树的节点
        
2、树的结点结构体        
      typedef union epoll_data {
               void        *ptr;
               int          fd;
               uint32_t     u32;
               uint64_t     u64;
           } epoll_data_t;
          
           struct epoll_event {
               uint32_t     events;      /* Epoll events */  需要监听的事件
               epoll_data_t data;        /* User data variable */ 需要监听的文件描述符
           };
```

## 5、循环监听

> epoll_wait(epfd,evs,1024,-1);
>

```c
#include <sys/epoll.h>
int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);
功能: 监听树上文件描述符的变化
参数：
            epfd : 数的句柄
            events : 接收变化的节点的数组的首地址
    	maxevents :  数组元素的个数
    	timeout : -1 永久监听  大于等于0 限时等待
返回值: 返回的是变化的文件描述符个数
```

## 6、提取连接

>  Accept(lfd,(struct sockaddr*)&cliaddr,&len);
>
> 参数：
>
> lfd：套接字
>
> (struct sockaddr*)&cliaddr：获取的客户端的的ip和端口信息  iPv4套接字结构体地址
>
> len：iPv4套接字结构体的大小的地址
>
> 返回值：新的已连接套接字的文件描述符

```c
int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr)
{
	int n;
again:
	if ((n = accept(fd, sa, salenptr)) < 0) {
		if ((errno == ECONNABORTED) || (errno == EINTR))//如果是被信号中断和软件层次中断,不能退出
			goto again;
		else
			perr_exit("accept error");
	}
	return n;
}
```

### 函数详解

```c
//从已完成连接队列提取连接
#include <sys/socket.h>
 int accept(int socket, struct sockaddr *restrict address,
           socklen_t *restrict address_len);

功能: 从已完成连接队列提取新的连接，如果连接队列没有新的连接,accept会阻塞
参数:
    socket : 套接字
    address : 获取的客户端的的ip和端口信息  iPv4套接字结构体地址
    address_len: iPv4套接字结构体的大小的地址
                        socklen_t len = sizeof(struct sockaddr );

返回值:  新的已连接套接字的文件描述符
```

## 7、设置cfd为非阻塞

```c
//获取的cfd的标志位		
int flag = fcntl(cfd,F_GETFL);					
flag |= O_NONBLOCK;				
fcntl(cfd,F_SETFL,flag);
```

## 8、新的连接，cfd上树

```c
ev.data.fd = cfd;					
ev.events = EPOLLIN;				
epoll_ctl(epfd,EPOLL_CTL_ADD,cfd,&ev);
```

## 9、cfd变化，读取事件

> read_client_request(epfd,&evs[i]);
>
> 参数：
>
> epfd：树的句柄
>
> evs[i]：树的结点

```c
void read_client_request(int epfd ,struct epoll_event *ev)
{
	//读取请求(先读取一行,在把其他行读取,扔掉)
	char buf[1024]="";
	char tmp[1024]="";
	 int n = Readline(ev->data.fd, buf, sizeof(buf));
         //无变化，客户端关闭连接
	 if(n <= 0)
	 {
	 	printf("close or err\n");
                //cfd下树EPOLL_CTL_DEL 下树
	 	epoll_ctl(epfd,EPOLL_CTL_DEL,ev->data.fd,ev);
                //关闭监听的文件描述符即cfd
	 	close(ev->data.fd);
	 	return ;
	 }
          //有变化，打印数据
	 printf("[%s]\n", buf);
	 int ret =0;
	 while(  (ret = Readline(ev->data.fd, tmp, sizeof(tmp))) >0);  
	 //解析请求行  GET /a.txt  HTTP/1.1\R\N
	 char method[256]="";
	 char content[256]="";
	 char protocol[256]="";
	 sscanf(buf,"%[^ ] %[^ ] %[^ \r\n]",method,content,protocol);
	 printf("[%s]  [%s]  [%s]\n",method,content,protocol );
	 //判断是否为get请求  get   GET
	 //忽略大小写strcasecmp
	 if( strcasecmp(method,"get") == 0)
	 {
	 	//[GET]  [/%E8%8B%A6%E7%93%9C.txt]  [HTTP/1.1]
	 		char *strfile = content+1;
	 		strdecode(strfile,strfile);
	 		 //GET / HTTP/1.1\R\N
	 		if(*strfile == 0)//如果没有请求文件,默认请求当前目录
	 			strfile= "./";
	 		//判断请求的文件在不在
	 		struct stat s;
	 		if(stat(strfile,&s)< 0)//文件不存在
	 		{
	 			printf("file not fount\n");
	 			//先发送 报头(状态行  消息头  空行)
	 			send_header(ev->data.fd, 404,"NOT FOUND",get_mime_type("*.html"),0);
	 			//发送文件 error.html
	 			send_file(ev->data.fd,"error.html",ev,epfd,1);
	 		}
	 		else
	 		{			
	 			//请求的是一个普通的文件
	 			if(S_ISREG(s.st_mode))
	 			{
	 				printf("file\n");
	 				//先发送 报头(状态行  消息头  空行)
	 				send_header(ev->data.fd, 200,"OK",get_mime_type(strfile),s.st_size);
	 				//发送文件
	 				send_file(ev->data.fd,strfile,ev,epfd,1);
	 			}
	 			else if(S_ISDIR(s.st_mode))//请求的是一个目录
	 			{
						printf("dir\n");
						//发送一个列表  网页
						send_header(ev->data.fd, 200,"OK",get_mime_type("*.html"),0);
						//发送header.html
						send_file(ev->data.fd,"dir_header.html",ev,epfd,0);
						struct dirent **mylist=NULL;
						char buf[1024]="";
						int len =0;
						int n = scandir(strfile,&mylist,NULL,alphasort);
						for(int i=0;i<n;i++)
						{
							//printf("%s\n", mylist[i]->d_name);
							if(mylist[i]->d_type == DT_DIR)//如果是目录
							{
								len = sprintf(buf,"<li><a href=%s/ >%s</a></li>",mylist[i]->d_name,mylist[i]->d_name);
							}
							else
							{
								len = sprintf(buf,"<li><a href=%s >%s</a></li>",mylist[i]->d_name,mylist[i]->d_name);
							}
							send(ev->data.fd,buf,len ,0);
							free(mylist[i]);
						}
						free(mylist);
						send_file(ev->data.fd,"dir_tail.html",ev,epfd,1);
	 			}
	 		}
	 }
}
```

## 10、判断请求文件存不存在

```c
struct stat s; 
(stat(strfile,&s)
 功能：通过文件名filename获取文件信息，并保存在buf所指的结构体stat中
 返回值：成功返回0，失败返回-1，并且将详细错误信息赋值给errno全局变
```

## 11、发送响应消息

### 发送流程

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207152128561.png)

发送的数据包括：

- 状态行
- 消息报头
- 空行
- 消息正文

![](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207152121988.png)



```c
void send_header(int cfd, int code,char *info,char *filetype,int length)
{	//1、发送状态行
	char buf[1024]="";
	int len =0;
         //将发行状态行存储在buf中，并返回字符总数
	len = sprintf(buf,"HTTP/1.1 %d %s\r\n",code,info);
	send(cfd,buf,len,0);
	//2、发送消息头
	len = sprintf(buf,"Content-Type:%s\r\n",filetype);
	send(cfd,buf,len,0);
	if(length > 0)
	{
		//发送消息头
		len = sprintf(buf,"Content-Length:%d\r\n",length);
		send(cfd,buf,len,0);
	}
	//3、空行
	send(cfd,"\r\n",2,0);
}
```

## 12、发送响应文件

```c
void send_file(int cfd,char *path,struct epoll_event *ev,int epfd,int flag)
{               
                 //以只读的方式打开浏览器请求的响应文件
		int fd = open(path,O_RDONLY);
                 //fd小于0，说明文件内容为空，或文件出现其他错误
		if(fd <0)
		{
			perror("");
			return ;
		}
		char buf[1024]="";
		int len =0;
		while( 1)
		{
			len = read(fd,buf,sizeof(buf));
                          //小于0，没有内容，读文件出错
			if(len < 0)
			{
				perror("");
				break;
			}
            		//读到文件末尾。退出
			else if(len == 0)
			{
				break;
			}
			else
			{
				int n=0;
				n =  send(cfd,buf,len,0);
				printf("len=%d\n", n);
			}
		}
		close(fd);
		//关闭cfd,下树
		if(flag==1)
		{
			close(cfd);
			epoll_ctl(epfd,EPOLL_CTL_DEL,cfd,ev);
		}
}
```

### send函数API

```c
ssize_t send(int sockfd, const void *buf, size_t len, int flags);//flags=1 紧急数据
功能：发送数据
参数：
    sockfd：套接字
    buf：数据
    len：数据长度
    flags：标志位，用于判断下不下树
```

### 1、发送文不存在

```c
			char *strfile = content+1;
			 //解码请求文件
			strdecode(strfile,strfile);
			//判断如果没有请求文件,默认请求当前目录
			if(*strfile == 0)
			strfile= "./";
			//判断请求的文件在不在
	 		struct stat s;
	 		if(stat(strfile,&s)< 0)//1、文件不存在
	 		{
	 			printf("file not fount\n");
	 			//先发送报头(状态行  消息头  空行)
	 			send_header(ev->data.fd, 404,"NOT FOUND",get_mime_type("*.html"),0);
	 			//发送文件 error.html
	 			send_file(ev->data.fd,"error.html",ev,epfd,1);
	 		}
```

### 2、发送普通文件

```c
	                          //2、请求的是一个普通的文件
                		//S_ISREG判断是不是一个常规文件
	 			if(S_ISREG(s.st_mode))
	 			{
	 				printf("file\n");
	 				//先发送 报头(状态行  消息头  空行)
	 				send_header(ev->data.fd, 200,"OK",get_mime_type(strfile),s.st_size);
	 				//发送文件
	 				send_file(ev->data.fd,strfile,ev,epfd,1);
	 			}
```

### 3、发送目录

```c
	else if(S_ISDIR(s.st_mode))//请求的是一个目录
	 			{
						printf("dir\n");
						//发送一个列表  网页
						send_header(ev->data.fd, 200,"OK",get_mime_type("*.html"),0);
						//发送header.html
						send_file(ev->data.fd,"dir_header.html",ev,epfd,0);
						struct dirent **mylist=NULL;
						char buf[1024]="";
						int len =0;
						int n = scandir(strfile,&mylist,NULL,alphasort);
                                                    //将目录中的文件以超链接的形式展示再header.html上
						for(int i=0;i<n;i++)
						{
							//printf("%s\n", mylist[i]->d_name);
							if(mylist[i]->d_type == DT_DIR)//如果是目录
							{
								len = sprintf(buf,"<li><a href=%s/ >%s</a></li>",mylist[i]->d_name,mylist[i]->d_name);
							}
							else
							{
								len = sprintf(buf,"<li><a href=%s >%s</a></li>",mylist[i]->d_name,mylist[i]->d_name);
							}
							send(ev->data.fd,buf,len ,0);
							free(mylist[i]);
						}
						free(mylist);
						send_file(ev->data.fd,"dir_tail.html",ev,epfd,1);
	 			}
	 		}
```

### scandirAPI 读取目录下所有文件名

```c
 功能：读取目录下所有文件名
   struct dirent {
               ino_t          d_ino;       /* inode number */
               off_t          d_off;       /* not an offset; see NOTES */
               unsigned short d_reclen;    /* length of this record */
               unsigned char  d_type;      /* type of file; not supported
                                              by all filesystem types */
               char           d_name[256]; /* filename */
           };

scandir 读取目录下的文件
struct dirent **mylist : //指向指针数组的首元素的地址
int scandir(const char *dirp, struct dirent ***namelist,
              int (*filter)(const struct dirent *),
              int (*compar)(const struct dirent **, const struct dirent **));
参数: 
        dirp: 目录的路径名
        namelist:  mylist地址
        filter: 过滤的函数入口地址
        compar : 排序函数入口地址   写 alphasort(字母排序)
返回值: 读取文件的个数
```

## 13、处理中文请求

```c
void strdecode(char *to, char *from)
{
    for ( ; *from != '\0'; ++to, ++from) {

        if (from[0] == '%' && isxdigit(from[1]) && isxdigit(from[2])) { //依次判断from中 %20 三个字符

            *to = hexit(from[1])*16 + hexit(from[2]);//字符串E8变成了真正的16进制的E8
            from += 2;                      //移过已经处理的两个字符(%21指针指向1),表达式3的++from还会再向后移一个字符
        } else
            *to = *from;
    }
    *to = '\0';
}

```

```c
sh
[GET]  [/%E8%8B%A6%E7%93%9C.txt]  [HTTP/1.1]

char buf[128]={  0xe8,0x8b,0xa6,0xe7,0x93,0x9c,'.','t','x','t'};

"e8"  => 0xe8

'8'   =>   '8'-'0' = 8
  
e =  'e' -'a'+10 =14
'
'e8' =    ('e' -'a'+10)*16 + ('8'-'0')*1 = 0xe8

```

