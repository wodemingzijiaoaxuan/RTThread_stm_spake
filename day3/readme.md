# 2024-RSOC-DAY3

## 学习内容

今天主要学习了RTThread的内核中的一些典型概念

*对应的图片可以通过[]()查看*

### IPC机制
#### 线程同步
1. 信号量（Semaphore）
2. 互斥量（mutex）
3. 事件集（event）

#### 线程间通信
1. 消息邮箱（mailbox）
2. 消息队列（msgqueue）

### RTThread内核概念：

- 临界区
- 阻塞与非阻塞
- 挂起
- 死锁

### RT的内核主要概念
### 消息同步（三种）

~~在裸机开发中会设置一些全局标志位flag，flag的变化使代码做出不同的反应~~


#### 信号量
> 对信号量的操作是原子操作，不可被打断
* ~~互斥信号量~~ *用于解决互斥问题。* ***但可能引起优先级反转问题（后面会提到）***

* 二值信号量 *用于解决同步问题*

* 计数信号量 *用于解决资源记数问题*

将信号量进行种类的细分，可以根据其用途，进行按情况使用

##### API的调用
* 创建信号量（动态信号量）

```
rt_sem_t rt_sem_create(const char *name, rt_uint32_t value, rt_uint8_t flag);
```
* 删除信号量

```
rt_err_t rt_sem_delete(rt_sem_t sem);
```
* 初始化信号量（初始化静态信号量）

```
rt_err_t rt_sem_init(rt_sem_t    sem,
                     const char *name,
                     rt_uint32_t value,
                     rt_uint8_t  flag);
```
* 脱离信号量

```
rt_err_t rt_sem_detach(rt_sem_t sem);
```
* 获取信号量

```
rt_err_t rt_sem_take(rt_sem_t sem, rt_int32_t timeout);
```
* 无等待获取信号量（类似于timeout == 0）
```
rt_err_t rt_sem_trytake(rt_sem_t sem);
```
* 释放信号量

```
rt_err_t rt_sem_release(rt_sem_t sem);
```



#### 互斥量
*（在信号量上引出的保护临界资源的任务）*
互斥量又叫相对排斥量，是一种特殊的二值信号量它具有
* 互斥量所有权
* 递归访问
* 防止优先级翻转 ***低优先级线程长时间阻塞高优先级线程***

##### API的调用
* 创建/初始化互斥量

```
rt_mutex_t rt_mutex_create(const char *name, rt_uint8_t flag);
rt_err_t rt_mutex_init(rt_mutex_t mutex, const char *name, rt_uint8_t flag);

```
* 删除互斥量

```
rt_err_t rt_mutex_delete(rt_mutex_t mutex);
```
* 获取互斥量

```
rt_err_t rt_mutex_take(rt_mutex_t mutex, rt_int32_t timeout);
```
* 无等待获取互斥量（类似于timeout == 0）
```
rt_err_t rt_mutex_trytake(rt_mutex_t mutex);
```
* 释放互斥量

```
rt_err_t rt_mutex_release(rt_mutex_t mutex);
```

#### 事件集（相比于信号量的单信号传递，这事件集是一个32bit的数）
##### 触发方式
1. 与触发 
2. 或触发 
* 发送：可以从中断和线程中发送
* 接收：线程接收，条件检查（逻辑与、逻辑或）

##### API的调用
* 创建/初始化事件

```
rt_event_t rt_event_create(const char *name, rt_uint8_t flag);
rt_err_t rt_event_init(rt_event_t event, const char *name, rt_uint8_t flag);


```
* 删除事件

```
rt_err_t rt_event_delete(rt_event_t event);
```
* 脱离事件

```
rt_err_t rt_event_detach(rt_event_t event);
```
* 发送事件
```
rt_err_t rt_event_send(rt_event_t event, rt_uint32_t set);
```
* 接收事件

```
rt_err_t rt_event_recv(rt_event_t   event,
                       rt_uint32_t  set,
                       rt_uint8_t   opt,
                       rt_int32_t   timeout,
                       rt_uint32_t *recved);
```

### 消息传递（两种）

#### 消息邮箱
*消息邮箱（一封邮件）固定容纳四个字节*


##### API的调用
* 创建/初始化邮箱

```
rt_mailbox_t rt_mb_create(const char *name, rt_size_t size, rt_uint8_t flag);
rt_err_t rt_mb_init(rt_mailbox_t mb,
                    const char  *name,
                    void        *msgpool,
                    rt_size_t    size,
                  rt_uint8_t   flag);


```
* 删除邮箱

```
rt_err_t rt_mb_delete(rt_mailbox_t mb);
```
* 发送邮件/等待方式发送邮件/发送紧急邮件

```
rt_err_t rt_mb_send(rt_mailbox_t mb, rt_ubase_t value);
rt_err_t rt_mb_send_wait(rt_mailbox_t mb,
                         rt_ubase_t  value,
                         rt_int32_t   timeout);
rt_err_t rt_mb_urgent(rt_mailbox_t mb, rt_ubase_t value);						 
```

* 接收邮件

```
rt_err_t rt_mb_recv(rt_mailbox_t mb, rt_ubase_t *value, rt_int32_t timeout);
```

#### 消息队列
*消息邮箱可容纳多个字节*
***消息队列实现线程间的异步通信***
* 支持读消息超时机制
* 支持等待方式发送消息
* 允许不同长度的任意类型消息*超过队列最大节点可能截断消息*
* 支持发送紧急消息*类似于紧急邮箱*

######  消息队列参数flag
* 先进先出方式
* 优先级等待方式（常用）


##### API的调用
* 创建/初始化队列

```
rt_mq_t rt_mq_create(const char *name,
                     rt_size_t   msg_size,
                     rt_size_t   max_msgs,
                     rt_uint8_t  flag);

rt_err_t rt_mq_init(rt_mq_t     mq,
                    const char *name,
                    void       *msgpool,
                    rt_size_t   msg_size,
                    rt_size_t   pool_size,
                    rt_uint8_t  flag);

```
* 发送消息/等待方式发送消息/发送紧急消息

```
rt_err_t rt_mq_send(rt_mq_t mq, const void *buffer, rt_size_t size);
rt_err_t rt_mq_send_wait(rt_mq_t     mq,
                         const void *buffer,
                         rt_size_t   size,
                         rt_int32_t  timeout);
rt_err_t rt_mq_urgent(rt_mq_t mq, const void *buffer, rt_size_t size);					 
```

* 接收邮件

```
rt_err_t rt_mq_recv(rt_mq_t    mq,
                    void      *buffer,
                    rt_size_t  size,
                    rt_int32_t timeout);
```

## 学习成果

调用线程，使在打印线程运行状态时，同时存在外设（LED）的调用
_星火一号bsp中有些小问题，LED为低电平🔋，但例程想让亮的时候给的PIN_HIGH，无伤大雅，但是如果一个状态对应一个颜色的话会导致误判断_
