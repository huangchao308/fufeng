# 为什么说 NodeJs 适合 I/O 密集型系统？


## 一句话答案

主要因为两点，一是 NodeJs 采用了非阻塞异步 I/O 模型，二是 NodeJs 是基于事件驱动的编程模型。

## 详细解答

* Node.js 是单线程的，采用了非阻塞异步 I/O 模型，可以在等待 I/O 操作（如读取文件、网络请求等）时让 CPU 去做其他工作，等数据返回后再去执行回调函数。这样可以充分利用 CPU 的时间，提高处理 I/O 密集型任务的效率。 
* Node.js 基于事件驱动的编程模型，通过注册事件监听器来处理请求。当有请求到来时，Node.js 会把请求交给事件循环处理，等待响应后再把结果返回。这样可以实现高并发、低延迟的服务。同时，事件驱动的编程模型也适用于处理数据流、实现内存占用小等场景。 

 综上所述，Node.js 适合处理高并发、响应时间短、I/O 密集型任务，如 Web 应用中的 HTTP 请求、文件读写、数据库操作等。

 ### NodeJs 如何实现事件驱动的异步 I/O 模型？

 NodeJs 通过 libuv 库实现非阻塞异步 I/O 模型。libuv 为 NodeJs 提供了统一的 I/O 接口，抹平了不同操作系统之间的 I/O 操作差异。下图是 libuv 的架构图： ![architecture.png](/fufeng/images/node-libuv-architecture.png)

#### libuv 的事件循环

 libuv 的核心是事件循环，事件循环采用单线程异步I/O的方法：所有（网络）I/O都在非阻塞的套接字上执行，由操作系统提供最佳机制进行轮询：Linux上的epoll、OSX和其他BSD上的kqueue、SunOS上的事件端口和Windows上的IOCP。在循环迭代的一部分中，循环将阻塞等待已添加到轮询器中的套接字上的I/O活动，并触发回调以指示套接字状态（可读、可写、挂起），以便句柄可以执行所需的读取、写入或I/O操作。下图展示了事件循环的各个阶段：
 
 ![loop_iteration.png](/fufeng/images/node-libuv-loop_iteration.png)

* 在每轮事件循环迭代开始之前，NodeJs 会缓存当前时间，用于后续判断定时器是否到达执行时间。如果事件循环仍然存活着，那么开始进入事件循环，否则结束循环，杀掉线程。
* 事件循环的第一个阶段是 `Run due timers` 阶段（定时器阶段），用于执行已经到期的定时器回调函数，即通过 `setTimeout` 或者 `setInterval` 函数注册的回调函数。
* 事件循环的第二个阶段是 `Call pending callbacks` 阶段，这个阶段是执行在上个迭代中没有执行完毕的 I/O 回调函数
* 事件循环的第三个阶段是 `Run idle handles` 阶段，这个阶段执行 nodejs 内部注册的回调函数
* 事件循环的第四个阶段是 `Run prepare handles` 阶段，本阶段执行 nodejs 内部注册的回调函数，用于准备即将启动的`Poll for I/O`阶段。它的主要作用是在执行轮询阶段之前运行一些准备工作，例如打开或关闭某些资源，初始化或重置一些状态等。
* 事件循环的第五个阶段是 `Poll for I/O` 阶段, 在这个阶段会阻塞等待操作系统的 I/O 操作，如果有写/读操作完成，则会在这个时候调用相关的回调函数。这个阶段的阻塞等待时长是按照如下规则计算的：
  * 如果事件循环使用 `UV_RUN_NOWAIT` 标志运行，则等待时长为 0
  * 如果事件循环正在被中止(`uv_stop`函数被调用)，则等待时长为 0
  * 如果没有活动的句柄或请求(没有打开文件描述符且没有 socket 连接)，则等待时长为 0
  * 如果有 `idle handles` 存在，则等待时长为 0
  * 如果有句柄正在等待关闭，则等待时长为 0
  * 如果上述情况都不符合，则取最接近的定时器的超时时间；如果没有注册定时器，则等待时长为无限（直到某个 I/O 操作完成）
* 事件循环的第六个阶段是 `Run check handles` 阶段，这个阶段执行 `setImmediate` 注册的回调函数
* 事件循环的第七个阶段是 `Call close callbacks` 阶段，所有 `close` 事件注册的回调函数在这个阶段执行

#### nodejs 的完整事件循环

nodejs 在 libuv 的事件循环的基础上增加了两个任务队列，分别是 `nextTick` 和 `promise 微任务队列`。完整的 nodejs 事件循环可参考图：
![full_nodejs_event_loop.png](/fufeng/images/full_nodejs_event_loop.png)

* 这两个任务队列并非在 libuv 中被执行，它们都是在 nodejs 层执行的，在 libuv 层处理每一个阶段的任务之后，会和 node 层进行通讯，那么会优先处理两个队列中的任务。**注意： 这两个任务队列的执行会阻塞 `libuv` 的事件循环**

* nextTick 任务的优先级要大于 Microtasks 任务中的 Promise 回调。也就是说 node 会首先清空 nextTick 中的任务，然后才是 Promise 中的任务。


## 参考文档

[libuv 1.45.1-dev documentation » Design overview](http://docs.libuv.org/en/v1.x/design.html)

[「Nodejs万字进阶」一文吃透异步I/O和事件循环](https://juejin.cn/post/7002106372200333319)



