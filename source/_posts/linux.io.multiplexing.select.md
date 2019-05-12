---
layout: post
title: 再谈IO多路复用之select
date: 2019-05-06 15:19:22
---
# API
```c
#include <sys/select.h>
#include <sys/time.h>

int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout);
```
* maxfdp1：待测试的描述符个数，最大描述符加1。因为描述符是从0开始的。

* readset：让内核测试读的描述字，如果不敢兴趣则置为空指针。

* writeset：让内核测试写的描述字，如果不敢兴趣则置为空指针。

* exceptset：让内核测试异常的描述字，如果不敢兴趣则置为空指针。

* timeout：告知内核等待所指定描述字中的任何一个就绪可花多少时间。这个参数有以下几种可能性：

  1. 永远等待下去：仅在有一个描述字准备好I/O时才返回。为此，把该参数设置为空指针NULL。
  2. 等待一段固定时间：在有一个描述字准备好I/O时返回，但是不超过由该参数所指向的timeval结构中指定的秒数和微秒数。
  3. 根本不等待：检查描述字后立即返回，这称为轮询。为此，该参数必须指向一个timeval结构，而且其中的定时器值必须为0。

  timeval结构体如下：

  ```c
  struct timeval {
      long tv_sec;   //seconds
      long tv_usec;  //microseconds
  };
  ```

# 缺点
1. 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
2. 同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
3. select支持的文件描述符数量太小了，默认是1024

# Example
下面的程序是基于socket的tcp应答程序。
* 服务端

  ```c
  #include<stdio.h>
  #include<stdlib.h>
  #include<netinet/in.h>
  #include<sys/socket.h>
  #include<arpa/inet.h>
  #include<string.h>
  #include<unistd.h>
  #define BACKLOG 5     //完成三次握手但没有accept的队列的长度
  #define CONCURRENT_MAX 8   //应用层同时可以处理的连接
  #define SERVER_PORT 11332
  #define BUFFER_SIZE 1024
  #define QUIT_CMD ".quit\n"
  int client_fds[CONCURRENT_MAX];
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
      //fd_set
      fd_set server_fd_set;
      int max_fd = -1;
      struct timeval tv;  //超时时间设置
      while(1)
      {
          tv.tv_sec = 20;
          tv.tv_usec = 0;
          FD_ZERO(&server_fd_set);
          FD_SET(STDIN_FILENO, &server_fd_set);
          if(max_fd <STDIN_FILENO)
          {
              max_fd = STDIN_FILENO;
          }
          //printf("STDIN_FILENO=%d\n", STDIN_FILENO);
          //服务器端socket
          FD_SET(server_sock_fd, &server_fd_set);
          // printf("server_sock_fd=%d\n", server_sock_fd);
          if(max_fd < server_sock_fd)
          {
              max_fd = server_sock_fd;
          }
          //客户端连接
          for(int i =0; i < CONCURRENT_MAX; i++)
          {
              //printf("client_fds[%d]=%d\n", i, client_fds[i]);
              if(client_fds[i] != 0)
              {
                  FD_SET(client_fds[i], &server_fd_set);
                  if(max_fd < client_fds[i])
                  {
                      max_fd = client_fds[i];
                  }
              }
          }
          int ret = select(max_fd + 1, &server_fd_set, NULL, NULL, &tv);
          if(ret < 0)
          {
              perror("select 出错\n");
              continue;
          }
          else if(ret == 0)
          {
              printf("select 超时\n");
              continue;
          }
          else
          {
              //ret 为未状态发生变化的文件描述符的个数
              if(FD_ISSET(STDIN_FILENO, &server_fd_set))
              {
                  printf("发送消息：\n");
                  bzero(input_msg, BUFFER_SIZE);
                  fgets(input_msg, BUFFER_SIZE, stdin);
                  //输入“.quit"则退出服务器
                  if(strcmp(input_msg, QUIT_CMD) == 0)
                  {
                      exit(0);
                  }
                  for(int i = 0; i < CONCURRENT_MAX; i++)
                  {
                      if(client_fds[i] != 0)
                      {
                          printf("client_fds[%d]=%d\n", i, client_fds[i]);
                          send(client_fds[i], input_msg, BUFFER_SIZE, 0);
                      }
                  }
              }
              if(FD_ISSET(server_sock_fd, &server_fd_set))
              {
                  //有新的连接请求
                  struct sockaddr_in client_address;
                  socklen_t address_len;
                  int client_sock_fd = accept(server_sock_fd, (struct sockaddr *)&client_address, &address_len);
                  printf("new connection client_sock_fd = %d\n", client_sock_fd);
                  if(client_sock_fd > 0)
                  {
                      int index = -1;
                      for(int i = 0; i < CONCURRENT_MAX; i++)
                      {
                          if(client_fds[i] == 0)
                          {
                              index = i;
                              client_fds[i] = client_sock_fd;
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
              for(int i =0; i < CONCURRENT_MAX; i++)
              {
                  if(client_fds[i] !=0)
                  {
                      if(FD_ISSET(client_fds[i], &server_fd_set))
                      {
                          //处理某个客户端过来的消息
                          bzero(recv_msg, BUFFER_SIZE);
                          long byte_num = recv(client_fds[i], recv_msg, BUFFER_SIZE, 0);
                          if (byte_num > 0)
                          {
                              if(byte_num > BUFFER_SIZE)
                              {
                                  byte_num = BUFFER_SIZE;
                              }
                              recv_msg[byte_num] = '\0';
                              printf("客户端(%d):%s\n", i, recv_msg);
                          }
                          else if(byte_num < 0)
                          {
                              printf("从客户端(%d)接受消息出错.\n", i);
                          }
                          else
                          {
                              FD_CLR(client_fds[i], &server_fd_set);
                              client_fds[i] = 0;
                              printf("客户端(%d)退出了\n", i);
                          }
                      }
                  }
              }
          }
      }
      return 0;
  }
  ```
  
* 客户端
  
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
> http://www.cnblogs.com/Anker/p/3265058.html