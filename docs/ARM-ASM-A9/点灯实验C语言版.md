#点灯实验C语言版

```  
/*
	1、控制两个灯：
	#define  GPX1CON        (*(volatile unsigned int *)0x11000c20)
	#define  GPX1DAT        (*(volatile unsigned int *)0x11000c24)
	GPX1CON = GPX1CON & ~(0xf)|(0x1);
	GPX1DAT = GPX1DAT & ~(0x01)   |(0x01)   ;
*/


/*
2、控制四个灯
  功能:   按键K2按下后能触发中断输出字符

 作业:  实现K3 按下后能触发中断输出字符
*/

//----------------uart
#define  GPA1CON    (*(volatile unsigned int *)0x11400020)
#define  ULCON2     (*(volatile unsigned int *)0x13820000) 
#define  UCON2      (*(volatile unsigned int *)0x13820004) 
#define  UBRDIV2    (*(volatile unsigned int *)0x13820028) 
#define  UFRACVAL2  (*(volatile unsigned int *)0x1382002C) 
#define  UTXH2      (*(volatile unsigned int *)0x13820020) 
#define  UTRSTAT2   (*(volatile unsigned int *)0x13820010) 


//---------------interrupt
#define  GPX1CON        (*(volatile unsigned int *)0x11000c20)
#define  EXT_INT41CON   (*(volatile  int *)0x11000E04)
#define  EXT_INT41_MASK (*(volatile  int *)0x11000F04)
#define ICDISER1_CPU0  (*(volatile  int *)0x10490104)
#define ICDIPTR14_CPU0 (*(volatile  int *)0x10490838)
#define ICDDCR (*(volatile  int *)0x10490000)
#define ICCICR_CPU0  (*(volatile  int *)0x10480000)
#define ICCPMR_CPU0  (*(volatile  int *)0x10480004)
#define EXT_INT41_PEND (*(volatile  int *)0x11000f44)
#define ICCIAR_CPU0  (*(volatile  int *)0x1048000C)
#define ICCEOIR_CPU0 (*(volatile  int *)0x10480010)
#define ICDICPR1_CPU0 (*(volatile  int *)0x10490284)

//----------------led
#define  GPX2CON        (*(volatile unsigned int *)0x11000c40)
#define  GPX1CON        (*(volatile unsigned int *)0x11000c20)
#define  GPF3CON        (*(volatile unsigned int *)0x114001e0)

#define  GPX2DAT        (*(volatile unsigned int *)0x11000c44)
#define  GPX1DAT        (*(volatile unsigned int *)0x11000c24)
#define  GPF3DAT        (*(volatile unsigned int *)0x114001e4)

void led_init()
{
	GPX2CON = GPX2CON & ~(0xf<<7)|(0x1<<7);
	GPX1CON = GPX1CON & ~(0xf)|(0x1);
	GPF3CON = GPF3CON & (~(0xF << 16)) | (0x1 << 16);
	GPF3CON = GPF3CON & (~(0xF << 20)) | (0x1 << 20);

}

void interrupt_init(void)
{
	//-----外: 配置管脚的工作模式
	GPX1CON = (GPX1CON & ~(0xF<<4))|(0xF<<4);  //配置 GPX1_1为中断模式
	
	EXT_INT41CON = (EXT_INT41CON & ~(0x7<<4))|(0x2<<4);  //设置GPX1_1的触发方式为 下降沿触发
	EXT_INT41_MASK = EXT_INT41_MASK & (~0x02);		//GPX1_1 中断使能

	
	
	//-----内: 功能块设置
	ICDISER1_CPU0 = ICDISER1_CPU0 | (1<<25);	//EINT9 (GPX1_1)  GIC中断使能
	ICDIPTR14_CPU0 = 0x01010101;   //参考例子背景，用默认设置 
	ICDDCR = ICDDCR|1; //GIC 分发总使能
	ICCICR_CPU0 = 1;  // CPU0  中断使能
	ICCPMR_CPU0 = 0XFF;   //设置CPU0的优先级门槛为最低

	//KEY3
	//外
	GPX1CON = (GPX1CON & ~(0xF<<8))|(0xF<<8);  //配置 GPX1_2为中断模式
	EXT_INT41CON = (EXT_INT41CON & ~(0x7<<8))|(0x2<<8);  //设置GPX1_2的触发方式未 下降沿触发
	EXT_INT41_MASK = EXT_INT41_MASK & (~0x04);		//GPX1_2 中断使能

	//内
	ICDISER1_CPU0 = ICDISER1_CPU0 | (1<<26);	//EINT9 (GPX1_2)  GIC中断使能
}



void uart_init(void)
{
  //-----外: 配置管脚的工作模式
  GPA1CON = 0x22;  //配置GPA1_1 GPA10 为uart串口模式
  
  //-----内: 功能块设置
  //1.  uart 通讯协议格式的设置
  ULCON2 = 0x03;  //设置协议格式(  无校验位  1个停止位 8个数据位)
  UCON2 = 0x05;   //设置串口发送接收模式为polling模式
  
  /*2.  设置uart 的速度为115200
  DIV_VAL = (100000000/(115200 *16)) - 1 = 53.253
  UBRDIVn =  53
  UFRACVALn/16 = 0.253
  Therefore, UFRACVALn = 4
  */
  UBRDIV2 = 53;
  UFRACVAL2 = 4;
    
}



void putc(char c)
{
   while(1)
   {
      if(UTRSTAT2&0x02)  //检测发送是否为空
      {
         break;
      }
   }
   
   UTXH2 = c;  //发送字符
}


void do_irq(void )
{

  int irq_num;

   irq_num = ICCIAR_CPU0&0x3ff; ////中断ID号

   switch(irq_num)
   {
   case 57:

   	   EXT_INT41_PEND = EXT_INT41_PEND|(1<<1);  //清GPX1_1中断标志	   
  	   ICDICPR1_CPU0 = ICDICPR1_CPU0|(1<<25);    //清GIC GPX1_1中断标志
   	   break;
	case 58:
   	   putc('k');
   	   EXT_INT41_PEND = EXT_INT41_PEND|(1<<2);  //清GPX1_2中断标志	   
  	   ICDICPR1_CPU0 = ICDICPR1_CPU0|(1<<26);    //清GIC GPX1_2中断标志
   	   break;
   default:
   	   putc('e');
   	   break;
   }
   	ICCEOIR_CPU0 = (ICCEOIR_CPU0&0x3FF)|irq_num;    //结束中断	将处理完成的中断ID号写入该寄存器，则表示相应的中断处理完成
}

int main(void) 
{
	led_init();
    uart_init();
    interrupt_init();

	GPX2DAT = GPX2DAT & ~(0x01<<7)|(0x01<<7);
	GPX1DAT = GPX1DAT & ~(0x01)   |(0x01)   ;
	GPF3DAT = GPF3DAT & ~(0x01<<4)|(0x01<<4);
	GPF3DAT = GPF3DAT & ~(0X01<<5)|(0X01<<5);
	while(1)
	{
  	
	}
	return 0;
}
```