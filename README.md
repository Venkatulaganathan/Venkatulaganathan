/*can2_driver.c*/
#include <LPC21xx.H>
#include "header.h"
void can2_init(void){
	PINSEL1|=0x00014000;//P0.23-->RD2 & P0.24-->TD2
	VPBDIV=1; //PCLK=60MHz
	C2MOD=0x1; //CAN2 into Reset  Mode 
	AFMR=0x2; //accept all receiving messages(data/remote)
	C2BTR=0x001C001D; //B125Kbps @ PLCK=60MHz(BRP=Pclk/bit rate*16)
	C2MOD=0x0; //CAN1 into Normal Mode
}
void can2_tx(CAN2_MSG m1)
{
  C2TID1=m1.id;
C2TFI1=(m1.dlc<<16);	
	if(m1.rtr==0) //if data frame
	{
		C2TFI1&=~(1<<30);//RTR=0
		C2TDA1=m1.byteA; //lower 4bytes of data
		C2TDB1=m1.byteB; //upper 4bytes of data
	}
	else{
		C2TFI1|=(1<<30); //RTR=1
	}
	C2CMR=(1<<0)|(1<<5);//Start Xmission & select Tx Buf1
	//C2CMR=(3<<0)|(1<<5);//Start Xmission & select Tx Buf1
	while((C2GSR&(1<<3))==0);//wait for data Xmission.
}
void can2_rx(CAN2_MSG *m1){
	while((C2GSR&0x1)==0);
	m1->id=C2RID;
	m1->dlc=(C2RFS>>16)&0xF;
	m1->rtr=(C2RFS>>30)&0x1;
	if(m1->rtr==0){ //if data frame
		m1->byteA=C2RDA;
		m1->byteB=C2RDB;
	}
	C2CMR=(1<<2);//free receiver buffer(imp)
}


/* main_nodeA_TX.c */
#include "header.h"
main(){	
	CAN2_MSG m1;
	//m2;
	can2_init();
	/*sending data frame*/
	m1.id=0x000001AF;
	m1.rtr=0;//data frame
	m1.dlc=4;
	m1.byteA=0xDDCCBBAA;
	m1.byteB=0;
	
	/*m2.id=0x5;
	m2.dlc=7;
	m2.rtr=1;*/
	while(1)
		{		
		can2_tx(m1);//data-frame
		delay_s(1);
		/*can2_tx(m2);//remote frame
		delay_ms(2000);*/
		//can2_rx(&m2);
		}
}









