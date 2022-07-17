# WebServer项目

# 什么是WebServer？

> Webserver能够解析HTTP协议。当Webserver接收到一个HTTP请求,会返回一个HTTP响应,比如送回一个HTML页面。为了处理一个请求Webserver能够响应一个静态页面或图片，进行页面跳转或者把动态响应的产生托付给一些其他的程序比如CGI脚本，JSP脚本，servlets，ASP脚本，server端JavaScript，或者一些其他的server端技术。不管它们(译者注：脚本)的目的怎样，这些server端的程序通常产生一个HTML的响应来让浏览器能够浏览。 

# 一、功能

> 实现一个web服务器，可以在浏览器页面请求资源页面

# 二、原理

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207151635277.png)

# 三、必备知识

## 1、http超文本传输协议

> http就是为了传输html这样的文件,http位于应用层,侧重于解释.

> http协议对消息区分可以分为请求消息和响应消息.

## 2、http请求消息

> 我们要开发的服务器与浏览器通信采用的就是http协议,在浏览器想访问一个资源的时候,在浏览器输入访问地址(例如http://127.0.0.1:8000),地址输入完成后当敲击回车键的时候,浏览器就将请求消息发送给服务器。我们可以先用测试工具创建一个socket服务器，之后通过浏览器请求地址,就会看到浏览器发送过来的请求消息。

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207151635927.jpg)

1. 请求行：说明请求类型,要访问的资源,以及使用的http版本

2. 请求头：说明服务器使用的附加信息,都是键值对,比如表明浏览器类型

3. 空行：不能省略-而且是\r\n,包括请求行和请求头都是以\r\n结尾

4. 请求数据：表明请求的特定数据内容,可以省略-如登陆时,会将用户名和密码内容作为请求数据

## 3、请求类型    

> http协议有很多种请求类型,对我们来说常见的用的最多的是get和post请求。常见的请求类型如下：

1. Get 请求指定的页面信息，并返回实体主体

2. Post 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。

3. Head 类似于get请求，但是响应消息没有内容，只是获得报头

4. Put 从客户端向浏览器传送的数据取代指定的文档内容

5. Delete 请求服务器删除指定的页面

6. Connect HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器

7. Options 允许客户端查看浏览器的性能

8. Trace 回显服务器收到的请求，主要用于测试和诊断

>  get 和 post 请求都是请求资源,而且都会提交数据,如果提交密码信息用get请求,就会明文显示,而post则不会显示出涉密信息.

## 4、http响应消息

> 响应消息是代表服务器收到请求消息后,给浏览器做的反馈,所以响应消息是服务器发送给浏览器的,响应消息也分为四部分:

1. 状态行 包括http版本号,状态码,状态信息

2. 消息报头 说明客户端要使用的一些附加信息,也是键值对

3. 空行 \r\n 同样不能省略

4. 响应正文 服务器返回给客户端的文本信息

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207151812391.png)

### http常见状态码

> http状态码由三位数字组成,第一个数字代表响应的类别,有五种分类:

1. 1xx  指示信息--表示请求已接收，继续处理

2. 2xx 成功--表示请求已被成功接收、理解、接受

3. 3xx 重定向--要完成请求必须进行更进一步的操作

4. 4xx 客户端错误--请求有语法错误或请求无法实现

5. 5xx 服务器端错误--服务器未能实现合法的请求

## 5、常见的状态码

- 200 OK 客户端请求成功
- 301  Moved Permanently 重定向
- 400 Bad Request    客户端请求有语法错误，不能被服务器所理解
- 401 Unauthorized   请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 
- 403 Forbidden      服务器收到请求，但是拒绝提供服务
- 404 Not Found      请求资源不存在，eg：输入了错误的URL
- 500 Internal Server Error       服务器发生不可预期的错误
- 503 Server Unavailable         服务器当前不能处理客户端的请求，一段时间后可能恢复正常

## 6、http常见文件类型分类

> http与浏览器交互时,为使浏览器能够识别文件信息,所以需要传递文件类型,这也是响应消息必填项,常见的类型如下:

-  普通文件:  text/plain; charset=utf-8
-  *.html:   text/html; charset=utf-8
-  *.jpg:   image/jpeg
-  *.gif:   image/gif
-  *.png:   image/png
-  *.wav:   audio/wav
- *.avi:   video/x-msvideo
-  *.mov:   video/quicktime
-  *.mp3:   audio/mpeg

**特别说明** 

> charset=iso-8859-1  西欧的编码，说明网站采用的编码是英文；
>
> charset=gb2312     说明网站采用的编码是简体中文；
>
> charset=utf-8       代表世界通用的语言编码；可以用到中文、韩文、日文等世界上所有语言编码上
>
> charset=euc-kr     说明网站采用的编码是韩文；
>
> charset=big5       说明网站采用的编码是繁体中文；

##  四、Web服务器开发

> 我们要开发web服务器已经明确要使用http协议传送html文件,http只是应用层协议,我们需要选择一个传输层的协议来完成我们的传输数据工作,所以开发协议选择是TCP+HTTP,也就是说服务器搭建浏览依照TCP,对数据进行解析和响应工作遵循HTTP的原则.

> 这样我们的思路很清晰,编写一个TCP并发服务器,只不过收发消息的格式采用的是HTTP协议,如下图:
>

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207151925531.jpg)

> 为了支持并发服务器,我们可以有多个选择,比如多进程服务器,多线程服务器,select,poll,epoll等多路IO工具都可以,也可以使用libevent进行开发.

### 基于epoll的web服务器

> epoll在大量并发少量活跃的情况下效率很高,epoll开发的主体流程:
>

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207151927687.jpg)

> 将上述框架封装成一个函数，函数参数处理客户端请求流程：
>

![img](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207151928125.png)

### epool_web代码实现

```c
#include "stdio.h"
#include "wrap.h"
#include "sys/epoll.h"
#include <fcntl.h>
#include <sys/stat.h>
#include "pub.h"
#include "dirent.h"
#include "signal.h"
#define PORT 8889
//发怂响应消息函数
void send_header(int cfd, int code,char *info,char *filetype,int length)
{	//发送状态行
	char buf[1024]="";
	int len =0;
	len = sprintf(buf,"HTTP/1.1 %d %s\r\n",code,info);
	send(cfd,buf,len,0);
	//发送消息头
	len = sprintf(buf,"Content-Type:%s\r\n",filetype);
	send(cfd,buf,len,0);
	if(length > 0)
	{
	        //发送消息头
		len = sprintf(buf,"Content-Length:%d\r\n",length);
		send(cfd,buf,len,0);
	}
	//发送空行
	send(cfd,"\r\n",2,0);
}
void send_file(int cfd,char *path,struct epoll_event *ev,int epfd,int flag)
{                 //打开这个文件，O_RDONLY只读
		int fd = open(path,，O_RDONLY);
		if(fd <0)
		{
			perror("");
			return ;
		}
		char buf[1024]="";
		int len =0;
    	     //发送数据
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
                 //关闭文件
		close(fd);
		//关闭cfd,下树
                 //flag==1就关闭
		if(flag==1)
		{
			close(cfd);
			epoll_ctl(epfd,EPOLL_CTL_DEL,cfd,ev);
		}
}
void read_client_request(int epfd ,struct epoll_event *ev)
{
	//读取请求(先读取一行,在把其他行读取,扔掉)
	char buf[1024]="";
	char tmp[1024]="";
         //先读取一行储存在buf中
	 int n = Readline(ev->data.fd, buf, sizeof(buf));
	 if(n <= 0)
	 {
	 	printf("close or err\n");
	 	epoll_ctl(epfd,EPOLL_CTL_DEL,ev->data.fd,ev);
	 	close(ev->data.fd);
	 	return ;
	 }
	 printf("[%s]\n", buf);
	 int ret =0;
          //再读取存储再tmp
	 while(  (ret = Readline(ev->data.fd, tmp, sizeof(tmp))) >0);  
	 //解析请求行  GET /a.txt  HTTP/1.1\R\N 
	 char method[256]="";
	 char content[256]="";
	 char protocol[256]="";
       //参数str的字符串根据参数format字符串来转换并格式化数据。格式转换形式请
        //参考scanf()。转换后的结果存于对应的参数内。
	 sscanf(buf,"%[^ ] %[^ ] %[^ \r\n]",method,content,protocol);
	 printf("[%s]  [%s]  [%s]\n",method,content,protocol );
	 //判断是否为get请求  get   GET
	 //忽略大小写strcasecmp
	 if( strcasecmp(method,"get") == 0)
	 {
	 	//[GET]  [/%E8%8B%A6%E7%93%9C.txt]  [HTTP/1.1]
               //content后面是文件名
	 		char *strfile = content+1;
                         //解码请求文件
	 		strdecode(strfile,strfile);
	 		 //GET / HTTP/1.1\R\N
	 		if(*strfile == 0)//如果没有请求文件,默认请求当前目录
	 			strfile= "./";
	 		//判断请求的文件在不在
	 		struct stat s;
	 		if(stat(strfile,&s)< 0)//1、文件不存在
	 		{
	 			printf("file not fount\n");
	 			//先发送 报头(状态行  消息头  空行)
	 			send_header(ev->data.fd, 404,"NOT FOUND",get_mime_type("*.html"),0);
	 			//发送文件 error.html
	 			send_file(ev->data.fd,"error.html",ev,epfd,1);
	 		}
	 		else
	 		{ 			
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
	 			else if(S_ISDIR(s.st_mode))//3、请求的是一个目录
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
int main(int argc, char const *argv[])
{    
	signal(SIGPIPE,SIG_IGN);
	//切换工作目录
	//获取当前目录的工作路径
	char pwd_path[256]="";
	char * path = getenv("PWD");
	///home/itheima/share/bjc++34/07day/web-http
	strcpy(pwd_path,path);
	strcat(pwd_path,"/web-http");
	chdir(pwd_path);
	//创建套接字 绑定
	int lfd = tcp4bind(PORT,NULL);
	//监听
	Listen(lfd,128);
	//创建树
	int epfd = epoll_create(1);
	//将lfd上树
        //创建树的结点
	struct epoll_event ev,evs[1024];
	ev.data.fd = lfd;   //套接字
	ev.events = EPOLLIN;  //需要监听的事件->读事件
        //（树的句柄，上树，上下树的文件描述符，上树的结点)
	epoll_ctl(epfd,EPOLL_CTL_ADD,lfd,&ev);
	//循环监听
	while(1)
	{        // -1 永久监听  大于等于0 限时等待，返回的是变化的文件描述符个数
		int nready = epoll_wait(epfd,evs,1024,-1);
                 //无变化
		if(nready < 0)
		{
			perror("");
			break;
		}
		else
		{        //遍历lfd
			for(int i=0;i<nready;i++)
			{
				printf("001\n");
				//判断是否是lfd，EPOLLIN代表读事件
				if(evs[i].data.fd == lfd && evs[i].events & EPOLLIN)
				{
					struct sockaddr_in cliaddr;
					char ip[16]="";
					socklen_t len = sizeof(cliaddr);
                    		     //提取连接
                    		//返回新的已连接套接字的文件描述符
					int cfd = Accept(lfd,(struct sockaddr*)&cliaddr,&len);
                    //输出已连接的ip地址和端口号
					printf("new client ip=%s port=%d\n",inet_ntop(AF_INET,&cliaddr.sin_addr.s_addr,ip,16),
						ntohs(cliaddr.sin_port));
					//设置cfd为非阻塞
					int flag = fcntl(cfd,F_GETFL);
					flag |= O_NONBLOCK;
					fcntl(cfd,F_SETFL,flag);
					//上树
					ev.data.fd = cfd;
					ev.events = EPOLLIN;
					epoll_ctl(epfd,EPOLL_CTL_ADD,cfd,&ev);
				}
                //cfd变化，lfd负责连接，cfd变化说明有数据发送过来
				else if(evs[i].events & EPOLLIN)//cfd变化
				{
					read_client_request(epfd,&evs[i]);
				}
			}
		}
	}
	//=收尾
	return 0;
}

```

## 五、效果

**开启服务器：**

![image-20220717101457685](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207171015778.png)

**访问浏览器：**

![image-20220717102505280](https://sx-image799.oss-cn-hangzhou.aliyuncs.com/image/202207171025444.png)

**以上属于黑马程序员的WebServer项目，如有侵权，请告知**
