---
layout: post
title: 再谈IO多路复用之poll
date: 2019-05-09 17:08:22
---
# API
```c
#include <poll.h>

int poll( struct pollfd *fds, unsigned int nfds, int timeout);
```
* fds：待测试的描述以及待测试的事件等，可以是多个。
  pollfd结构体如下：
  
  ```c
  struct pollfd {
  	int fd;			/* 文件描述符 */
  	short events;	/* 等待的事件 */
  	short revents;	/* 实际发生了的事件 */
  };
  ```
  每一个pollfd结构体指定了一个被监视的文件描述符，可以传递多个结构体，指示poll()监视多个文件描述符。每个结构体的events域是监视该文件描述符的事件掩码，由用户来设置这个域。revents域是文件描述符的操作结果事件掩码，内核在调用返回时设置这个域。events域中请求的任何事件都可能在revents域中返回。合法的事件如下：
  - POLLIN：有数据可读
  - POLLRDNORM：有普通数据可读
  - POLLRDBAND：有优先数据可读
  - POLLPRI：有紧迫数据可读
  - POLLOUT：写数据不会导致阻塞
  - POLLWRNORM：写普通数据不会导致阻塞
  - POLLWRBAND：写优先数据不会导致阻塞
  - POLLMSGSIGPOLL：消息可用
  
  此外，revents域中还可能返回下列事件：
  - POLLER：指定的文件描述符发生错误
  - POLLHUP：指定的文件描述符挂起事件
- POLLNVAL：指定的文件描述符非法
  
  POLLIN | POLLPRI等价于select()的读事件，POLLOUT | POLLWRBAND等价于select()的写事件。POLLIN等价于POLLRDNORM | POLLRDBAND，而POLLOUT则等价于POLLWRNORM。例如，要同时监视一个文件描述符是否可读和可写，我们可以设置 events为POLLIN | POLLOUT。在poll返回时，我们可以检查revents中的标志，对应于文件描述符请求的events结构体。如果POLLIN事件被设置，则文件描述符可以被读取而不阻塞。如果POLLOUT被设置，则文件描述符可以写入而不导致阻塞。这些标志并不是互斥的：它们可能被同时设置，表示这个文件描述符的读取和写入操作都会正常返回而不阻塞。
* nfds：fds的个数。
* timeout：参数指定等待的毫秒数，无论I/O是否准备好，poll都会返回。timeout指定为负数值表示无限超时，使poll()一直挂起直到一个指定事件发生；timeout为0指示poll调用立即返回并列出准备好I/O的文件描述符，但并不等待其它的事件。这种情况下，poll()就像它的名字那样，一旦选举出来，立即返回。
* 返回值：成功时，poll()返回结构体中revents域不为0的文件描述符个数；如果在超时前没有任何事件发生，poll()返回0；失败时，poll()返回-1，并设置errno为下列值之一：
  - EBADF：一个或多个结构体中指定的文件描述符无效
  - EFAULTfds：指针指向的地址超出进程的地址空间
  - EINTR：请求的事件之前产生一个信号，调用可以重新发起
  - EINVALnfds：参数超出PLIMIT_NOFILE值
  - ENOMEM：可用内存不足，无法完成请求
  
# 缺点
poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

# Example
下面的程序是基于socket的tcp应答程序。
* 服务端
  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <poll.h>
  #include <netinet/in.h>
  #include <sys/socket.h>
  #include <arpa/inet.h>
  #include <string.h>
  #include <unistd.h>
  #define BACKLOG 5     //完成三次握手但没有accept的队列的长度
  #define CONCURRENT_MAX 8   //应用层同时可以处理的连接
  #define SERVER_PORT 11332
  #define BUFFER_SIZE 1024
  #define QUIT_CMD ".quit\n"
  
  int main(int argc, const char * argv[])
  {
      char input_msg[BUFFER_SIZE];
      char recv_msg[BUFFER_SIZE];
      //本地地址
      struct sockaddr_in server_addr;
      server_addr.sin_family = AF_INET;
      server_addr.sin_port = htons(SERVER_PORT);
      server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
      bzero(&(server_addr.sin_zero), 8);
      //创建socket
      int server_sock_fd = socket(AF_INET, SOCK_STREAM, 0);
      if(server_sock_fd == -1)
      {
          perror("socket error");
          return 1;
      }
      //绑定socket
      int bind_result = bind(server_sock_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
      if(bind_result == -1)
      {
          perror("bind error");
          return 1;
      }
      //listen
      if(listen(server_sock_fd, BACKLOG) == -1)
      {
          perror("listen error");
          return 1;
      }
  
      //pollfd
      int timeout = 20 * 1000;
      int fds_len = 2 + CONCURRENT_MAX;
      struct pollfd fds[fds_len];
      for(int i = 0;i < fds_len;i++)
      {
          fds[i].fd = -1;
          fds[i].events = POLLIN;
          fds[i].revents = 0;
      }
      fds[0].fd = STDIN_FILENO;
      fds[1].fd = server_sock_fd;
  
      //do poll
      while(1)
      {
          int ret = poll(fds, fds_len, timeout);
          if(ret < 0)
          {
              perror("poll 出错\n");
              continue;
          }
          else if(ret == 0)
          {
              printf("poll 超时\n");
              continue;
          }
          else
          {
              for(int i = 0;i < fds_len;i++)
              {
                  if(fds[i].revents & fds[i].events)
                  {
                      fds[i].revents = 0;
  
                      if(i == 0)
                      {
                          printf("发送消息：\n");
                          bzero(input_msg, BUFFER_SIZE);
                          fgets(input_msg, BUFFER_SIZE, stdin);
                          //输入“.quit"则退出服务器
                          if(strcmp(input_msg, QUIT_CMD) == 0)
                          {
                              exit(0);
                          }
                          for(int client_i = 0;client_i < CONCURRENT_MAX;client_i++)
                          {
                              if(fds[client_i + 2].fd > 0)
                              {
                                  printf("向客户端(%d)发送消息\n", client_i);
                                  send(fds[client_i + 2].fd, input_msg, BUFFER_SIZE, 0);
                              }
                          }
                      }
                      else if(i == 1)
                      {
                          //有新的连接请求
                          struct sockaddr_in client_address;
                          socklen_t address_len;
                          int client_sock_fd = accept(server_sock_fd, (struct sockaddr *)&client_address, &address_len);
                          printf("new connection client_sock_fd = %d\n", client_sock_fd);
                          if(client_sock_fd > 0)
                          {
                              int index = -1;
                              for(int client_i = 0;client_i < CONCURRENT_MAX;client_i++)
                              {
                                  if(fds[client_i + 2].fd == -1)
                                  {
                                      index = client_i;
                                      fds[client_i + 2].fd = client_sock_fd;
                                      break;
                                  }
                              }
                              if(index >= 0)
                              {
                                  printf("新客户端(%d)加入成功 %s:%d\n", index, inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));
                              }
                              else
                              {
                                  bzero(input_msg, BUFFER_SIZE);
                                  strcpy(input_msg, "服务器加入的客户端数达到最大值,无法加入!\n");
                                  send(client_sock_fd, input_msg, BUFFER_SIZE, 0);
                                  printf("客户端连接数达到最大值，新客户端加入失败 %s:%d\n", inet_ntoa(client_address.sin_addr), ntohs(client_address.sin_port));
                              }
                          }
                      }
                      else
                      {
                          //处理某个客户端过来的消息
                          bzero(recv_msg, BUFFER_SIZE);
                          long byte_num = recv(fds[i].fd, recv_msg, BUFFER_SIZE, 0);
                          if(byte_num > 0)
                          {
                              if(byte_num > BUFFER_SIZE)
                              {
                                  byte_num = BUFFER_SIZE;
                              }
                              recv_msg[byte_num] = '\0';
                              printf("客户端(%d):%s\n", i - 2, recv_msg);
                          }
                          else if(byte_num < 0)
                          {
                              printf("从客户端(%d)接受消息出错.\n", i - 2);
                          }
                          else
                          {
                              fds[i].fd = -1;
                              fds[i].revents = 0;
                              printf("客户端(%d)退出了\n", i - 2);
                          }
                      }
                  }
              }
          }
      }
      return 0;
  }
  ```
  
* 客户端（仍然采用select）
  ```c
  #include<stdio.h>
  #include<stdlib.h>
  #include<netinet/in.h>
  #include<sys/socket.h>
  #include<arpa/inet.h>
  #include<string.h>
  #include<unistd.h>
  #define BUFFER_SIZE 1024
  
  int main(int argc, const char * argv[])
  {
      struct sockaddr_in server_addr;
      server_addr.sin_family = AF_INET;
      server_addr.sin_port = htons(11332);
      server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
      bzero(&(server_addr.sin_zero), 8);
  
      int server_sock_fd = socket(AF_INET, SOCK_STREAM, 0);
      if(server_sock_fd == -1)
      {
          perror("socket error");
          return 1;
      }
      char recv_msg[BUFFER_SIZE];
      char input_msg[BUFFER_SIZE];
  
      if(connect(server_sock_fd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr_in)) == 0)
      {
          fd_set client_fd_set;
          struct timeval tv;
  
          while(1)
          {
              tv.tv_sec = 20;
              tv.tv_usec = 0;
              FD_ZERO(&client_fd_set);
              FD_SET(STDIN_FILENO, &client_fd_set);
              FD_SET(server_sock_fd, &client_fd_set);
  
              select(server_sock_fd + 1, &client_fd_set, NULL, NULL, &tv);
              if(FD_ISSET(STDIN_FILENO, &client_fd_set))
              {
                  bzero(input_msg, BUFFER_SIZE);
                  fgets(input_msg, BUFFER_SIZE, stdin);
                  if(send(server_sock_fd, input_msg, BUFFER_SIZE, 0) == -1)
                  {
                      perror("发送消息出错!\n");
                  }
              }
              if(FD_ISSET(server_sock_fd, &client_fd_set))
              {
                  bzero(recv_msg, BUFFER_SIZE);
                  long byte_num = recv(server_sock_fd, recv_msg, BUFFER_SIZE, 0);
                  if(byte_num > 0)
                  {
                      if(byte_num > BUFFER_SIZE)
                      {
                          byte_num = BUFFER_SIZE;
                      }
                      recv_msg[byte_num] = '\0';
                      printf("服务器:%s\n", recv_msg);
                  }
                  else if(byte_num < 0)
                  {
                      printf("接受消息出错!\n");
                  }
                  else
                  {
                      printf("服务器端退出!\n");
                      exit(0);
                  }
              }
          }
          //}
      }
      return 0;
  }
  ```

# 参考
> http://www.cnblogs.com/Anker/archive/2013/08/15/3261006.html

