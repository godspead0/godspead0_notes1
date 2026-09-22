**使用的开发板**：`STM32F103C8T6`

> [!NOTE]
> *此处为开发板与面包板实物图*

---

### 一、GPIO 输出

#### 1. LED 闪烁

LED 灯亮的逻辑是正极接 3.3V，负极接 GPIO 引脚，通过设定引脚电平来控制灯亮：高电平不亮，低电平亮。如果反过来负极接地、正极接引脚，那高低电平控制逻辑也随之反转。

其中设定 GPIO 输出模式为**推挽输出**（`Out_PP`），这样能正反都能控制；如果是开漏输出，那么高电平将不能和地产生能使 LED 灯亮的电流。

为了模块化编程，先写一个文件来专门控制 LED 灯开关。首先初始化引脚，编写初始化函数 `void LED_init(void)`：

```c
void LED_init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_LEDINIT;
    GPIO_LEDINIT.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_LEDINIT.GPIO_Pin = GPIO_Pin_1;
    GPIO_LEDINIT.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_LEDINIT);
    
    GPIO_SetBits(GPIOA, GPIO_Pin_1);
}
```

* `RCC_APB2PeriphClockCmd`：用于开启 GPIOA 的外设时钟。
* `GPIO_InitTypeDef GPIO_LEDINIT`：定义一个结构体变量，用于储存初始化参数。
* 输出模式为推挽输出 `GPIO_Mode_Out_PP`。
* 引脚配置为 `GPIO_Pin_1`。
* 输出速率配置为 `GPIO_Speed_50MHz`。
* 调用库函数 `GPIO_Init` 写入配置。
* `GPIO_SetBits`：防止一开始初始化后默认低电平导致灯亮，将默认电平置为高电平，保持初始状态熄灭。

然后编写模块化控制灯亮灭的函数：`void LED1_ON(void)` 和 `void LED1_OFF(void)`（将 PA1 引脚连接的灯命名为 LED1）：

```c
void LED1_ON(void)
{
    GPIO_ResetBits(GPIOA, GPIO_Pin_1);
}

void LED1_OFF(void)
{
    GPIO_SetBits(GPIOA, GPIO_Pin_1);
}
```

* `GPIO_ResetBits`：设置引脚为低电平（对应 LED 点亮）。
* `GPIO_SetBits`：设置引脚为高电平（对应 LED 熄灭）。

将函数在新建的 `LED.h` 头文件中声明：

```c
void LED_init(void); 
void LED1_ON(void);
void LED1_OFF(void);
```

主函数引用头文件 `#include "LED.h"` 后即可直接调用。

**主函数代码：**

```c
int main(void)
{
    LED_init();
    while(1)
    {
        LED1_ON();
        Delay_ms(500);
        LED1_OFF();
        Delay_ms(500);
    }
}
```

> **延时函数说明**：工程中包含了 `#include "Delay.h"` 头文件，包含 `Delay_us`（微秒级）、`Delay_ms`（毫秒级）和 `Delay_s`（秒级）三个函数，参数为对应单位的时长。分析其底层原理，发现是在定时器设定好后，通过执行空循环语句的数量来精确控制延时时长的。

程序循环点亮 LED 500ms、熄灭 500ms，实现了 LED 闪烁效果。

---

#### 2. LED 流水灯

沿用上述控制逻辑增加一路 LED，实现两颗灯交替闪烁。

在初始化函数中加入第二个引脚 PA2：

```c
void LED_init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_LEDINIT;
    GPIO_LEDINIT.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_LEDINIT.GPIO_Pin = GPIO_Pin_2 | GPIO_Pin_1;
    GPIO_LEDINIT.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_LEDINIT);
    
    GPIO_SetBits(GPIOA, GPIO_Pin_1 | GPIO_Pin_2);
}
```

> 初始化同组端口的多个引脚时，可以使用按位或运算符 `|` 同时配置。若需要同时初始化 PA 和 PB 两组不同端口，则需要分别编写两套初始化代码。

添加控制第二个 LED（PA2，命名为 LED2）的函数：

```c
void LED2_ON(void)
{
    GPIO_ResetBits(GPIOA, GPIO_Pin_2);
}

void LED2_OFF(void)
{
    GPIO_SetBits(GPIOA, GPIO_Pin_2);
}
```

在头文件声明后，编写主函数交替控制两颗灯的亮灭：

```c
int main(void)
{
    LED_init();
    while(1)
    {
        LED1_ON();
        LED2_OFF();
        Delay_ms(500);
        LED1_OFF();
        LED2_ON();
        Delay_ms(500);
    }
}
```

> **后记思考**：两个引脚初始配置均为推挽输出，后续尝试开漏输出模式时发现：开漏输出设置为高电平时，引脚与地之间虽有电位差但无驱动电流。  
> **（已解决，来自张凤希学长指导）**：开漏输出高电平状态下内部断开，呈现高阻态（阻值近似无穷大），无法对外主动输出电流。开漏模式的优势在于便于实现电平转换与“线与”逻辑，可通过外部上拉电阻将输出电平拉至任意所需电压。

---

### 二、GPIO 输入

#### 按键控制 LED 灯

通过检测按键连接引脚的电平状态变化，产生开关信号来控制 LED。

硬件接线：按键一端接引脚（PB1、PB11），另一端接地。  
引脚配置为**上拉输入**（`IPU`），默认状态下为高电平；当按键按下导通接地时，引脚电平被拉低，检测到低电平信号。

**按键初始化代码：**

```c
void KEY_INIT(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
    
    GPIO_InitTypeDef GPIO_KEYINIT;
    GPIO_KEYINIT.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_KEYINIT.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_11;
    GPIO_KEYINIT.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB, &GPIO_KEYINIT);
}
```

电平检测使用标准库函数 `GPIO_ReadInputDataBit`。由于机械按键在闭合和断开瞬间存在机械抖动，需要进行软件消抖。

**按键检测函数：**

```c
uint8_t KEY_IO(void)
{
    uint8_t KEYRET = 0;
    
    if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)
    {
        Delay_ms(20); // 消抖
        while(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0); // 等待按键松开
        Delay_ms(20);
        KEYRET = 1;
    }
    if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11) == 0)
    {
        Delay_ms(20);
        while(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11) == 0);
        Delay_ms(20);
        KEYRET = 2;
    }
    return KEYRET;
}
```

**初始主函数（按下亮灯）：**

```c
int main(void)
{
    LED_init();
    KEY_INIT();
    while(1)
    {
        num = KEY_IO();
        if(num == 1)
        {
            LED1_ON();
        }
        if(num == 2)
        {
            LED2_ON();
        }
    }
}
```

为了实现“按一下开、再按一下关”的状态翻转，在 `LED.c` 中添加电平反转函数：

```c
void LED1_TURN(void)
{
    if(GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_1) == 0)
    {
        GPIO_SetBits(GPIOA, GPIO_Pin_1);
    }
    else
    {
        GPIO_ResetBits(GPIOA, GPIO_Pin_1);
    }
}

void LED2_TURN(void)
{
    if(GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_2) == 0)
    {
        GPIO_SetBits(GPIOA, GPIO_Pin_2);
    }
    else
    {
        GPIO_ResetBits(GPIOA, GPIO_Pin_2);
    }
}
```

**改进后的主函数代码：**

```c
uint8_t num;

int main(void)
{
    LED_init();
    KEY_INIT();
    while(1)
    {
        num = KEY_IO();
        if(num == 1)
        {
            LED1_TURN();
        }
        if(num == 2)
        {
            LED2_TURN();
        }
    }
}
```

> **后记**：除 `GPIO_ReadInputDataBit` 外，库中还有读取整组端口电平的输入函数以及多种输出库函数，均在固件库头文件中作了详尽定义。

---

### 三、OLED 显示屏使用

采用 $I^2C$ 接口的 4 脚 OLED 屏：

* `VCC` / `GND`：电源供电
* `SCK`：接 `PB8`
* `SDA`：接 `PB9`

**显示测试主函数：**

```c
int main(void)
{
    OLED_Init();
    OLED_ShowChar(1, 2, 'h');
    OLED_ShowString(2, 2, "Hello world");
    OLED_ShowNum(3, 2, 3, 1);
    while(1)
    {
        
    }
}
```

**核心驱动库函数参数说明：**

* `OLED_Init()`：初始化 OLED 引脚与控制器。
* `OLED_ShowChar(Row, Col, Char)`：在指定行列（行 $1\sim4$，列 $1\sim16$）显示 ASCII 字符。字符点阵字模预置在 `OLED_Font.h` 中。
* `OLED_ShowString(Row, Col, String)`：在指定位置显示字符串。
* `OLED_ShowNum(Row, Col, Number, Length)`：显示无符号整型数值（范围 $0\sim4294967295$），参数包含显示长度；显示带符号数字可配合 `OLED_ShowSignedNum()` 使用。

---

### 四、EXTI 外部中断

#### 1. 对射式红外传感器计次

传感器为 4 引脚模块，使用 3 个引脚：`VCC`、`GND`、`DO`（数字输出脚接 `PB14`）。

**中断原理简述**：当触发中断源事件时，CPU 暂停当前程序并保护现场，跳转执行中断服务函数（ISR），执行完毕后恢复现场并返回主流程。外部中断通过监测 GPIO 引脚的边沿变化（上升沿/下降沿）产生中断请求并送达 NVIC 控制器。

**初始化流程**：

1. 开启 GPIOB 和 AFIO 外设时钟。
2. 配置 PB14 为上拉输入。
3. 通过 AFIO 进行外部中断引脚映射绑定。
4. 配置 EXTI 触发模式（双边沿触发）。
5. 配置 NVIC 中断通道及抢占/响应优先级。

**初始化代码：**

```c
void Encoder_Init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStr;
    GPIO_InitStr.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStr.GPIO_Pin = GPIO_Pin_14;
    GPIO_InitStr.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB, &GPIO_InitStr);
    
    GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);
    
    EXTI_InitTypeDef EXTI_InitStr;
    EXTI_InitStr.EXTI_Line = EXTI_Line14;
    EXTI_InitStr.EXTI_LineCmd = ENABLE;
    EXTI_InitStr.EXTI_Mode = EXTI_Mode_Interrupt;
    EXTI_InitStr.EXTI_Trigger = EXTI_Trigger_Rising_Falling;
    EXTI_Init(&EXTI_InitStr);
    
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    
    NVIC_InitTypeDef NVIC_InitStr;
    NVIC_InitStr.NVIC_IRQChannel = EXTI15_10_IRQn;
    NVIC_InitStr.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStr.NVIC_IRQChannelPreemptionPriority = 1;
    NVIC_InitStr.NVIC_IRQChannelSubPriority = 1;
    NVIC_Init(&NVIC_InitStr);
}
```

**中断处理与计次函数：**

```c
uint16_t CountS;

uint16_t CountS_return(void)
{
    return CountS;
}

void EXTI15_10_IRQHandler(void)
{
    if (EXTI_GetITStatus(EXTI_Line14) == SET)
    {
        Delay_ms(200); // 延时防抖
        CountS++;
        EXTI_ClearITPendingBit(EXTI_Line14); // 清除中断挂起标志位
    }
}
```

> **调试技巧**：在 Keil5 中点击 Debug 按钮进入在线调试模式，在中断函数内部打上断点。通过遮挡传感器观察全速运行是否准确停在断点处，从而验证中断通路配置正确。

**主函数：**

```c
int main(void)
{
    OLED_Init();
    Encoder_Init();
    OLED_ShowString(1, 1, "Count:");
    
    while(1)
    {
        OLED_ShowNum(1, 7, CountS_return(), 5);
    }
}
```

---

#### 2. 旋转编码器计次

旋转编码器引脚连接：`A 相`接 `PB0`，`B 相`接 `PB1`。  
原理：正转时 B 相下降沿处检测到 A 相应处于低电平；反转时 A 相下降沿处检测到 B 相应处于低电平。通过配置两路外部中断检测下降沿，并在中断中读取另一相状态判断旋向。

**初始化配置：**

```c
void Encoder_Init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStr;
    GPIO_InitStr.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStr.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;
    GPIO_InitStr.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB, &GPIO_InitStr);
    
    GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource0);
    GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource1);
    
    EXTI_InitTypeDef EXTI_InitStr;
    EXTI_InitStr.EXTI_Line = EXTI_Line1 | EXTI_Line0;
    EXTI_InitStr.EXTI_LineCmd = ENABLE;
    EXTI_InitStr.EXTI_Mode = EXTI_Mode_Interrupt;
    EXTI_InitStr.EXTI_Trigger = EXTI_Trigger_Falling;
    EXTI_Init(&EXTI_InitStr);
    
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    
    NVIC_InitTypeDef NVIC_InitStr;
    NVIC_InitStr.NVIC_IRQChannel = EXTI0_IRQn;
    NVIC_InitStr.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStr.NVIC_IRQChannelPreemptionPriority = 1;
    NVIC_InitStr.NVIC_IRQChannelSubPriority = 1;
    NVIC_Init(&NVIC_InitStr);
    
    NVIC_InitStr.NVIC_IRQChannel = EXTI1_IRQn;
    NVIC_InitStr.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStr.NVIC_IRQChannelPreemptionPriority = 1;
    NVIC_InitStr.NVIC_IRQChannelSubPriority = 2;
    NVIC_Init(&NVIC_InitStr);
}
```

**中断处理与清零提取：**

```c
int16_t CountS;

void EXTI0_IRQHandler(void)
{
    if (EXTI_GetITStatus(EXTI_Line0) == SET)
    {
        if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)
        {
            CountS--;
        }
        EXTI_ClearITPendingBit(EXTI_Line0);
    }
}

void EXTI1_IRQHandler(void)
{
    if (EXTI_GetITStatus(EXTI_Line1) == SET)
    {
        if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_0) == 0)
        {
            CountS++;
        }
        EXTI_ClearITPendingBit(EXTI_Line1);
    }
}

int16_t CountS_return(void)
{
    int16_t tp = CountS;
    CountS = 0;
    return tp;
}
```

**主函数：**

```c
int16_t num;

int main(void)
{
    OLED_Init();
    Encoder_Init();
    OLED_ShowString(1, 1, "num:");
    
    while(1)
    {
        num += CountS_return();
        OLED_ShowSignedNum(1, 5, num, 5);
    }
}
```

> **后记**：中断服务函数不需要在头文件中声明，也不由主函数调用。  
> **（已解决）**：当硬件中断触发时，NVIC 自动根据启动文件中的中断向量表直接寻址跳转执行，因此中断函数名必须严格与向量表定义一致。

---

### 五、TIM 定时中断

#### 1. 定时中断秒表

通过定时器内部时钟源与计数器溢出产生周期性中断。系统主频为 $72\text{ MHz}$，设定溢出时间为 $1\text{ s}$（频率 $1\text{ Hz}$）：

$$f = \frac{72\text{ MHz}}{(\text{Period} + 1) \times (\text{Prescaler} + 1)} = \frac{72\text{ MHz}}{10000 \times 7200} = 1\text{ Hz}$$

**定时器初始化（含初始溢出标志清除）：**

```c
void Timer_Init(void)
{
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
    
    TIM_InternalClockConfig(TIM2);
    
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseStructure.TIM_Period = 10000 - 1;
    TIM_TimeBaseStructure.TIM_Prescaler = 7200 - 1;
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);
    
    // 关键修正：清除初始化时产生的预置更新标志位，避免复位后从 1 开始计数
    TIM_ClearFlag(TIM2, TIM_FLAG_Update);
    
    TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
    
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    
    NVIC_InitTypeDef NVIC_InitStructure;
    NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    NVIC_Init(&NVIC_InitStructure);
    
    TIM_Cmd(TIM2, ENABLE);
}
```

**中断处理与主函数：**

```c
extern uint16_t num;

void TIM2_IRQHandler(void)
{
    if(TIM_GetITStatus(TIM2, TIM_IT_Update) == SET)
    {
        num++;
        TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
    }
}
```

```c
uint16_t num;

int main(void)
{
    OLED_Init();
    Timer_Init();
    
    OLED_ShowString(1, 1, "Num:");
    
    while(1)
    {
        OLED_ShowNum(1, 5, num, 5);
        OLED_ShowNum(2, 5, TIM_GetCounter(TIM2), 5); // 实时监测底层 CNT 计数值变化
    }
}
```

---

#### 2. 定时器外部时钟

将 TIM2 的时钟源由内部时钟替换为通过 ETR（引脚 PA0）引入的外部脉冲。每次外部信号产生脉冲使计数器加 1，当计数达到自动重装值 10 时触发更新中断。

**关键配置代码：**

```c
void Timer_Init(void)
{
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    // 配置 ETR 外部时钟模式 2
    TIM_ETRClockMode2Config(TIM2, TIM_ExtTRGPSC_OFF, TIM_ExtTRGPolarity_NonInverted, 0x0F);
    
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseStructure.TIM_Period = 10 - 1;       // 计满 10 次溢出
    TIM_TimeBaseStructure.TIM_Prescaler = 1 - 1;      // 不分频
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);
    
    TIM_ClearFlag(TIM2, TIM_FLAG_Update);
    TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
    
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    NVIC_InitTypeDef NVIC_InitStructure;
    NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    NVIC_Init(&NVIC_InitStructure);
    
    TIM_Cmd(TIM2, ENABLE);
}

uint16_t TimerCount_return(void)
{
    return TIM_GetCounter(TIM2);
}
```

---

### 六、PWM 输出与应用

#### 1. PWM 呼吸灯

利用定时器输出比较（OC）功能生成可变占空比的方波信号驱动 LED。引脚设为复用推挽输出 `GPIO_Mode_AF_PP`，ARR 设置为 $100 - 1$，此时比较值 CCR 即直观对应 $0\% \sim 100\%$ 的占空比。

**初始化与控制函数：**

```c
void PWM_Init(void)
{
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_LEDINIT;
    GPIO_LEDINIT.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_LEDINIT.GPIO_Pin = GPIO_Pin_0;
    GPIO_LEDINIT.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_LEDINIT);
    
    TIM_InternalClockConfig(TIM2);
    
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseStructure.TIM_Period = 100 - 1;
    TIM_TimeBaseStructure.TIM_Prescaler = 720 - 1;
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);
    
    TIM_OCInitTypeDef TIM_OCInitStructure;
    TIM_OCStructInit(&TIM_OCInitStructure); // 给结构体成员赋初值
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
    TIM_OCInitStructure.TIM_Pulse = 90;
    TIM_OC1Init(TIM2, &TIM_OCInitStructure);
    
    TIM_Cmd(TIM2, ENABLE);
}

void PWM_SetCompare1(uint16_t Compare)
{
    TIM_SetCompare1(TIM2, Compare);
}
```

**呼吸灯主循环逻辑：**

```c
uint8_t i;

int main(void)
{
    OLED_Init();
    PWM_Init();
    
    while(1)
    {
        for(i = 0; i <= 100; i++)
        {
            PWM_SetCompare1(i);
            Delay_ms(10);
        }
        for(i = 0; i <= 100; i++)
        {
            PWM_SetCompare1(100 - i);
            Delay_ms(10);
        }
    }
}
```

---

#### 2. PWM 驱动舵机

驱动舵机（SG90 等）需要周期为 $20\text{ ms}$（$50\text{ Hz}$）的控制信号。高电平时间在 $0.5\text{ ms} \sim 2.5\text{ ms}$ 之间，对应控制角 $0^\circ \sim 180^\circ$。

设置参数：

* 定时器时钟频率经分频后为 $1\text{ MHz}$（$\text{PSC} = 72 - 1$），即计数器步长为 $1\text{ }\mu\text{s}$。
* 周期为 $20\text{ ms}$，对应 $\text{ARR} = 20000 - 1$。
* 占空比映射公式：
    $$\text{CCR} = \frac{\text{Angle}}{180^\circ} \times 2000 + 500$$
    * $\text{Angle} = 0^\circ \implies \text{CCR} = 500\text{ }(0.5\text{ ms})$
    * $\text{Angle} = 90^\circ \implies \text{CCR} = 1500\text{ }(1.5\text{ ms})$
    * $\text{Angle} = 180^\circ \implies \text{CCR} = 2500\text{ }(2.5\text{ ms})$

**舵机驱动模块封装：**

```c
void Servo_Init(void)
{
    PWM_Init();
}

void Servo_SetAngle(float Angle)
{
    PWM_SetCompare2((uint16_t)(Angle / 180.0f * 2000.0f + 500.0f));
}
```

**按键步进调角主函数（每按一次加 $30^\circ$）：**

```c
uint8_t KeyNum;
float Angle;

int main(void)
{
    OLED_Init();
    Servo_Init();
    KEY_INIT();
    
    OLED_ShowString(1, 1, "Angle:");
    
    while(1)
    {
        KeyNum = KEY_IO();
        if(KeyNum == 1)
        {
            Angle += 30;
            if(Angle > 180)
                Angle = 0;
        }
        Servo_SetAngle(Angle);
        OLED_ShowNum(1, 7, Angle, 3);
    }
}
```

---

#### 3. PWM 直流电机调速与换向

利用电机驱动芯片驱动电机，PA2 输出 PWM 控制转速，PA4 与 PA5 控制 H 桥方向。

**电机模块封装：**

```c
void Motor_Init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_LEDINIT;
    GPIO_LEDINIT.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_LEDINIT.GPIO_Pin = GPIO_Pin_4 | GPIO_Pin_5;
    GPIO_LEDINIT.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_LEDINIT);
    
    PWM_Init(); // PA2 对应 TIM2_CH3
}

void Motor_SetSpeed(int8_t Speed)
{
    if(Speed >= 0)
    {
        GPIO_SetBits(GPIOA, GPIO_Pin_4);
        GPIO_ResetBits(GPIOA, GPIO_Pin_5);
        PWM_SetCompare3(Speed);
    }
    else
    {
        GPIO_SetBits(GPIOA, GPIO_Pin_5);
        GPIO_ResetBits(GPIOA, GPIO_Pin_4);
        PWM_SetCompare3(-Speed);
    }
}
```

**主函数（按键循环调节速度 $-100 \sim +100$）：**

```c
uint8_t KeyNum;
int8_t Speed;

int main(void)
{
    OLED_Init();
    Motor_Init();
    KEY_INIT();
    
    OLED_ShowString(1, 1, "Speed:");
    while(1)
    {
        KeyNum = KEY_IO();
        if(KeyNum)
        {
            Speed += 20;
            if(Speed > 100)
            {
                Speed = -100;
            }
        }
        Motor_SetSpeed(Speed);
        OLED_ShowSignedNum(1, 7, Speed, 3);
    }
}
```

---

### 七、输入捕获（IC）与频率/占空比测量

#### 1. 输入捕获测频率

利用定时器从模式的复位功能（Reset Mode）：当输入端检测到上升沿时，把计数器值捕获至 CCR1，并由硬件自动清零 CNT 重新开始计数。相邻两次上升沿之间的计数值即对应输入信号的周期。

$$f = \frac{f_{\text{count}}}{\text{CCR1}} = \frac{1\text{ MHz}}{\text{CCR1}}$$

```c
void IC_Init(void)
{
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitTypeDef GPIO_LEDINIT;
    GPIO_LEDINIT.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_LEDINIT.GPIO_Pin = GPIO_Pin_6; // PA6 对应 TIM3_CH1
    GPIO_LEDINIT.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA, &GPIO_LEDINIT);
    
    TIM_InternalClockConfig(TIM3);
    
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseStructure.TIM_Period = 65536 - 1;
    TIM_TimeBaseStructure.TIM_Prescaler = 72 - 1; // 1MHz 计数频率
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);
    
    TIM_ICInitTypeDef TIM_ICInitStructure;
    TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;
    TIM_ICInitStructure.TIM_ICFilter = 0xF;
    TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;
    TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
    TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;
    TIM_ICInit(TIM3, &TIM_ICInitStructure);
    
    TIM_SelectInputTrigger(TIM3, TIM_TS_TI1FP1);
    TIM_SelectSlaveMode(TIM3, TIM_SlaveMode_Reset);
    
    TIM_Cmd(TIM3, ENABLE);
}

uint32_t IC_GetFreq(void)
{
    return 1000000 / TIM_GetCapture1(TIM3);
}
```

---

#### 2. PWMI 模式测量频率与占空比

通过配置 `TIM_PWMIConfig`，硬件自动将 CH1 设为上升沿捕获周期（对应整个周期计数值 CCR1），将 CH2 设为下降沿捕获高电平时间（对应高电平计数值 CCR2）。

占空比计算公式：

$$\text{Duty} = \frac{\text{CCR2} + 1}{\text{CCR1} + 1} \times 100\%$$

```c
uint32_t IC_GetDuty(void)
{
    return (TIM_GetCapture2(TIM3) + 1) * 100 / (TIM_GetCapture1(TIM3) + 1);
}
```

**主函数测量显示：**

```c
int main(void)
{
    OLED_Init();
    PWM_Init();
    IC_Init();
    
    OLED_ShowString(1, 1, "Freq:00000Hz");
    OLED_ShowString(2, 1, "Duty:00%");
    
    PWM_SetPSC(720 - 1);
    PWM_SetCompare1(30);
    
    while(1)
    {
        OLED_ShowNum(1, 6, IC_GetFreq(), 5);
        OLED_ShowNum(2, 6, IC_GetDuty(), 2);
    }
}
```

