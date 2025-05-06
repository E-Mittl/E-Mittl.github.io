---
title: Resources
---

The code and MPLabX resources for this project can be found as a .zip file [here](ZIP/Individual_Subsystem_Verification.zip)

## Code
```
#include "mcc_generated_files/system/system.h"
#include <math.h>
#include <stdlib.h>
#include <stdio.h>

#define BUFSIZE 16
#define MSGSIZE 64
#define TEAMSIZE 5
#define TYPESIZE 4
#define MSGTESTSIZE 64
#define MSGTESTCHAR 0

#define OPT4048_ADDR 0x45 //0b1000101

//Control register for startup
#define CTRL_REG 0x0A // CONFIG REGISTER
#define RANGE 0x0C // AUTO SCALE
#define CONV_TIME 0x07 //50 MS CONVERSION TIME
#define OP_MODE_CONT 0x03 // CONTINUOUS MODE
#define LATCH_MODE 0x00 // TRANSPARENT HYSTERESIS MODE
#define INT_POL 0x00 // INT PIN ACTIVE LOW
#define FAULT_COUNT 0x02 

const char id = 'S';
const char team_id[TEAMSIZE + 1] = "MASHX";
const uint8_t message_types[TYPESIZE] = {0x00, 0x01,0x02,0x03};
const uint8_t status[TYPESIZE] = {0x00,0x01,0x02,0x03};
const uint8_t drive[TYPESIZE] = {0x00,0x01};
const uint8_t sens[TYPESIZE] = {0x00,0x01,0x02,0x03};
const uint8_t dir[TYPESIZE] = {0x00,0x01,0x02,0x03};
char buffer_in[BUFSIZE + 1];
char message_in[MSGSIZE + 1];
char message_out[MSGSIZE+1];

uint8_t val;

float timeCurrent = 0.0;
float timeLast = 0.0;
float timeHeldStart = 0.0;
int secHeld = 0;
int time_ms = 0;
int time_s = 0;
int i = 0;

float timeDown = 0.0;
float timeUp = 0.0;

int SWCurrent;
int SWLast;
int lastSecondBlink;
float debounceDelay = 0.150;

// FOR BLINKING FUNCTION
unsigned int blinkCount = 0;
unsigned int blinkTarget = 0;
unsigned long lastBlinkTime = 0;
bool ledState = false;
bool blinking = false;

void fill_string(char * mystring, char value, unsigned int size){
    for(int ii=0;ii<size;ii++){
        mystring[ii]=value;
    }
}


unsigned int find_char(const char * mystring, char value, unsigned int size){
    char c = 0;
    for(int ii = 0; ii<size; ii++){
        c = mystring[ii];
        //printf("%c,%c; ",value, mystring[ii]);
        if(c==value){
            return 1;
        }
    }
    return 0;
}

void test_message(char type, char val){
    if(message_in[4] == type && message_in[5] == val){
        for(i = 0; i < type; i++){
            GLED_SetHigh();
            __delay_ms(200);
            GLED_SetLow();
            __delay_ms(200);
        }
        for(i = 0; i < val; i++){
            RLED_SetHigh();
            __delay_ms(200);
            RLED_SetLow();
            __delay_ms(200);
        }
    }
}
void sens_setup(void){
    uint8_t controlData[3];
    uint16_t controlWord = 0;

    controlWord |= (RANGE << 10);        // Bits 13:10
    controlWord |= (CONV_TIME << 6);     // Bits 9:6
    controlWord |= (OP_MODE_CONT << 4);  // Bits 5:4
    controlWord |= (LATCH_MODE << 3);    // Bit 3
    controlWord |= (INT_POL << 2);       // Bit 2
    controlWord |= (FAULT_COUNT);        // Bits 1:0

    controlData[0] = CTRL_REG;               // 0x0A
    controlData[1] = (controlWord >> 8) & 0xFF; // MSB
    controlData[2] = (controlWord & 0xFF);      // LSB
    WLED_SetHigh();
    __delay_ms(300);
    I2C1_Write(OPT4048_ADDR, controlData, 3);
    __delay_ms(300);
    WLED_SetLow();
}
uint32_t sensorRead(char c) {
    uint8_t reg = 0x00;
    uint8_t msb = 0, lsb = 0;
    uint8_t exp = 0;
    uint16_t mantissa = 0;
    uint32_t finalADC = 0;

    // Choose channel
    if (c == 'r') reg = 0x00;  // CH0 - Clear
    else if (c == 'g') reg = 0x02; // CH1 - Red
    else if (c == 'b') reg = 0x04; // CH2 - Green
    else if (c == 'w') reg = 0x06; // CH3 - Blue
    else return 0; // Invalid channel

    // Read MSB (reg), then LSB (reg + 1)
    I2C1_WriteRead(OPT4048_ADDR, &reg, 1, &msb, 1);
    __delay_ms(65);
    reg += 1;
    I2C1_WriteRead(OPT4048_ADDR, &reg, 1, &lsb, 1);
    __delay_ms(65);

    // Extract exponent and mantissa
    exp = (msb >> 4) & 0x0F;
    mantissa = ((msb & 0x0F) << 8) | lsb; // 12-bit mantissa

    // Scale using exponent
    finalADC = ((uint32_t)mantissa) << exp;

    return finalADC;
}
void sens_init(){
    printf("AZAH%c%cYB",0x00,0x00);
    for(i = 0; i <= 5;i++){
        RLED_SetHigh();
        __delay_ms(200);
        RLED_SetLow();
        __delay_ms(200);
    }
    WLED_SetHigh();
    __delay_ms(10);
    uint32_t RVal = (sensorRead('r')*255)/120000;
    __delay_ms(10);
    uint32_t GVal = (sensorRead('g')*255)/220000;

    float R2G = ((float)RVal / (float)GVal) * 100.0;
    if( R2G > 100.0){
        printf("AZSX%c%cYB",0x02,0x00);
    }
    else if(R2G < 100.0 && R2G >= 85.0){
        printf("AZSX%c%cYB",0x02,0x01);
    }
    else if(R2G < 85.0){
        printf("AZSX%c%cYB",0x02,0x02);
    }
    WLED_SetLow();
}

void eusart_callback(void){
    //ms++;
}

// Nonblocking code to keep track of time
void timer_callback(){
        time_ms = time_ms + 1;
        if(time_ms >= 1000){
            time_ms = time_ms - 1000;
            time_s = time_s + 1;
    }
}
void pin_down(){  
}
void getColor(){
    WLED_SetHigh();
    __delay_ms(300);
    uint32_t RVal = (sensorRead('r')*255)/120000;
    __delay_ms(75);
    uint32_t GVal = (sensorRead('g')*255)/220000;
    __delay_ms(750);
    float R2G = ((float)RVal / (float)GVal) * 100.0;
    if( R2G > 120.0){
        RLED_SetHigh();
        __delay_ms(1000);
        RLED_SetLow();
        printf("AZSH%c%cYB",0x02,0x00);
    }
    else if(R2G < 120.0 && R2G >= 85.0){
        RLED_SetHigh();
        GLED_SetHigh();
        __delay_ms(1000);
        RLED_SetLow();
        GLED_SetLow();
        printf("AZSH%c%cYB",0x02,0x01);
    }
    else if(R2G < 55.0){
        GLED_SetHigh();
        __delay_ms(1000);
        GLED_SetLow();
        printf("AZSH%c%cYB",0x02,0x02);
    }
    WLED_SetLow();
}
void handleMessage(){
    char sender = message_in[2];
    char receiver = message_in[3];
    char type = message_in[4];
    char data = message_in[5];
    
    if(find_char(team_id,sender,TEAMSIZE) == 0 || sender == 'S'){
        return;
    }
    if(find_char(team_id,receiver,TEAMSIZE) == 1){
        if(receiver == 'S'){
            if(find_char(message_types,type,TYPESIZE) == 1){
                if(type == 0x00){
                    if(data == 0x00){
                        printf("AZSH%c%cYB",0x00,0x00);
                        RLED_SetLow();
                        for(int i = 0; i < 5; i++){
                            GLED_SetHigh();
                            __delay_ms(250);
                            GLED_SetLow();
                            __delay_ms(250);
                        }
                        getColor();
                        }
                        else{
                            return;
                        }
                }
                if(type == 0x03){
                    getColor();
                }
            }
        }
        if(receiver == 'X'){
            if(find_char(message_types,type,TYPESIZE) == 1){
                printf("AZ%c%c%c%cYB",sender,receiver,type,data);
                if(type == 0x00){
                    if(data == 0x00){
                        printf("AZSH%c%cYB",0x00,0x00);
                        RLED_SetLow();
                        for(int i = 0; i < 5; i++){
                            GLED_SetHigh();
                            __delay_ms(250);
                            GLED_SetLow();
                            __delay_ms(250);
                        }
                        getColor();
                        return;
                        }
                        else{
                            return;
                        }
                }
                if(type == 0x03){
                    getColor();
                }
            }
        }
        else if(receiver == 'A' || receiver == 'M' || receiver == 'H'){
            printf("AZ%c%c%c%cYB",sender,receiver,type,data);
            return;
        }
        else{
            return;
        }
        
    }
    
    
}
void buttonCheck(){

        timeCurrent = time_s + time_ms/1000.0; // Calculate current time
        
        SWCurrent = SW_GetValue();
        
        if(SWCurrent != SWLast){
            if(fabs(timeCurrent - timeLast) > debounceDelay){
                GLED_SetHigh();
                printf("AZSX%c%cYB",0x01, 0x00);
            }
                
      }
        SWLast = SWCurrent;
        timeLast = time_s + time_ms/1000.0;       
}



int main(void)
{
    SYSTEM_Initialize();
    INTERRUPT_GlobalInterruptEnable();    
    
    
    UART1_Initialize();
    UART1_Enable();
    
    char c=0;
    char sender;
    char receiver;
    char type;
    char subtype;
    unsigned int pass = 0;
    unsigned int buffer_ii = 0;
    unsigned int buffer_last_ii = 0;
    unsigned int message_ii = 0;
    unsigned int message_last_ii = 0;
    buffer_in[BUFSIZE] = 0;
    message_in[MSGSIZE] = 0;
    unsigned int message_incoming = 0;
    fill_string(buffer_in, 'a', BUFSIZE);
    fill_string(message_in,'_',MSGSIZE);
    message_in[MSGTESTSIZE] = MSGTESTCHAR;
    sens_setup();
    //buttonWait();
    RLED_SetHigh();
    UART1_RxCompleteCallbackRegister(eusart_callback);
    SW_SetInterruptHandler(pin_down);
    TMR0_PeriodMatchCallbackRegister(timer_callback);
    TMR0_Start();
    while(1){
        if(UART1_IsRxReady())
            {
            
                c = UART1_Read();
                buffer_in[buffer_ii]=c;
                
                // If AZ is detected - start recording
                if (buffer_in[buffer_last_ii]=='A' && buffer_in[buffer_ii]=='Z'){
                    fill_string(message_in,'_',MSGSIZE);
                    message_in[MSGTESTSIZE]=MSGTESTCHAR;
                    message_incoming=1;
                    message_in[0] = buffer_in[buffer_last_ii]; // Set A to position 0 in message array
                    message_ii=1;
                    //RLED_SetHigh();
                    //GLED_SetLow();
                }
                
                // If YB is detected - end recording
                if (buffer_in[buffer_last_ii]=='Y' && buffer_in[buffer_ii]=='B'){
                    message_incoming=0;
                    message_in[message_ii] = buffer_in[buffer_ii];
                    message_in[message_ii + 1] = '\0'; 
                    message_last_ii= message_ii;
                    message_ii = message_ii+1;
                    handleMessage();
                    //GLED_SetHigh();
                    //RLED_SetLow();
                }
                if (message_incoming != 0){
                    message_in[message_ii] = buffer_in[buffer_ii];
                    
                    // Determine if message is too long
                    message_last_ii= message_ii;
                    message_ii = message_ii+1;
                    if (message_ii<MSGTESTSIZE){} else{
                        message_incoming=0;
                        message_ii=0;
                    }
                }
                buffer_last_ii= buffer_ii;
                buffer_ii = (buffer_ii+1)%BUFSIZE;
                
            }
        buttonCheck();
        }
}
```