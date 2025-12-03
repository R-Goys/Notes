# CS144 - Lab 0

---

## telnet 发送请求

如图，很简单，但是注意输入时间太久会超时

![image-20250528155327866](image-20250528155327866.png)

## 发邮箱

首先我们需要用命令行去发邮箱，这里我用企业微信邮箱给自己的 qq 邮箱发送~

整个命令如下！

![image-20250528144404189](image-20250528144404189.png)

对于其中的参数，其实从英文就可以看出来，首先连接到企业微信邮箱的发送邮箱的服务器，然后输入 HELO smtp.exmail.qq.com 表示开始进行和 smtp 服务器的会话，然后我们可以输入 AUTH LOGIN 来鉴权，首先输入你的邮箱的对应的 base64 的编码，比如我的邮箱是 rinai@g-rinai.cn 就将他转换为 base64 编码就可以了，然后我们还需要一个登录密码，**注意**，这个密码虽然也需要 base64 编码，但是并不是需要你的登录密码，而是一个密钥，基本每个邮箱客户端都会有这样一个位置给我们提供这样一个密钥，比如我是企业微信邮箱，获取密钥的位置在这里：

![image-20250528145040176](image-20250528145040176.png)

点击生成新密码，然后把对应的密码输入到 base64 编码器即可，注意，如果你的码暴露给别人了，请及时删除，比如我现在用完就删了😇

然后鉴权成功，我们可以输入发送方，接收方，输入内容，然后输入 "换行" + "." 就可以结束了。

![image-20250528145653036](image-20250528145653036.png)

结果是成功发送，通过这个方法，你可以给别人发邮件😍😍😍，那么，Next。

---

## netcat 启动服务器

我们可以使用 `netcat -v -l -p 9090` 在本地启动一个服务器，端口为 9090，我们在另一个窗口可以通过 telnet localhost 9090 来连接本地的这个服务器。

## 编写 webget

这里就是真正涉及到代码了，我们需要用套接字实现我们刚刚第一个 telnet 发送请求的功能，这里感觉并不是很难，就是我对套接字编程不太熟悉，很多系统调用都不知道😭，然后我去看了一下 linux 高性能服务器编程里面的套接字编程，就看了几个 API 就写出来了，哈哈。

我们首先通过 `gethostbyname` 来获取域名对应的 ip 信息，然后创建一个套接字，并与对应的 ip 建立连接，发送我们的请求的内容，这里很简单，就是我们刚刚 telnet 之后写的内容，只需要注意一下换行符就可以了，然后从套接字里面循环读取数据到缓冲区并打印就可以了！

```c++
void get_URL(const string& host, const string& path)
{
  cerr << "Function called: get_URL(" << host << ", " << path << ")\n";
  // 通过域名解析获取 ip
  struct hostent *server = gethostbyname(host.c_str());
  if (server == NULL) {
    cerr << "Error: Could not get host information for " << host << "\n";
    return;
  }
  // 这是一个用来表示 ipv4 地址信息的结构体
  // 用于套接字编程
  struct sockaddr_in server_addr;
  // 填充这个结构体
  server_addr.sin_family = AF_INET;
  server_addr.sin_addr.s_addr = *(u_long*)server->h_addr_list[0];
  server_addr.sin_port = htons(80);
  // 本地创建一个套接字
  int sockfd = socket(AF_INET, SOCK_STREAM, 0);
  if ( sockfd < 0 ) {
    cerr << "Error: Could not create socket\n";
    return;
  }
  // 连接到远程服务器
  if (connect(sockfd, (struct sockaddr*) &server_addr, sizeof(server_addr)) < 0 ) {
    cerr << "Error: Could not connect to server\n";
    return;
  }
  string request = "GET " + path + " HTTP/1.1\r\nHost: " + host + "\r\nConnection: close\r\n\r\n";
  if (send(sockfd, request.c_str(), request.size(), 0) < 0) {
    cerr << "Error: Could not send request\n";
    return;
  }
  char buffer[1024];
  int bytes_read = 0;
  while ((bytes_read = recv(sockfd, buffer, sizeof(buffer), 0)) > 0 ) {
    write(1, buffer, bytes_read);
  }
  if (bytes_read < 0) {
    cerr << "Error: Could not read response\n";
    return;
  }
  close(sockfd);
}
```

结果如下：

```bash
root@r-linux:/home/rinai/project/CS144# cmake --build build --target check_webget 
Test project /home/rinai/project/CS144/build
    Start 1: compile with bug-checkers
1/2 Test #1: compile with bug-checkers ........   Passed    0.20 sec
    Start 2: t_webget
2/2 Test #2: t_webget .........................   Passed    1.10 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   1.30 sec
Built target check_webget
```

---

## 内存可靠字节流

这里很简单，就是写一个读写缓冲区模型，其实跟咱消费者生产者模型还是挺像的哈，主要就是一个缓冲区的状态管理，这里我不太熟悉 cpp 的生态，这里直接用了 string 类型来表示字节流了。

我的实现：

首先我们需要查看注释，为所谓的 bytestream 类添加成员字段：

```cpp
class ByteStream
{
public:
  explicit ByteStream( uint64_t capacity );

  // Helper functions (provided) to access the ByteStream's Reader and Writer interfaces
  Reader& reader();
  const Reader& reader() const;
  Writer& writer();
  const Writer& writer() const;

  void set_error() { error_ = true; };       // Signal that the stream suffered an error.
  bool has_error() const { return error_; }; // Has the stream had an error?

protected:
  // Please add any additional state to the ByteStream here, and not to the Writer and Reader interfaces.
  uint64_t capacity_;
  bool error_ {};
  // added state:
  std::string buffer_; 
  uint64_t read_cnt_;
  uint64_t write_cnt_;
  bool closed_ {};
};
```

然后我们需要为 Writer 和 Reader 实现构造函数和对应的读写方法。

```cpp
#include "byte_stream.hh"

using namespace std;

ByteStream::ByteStream( uint64_t capacity )
    : capacity_(capacity),
      buffer_(),  
      read_cnt_(0),
      write_cnt_(0)
{
}
void Writer::push( string data )
{
  if (Writer::is_closed()) 
    return;

  Writer::write_cnt_ += min(Writer::available_capacity(), data.size());
  Writer::buffer_.append( data );
  Writer::buffer_.resize( min(Writer::buffer_.size(), capacity_) );
  
  return;
}

void Writer::close()
{
  Writer::closed_ = true;
}

bool Writer::is_closed() const
{
  return Writer::closed_;
}

uint64_t Writer::available_capacity() const
{
  return capacity_ - Writer::buffer_.size();
}

uint64_t Writer::bytes_pushed() const
{
  return Writer::write_cnt_;
}

string_view Reader::peek() const
{
  std::string_view view(Writer::ByteStream::buffer_.data(), Writer::ByteStream::buffer_.size());
  return view;
}

void Reader::pop( uint64_t len )
{
  if (Reader::is_finished())
    return;
  
  if (len > Reader::bytes_buffered())
    return;
  printf("before buffer size: %lu\n", Reader::buffer_.size());
  Reader::buffer_.erase(0, len);
  printf("after buffer size: %lu\n", Reader::buffer_.size());
  Reader::read_cnt_ += len;
  return;
}


bool Reader::is_finished() const
{
  return Reader::closed_ && Reader::buffer_.empty();
}

uint64_t Reader::bytes_buffered() const
{
  return Reader::buffer_.size();
}

uint64_t Reader::bytes_popped() const
{
  return Reader::read_cnt_;
}
```

写的时候也需要熟悉一下 cpp 的语法了，感觉这种面向对象的结构还挺新鲜的，跟 go 的结构完全不一样。

最后：

```bash
root@r-linux:/home/rinai/project/CS144# cmake --build build --target check0
Test project /home/rinai/project/CS144/build
      Start  1: compile with bug-checkers
 1/11 Test  #1: compile with bug-checkers ........   Passed    0.85 sec
      Start  2: t_webget
 2/11 Test  #2: t_webget .........................   Passed    1.27 sec
      Start  3: byte_stream_basics
 3/11 Test  #3: byte_stream_basics ...............   Passed    0.02 sec
      Start  4: byte_stream_capacity
 4/11 Test  #4: byte_stream_capacity .............   Passed    0.01 sec
      Start  5: byte_stream_one_write
 5/11 Test  #5: byte_stream_one_write ............   Passed    0.01 sec
      Start  6: byte_stream_two_writes
 6/11 Test  #6: byte_stream_two_writes ...........   Passed    0.02 sec
      Start  7: byte_stream_many_writes
 7/11 Test  #7: byte_stream_many_writes ..........   Passed    0.10 sec
      Start  8: byte_stream_stress_test
 8/11 Test  #8: byte_stream_stress_test ..........   Passed    0.03 sec
      Start 37: no_skip
 9/11 Test #37: no_skip ..........................   Passed    0.01 sec
      Start 38: compile with optimization
10/11 Test #38: compile with optimization ........   Passed    3.15 sec
      Start 39: byte_stream_speed_test
        ByteStream throughput (pop length 4096): 11.88 Gbit/s
        ByteStream throughput (pop length 128):   2.13 Gbit/s
        ByteStream throughput (pop length 32):    0.56 Gbit/s
11/11 Test #39: byte_stream_speed_test ...........   Passed    0.36 sec

100% tests passed, 0 tests failed out of 11

Total Test time (real) =   5.84 sec
Built target check0
```

clear



