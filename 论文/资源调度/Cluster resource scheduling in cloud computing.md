# 调度系统

Cluster resource scheduling in cloud computing: literature review and research challenges

- [调度系统](#调度系统)
  - [1.centralized:  拥有全局视图](#1centralized--拥有全局视图)
    - [1.1基于队列的(Queue-based)](#11基于队列的queue-based)
    - [1.2流式(Flow-based)](#12流式flow-based)
  - [2.two-level](#2two-level)
    - [offer-based](#offer-based)
    - [全连接两级调度(fully connected two-level)：Mira](#全连接两级调度fully-connected-two-levelmira)
  - [3.distributed](#3distributed)
    - [基于状态共享：Apollo，Omega，Tarcil](#基于状态共享apolloomegatarcil)
    - [完全分布式：Sparrow](#完全分布式sparrow)
  - [4.hybrid](#4hybrid)
  - [5. 集群调度的目的](#5-集群调度的目的)

## 1.centralized:  拥有全局视图

- Node manager: 部署在节点上，负责资源上报和任务状态收集
- Job master: 由每一个任务自己负责管理task的生命周期以及决定运行在哪个节点
- resource manager： 负责job master的资源申请和分配，基于全局资源状态

特点：

- 能支持多种调度策略：faireness，capacity，priority
- 能集成其他模块辅助调度

### 1.1基于队列的(Queue-based)

- YARN(多队列)
- Borg，K8S（单队列）

### 1.2流式(Flow-based)

- Quincy
- Firmament

## 2.two-level

优势: 可以将共享的计算集群分享给多个框架使用

- Marathon：容器调度器，用于调度长期运行的服务(类似K8S)
- Chronos: batch and repetitive job, airbnb 开发

劣势：

- 悲观锁，中心调度器分配出去的资源即使不被接受，也不在分给其他框架
- 中小调度器的资源分配延迟，以及中心调度器和计算框架之间通信延迟决定了调度延迟

改进：fully connected two-level

### offer-based

- Mesos，解耦资源分配与任务调度，中心调度器负责提供资源offer；各框架调度器可以接受offer或者拒绝offer，等待下次offer

### 全连接两级调度(fully connected two-level)：Mira

- 通过减少中心调度器和计算框架之间的通信延迟
- 及时重新分配被计算框架拒绝的资源，提高资源分配效率

## 3.distributed

状态共享的调度器通常使用共享的注册中心协调调度结果，而完全分布式调度器使用采样的形式进行分配，无需中心协调者

### 基于状态共享：Apollo，Omega，Tarcil

- 乐观锁
- 需要解决冲突；
- 无法满足对延迟需求小的任务调度

Omega: 任务提交的transaction存储在中心registry
Apollo：中心矩阵存储每个节点上可能等待的时间

the "cell state" in Omega, the "resource monitor" in Apollo, and the "plan queue" in Nomad.

### 完全分布式：Sparrow

- 局部资源视图：每个调度器只随机可见部分节点，去中心化决策
- 无协调者，可能导致资源竞争
- 无法实现全局需求，比如公平性，

缺点: 无法实现全局需求，比如fairless； 在集群负载重的情况下，无协调者导致资源冲突频繁，效率下降

## 4.hybrid

- 完全hybird： Mercury，
  - 一个中心调度器和若干分布式调度器；中心调度器负责少数长期运行的高优先级任务，往往占据集群大部分资源
  - 许多
- 状态共享的hybird:分布式scheduler只共享中心调度器的部分节点信息
- 两调度器系统：一个负责长期运行服务，一个负责短期运行的任务;(类似离线和在线使用不同的调度器）

## 5. 集群调度的目的

- 公平性：Max-Min Fairshare, DRF, EDF
  - Max-Min Fairshare: 将所有资源平均分成若干份，没人能拿到的平均的资源；实际申请小于平均的，分完之后将剩余资源放入空闲资源池子；实际申请比平均量大的其他用户，继续平均分空闲资源池子，直到资源池子分尽或者没有待分配的用户
  - DRF：先计算主导资源(资源占比最大的种类的占用率)；然后优先给主导资源占用率最小的用户优先分配
  - EDRF：DRF的缺陷 1:zero-demand case; 2. weighted demands case:给资源类型分不同的权重 3.不满足不可分割的假设(indivisibility assumption),DRF允许接收一半的资源
  - DRFH
- 效率(efficiency)
  - 资源利用率
    - Borg,k8s, openstack nova:filtering-scoring：
    - Apollo: 将任务分成普通任务-高优，和机会主义任务-低优：普通任务优先调度，低优任务找空闲资源填充
  - 能源效率
    - 减少active节点
- 性能
  - 调度性能：
    - Apollo: 基于存储waiting-time的中心矩阵：每个节点预计等待时间，调度器将任务调度到预计等待最少时间的节点
    - Miro: 优化任务申请和释放资源的延迟
