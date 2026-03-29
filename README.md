# 引言
引言还没想好 >< 
总之这是一份教学README，能够尽快让大家看懂代码，学会在原有的代码上加减/重构

# 命名与注释参考
完全理想的命名能做到三件事：
1. 让人一眼就能明白它可能在哪定义&声明，并快速找到它      
    （如果别人突然发现某个文件里出现了陌生的变量，让他可以不靠F12，直接理解它“最早出现在哪”
    ——这能够极大提高**变量查找和修改**的效率，不必特别费心记住“每个变量原来在哪”）
2. 让人能快速明白它在被用来做什么了；并且在调试时，不需要回看上下文重新理解它
    （别人能够不靠“回看它的注释”就理解它的作用——或者只看一两遍注释就记住它的作用
    ——如果需要同时理解的上下文非常多，那么这样做无疑会降低**理解难度**——不管怎么说都没有坏处）
3. 让人能够根据上下文明白“我该如何使用它” 
    （别人看到一个函数，结合它的引用情况和命名，大部分情况下都能“自己学会它的用法”
    ——使用代码时，大部分情况下能**不依赖原作者线下指导**——这是异步协同开发的必要条件）

当然，人与人之间的相互理解是最困难的，更别说通过代码这种东西隔空对话了
因此在一般情况下，我们只要求代码的命名**形式统一，文能对题，有区分度，一个月后你还能看懂**即可：

1. **形式统一** 大小写的使用，下划线的使用，变量的首个单词的来源，缩写规则**有规律可循**
                而不是想到怎么写就怎么写，这个文件这样的规则，那个文件那样的习惯
2. **文能对题** 变量确实**和你的命名相关**
                而不是“写的是state，跑去做counter”；“说的是joint，结果在chassis里用了”
                或者甚至“用abc”
3. **有区分度** 变量之间能从名字**看出来差别**
                而不是“所有counter都长得一样都叫counter，全称缩写也差不多，只是调用位置不一样”
                或者缩写**存在歧义**
4. **一个月后你还能看懂** 顾名思义

为了实现这一目标，我们提出了如下的命名方法做参考
我们会以“格式-示例-解释”的形式给出

**非常鼓励大家形成自己的代码风格，在程序编写中找到创作感和归属感！**

## 独立变量和值定义命名参考
```C
<前缀> <类型名> <文件名>_<对象>_<做什么> = 0;

uint8_t upper_clamp_state = 0; 
upper.c里定义    夹爪的状态记录

uint8_t stateupd_upd_pcnt = 0; // for app_state_update(),program counter
// (How you use it)
app_state_update.c里定义    app_state_update()函数的执行次数计数器（program counter => pcnt）

uint16_t stateupd_upd_delaycnt = 0; // for app_state_update(),delaytime counter
// (How you use it)
app_state_update.c里定义    app_state_update()函数的时间计数器（delay counter => delaycnt）
别写dtcnt——歧义是“时间微分的计算值”——会产生歧义的缩写，宁愿写长名称也不要缩

#define JOINT_FOREARM_UPLIM 4095 // 编码器范围[0,8191]
joint.c里定义   小臂仰角极限值

禁止使用以下形式作为独立变量名：
tmp, test, data, flag, cnt（不带前缀）
```

## 结构体（对象的实例）命名参考
```C
<自定结构体类型名> <对象>_<做什么&什么性质>_obj = {0};
chassis_ctrl_obj    底盘控制对象（它本身就是对象的实例化）
dbus_ctrl_data      遥控器dbus的控制数据结构体

不需要再强调文件名：一般文件名就是“指这个对象”；结构体是多个文件都会用的“主要对象”，不存在不好找的问题
```
**“一个文件，一类对象，一个实例”是好文明**
强烈推荐一个.c文件里，只有一个和它名字强相关的结构体
当你想新加一个对象的时候，首先想到的不是“在文件里新建一个结构体”，而是“再在合适的文件夹里新建一个.c”
如果一个.c文件中出现 两个及以上同类型实例
**应立即考虑是否需要拆文件，修改或上移抽象层级**

## 结构体，枚举类型命名参考
```C
typedef __<自定结构体类型名> 
{
    <结构体内部变量>;
}<自定结构体名>;

自定结构体类型名：<对象>_<什么性质&和做什么相关>_sTypeDef
（当然HAL库有HandleTypeDef这样的，形如xxxTypeDef的区分——不过我们的层级没这么细）
（尽管如此，sTypeDef本身也是在说：这是结构体的类型名）
（极其不推荐枚举类型名也用xxxTypeDef这种叫法）
joint_ctrl_sTypeDef     关节的控制结构体定义
joint_data_sTypeDef     关节的数据结构体定义
rm_motor_piddata_sTypeDef  RM电机的pid参数结构体定义

结构体内部变量：不需要再强调“对象”——因为使用这些变量的时候，结构体名就说过了：
joint_data_sTypeDef.exp_data[2]
甚至
joint_data_sTypeDef.expect[2]（当然，一致性不是很强，也不够清楚强调这是“data”）
而不是
joint_data_sTypeDef.joint_exp_data[2]
但我们希望内部是清晰的，比如通过注释：
joint_data_sTypeDef.exp_data[2] // 0:Forearm 1:Backarm
再或者
joint_data_sTypeDef.tgt_spd[3] 
joint_data_sTypeDef.rel_spd[3] （你对“整个机构的实际速度”有定义需求）

typedef __<枚举类型名> enum
{
    <枚举>,
}<枚举类型名>;

枚举类型名：<对象>_<做什么> （首字母大写，更直观的和独立变量分开）
Joint_InitFlag  initflag    关节初始化的状态机

枚举：<大写字母>
DEFAULT = 0,
WAITING = 1,
DONE = 2,

MOTOR_1 = 0x01,
MOTOR_2 = 0x02,
MOTOR_3 = 0x04,
MOTOR_4 = 0x08,

枚举使用的位置几乎都是在结构体内做内部变量（独立变量不需要用枚举这种正式方式）
因此和结构体的命名一样，不需要再强调对象
即便是在函数里做参数，枚举类型名足够告诉你这个枚举的对象是谁了
```

## 函数，参数参考及include规范
### 函数及参数参考
```C
<返回值类型> <文件名>_<对象>_<做什么事>(<参数类型名> <参数类型缩写><参数对象（或者带上做什么事）（或其缩写）>);

Lib里面的函数类似这样
float lib_astar_calculate(float input) {
    float output
    /* astar codes */
    return output;
}
Drv里面的函数类似这样
typedef sturct __rm_motor_grp_ctrlTypeDef
{
    /* codes */
}rm_motor_ctrlTypeDef;
void rm_motor_ctrl_init(rm_motor_grp_ctrlTypeDef *hmgrp) {
    /* codes */
}
Bsp里面的函数类似这样
void bsp_tim_init(void) {
    HAL_TIM_Base_Start_IT(&htim1);
}
Mech里面的函数类似这样
void mech_joint_init(void) {
    /* codes */
}
App里面的函数类似这样
typedef struct __mech_stateTypeDef
void  app_state_update(mech_stateTypeDef *hms) {
    /* codes */
}
```
关于函数的分层，可以用
/* internal functions --------------------------*/
/* external functions --------------------------*/
这样的注释来区分“内部函数”和“外部接口”，当然更推荐使用vscode的“大纲”功能，或直接查看.h的函数声明，快速审阅函数

关于函数的注释，推荐用vscode自带的这种：
/**
 * @brief 电机控制组初始化
 * 
 * @param hmgrp 电机控制组实例
 * @param type  电机类型：M2006，M6020或M3508
 * @param group_stdid 电机组标识符：GRP1_4对应电调id1-4，GRP5_7对应电调id5-7
 * @param hcan can外设指针,如：&hcan1
 * @param mode Normal 常规pid控制；MIT mit控制
 */
void rm_motor_ctrl_init(rm_motor_grpCtrlTypeDef* hmgrp, 
                        MotorType type, GrpStdType group_stdid, CAN_HandleTypeDef* hcan)

键入/** 后按回车就可以了
当然文件开头的注释也可以如此写

### include规范
在注释里推荐这样部署（可以直接复制）
```C
/* HAL Includes --------------------------*/
#include "can.h"
/* Drv Includes --------------------------*/
#include "drv_xxx.h"
```
尽管我们的部分文件命名前面都有文件名前缀，但这些注释本身展示了include层级
如果你发现drv的.c的include里有app，mech等高层.h，重写吧，层级都乱了

# 工程文件结构

keil的.uvprojx 文件名一般就是项目名
我们非常建议“项目名=嵌入式平台型号名&称呼”或者“项目名=执行单元名”，因为同型号板子的配置几乎差不多，可以直接复制（尤其是Stm32CubeMX）
而存放项目的文件夹名形如“25_beijing_mech_arm_0913”，包含时间信息，开发组，开发的机器人等
或者把相关的信息在README里写好

以下是我们现行的工程文件结构：
```shell
ROOT:.
├─ Core         入口层（程序启动的入口）
│   └─ main.c   (重要！所有的高层init函数必须在此文件的while之前以合适顺序完成，允许在此处使用HAL_Delay)
│
├─ App          应用层（实例化与业务逻辑）
│   ├─ app_state_update.c
│   ├─ app_fast.c  (若赶进度，临时直接控制机器人运行，请在这里堆屎山，并把其他地方的必要函数注释掉)
│   └─ ...
│
├─ Mech         机构层（机器人机构抽象定义）
│   ├─ mech_joint.c
│   ├─ mech_upper.c
│   ├─ mech_chassis.c
│   └─ ...
│
├─ Bsp          板级层（根据任务配置外设初始化、回调行为）
│   ├─ bsp_gpio.c
│   ├─ bsp_uart.c
│   ├─ bsp_can.c
│   ├─ bsp_tim.c    (重要！App函数在此文件下的中断回调函数内循环执行——不要把它们放进main.c的while里！不允许在此处使用HAL_Delay，不允许任何放进此处的函数内部有HAL_Delay!!!HAL_Delay是最低优先级中断，会出现互相等待的情况，使程序转不起来)
│   ├─ bsp_exti.c
│   └─ ...
│
├─ Drv          驱动层（文件名看情况加不加drv_前缀）
│   ├─ rm_motor.c
│   ├─ imu.c
│   ├─ dbus.c
│   └─ ...
│
├─ Lib          算法库（文件名看情况加不加lib_前缀）
│   ├─ pid.c
│   ├─ filter.c
│   ├─ astar.c
│   └─ ...
│
├─ Drivers      芯片厂商提供的底层驱动 (HAL, CMSIS)
│   ├─ stm32f4xx_hal.c
│   ├─ stm32f4xx_it.c   (重要！有些高层update函数需要在“中断服务函数”里执行以确保持续运行，比如dbus)
│   └─ ...
│
├─ Test         测试库
│   └─ ...  (我们推荐屎山还是在这里堆比较好)
│
```