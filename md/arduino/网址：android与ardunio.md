android通过OTG连接ardunio读写数据

[http://www.geek-workshop.com/thread-2277-1-1.html](http://www.geek-workshop.com/thread-2277-1-1.html)

Arduino和Android通过OTG 通信

[http://blog.csdn.net/xiaomuzi0802/article/details/50441213](http://blog.csdn.net/xiaomuzi0802/article/details/50441213)

android上的Jarduino库，直接复写了arduino方法在android上调用

[https://github.com/SINTEF-9012/JArduino/wiki](https://github.com/SINTEF-9012/JArduino/wiki)

android用usb控制arduino上的led灯，9600波特率

[http://tieba.baidu.com/p/2709535034](http://tieba.baidu.com/p/2709535034)

使用hc05蓝牙和手机连接

[https://blog.csdn.net/weixin_37272286/article/details/78016497?locationNum=10&fps=1](https://blog.csdn.net/weixin_37272286/article/details/78016497?locationNum=10&fps=1)

HC05蓝牙AT模式设置

[https://blog.csdn.net/jim_cheng_0_0/article/details/78514824](https://blog.csdn.net/jim_cheng_0_0/article/details/78514824)

HC05介绍

[地址](http://zhongbest.com/2016/09/01/%E8%93%9D%E7%89%99%E6%A8%A1%E5%9D%97hc05/)

微雪L293D电机驱动板

[http://www.waveshare.net/shop/Motor-Control-Shield.htm](http://www.waveshare.net/shop/Motor-Control-Shield.htm)

微雪5V步进电机

[http://www.waveshare.net/shop/5V-Step-Motor.htm](http://www.waveshare.net/shop/5V-Step-Motor.htm)

uln2003APG步进电机控制代码

[https://blog.csdn.net/jackhuang2015/article/details/44044507](https://blog.csdn.net/jackhuang2015/article/details/44044507)

串口接收字符串

[http://www.51hei.com/bbs/dpj-48290-1.html](http://www.51hei.com/bbs/dpj-48290-1.html)

串口函数大全

[https://blog.csdn.net/iracer/article/details/50334041](https://blog.csdn.net/iracer/article/details/50334041)

EEPROM掉电也能保存信息

[https://www.arduino.cn/thread-1157-1-1.html](https://www.arduino.cn/thread-1157-1-1.html)

有源蜂鸣器

[https://www.cnblogs.com/xiaowuyi/p/3343757.html](https://www.cnblogs.com/xiaowuyi/p/3343757.html)

红色激光头与上一个一模一样，记得区分正负极

  

如何使用微动开关或者按键开关切换状态（用开关控制开关灯，按一次亮，再按一次灭）

//注：微动开关根部和中间是常开状态（按下时闭合），根部和前部是常闭状态（按下时打开）

int led = 13;//板载的led

int btn = 0;//从数字口0作为按键口（按键的两端分别接在数字口0和GND上）

int state = 0;//led状态（0灭1亮）

int nowState = 0;//按键当前状态（0抬起1按下）

void setup() {

pinMode(led, OUTPUT);//led输出

pinMode(btn, INPUT);//btn读取

}

  

void loop() {

int getState = digitalRead(btn);//读取当前状态

delay(10);//延时防止抖动

if (getState != nowState) {//如果这一次loop中btn状态切换了

nowState = getState;//更新btn状态

if (nowState == 0) {//如果当前状态为0，代表这一次切换是从按下切换到了抬起状态，也就是触发完成

if (state == 0) {//如果led为0，就点亮

digitalWrite(led, HIGH);

state = 1;

} else {//否则熄灭

digitalWrite(led, LOW);

state = 0;

}

}

}

}

  

继电器

[https://blog.csdn.net/c80486/article/details/52622031](https://blog.csdn.net/c80486/article/details/52622031)