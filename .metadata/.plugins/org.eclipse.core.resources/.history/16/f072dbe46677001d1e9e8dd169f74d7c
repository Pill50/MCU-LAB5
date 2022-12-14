#include "main.h"
#include "button.h"
#include "input_processing.h"
#include "SoftwareTimer.h"
#include "global.h"
#include "stdio.h"
#define TRUE 1
#define FALSE 0
#define T_7SEGLED 250

int RED_DURATION;
int YELLOW_DURATION ;
int GREEN_DURATION;

int pes_led_period;
int pes_led_count;

int red_temp_dur, yellow_temp_dur, green_temp_dur;

enum State{INIT ,MODE1, MODE2, MODE3, MODE4, INCREASE, INCREASE_500MS, UPDATE_DURATION} ;
enum LED{RED,YELLOW,GREEN};
enum State state = INIT;

/* WAY1 TRAFFIC LIGHT VAR START */
enum LED current_led_way1;
int leds_way1_count[3];
/* WAY1 TRAFFIC LIGHT VAR END */

/* WAY2 TRAFFIC LIGHT VAR START */
enum LED current_led_way2;
int leds_way2_count[3];
/* WAY2 TRAFFIC LIGHT VAR END */

int button1_executed=0;
int button2_increase1=0;

/* VAR NG DI BO */
int flag_pes=0;
int button4_executed=0;
/* VAR NG DI BO */

enum LED LedToChange;

const int MAX_LED=2;
int index_7SEGLed=0;
int led_buffer1[2];
int led_buffer2[2];
void update_led_buffer(int val);
void Switch7SEGLEDIndex();
void displayMode(int mode);
void update7SEG(int index);
void loadLedTempDuration(enum LED color);
void increaseTempDur(enum LED color);
void updateDuration(enum LED color);
void trafficLightCount(int*leds_count,enum LED*current_led);
void displaySingleLedsMode1();
void display7SEGMode1();
void turnOffSingleLed();
void PES_LED_ON();
void PES_LED_OFF();
void ledBlinking(enum LED led_to_change);
void turnOnGreen1();
void turnOnRed1();
void turnOnYellow1();
void turnOff1();
void turnOnGreen2();
void turnOnRed2();
void turnOnYellow2();
void turnOff2();
void ToggleRED();
void ToggleGREEN();
void ToggleYELLOW();

void fsm_for_input_processing(UART_HandleTypeDef*huart){
	switch(state){
	case INIT:
		/* INIT PORT OUTPUT START */
		turnOffSingleLed();
		/* INIT PORT OUTPUT END */

		/* INITIALIZE TEMPORARY DURATION START */
		red_temp_dur=RED_DURATION;
		yellow_temp_dur=YELLOW_DURATION;
		green_temp_dur=GREEN_DURATION;
		/* INITIALIZE TEMPORARY DURATION END */

		/* INITIALIZE COUNT START */
		leds_way1_count[0]=RED_DURATION;
		leds_way1_count[1]=GREEN_DURATION;
		leds_way1_count[2]=YELLOW_DURATION;
		turnOnRed1();
		current_led_way1=RED;
		leds_way2_count[0]=RED_DURATION;
		leds_way2_count[1]=GREEN_DURATION;
		leds_way2_count[2]=YELLOW_DURATION;
		turnOnGreen2();
		current_led_way2=GREEN;
		/* INITIALIZE COUNT END */

		state=MODE1;
		setTimer4(1000);  //SET TIMER4 FOR COUNT DOWN

		/* PES_LED SETUP START */
		pes_led_period=(RED_DURATION+YELLOW_DURATION+GREEN_DURATION)*2;
		pes_led_count=pes_led_period;
		/* PES_LED_SETUP END */
		break;
	case MODE1:
		/* COUNT DOWN START */
		if(timer4_flag==1){
			setTimer4(1000);
			pes_led_count--;
			trafficLightCount(leds_way1_count,&current_led_way1);
			trafficLightCount(leds_way2_count,&current_led_way2);
			displaySingleLedsMode1();
		}
		/* COUNT DOWN END */

		/* UART TRANSMIT START */
		if(current_led_way1==RED){
			update_led_buffer(leds_way1_count[0]);
		}else if(current_led_way1==GREEN){
			update_led_buffer(leds_way1_count[1]);
		}else if(current_led_way1==YELLOW){
			update_led_buffer(leds_way1_count[2]);
		}
		HAL_UART_Transmit(huart, traffic_light_num, sprintf(traffic_light_num,"!7SEG:%d%d#",led_buffer1[0],led_buffer1[1]), 100);
		/* UART TRANSMIT END */


		/*EXECUTE INPUT BUTTON1 START */
		if(button1_executed==0){
			if(button1_pressed()==TRUE){
				state=MODE2;
				button1_executed=1;
				setTimer1(250);   //set timer for red led blinking
				/* INITIALIZE TEMP DURATION START */
				red_temp_dur=RED_DURATION;
				yellow_temp_dur=YELLOW_DURATION;
				green_temp_dur=GREEN_DURATION;
				/* INITIALIZE TEMP DURATION END */
				turnOffSingleLed();
			}
		}else if(button1_executed==1){
			if(button1_pressed()==FALSE){
				button1_executed=0;
			}
		}
		/*EXECUTE INPUT BUTTON1 END */

		/* PES LED EXCUTION START */
		if(button4_pressed()==1){
			if(button4_executed==0){
				flag_pes=1;
				button4_executed=1;
				timer2_flag = 1;
				pes_led_count=pes_led_period;
			}
		}else if(button4_pressed()==0){
			button4_executed=0;
		}

		if(flag_pes==1){
			if(current_led_way1==RED){
				PES_LED_ON();
				if (leds_way1_count[0] <= 3){
					if (timer2_flag == 1){
						pwm = pwm +25;
						setTimer2(30);
					}
				}
			}else{
				PES_LED_OFF();
				timer2_flag = 1;
				pwm = 0;
			}
			if(pes_led_count<=0 && current_led_way1!=RED){
				PES_LED_OFF();
				pwm = 0;
				flag_pes=0;
			}
		}
		/* PES LED EXCUTION END */
		break;
	case MODE2:

		/* BLINK SINGLE RED LED START */
//		if(timer1_flag==1){
//			setTimer1(250);
//			HAL_GPIO_TogglePin(RED_LED1_GPIO_Port, RED_LED1_Pin);  //TOGGLE RED LEDS IN 2 WAY
//			HAL_GPIO_TogglePin(RED_LED2_GPIO_Port, RED_LED2_Pin);
//		}
		/* BLINK SINGLE RED LED END */

		/*EXECUTE INPUT BUTTON1 START */
		if(button1_executed==0){
			if(button1_pressed()==TRUE){
				state=MODE3;
				button1_executed=1;
				turnOffSingleLed();
			}
		}else if(button1_executed==1){
			if(button1_pressed()==FALSE){
				button1_executed=0;
			}
		}
		/*EXECUTE INPUT BUTTON1 END */

		/*EXECUTE INPUT BUTTON3 START */
		if(button3_pressed()==TRUE){
			state=UPDATE_DURATION;
			LedToChange=RED;
		}
		/*EXECUTE INPUT BUTTON3 END */

		/*EXECUTE INPUT BUTTON2 START */
		if(button2_pressed()==TRUE){
			state=INCREASE;
			LedToChange=RED;
			button2_increase1=0;
		}
		/*EXECUTE INPUT BUTTON2 END */
		//TO DO
		/* UART TRANSMIT START */
		if(LedToChange==RED){
			update_led_buffer(red_temp_dur);
		}
		HAL_UART_Transmit(huart, traffic_light_num, sprintf(traffic_light_num,"!7SEG:%d%d#",led_buffer1[0],led_buffer1[1]), 100);
		/* UART TRANSMIT END */

		break;
	case MODE3:

		/* BLINK SINGLE YELLOW LED START */
//		if(timer1_flag==1){
//			setTimer1(250);
//			HAL_GPIO_TogglePin(GREEN_LED1_GPIO_Port, GREEN_LED1_Pin);  //TOGGLE RED LEDS IN 2 WAY
//			HAL_GPIO_TogglePin(GREEN_LED2_GPIO_Port, GREEN_LED2_Pin);
//		}
		/* BLINK SINGLE YELLOW LED END */


		/*EXECUTE INPUT BUTTON1 START */
		if(button1_executed==0){
			if(button1_pressed()==TRUE){
				state=MODE4;
				button1_executed=1;
				turnOffSingleLed();
			}
		}else if(button1_executed==1){
			if(button1_pressed()==FALSE){
				button1_executed=0;
			}
		}
		/*EXECUTE INPUT BUTTON1 END */

		/*EXECUTE INPUT BUTTON3 START */
		if(button3_pressed()==TRUE){
			state=UPDATE_DURATION;
			LedToChange=GREEN;
		}
		/*EXECUTE INPUT BUTTON3 END */

		/*EXECUTE INPUT BUTTON2 START */
		if(button2_pressed()==TRUE){
			state=INCREASE;
			LedToChange=GREEN;
			button2_increase1=0;
		}
		/*EXECUTE INPUT BUTTON2 END */
		//TO DO
		/* UART TRANSMIT START */
		if(LedToChange==GREEN){
			update_led_buffer(green_temp_dur);
		}
		HAL_UART_Transmit(huart, traffic_light_num, sprintf(traffic_light_num,"!7SEG:%d%d#",led_buffer1[0],led_buffer1[1]), 100);
		/* UART TRANSMIT END */
		break;
	case MODE4:
		/* BLINK SINGLE GREEN LED START */
//		if(timer1_flag==1){
//			setTimer1(250);
//			HAL_GPIO_TogglePin(YELLOW_LED1_GPIO_Port, YELLOW_LED1_Pin);  //TOGGLE RED LEDS IN 2 WAY
//			HAL_GPIO_TogglePin(YELLOW_LED2_GPIO_Port, YELLOW_LED2_Pin);
//		}
		/* BLINK SINGLE GREEN LED END */

		/*EXECUTE INPUT BUTTON1 START */
		if(button1_executed==0){
			if(button1_pressed()==TRUE){
				state=INIT;
				button1_executed=1;
				setTimer1(0); //turn off or reset timer 1
				turnOffSingleLed();
			}
		}else if(button1_executed==1){
			if(button1_pressed()==FALSE){
				button1_executed=0;
			}
		}
		/*EXECUTE INPUT BUTTON1 END */

		/*EXECUTE INPUT BUTTON3 START */
		if(button3_pressed()==TRUE){
			state=UPDATE_DURATION;
			LedToChange=YELLOW;
		}
		/*EXECUTE INPUT BUTTON3 END */

		/*EXECUTE INPUT BUTTON2 START */
		if(button2_pressed()==TRUE){
			state=INCREASE;
			LedToChange=YELLOW;
			button2_increase1=0;
		}
		/*EXECUTE INPUT BUTTON2 END */
		//TO DO
		/* UART TRANSMIT START */
		if(LedToChange==YELLOW){
			update_led_buffer(yellow_temp_dur);
		}
		HAL_UART_Transmit(huart, traffic_light_num, sprintf(traffic_light_num,"!7SEG:%d%d#",led_buffer1[0],led_buffer1[1]), 100);
		/* UART TRANSMIT END */
		break;
	case INCREASE:
		/* BLINK SINGLE LED START */
//		if(timer1_flag==1){
//			setTimer1(250);
//			ledBlinking(LedToChange);
//		}
		/* BLINK SINGLE LED END */

		/* EXECUTE INPUT BUTTON2 START */
		if(button2_pressed_1s()==TRUE){
			state=INCREASE_500MS;
			setTimer3(500);
		}else if(button2_pressed()==FALSE){
			switch(LedToChange){
				case RED:
					state=MODE2;
					break;
				case GREEN:
					state=MODE3;
					break;
				case YELLOW:
					state=MODE4;
					break;
			}
		}
		/* EXECUTE INPUT BUTTON2 START */

		/* INCREASE DURATION START */
		if(button2_increase1==0){
			increaseTempDur(LedToChange);
			button2_increase1=1;
		}
		/* INCREASE DURATION END */
		//TO DO
		break;
	case INCREASE_500MS:
		/* BLINK SINGLE LED START */
//		if(timer1_flag==1){
//			setTimer1(250);
//			ledBlinking(LedToChange);
//		}
//		/* BLINK SINGLE LED END */

		/* EXECUTE INPUT BUTTON2 START */
		if(button2_pressed()==FALSE){
			switch(LedToChange){
				case RED:
					state=MODE2;
					break;
				case GREEN:
					state=MODE3;
					break;
				case YELLOW:
					state=MODE4;
					break;
			}
		}
		/* EXECUTE INPUT BUTTON2 START */

		/* INCREASE TEMP DURATION START */
		if(timer3_flag==1){
			setTimer3(500);
			increaseTempDur(LedToChange);
		}
		/* INCREASE TEMP DURATION END */
		//TO DO
		/* UART TRANSMIT START */
		if(LedToChange==RED){
			update_led_buffer(red_temp_dur);
		}else if(LedToChange==GREEN){
			update_led_buffer(green_temp_dur);
		}else if(LedToChange==YELLOW){
			update_led_buffer(yellow_temp_dur);
		}
		HAL_UART_Transmit(huart, traffic_light_num, sprintf(traffic_light_num,"!7SEG:%d%d#",led_buffer1[0],led_buffer1[1]), 100);
		/* UART TRANSMIT END */
		break;
	case UPDATE_DURATION:
		updateDuration(LedToChange);
		if(button3_pressed()==FALSE){
			switch(LedToChange){
				case RED:
					state=MODE2;
					break;
				case GREEN:
					state=MODE3;
					break;
				case YELLOW:
					state=MODE4;
					break;
			}
		}
		//TO DO
		break;
	}
}


void increaseTempDur(enum LED color){
	switch(color){
		case RED:
			red_temp_dur++;
			if(red_temp_dur>99)
				red_temp_dur=1;
			break;
		case YELLOW:
			yellow_temp_dur++;
			if(yellow_temp_dur>99)
				yellow_temp_dur=1;
			break;
		case GREEN:
			green_temp_dur++;
			if(red_temp_dur>99)
				green_temp_dur=1;
			break;
	}
}

void updateDuration(enum LED color){
	switch(color){
		case RED:
			RED_DURATION=red_temp_dur;
			break;
		case YELLOW:
			YELLOW_DURATION=yellow_temp_dur;
			break;
		case GREEN:
			GREEN_DURATION=green_temp_dur;
			break;
	}
}



void displaySingleLedsMode1(){
	if(current_led_way1==RED){
		turnOnRed1();  // TURN ON RED LED
	}
	if(current_led_way1==GREEN){
		turnOnGreen1();  // TURN ON GREEN LED
	}
	if(current_led_way1==YELLOW){
		turnOnYellow1();  // TURN ON YELLOW LED
	}

	if(current_led_way2==RED){
		turnOnRed2();  // TURN ON RED LED
	}
	if(current_led_way2==GREEN){
		turnOnGreen2();  // TURN ON GREEN LED
	}
	if(current_led_way2==YELLOW){
		turnOnYellow2();  // TURN ON YELLOW LED
	}
}

void trafficLightCount(int*leds_count,enum LED*current_led){
	if(*current_led==RED){
		leds_count[0]--;
		if(leds_count[0]<=0){
			leds_count[0]=RED_DURATION;
			*current_led=GREEN;
		}
	}
	else if(*current_led==GREEN){
		leds_count[1]--;
		if(leds_count[1]<=0){
			leds_count[1]=GREEN_DURATION;
			*current_led=YELLOW;
		}
	}
	else if(*current_led==YELLOW){
		leds_count[2]--;
		if(leds_count[2]<=0){
			leds_count[2]=YELLOW_DURATION;
			*current_led=RED;
		}
	}
}

void turnOffSingleLed(){
	turnOff1();
	turnOff2();
}

//void ledBlinking(enum LED led_to_change){
//	switch(led_to_change){
//	case RED:
//		HAL_GPIO_TogglePin(RED_LED1_GPIO_Port, RED_LED1_Pin);  //TOGGLE RED LEDS IN 2 WAY
//		HAL_GPIO_TogglePin(RED_LED2_GPIO_Port, RED_LED2_Pin);
//		break;
//	case YELLOW:
//		HAL_GPIO_TogglePin(YELLOW_LED1_GPIO_Port, YELLOW_LED1_Pin);  //TOGGLE RED LEDS IN 2 WAY
//		HAL_GPIO_TogglePin(YELLOW_LED2_GPIO_Port, YELLOW_LED2_Pin);
//		break;
//	case GREEN:
//		HAL_GPIO_TogglePin(GREEN_LED1_GPIO_Port, GREEN_LED1_Pin);  //TOGGLE RED LEDS IN 2 WAY
//		HAL_GPIO_TogglePin(GREEN_LED2_GPIO_Port, GREEN_LED2_Pin);
//		break;
//	}
//}

void PES_LED_ON(){
	HAL_GPIO_WritePin(D6_GPIO_Port, D6_Pin, 0);
	HAL_GPIO_WritePin(D7_GPIO_Port, D7_Pin, 1);
}
void PES_LED_OFF(){
	HAL_GPIO_WritePin(D6_GPIO_Port, D6_Pin, 1);
	HAL_GPIO_WritePin(D7_GPIO_Port, D7_Pin, 0);
}
void turnOnGreen1(){
	HAL_GPIO_WritePin(D2_GPIO_Port, D2_Pin, 0);
	HAL_GPIO_WritePin(D3_GPIO_Port, D3_Pin, 1);
}
void turnOnRed1(){
	HAL_GPIO_WritePin(D2_GPIO_Port, D2_Pin, 1);
	HAL_GPIO_WritePin(D3_GPIO_Port, D3_Pin, 0);
}
void turnOnYellow1(){
	HAL_GPIO_WritePin(D2_GPIO_Port, D2_Pin, 1);
	HAL_GPIO_WritePin(D3_GPIO_Port, D3_Pin, 1);
}
void turnOff1(){
	HAL_GPIO_WritePin(D2_GPIO_Port, D2_Pin, 0);
	HAL_GPIO_WritePin(D3_GPIO_Port, D3_Pin, 0);
}
void turnOnGreen2(){
	HAL_GPIO_WritePin(D4_GPIO_Port, D4_Pin, 0);
	HAL_GPIO_WritePin(D5_GPIO_Port, D5_Pin, 1);
}
void turnOnRed2(){
	HAL_GPIO_WritePin(D4_GPIO_Port, D4_Pin, 1);
	HAL_GPIO_WritePin(D5_GPIO_Port, D5_Pin, 0);
}
void turnOnYellow2(){
	HAL_GPIO_WritePin(D4_GPIO_Port, D4_Pin, 1);
	HAL_GPIO_WritePin(D5_GPIO_Port, D5_Pin, 1);
}
void turnOff2(){
	HAL_GPIO_WritePin(D4_GPIO_Port, D4_Pin, 0);
	HAL_GPIO_WritePin(D5_GPIO_Port, D5_Pin, 0);
}
void update_led_buffer(int val){
	led_buffer1[0]=val/10;
	led_buffer1[1]=val%10;
}
int toggle_state=0;


