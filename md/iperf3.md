## iperf3发送流量的源码解析

iperf3跑流量过程分为client模式和server模式。

通过main函数获取的参数输入判断启动模式：

#### client模式：运行iperf_run_client()

- iperf_connect (test)客户端到服务器的连接函数，负责设置控制通道（TCP连接）、测试连接以及获取控制通道的MSS（最大分段大小）来合理设置UDP测试的块大小。控制通道主要用于传输一些控制信息（socket创建）

  其中`TCP_NODELAY`选项用于禁用Nagle算法，降低小数据包的发送延迟，以此提高控制信息传输的速率

- 进行初始化，然后通过result = select(test->max_fd + 1, &read_set, &write_set, NULL, timeout);来监控 `test->read_set` 和 `test->write_set` 中的文件描述符，看它们是否可读或可写。一旦某个文件描述符可读（例如控制通道有数据到达），就可以获取控制通道传来的消息

- 获取消息后调用iperf_handle_message_client()进行处理，该用于在客户端处理从服务器接收到的状态消息。客户端使用 `read()` 从控制通道（`test->ctrl_sck`）读取服务器发送的状态信息。读取到的状态被存储在 `test->state` 中。`read()` 返回 `rval`，其值表示读取的字节数：

  - `rval == 0`：服务器关闭了控制通道，设置错误代码为 `IECTRLCLOSE` 并返回 `-1`。
  - `rval < 0`：读取失败，设置错误代码为 `IERECVMESSAGE` 并返回 `-1`。
  - `rval > 0`：成功读取服务器状态，继续执行

  并根据test->state的状态值执行相应的操作：

  - `PARAM_EXCHANGE`（参数交换）

  - `CREATE_STREAMS`（创建数据流）

    iperf_create_streams创建数据流，绑定端口，连接数据流，设置 TCP 拥塞控制算法

    这其中就包括要发送数据的创建：

    在iperf_new_stream函数创建数据流时：会查看是否提供磁盘文件作为数据写入文件，然后创建临时文件，将该文件映射到内存中，作为数据缓冲区，根据配置，填充数据缓冲区。如果没提供磁盘文件，则会使用重复模式或者熵数据填充（即`readentropy` 函数从系统中读取随机数据填充缓冲区）。

  - `TEST_START`（开始测试）

    iperf_init_test中进行协议和流的初始化，create_client_timers创建和初始化客户端的定时器

  - `TEST_RUNNING`（测试进行中）

  - `EXCHANGE_RESULTS`（交换测试结果）

  - `DISPLAY_RESULTS`（显示结果）

  - `IPERF_DONE`（测试完成）

  - ......

- 状态为`TEST_RUNNING`时，会初始化并创建线程，运行定时器，并检查测试是否完成，从而取消和停止线程。其中，创建线程时调用的pthread_create(&(sp->thr), &attr, &iperf_client_worker_run, sp)里面运行iperf_client_worker_run函数，该函数在while循环里面根据当前对象调用iperf_send_mt(sp)或iperf_recv_mt(sp)函数。

  iperf_send_mt：根据参数判断是否要多次发送，multisend代表发送次数，如果设置了速率且不是突发模式，则每次发送检查比特率，然后根据multisend循环发送数据，通过sp->snd(sp)发送数据并进行字节数累加。

- 后续还要判断是否满足结束条件之一，无论是按时间、字节数还是按块数。字节和块测试需要处理客户端作为发送方和客户端作为接收方的情况；并在测试结束后停止线程。

### server模式：运行iperf_run_server()

​	主要包括 iperf_server_listen() 打开套接字并监听、使用 `select` 函数等待是否有新客户端连接和新数据的到来、接收后调用iperf_accept函数对客户端发起的控制连接进行处理并从客户端读取验证信息、调用iperf_handle_message_server()根据状态信息对client发送的消息进行处理等......。流程与client类似，故不赘述。