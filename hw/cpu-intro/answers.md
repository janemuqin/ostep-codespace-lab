
## Q1
- Prediction / 预测:
预测总时间：10
CPU 利用率：100%
```text
Time PID 0    PID 1   CPU IOs
1    RUN:cpu  READY   1
2    RUN:cpu  READY   1
3    RUN:cpu  READY   1
4    RUN:cpu  READY   1
5    RUN:cpu  READY   1
6    DONE     RUN:cpu 1
7    DONE     RUN:cpu 1
8    DONE     RUN:cpu 1
9    DONE     RUN:cpu 1
10   DONE     RUN:cpu 1
```
- Reasoning / 理由:
本题包含两个仅执行CPU计算的进程，无任何I/O操作。默认调度策略下，CPU密集型进程在执行过程中不会主动让出处理器，因此PID 0先连续执行5条CPU指令，期间PID 1处于就绪队列等待。PID 0执行完毕后，PID 1依次执行5条CPU指令。整个过程中每个时间片都有进程占用CPU，不存在空闲周期。总运行时间为10个tick，CPU利用率为100%，全程无I/O设备活动。
- Verified result / 验证结果:
- Analysis / 分析:

## Q2
- Prediction / 预测:
预测总时间：11
CPU 利用率：6/11 ≈ 54.55%
```text
Time PID 0    PID 1        CPU IOs
1    RUN:cpu  READY        1
2    RUN:cpu  READY        1
3    RUN:cpu  READY        1
4    RUN:cpu  READY        1
5    DONE     RUN:io       1
6    DONE     BLOCKED          1
7    DONE     BLOCKED          1
8    DONE     BLOCKED          1
9    DONE     BLOCKED          1
10   DONE     BLOCKED          1
11*  DONE     RUN:io_done  1
```
- Reasoning / 理由:
PID 0包含4条CPU指令，PID 1包含1次I/O操作。默认的SWITCH_ON_IO策略仅在进程发起I/O时触发上下文切换，纯CPU计算过程中不会发生调度。因此前4个时间片内CPU由PID 0独占，PID 1保持就绪状态。PID 0执行完成后，PID 1开始执行完整的I/O流程：1个tick发起I/O请求，5个tick处于阻塞状态等待设备响应，最后1个tick处理I/O完成中断。由于此时没有其他就绪进程，阻塞阶段CPU全程空闲。总运行时间为11个tick，其中CPU忙碌6个tick，利用率约为54.55%。
- Verified result / 验证结果:
- Analysis / 分析:

## Q3
- Prediction / 预测:
预测总时间：7
CPU 利用率：6/7 ≈ 85.71%
```text
Time PID 0        PID 1        CPU IOs
1    RUN:io       READY        1
2    BLOCKED      RUN:cpu      1   1
3    BLOCKED      RUN:cpu      1   1
4    BLOCKED      RUN:cpu      1   1
5    BLOCKED      RUN:cpu      1   1
6    BLOCKED      DONE             1
7*   RUN:io_done  DONE         1
```
- Reasoning / 理由:
Q3的进程工作量与Q2完全一致，仅两个进程的启动顺序调换。PID 0先执行，在第1个tick发起I/O请求后进入阻塞状态；依据SWITCH_ON_IO策略，系统立即切换至PID 1执行。PID 1的4条CPU指令刚好在PID 0阻塞的前4个tick内执行完毕。第6个tick时PID 1已运行结束，PID 0仍处于阻塞期，该时间片CPU处于空闲。第7个tick PID 0处理I/O完成，全部进程结束。总运行时间为7个tick，CPU利用率约为85.71%。与Q2相比总时长明显缩短，核心原因是I/O阻塞期间CPU可以调度其他进程，避免了计算资源闲置
- Verified result / 验证结果:
- Analysis / 分析:

## Q4
- Prediction / 预测:
预测总时间：11
CPU 利用率：6/11 ≈ 54.55%
```text
Time PID 0        PID 1    CPU IOs
1    RUN:io       READY    1
2    BLOCKED      READY        1
3    BLOCKED      READY        1
4    BLOCKED      READY        1
5    BLOCKED      READY        1
6    BLOCKED      READY        1
7*   RUN:io_done  READY    1
8    DONE         RUN:cpu  1
9    DONE         RUN:cpu  1
10   DONE         RUN:cpu  1
11   DONE         RUN:cpu  1
```
- Reasoning / 理由:
本题进程配置与Q3相同，调度策略更换为SWITCH_ON_END：仅当当前进程完全执行完毕后，系统才会切换至下一个进程，即使进程因I/O阻塞也不会让出CPU。因此PID 0在第1个tick发起I/O后，后续5个阻塞周期内CPU始终空闲，PID 1持续处于就绪队列等待。直到第7个tick PID 0处理完I/O并结束后，PID 1才开始执行4条CPU指令。总运行时间为11个tick，CPU利用率约为54.55%。对比Q3可以看出，不支持I/O触发切换的调度策略会造成CPU资源的大量浪费，整体运行效率更低。
- Verified result / 验证结果:
- Analysis / 分析:

## Q5
- Prediction / 预测:
预测总时间：7
CPU 利用率：6/7 ≈ 85.71%
```text
Time PID 0        PID 1        CPU IOs
1    RUN:io       READY        1
2    BLOCKED      RUN:cpu      1   1
3    BLOCKED      RUN:cpu      1   1
4    BLOCKED      RUN:cpu      1   1
5    BLOCKED      RUN:cpu      1   1
6    BLOCKED      DONE             1
7*   RUN:io_done  DONE         1
```
- Reasoning / 理由:
本题显式指定-S SWITCH_ON_IO调度策略，该策略为模拟器的默认配置，因此运行过程与最终结果均与Q3一致。进程发起I/O后立即触发上下文切换，I/O阻塞期间CPU可调度其他就绪进程，仅在阻塞末期存在1个tick的CPU空闲。总运行时间为7个tick。与Q4的SWITCH_ON_END策略相比，总时长明显缩短，体现了I/O触发调度对CPU资源利用率的提升作用。
- Verified result / 验证结果:
- Analysis / 分析:

## Q6
- Prediction / 预测:
预测总时间：31
CPU 利用率：21/31 ≈ 67.74%
```text
Time PID 0        PID 1        PID 2        PID 3        CPU IOs
1    RUN:io       READY        READY        READY        1
2    BLOCKED      RUN:cpu      READY        READY        1   1
3    BLOCKED      RUN:cpu      READY        READY        1   1
4    BLOCKED      RUN:cpu      READY        READY        1   1
5    BLOCKED      RUN:cpu      READY        READY        1   1
6    BLOCKED      RUN:cpu      READY        READY        1   1
7*   READY        DONE         RUN:cpu      READY        1
8    READY        DONE         RUN:cpu      READY        1
9    READY        DONE         RUN:cpu      READY        1
10   READY        DONE         RUN:cpu      READY        1
11   READY        DONE         RUN:cpu      READY        1
12   READY        DONE         DONE         RUN:cpu      1
13   READY        DONE         DONE         RUN:cpu      1
14   READY        DONE         DONE         RUN:cpu      1
15   READY        DONE         DONE         RUN:cpu      1
16   READY        DONE         DONE         RUN:cpu      1
17   RUN:io_done  DONE         DONE         DONE         1
18   RUN:io       DONE         DONE         DONE         1
19   BLOCKED      DONE         DONE         DONE             1
20   BLOCKED      DONE         DONE         DONE             1
21   BLOCKED      DONE         DONE         DONE             1
22   BLOCKED      DONE         DONE         DONE             1
23   BLOCKED      DONE         DONE         DONE             1
24*  RUN:io_done  DONE         DONE         DONE         1
25   RUN:io       DONE         DONE         DONE         1
26   BLOCKED      DONE         DONE         DONE             1
27   BLOCKED      DONE         DONE         DONE             1
28   BLOCKED      DONE         DONE         DONE             1
29   BLOCKED      DONE         DONE         DONE             1
30   BLOCKED      DONE         DONE         DONE             1
31*  RUN:io_done  DONE         DONE         DONE         1
```
- Reasoning / 理由:
PID 0包含3次I/O操作，PID 1、PID 2、PID 3各包含5条CPU指令，调度策略为SWITCH_ON_IO搭配IO_RUN_LATER：I/O完成后，对应进程进入就绪队列尾部等待调度，不会抢占当前运行的进程。第1个tick PID 0发起第一次I/O后阻塞，系统切换至PID 1执行，第2至6个tick PID 1运行完毕。第7个tick时PID 0的I/O已完成，但依据IO_RUN_LATER规则，需按队列顺序依次调度PID 2、PID 3，各执行5个tick。第17个tick PID 0才被调度，处理第一次I/O完成后立即发起第二次I/O。后两次I/O执行期间，其余进程均已结束，没有就绪进程可调度，CPU全程空闲，每次I/O各占7个tick。总运行时间为31个tick，其中CPU忙碌21个tick，利用率约为67.74%。同时I/O设备存在明显的空闲期，第一次I/O完成后间隔了较长时间才收到下一次请求。
- Verified result / 验证结果:
- Analysis / 分析:

## Q7
- Prediction / 预测:
预测总时间：21
CPU 利用率：21/21 = 100%
```text
Time PID 0        PID 1        PID 2        PID 3        CPU IOs
1    RUN:io       READY        READY        READY        1
2    BLOCKED      RUN:cpu      READY        READY        1   1
3    BLOCKED      RUN:cpu      READY        READY        1   1
4    BLOCKED      RUN:cpu      READY        READY        1   1
5    BLOCKED      RUN:cpu      READY        READY        1   1
6    BLOCKED      RUN:cpu      READY        READY        1   1
7*   RUN:io_done  DONE         READY        READY        1
8    RUN:io       DONE         READY        READY        1
9    BLOCKED      DONE         RUN:cpu      READY        1   1
10   BLOCKED      DONE         RUN:cpu      READY        1   1
11   BLOCKED      DONE         RUN:cpu      READY        1   1
12   BLOCKED      DONE         RUN:cpu      READY        1   1
13   BLOCKED      DONE         RUN:cpu      READY        1   1
14*  RUN:io_done  DONE         DONE         READY        1
15   RUN:io       DONE         DONE         READY        1
16   BLOCKED      DONE         DONE         RUN:cpu      1   1
17   BLOCKED      DONE         DONE         RUN:cpu      1   1
18   BLOCKED      DONE         DONE         RUN:cpu      1   1
19   BLOCKED      DONE         DONE         RUN:cpu      1   1
20   BLOCKED      DONE         DONE         RUN:cpu      1   1
21*  RUN:io_done  DONE         DONE         DONE         1
```
- Reasoning / 理由:
本题进程配置与Q6一致，调度策略更换为IO_RUN_IMMEDIATE：I/O完成后立即调度对应进程运行，无需进入就绪队列排队。第1个tick PID 0发起第一次I/O后阻塞，第2至6个tick PID 1执行。第7个tick I/O完成，立即调度PID 0处理完成中断，第8个tick随即发起第二次I/O，阻塞期间切换至PID 2执行5个tick。第14个tick第二次I/O完成，立即调度PID 0，第15个tick发起第三次I/O，阻塞期间PID 3执行5个tick。第21个tick第三次I/O处理完成，所有进程结束。整个过程中CPU没有空闲周期，总运行时间为21个tick，CPU利用率为100%。与Q6相比总时长显著缩短，原因在于I/O密集型进程完成后立即继续执行，既保证了I/O设备持续忙碌，也让CPU在I/O阻塞阶段始终有进程可运行，整体资源利用率更高。
- Verified result / 验证结果:
- Analysis / 分析:

## Q8
- Prediction / 预测:

### 种子 1（-s 1）
指令列表：PID 0 = cpu, io, io（共 3 条）；PID 1 = cpu, cpu, cpu。

默认设置：预测总时间 15，CPU 利用率 8/15 ≈ 53.33%，I/O 忙 10/15 ≈ 66.67%
```text
Time PID 0        PID 1        CPU IOs
1    RUN:cpu      READY        1
2    RUN:io       READY        1
3    BLOCKED      RUN:cpu      1   1
4    BLOCKED      RUN:cpu      1   1
5    BLOCKED      RUN:cpu      1   1
6    BLOCKED      DONE             1
7    BLOCKED      DONE             1
8*   RUN:io_done  DONE         1
9    RUN:io       DONE         1
10   BLOCKED      DONE             1
11   BLOCKED      DONE             1
12   BLOCKED      DONE             1
13   BLOCKED      DONE             1
14   BLOCKED      DONE             1
15*  RUN:io_done  DONE         1
```
-I IO_RUN_IMMEDIATE：总时间 15，CPU 利用率 8/15 ≈ 53.33%（与默认相同）
-S SWITCH_ON_END：总时间 18，CPU 利用率 8/18 ≈ 44.44%

### 种子 2（-s 2）
指令列表：PID 0 = io, io, cpu；PID 1 = cpu, io, io。

默认设置：预测总时间 16，CPU 利用率 10/16 = 62.5%，I/O 忙 14/16 = 87.5%
```text
Time PID 0        PID 1        CPU IOs
1    RUN:io       READY        1
2    BLOCKED      RUN:cpu      1   1
3    BLOCKED      RUN:io       1   1
4    BLOCKED      BLOCKED          2
5    BLOCKED      BLOCKED          2
6    BLOCKED      BLOCKED          2
7*   RUN:io_done  BLOCKED      1   1
8    RUN:io       BLOCKED      1   1
9*   BLOCKED      RUN:io_done  1   1
10   BLOCKED      RUN:io       1   1
11   BLOCKED      BLOCKED          2
12   BLOCKED      BLOCKED          2
13   BLOCKED      BLOCKED          2
14*  RUN:io_done  BLOCKED      1   1
15   RUN:cpu      BLOCKED      1   1
16*  DONE         RUN:io_done  1
```
-I IO_RUN_IMMEDIATE：总时间 16，CPU 利用率 10/16 = 62.5%（与默认相同）
-S SWITCH_ON_END：总时间 30，CPU 利用率 10/30 ≈ 33.33%

### 种子 3（-s 3）
指令列表：PID 0 = cpu, io, cpu；PID 1 = io, io, cpu。

默认设置：预测总时间 18，CPU 利用率 9/18 = 50%，I/O 忙 11/18 ≈ 61.11%
```text
Time PID 0        PID 1        CPU IOs
1    RUN:cpu      READY        1
2    RUN:io       READY        1
3    BLOCKED      RUN:io       1   1
4    BLOCKED      BLOCKED          2
5    BLOCKED      BLOCKED          2
6    BLOCKED      BLOCKED          2
7    BLOCKED      BLOCKED          2
8*   RUN:io_done  BLOCKED      1   1
9*   RUN:cpu      READY        1
10   DONE         RUN:io_done  1
11   DONE         RUN:io       1
12   DONE         BLOCKED          1
13   DONE         BLOCKED          1
14   DONE         BLOCKED          1
15   DONE         BLOCKED          1
16   DONE         BLOCKED          1
17*  DONE         RUN:io_done  1
18   DONE         RUN:cpu      1
```
-I IO_RUN_IMMEDIATE：总时间 17，CPU 利用率 9/17 ≈ 52.94%
-S SWITCH_ON_END：总时间 24，CPU 利用率 9/24 = 37.5%

- Reasoning / 理由:
通过三个不同随机种子生成进程指令序列，对比三种调度策略下的运行差异。首先明确基础规则：单次I/O的完整执行周期为7个tick，包含1个tick的发起阶段、5个tick的阻塞阶段和1个tick的完成处理阶段。所有策略下进程的CPU计算总量是固定的，总运行时间的差异本质上来源于CPU空闲时长的不同。
默认设置 SWITCH_ON_IO 会在进程发起 I/O 后切换到别的进程，这样一个进程阻塞时，另一个进程可以用 CPU。种子 1 PID 1 仅包含 3 条 CPU 指令，刚好填满 PID 0 第一次 I/O 阻塞的前 3 个 tick；PID 1 结束后，PID 0第一次I/O剩下的阻塞时间和第二次I/O都没有其他进程可调度，CPU共空闲7个tick，总运行时间为 15 个 tick。种子 2 两个进程均包含 I/O 指令，阻塞期存在部分重叠，其中第4至6个tick和第11至13个tick共6个tick双进程同时阻塞，CPU无进程可运行；其余时间始终有进程占用 CPU，总运行时间为 16 个 tick。种子 3 里 PID 1 第一次 I/O 完成时，PID 0 仍在占用 CPU 执行；默认策略下 PID 1 只能进入就绪队列等待，直到 PID 0 完全结束后才继续执行，因此总运行时间延长至 18 个 tick。
IO_RUN_IMMEDIATE 该策略仅在“I/O完成时刻CPU正被其他进程占用”的场景下与默认策略产生差异：I/O完成后直接抢占CPU运行对应进程，无需排队。
- 种子1与种子2中，每次I/O完成时，CPU要么处于空闲状态，要么另一进程处于阻塞态，因此“立即运行”与“排队等待”的效果一致，总运行时间与默认设置没有差异。
- 种子3中，PID 1的I/O完成时PID 0正在运行，立即调度可让PID 1提前发起下一次I/O；而PID 0剩余的最后一条CPU指令，可安排在PID 1第二次I/O的阻塞期内执行，总运行时间较默认策略缩短1个tick，变为17个tick。
SWITCH_ON_END 要等当前进程完全结束才切换，I/O 阻塞期间 CPU 不会去运行别的进程，所以三个种子都最慢：每个进程的 I/O 等待时间都完全浪费，总时间等于两个进程各自单独运行的时间之和（种子 1：15 + 3 = 18；种子 2：15 + 15 = 30；种子 3：9 + 15 = 24）。
- Verified result / 验证结果:
- Analysis / 分析:

