---
layout: default
title: 录制与回放
has_children: true
parent: 自动驾驶系统
---

# 录制与回放

录制和回放功能是自动驾驶系统功能调试和优化的一个重要手段。通过录制与回放功能，我们可以将道路测试的资源最大化利用：识别路测时功能不及预期的片段，保存数据，工程师可以通过离线回放模拟路测环境来查找问题和进行迭代优化。

## 录制内容的选择

ROS提供了丰富的开发调试工具，其中一个很重要的工具就是rosbag。通过rosbag工具，我们可以录制感兴趣的ros消息形成bag文件，可以修改录制的bag文件，并且可以对bag文件进行回放。回放的过程中，也可以控制启动、暂停、播放速率等。

```
Usage: rosbag <subcommand> [options] [args]

A bag is a file format in ROS for storing ROS message data. The rosbag command can record, replay and manipulate bags.

Available subcommands:
   check  	Determine whether a bag is playable in the current system, or if it can be migrated.
   compress  	Compress one or more bag files.
   decompress  	Decompress one or more bag files.
   filter  	Filter the contents of the bag.
   fix  	Repair the messages in a bag file so that it can be played in the current system.
   help
   info  	Summarize the contents of one or more bag files.
   play  	Play back the contents of one or more bag files in a time-synchronized fashion.
   record  	Record a bag file with the contents of specified topics.
   reindex  	Reindexes one or more bag files.
```

对于使用ROS的自动驾驶系统，rosbag可以成为一个有力的工具。但rosbag回放也存在一些弊端，比如录制的数据量会非常大（尤其lidar和图像数据），回放的时候会出现卡顿等。

在一些场景下，我们也可以选择直接录制传感器的数据，以便于中间件解耦。

## 回放时的时间戳

进行回放的时候，需要注意系统时间的同步。不同传感器数据回放的时候需要进行时间同步。同一数据流的上下游也应保持时间的一致。由于录制的传感器数据通常被打上了时间戳，在回放的时候，应该让所有的节点都认为自己在录制的时间窗口运行，而不能使用当前的系统时间（当前的系统时间对功能模块来说没有意义）。因此，应当提供一个时间API，当节点通过这个时间API获取时间的时候，能够根据当前是在线运行还是回访状态而从不同的时钟获取时间。

为了这个目的，ROS提供了`ros::Time::now()`等接口提供获取时间的能力。通过`/use_sim_time`参数来控制这个接口的行为。

实现上，ROS在回放rosbag的时候，可以通过"--clock"标志来生成并发送一个高频的`/clock`话题的消息。这个消息的内容是当前的模拟时间，是递增的。对应的录制的ros消息只有在到时时才会被发出。而各个节点自动订阅`/clock`话题，并据此更新`ros::Time`接口内维护的当前时间的变量。当应用程序通过`ros::Time::now()`获取当前时间时，会根据收到的最新的`/clock`的时间戳返回当前时间。