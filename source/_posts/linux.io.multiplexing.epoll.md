---
layout: post
title: 再谈IO多路复用之epoll
date: 2019-05-12 09:00:15
---
# API
#### epoll_create
```c
#include <sys/epoll.h>

int epoll_create(int size);
```
创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大。这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值。需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

```bash
lrwx------. 1 root root 64 5月  12 11:45 0 -> /dev/pts/0
lrwx------. 1 root root 64 5月  12 11:45 1 -> /dev/pts/0
lrwx------. 1 root root 64 5月  12 11:45 2 -> /dev/pts/2
lrwx------. 1 root root 64 5月  12 11:45 3 -> socket:[251991]
lrwx------. 1 root root 64 5月  12 11:45 4 -> anon_inode:[eventpoll]
```

* size：告诉内核这个监听的数目一共有多大。

#### epoll_ctl
```c
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```
epoll的事件注册函数，它不同与select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。
* epfd：epoll_create()的返回值。
* op：动作，用三个宏来表示：
  1. EPOLL_CTL_ADD：注册新的fd到epfd中。
  2. EPOLL_CTL_MOD：修改已经注册的fd的监听事件。
  3. EPOLL_CTL_DEL：从epfd中删除一个fd。
* fd：需要监听的fd。
* event：告诉内核需要监听什么事，struct epoll_event结构如下：
  ```c
  struct epoll_event {
  	__uint32_t events;  /* Epoll events */
  	epoll_data_t data;  /* User data variable */
  };
  ```
  events可以是以下几个宏的集合：
  1. EPOLLIN：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）。
  2. EPOLLOUT：表示对应的文件描述符可以写。
  3. EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）。
  4. EPOLLERR：表示对应的文件描述符发生错误。
  5. EPOLLHUP：表示对应的文件描述符被挂断。
  6. EPOLLET：将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
    + LT模式：默认为此模式，当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。
    + ET模式：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

#### epoll_wait
```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

等待事件的产生，类似于select()调用。
* epfd：epoll_create()的返回值。
* events：从内核得到事件的集合。
* maxevents：告诉内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size。
* timeout：超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。
* 返回值：需要处理的事件数目，如返回0表示已超时，-1表示有错误发生。
  
# 特点
epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

# Example
下面的程序是基于socket的tcp应答程序。
* 服务端
  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <sys/epoll.h>
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
  
      //create epollfd
      int events_len = CONCURRENT_MAX + 2;
      int epollfd = epoll_create(events_len);
      if(epollfd == -1)
      {
          fprintf(stderr, "create epollfd failed!\n");
          exit(-1);
      }
  
      //clientfd
      int clientfds[CONCURRENT_MAX];
      for(int i = 0;i < CONCURRENT_MAX;i++)
      {
          clientfds[i] = -1;
      }
  
      //add event to kernel
      struct epoll_event stdin_event;
      stdin_event.events = EPOLLIN;
      stdin_event.data.fd = STDIN_FILENO;
      epoll_ctl(epollfd, EPOLL_CTL_ADD, STDIN_FILENO, &stdin_event);
  
      struct epoll_event server_event;
      server_event.events = EPOLLIN;
      server_event.data.fd = server_sock_fd;
      epoll_ctl(epollfd, EPOLL_CTL_ADD, server_sock_fd, &server_event);
  
      //get events from kernel
      int timeout = 20 * 1000;
      struct epoll_event events[events_len];
  
      //do epoll
      while(1)
      {
          int ret = epoll_wait(epollfd, events, events_len, timeout);
          if(ret < 0)
          {
              perror("epoll 出错\n");
              continue;
          }
          else if(ret == 0)
          {
              printf("epoll 超时\n");
              continue;
          }
          else
          {
              for(int i = 0;i < ret;i++)
              {
                  int fd = events->data.fd;
                  if(fd == server_sock_fd && (events->events & server_event.events))
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
                              if(clientfds[client_i] == -1)
                              {
                                  index = client_i;
                                  clientfds[client_i] = client_sock_fd;
  
                                  // add event to kernel
                                  struct epoll_event client_event;
                                  client_event.events = EPOLLIN;
                                  client_event.data.fd = client_sock_fd;
                                  epoll_ctl(epollfd, EPOLL_CTL_ADD, client_sock_fd, &client_event);
  
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
                  else if(fd == STDIN_FILENO && (events->events & stdin_event.events))
                  {
                      bzero(input_msg, BUFFER_SIZE);
                      fgets(input_msg, BUFFER_SIZE, stdin);
                      //输入“.quit"则退出服务器
                      if(strcmp(input_msg, QUIT_CMD) == 0)
                      {
                          exit(0);
                      }
                      for(int client_i = 0;client_i < CONCURRENT_MAX;client_i++)
                      {
                          if(clientfds[client_i] > 0)
                          {
                              printf("向客户端(%d)发送消息\n", client_i);
                              send(clientfds[client_i], input_msg, BUFFER_SIZE, 0);
                          }
                      }
                  }
                  else
                  {
                      //处理某个客户端过来的消息
                      bzero(recv_msg, BUFFER_SIZE);
                      long byte_num = recv(events[i].data.fd, recv_msg, BUFFER_SIZE, 0);
                      if(byte_num > 0)
                      {
                          if(byte_num > BUFFER_SIZE)
                          {
                              byte_num = BUFFER_SIZE;
                          }
                          recv_msg[byte_num] = '\0';
                          for(int client_i = 0;client_i < CONCURRENT_MAX;client_i++)
                          {
                              if(clientfds[client_i] == events[i].data.fd)
                              {
                                  printf("客户端(%d):%s\n", client_i, recv_msg);
                                  break;
                              }
                          }
                      }
                      else if(byte_num < 0)
                      {
                          for(int client_i = 0;client_i < CONCURRENT_MAX;client_i++)
                          {
                              if(clientfds[client_i] == events[i].data.fd)
                              {
                                  printf("从客户端(%d)接受消息出错.\n", client_i);
                                  break;
                              }
                          }
                      }
                      else
                      {
                          // delete event in kernel
                          struct epoll_event client_event;
                          client_event.events = EPOLLIN;
                          client_event.data.fd = events[i].data.fd;
                          epoll_ctl(epollfd, EPOLL_CTL_DEL, events[i].data.fd, &client_event);
  
                          for(int client_i = 0;client_i < CONCURRENT_MAX;client_i++)
                          {
                              if(clientfds[client_i] == events[i].data.fd)
                              {
                                  clientfds[client_i] = -1;
                                  printf("客户端(%d)退出了.\n", client_i);
                                  break;
                              }
                          }
                      }
                  }
              }
          }
      }
      close(epollfd);
  
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
> http://www.cnblogs.com/Anker/archive/2013/08/17/3263780.html

