# 加速同调连光都可以超越项目文档

## 前言

本项目为临时项目，主要用于前期对于mpu6050的调试以及摸索。主要根据整点原子的代码进行二次开发。

后期会大幅改动该部分的代码并将其作为驱动层进行使用（正点的代码写的。。。）

## 如何计算速度

速度的计算主要依据模块内部的加速度模块进行计算，利用公式 V = V0 + a * t，得出当前速度，利用S = S0 + V * t，得出行驶路程。暂时感觉不需要GPS传感器单独进行速度计算。

所以当前问题在于如何获取精准的时段，用于速度以及路程的计算。暂定使用一个通用定时器，该定时器设置为秒级定时器（若为毫秒级定时器预计出现大量的计算过程以及速度及加速度的计算间隔过短，出现速度太过灵敏的情况。该处后续可更改。）在获取加速度以及计算速度时，获取定时器的值，并清空定时器。将加速度信息以及定时器信息打包进行下一步的计算。伪代码如下：

```C
struct aac_t
{
    uint32_t aac;
    uint32_t time;
};

struct velocity_t
{
    uint32_t velocity;
    uint32_t time;
};

uint32_t getAACX(struct acc_t* acc_t)//获取X轴加速度信息
{
    uint32_t aacx = 0, time = 0;
    if (mpu_6050_get_aacx(&aacx))
    {
        return 1;//返回获取x轴加速度信息错误
    }
    
    if (get_aac_time(&time))
    {
        return 2;//返回获取时段信息错误
    }
    acc_t.acc = accx;
    acc_t.time = time;
    time = 0;
    
    if (set_aac_time(&time))//清楚定时器时间
    {
        return 3;//返回时间设置错误信息
    }
    
    return 0;
}

uint32_t getVelocityX(struct velocity_t*, struct acc_t* acc_t)//获取X轴速度信息
{
    static uint32_t XVelocity = 0;
    uint32_t time = 0;
    
    if (acc_t == NULL)
    {
        return 1;
    }
    
    XVelocity = XVelocity + acc_t->acc * acc_t->time;
    
    if (get_velocity_time(&time))
    {
        return 2;
    }
    
    velocity_t->velocity = velocity;
    velocity_t->time = time;
    time = 0;
    
    if (set_velocity_time(&time))
    {
        return 3;
    }
    
    return 0;
}

uint32_t getDistanceX(uint32_t distance*, struct velocity_t*)
{
    if (velocity_t == NULL)
    {
        return 1;
    }
    
    distance = distance +  velocity_t->velocity * velocity_t->time;
    
    return 0;
}

```

## 如何识别压弯（生命安全考虑，谨慎压弯）

利用mpu6050的x轴与y轴的角度进行识别，识别出车辆的舵角与倾角。

欢迎大伙补充