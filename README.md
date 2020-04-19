# uv-cpp
<a href="https://github.com/wlgq2/libuv_cpp11/releases"><img src="https://img.shields.io/github/release/wlgq2/libuv_cpp11.svg" alt="Github release"></a>
[![Platform](https://img.shields.io/badge/platform-%20%20%20%20Linux,%20Windows-green.svg?style=flat)](https://github.com/wlgq2/libuv_cpp11)
[![License](https://img.shields.io/badge/license-%20%20MIT-yellow.svg?style=flat)](LICENSE)
[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)

<br>Language Translations:</br>
* [Englishi](README.md)
* [简体中文](README_cn.md)
** **
uv-cpp is a simple interface, high-performance network library based on C++11.

## Dependencies
 * [libuv][1]
## Features
* C++11 functional/bind style callback instead of C-style function pointer.
* `TCP` and `UDP` wrapper.
* `DNS`and`Http`：DNS query and http support，Http routing based on radix tree.
* `Timer`and`TimerWheel`：Heartbeat timeout judgment with time complexity of O(1).
* `Async`：libuv async wrapper，but optimized the [problem][2] of calling multiple times  but callback  will only be called once. 
* `Packet`and`PacketBuffer`：Send and receive packet of Tcp data stream. Support custom data packet structure (such as [uvnsq][3])
* Log interface.
** **
## Build Instructions
* VS2017 (windows)
* Codeblocks (linux)
* CMake (linux)
** **
## Benchmark
### ping-pong VS boost.asio 1.67
<br>environment：Intel Core i5 8265U + debian8 + gcc8.3.0 + libuv1.30.0 + '-O2'</br>

   libuv_cpp | no use PacketBuffer|CycleBuffer|ListBuffer|
:---------:|:--------:|:--------:|:--------:|
Times/Sec | 192857 |141487|12594|

### Apache bench VS nginx 1.14.2
<br>environment：Intel Core i5 8265U + debian8 + gcc8.3.0 + libuv1.30.0 + '-O2'</br>

## Quick start
A simple echo server
```C++
#include <iostream>
#include <uv/include/uv11.h>

int main(int argc, char** args)
{
    uv::EventLoop* loop = uv::EventLoop::DefaultLoop();

    uv::SocketAddr addr("0.0.0.0", 10005, uv::SocketAddr::Ipv4);
    uv::TcpServer server(loop);
    server.setMessageCallback([](uv::TcpConnectionPtr ptr,const char* data, ssize_t size)
    {
        ptr->write(data, size, nullptr);
    });
    //heartbeat timeout.
    //server.setTimeout(60);
    server.bindAndListen(addr);
    loop->run();
}

```

A simple http service router which based on radix tree.
```C++
int main(int argc, char** args)
{
    uv::EventLoop loop;
    uv::http::HttpServer::SetBufferMode(uv::GlobalConfig::BufferMode::CycleBuffer);

    uv::http::HttpServer server(&loop);
    //example:  127.0.0.1:10010/test
    server.Get("/test",std::bind(&func1,std::placeholders::_1,std::placeholders::_2));
    
    //example:  127.0.0.1:10010/some123abc
    server.Get("/some*",std::bind(&func2, std::placeholders::_1, std::placeholders::_2));
    
    //example:  127.0.0.1:10010/value:1234
    server.Get("/value:",std::bind(&func3, std::placeholders::_1, std::placeholders::_2));
    
    //example:  127.0.0.1:10010/sum?param1=100&param2=23
    server.Get("/sum",std::bind(&func4, std::placeholders::_1, std::placeholders::_2));
    
    uv::SocketAddr addr("127.0.0.1", 10010);
    server.bindAndListen(addr);
    loop.run();
}

```
More examples [here][4].

[1]: https://github.com/libuv/libuv
[2]: http://docs.libuv.org/en/v1.x/async.html
[3]: https://github.com/wlgq2/uvnsq
[4]: https://github.com/wlgq2/uv-cpp/tree/master/examples

