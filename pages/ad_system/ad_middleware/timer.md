---
layout: default
title: 定时器
parent: 系统中间件
grand_parent: 自动驾驶系统
---

# 定时器

对于节点来说，会有很多定时触发的任务。我们可以通过注册一个定时器来定时调用一个函数，比如ROS中的`ros::Timer`对象和`ros::NodeHandle::addTimer`接口。

定时器应当能够支持周期性执行的任务，或者一次性执行的预约任务。

## 实现
可以参考ROS中的实现。以下给出一个精简版本：

我们需要一个定时器线程来管理进程内注册的所有定时器，并在对应的时间去构建一个工作线程来执行任务，或者在对应的时间后向任务队列中添加一个任务。

因此我们可以抽象出一个基础类：定时器管理员（`TimerManager`）。

定时器管理员的接口可以设计如下：

```

class TimerManager {
 public:
    typedef void (*CallbackHandleType)();
    typedef int TimerHandle;

    virtual ~TimerManager() = default;
    TimerManager *GetInstance();
    TimerHandle &RegisterTimer(CallbackHandleType h, float period_ms, bool oneshot = false);
 private:
    TimerManager() = default;
    TimerManager(const TimerManager &) = default;
    void ThreadWorker();
};
```

其中，`TimerManager::RegisterTimer(CallbackHandleType h,float period_ms, bool oneshot)`接口可以被进一步封装以支持不同的回调类型（比如类的成员函数、functor、`std::function<T>`等）。

`TimerManager`构造的时候，会启动一个`ThreadWorker`工作线程。`RegisterTimer`函数会向一个timer列表中添加新的timer，`ThreadWorker`线程则需要根据timer列表，判断当前距离下一个timer所需要休眠的时间，执行必要的休眠，并在休眠结束后向任务队列中派发一个到时的timer所对应的任务，随后，应当重新计算出下一个休眠时间。需要注意的是，`ThreadWorker`应当使用一个`std::condition_variable::wait_until(XXX)`接口来执行休眠，而不是`std::this_thread::sleep_for(XXX)`接口，因为在新的timer被添加进来之后，应当唤醒`ThreadWorker`来判断是否新的timer就是下一个到时的timer。