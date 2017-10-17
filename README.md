# -
一个项目的我的思考过程的记录





主代码如下：
#include"stdafx.h";
#include"conio.h";
#include"gts.h"

//定义轴号
#define AXIS 2
#define AXUS 1
void commandhandler(char *command, short error)
{
	// 如果指令执行返回值为非0，说明指令执行错误，向屏幕输出错误结果
	if (error)
	{
		printf("%s = %d\n", command, error);
	}
}
int main(int argc, char* argv[])
{
	short sRtn;
	TJogPrm jog,jug;
	long sts,sus;
	double prfPos, prfVel,Pos,Vel;
//打开运动控制器
	sRtn = GT_Open();
	commandhandler("GT_Open", sRtn);
	//复位运动控制器
	sRtn = GT_Reset();
	commandhandler("GT_Reset", sRtn);
	//配置运动控制器
	sRtn = GT_LoadConfig("test.cfg");
	commandhandler("GT_LoadConfig", sRtn);
	//清除隔周的报警和限位
	sRtn = GT_ClrSts(1, 4);
	commandhandler("GT_ClrSts", sRtn);
	//将AXIS轴设为jog运动模式
	sRtn = GT_PrfJog(AXIS);
	commandhandler("GT_PrfJog", sRtn);
	sRtn = GT_PrfJog(AXUS);
	commandhandler("GT_PrfJog", sRtn);
	//读取jog运动参数
	sRtn = GT_GetJogPrm(AXIS, &jog);
	commandhandler("GT_GetJogPrm", sRtn);
	jog.acc = 0.125;
	jog.dec = 0.25;
	sRtn = GT_GetJogPrm(AXUS, &jug);
	commandhandler("GT_GetJogPrm", sRtn);
	jug.acc = 0.125;
	jug.dec = 0.25;
	//设置jog运动参数
	sRtn = GT_SetJogPrm(AXIS, &jog);
	commandhandler("GT_SetJogPrm", sRtn);
	sRtn = GT_SetJogPrm(AXUS, &jug);
	commandhandler("GT_SetJogPrm", sRtn);
		
	for (int i = 0; i < 2; i++)
	{
		//设置AXIS轴的目标速度
		sRtn = GT_SetVel(AXIS, 12);
		commandhandler("GT_SetVel", sRtn);
		//启动AXIS轴的运动
		sRtn = GT_Update(1 << (AXIS - 1));
		commandhandler("GT_Update", sRtn);
		//设置AXIS轴的目标速度
		sRtn = GT_SetVel(AXUS, 6);
		commandhandler("GT_SetVel", sRtn);
		//启动AXIS轴的运动
		sRtn = GT_Update(1 << (AXUS - 1));
		commandhandler("GT_Update", sRtn);
		while (1)
		{
			//读取AXIS轴的状态
			sRtn = GT_GetSts(AXIS, &sts);
			//读取规划位置
			sRtn = GT_GetPrfPos(AXIS, &prfPos);
			//读取规划速度
			sRtn = GT_GetPrfVel(AXIS, &prfVel);
			printf("sts=0x%-101xprfPos=%-10.11fprfVel=%-10.11f\r", sts, prfPos, prfVel);
			if (prfPos >= 100000)
			{
				sRtn = GT_SetVel(AXIS, -12);
				commandhandler("GT_SetVel", sRtn);
				// AXIS轴新的目标速度生效
				sRtn = GT_Update(1 << (AXIS - 1));
				commandhandler("GT_Update", sRtn);
				while (1)
				{
					//读取AXIS轴的状态
					sRtn = GT_GetSts(AXIS, &sts);
					//读取规划位置
					sRtn = GT_GetPrfPos(AXIS, &prfPos);
					//读取规划速度
					sRtn = GT_GetPrfVel(AXIS, &prfVel);
					printf("sts=0x%-101xprfPos=%-10.11fprfVel=%-10.11f\r", sts, prfPos, prfVel);
					if (prfPos <= 400)
					{
						sRtn = GT_SetVel(AXIS, 0);
						commandhandler("GT_SetVel", sRtn);
						// AXIS轴新的目标速度生效
						sRtn = GT_Update(1 << (AXIS - 1));
						commandhandler("GT_Update", sRtn);
						break;
					}
				}
				break;
			}
		}
		while (1)
		{
			//读取AXIS轴的状态
			sRtn = GT_GetSts(AXUS, &sus);
			//读取规划位置
			sRtn = GT_GetPrfPos(AXUS, &Pos);
			//读取规划速度
			sRtn = GT_GetPrfVel(AXUS, &Vel);
			printf("sus=0x%-101xPos=%-10.11fVel=%-10.11f\r", sus, Pos, Vel);
			if (Pos >= 50000)
			{
				sRtn = GT_SetVel(AXUS, -6);
				commandhandler("GT_SetVel", sRtn);
				// AXIS轴新的目标速度生效
				sRtn = GT_Update(1 << (AXUS - 1));
				commandhandler("GT_Update", sRtn);
				while (1)
				{
					//读取AXIS轴的状态
					sRtn = GT_GetSts(AXUS, &sus);
					//读取规划位置
					sRtn = GT_GetPrfPos(AXUS, &Pos);
					//读取规划速度
					sRtn = GT_GetPrfVel(AXUS, &Vel);
					printf("sus=0x%-101xPos=%-10.11fVel=%-10.11f\r", sus, Pos, Vel);
					if (Pos <= 400)
					{
						sRtn = GT_SetVel(AXUS, 0);
						commandhandler("GT_SetVel", sRtn);
						// AXIS轴新的目标速度生效
						sRtn = GT_Update(1 << (AXUS - 1));
						commandhandler("GT_Update", sRtn);
						break;
					}
				}
				break;
			}
		}
	}
	while (!_kbhit())
	{
		//读取AXIS轴的状态
		sRtn = GT_GetSts(AXIS, &sts);
		//读取规划位置
		sRtn = GT_GetPrfPos(AXIS, &prfPos);
		//读取规划速度
		sRtn = GT_GetPrfVel(AXIS, &prfVel);
		//读取AXIS轴的状态
		printf("sts=0x%-101xprfPos=%-10.11fprfVel=%-10.11f\r", sts, prfPos, prfVel);
		//读取AXIS轴的状态
		sRtn = GT_GetSts(AXUS, &sus);
		//读取规划位置
		sRtn = GT_GetPrfPos(AXUS, &Pos);
		//读取规划速度
		sRtn = GT_GetPrfVel(AXUS, &Vel);
		printf("sus=0x%-101xPos=%-10.11fVel=%-10.11f\r", sus, Pos, Vel);
	}
	_getch();
	return 0;
}
