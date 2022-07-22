## 多线程渲染

主要指将渲染线程分离出来，减少主线程因调用API，改变驱动层状态而产生的卡顿。

除了要保证线程数据通讯外，还要保证主线程与渲染线程同步。一般瓶颈会出现在渲染线程，即缓冲区存放了大量的指令，渲染线程落后主线程多帧而导致的画面延迟。

- 方案是双队列+帧同步等待协同

  主线程在每一帧的最后，将渲染指令进行交换。在主线程下一帧开始唤醒渲染线程执行渲染（此时渲染的是上一帧的画面），如果主线程执行完毕，渲染线程还未完成，主线程会等待其完成后再进行新一帧的指令交换

## FBX SDK

fbx是一种不公开的三维数据交换格式。因为其的不公开性，fbx的所属公司Autodesk提供了基于C++的SDK来实现对FBX格式的各种读写、修改以及转换等操作。

# sim-ig数据分析

## 帧结构

- ethentic header 14 bytes（源mac，目的 mac，长度

- buffer format
  - cae header 16 bytes （4 words）
  - message number 4 bytes + visual header 8bytes
  - command header 8 bytes（包含opcode，数据长度）

## 重要 operation code（opcode

- 21h：大地坐标系统更新 host->ig
- 86h：通用信息feedback ig->host

## 问题

说是要验证sim与ig之间的通信内容，文档是做什么用的，struct不也能反应内容，到底是要验证什么东西。

打印struct是什么意思