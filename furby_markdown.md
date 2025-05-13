# TITLE PAGE 

INTERACTIVE TOY
(FURBY.ASM - Version 25)

INVENTOR: Dave Hampton

Attorney Docket No. 64799
FITCH, EVEN, TABIN \\& FLANNERY
Suite 900
135 South LaSalle Street
Chicago, Illinois 60603-4277
Telephone (312) 372-7842
      ```
;EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII!
;*
;* SPC81A Source Code (Version 25)
;*
;* Written by: Dave Hampton / Wr & Schulz
;* Date: July 30, 1998
;*
;* Copyright (C) 1996,1997,1998 by Sounds Amazing!
;* All rights reserved.
;EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII!
;*
;***************************************************************
; remember SBC if there is a borrow carry is CLEARED
; also SBC if the two numbers are equal you still get a negative
result
;
;EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII;
;* MODIFICATION LIST :
;
; Furby29/30/31/32
    Final testing for shipment of code on 8/2/98.
    Tables updated for speed updated, wake up/name fix
    sequential tab...s never getting first entry.fixed.
    New diag5.asm, Light3.asm (if light osc stalls it wont hang
system).
;
; Furby33
    In motor brake routine, turn motors off before turning reverse
    braking pulse on to save transistors.
;
; Furby34
    Cleanup start code and wake routines.
    Light sensor goes max dark and stays there to reff time, then
    call sleep macro and shut down.
;
; Furby35
    Adds four new easter eggs,BURP ATTACK, SAY NAME, TWINKLE SONG,
    and ROOSTER LOVES YOU. Also add new names.
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
;
      Release 3

: File \"testR3a\"
: 1. Light sensor has a hysteresis point of continually triggering sensor.
: 2. Light sensor decrements two instead of one on hungry counter.
: 3. Diagnos ' - de for light sensor wont trigger very easily.
: 4. When a furpy eceives the I.R. sleep command he sends the same command
: out before goi. - to sleep.
: 5. When hungry is ow enough to trigger sick counter, each sensor deducts two instead of one for each hit.
: 6. When diagnostics complete c1ear memory, reset hungry \\& sick to FF randomly choose new name an voice, then write EEPROM before going to sleep. Also extend EEPROM diagnostic to test all locations for pass/fail of device.
: 7. Add new light routine
: 8. Change hide and seek egg to light, light, light, tummy.
: 9. Change sick/hungry counter so that it can only get so sick and not continue down to zero. (MAX_SICK)
:10. In diagnostics, motor position test ,, , first goes forward continuously until the front switch is pressed, then goes reverse continuously until the front switch is pressed again, and then does normal position calibration stopping at the calibration switch.
:11. On power up we still use tilt and invert to generate startup random numbers, but if feed switch is pressed for cold boot, we use it to generate random numbers, because it is controlled by the user where the tilt and invert are more flaky.
:12. No matter what age, 25% of time he randomly pulls speech from age to generate more purbish at older ages.
:13. Twinkle song egg
: When song is complete, if both front and back switches are pressed we goto deep sleep. That means only the invert can wake us up, not the tilt switch.
      Actual numeric value for TI pitch control

```
bit 7 set = subtract value from current course value
    clr = add value to cur ent course value
bit 6 set = select music pitch table
    clr = select normal speech pitch table
bit 0-5 value to change course value (no change = 0)
A math routine in 'say_0' converts the value for + or -
if <80 then subtracts from 80 to get the minus version of 00
ie, if numbur is 70 then TI gets sent 10 (which is -10)
If number is 80 or > 80 then get sent literal as positive.
NOTE: MAX POSITIVE IS 8F (+16 from normal voice of 00)
    MAX NEGATIVE is 2F (-47 from normal voice of 00)
This is a difference of 80h - 2Fh or 51h
8Fh is hi voice (8f is very squeeeeeke)
2Fh lo voice ( very low)
The math routine in 'say_0' allows a +-decimal number in the speech
table.
A value of 80 = no change or 00 sent to TI
: 81 = +1
: 8f = +16
; 2value of 7F = -1 from normal voice
:70 = -16
The voice selection should take into consideration that the hi voice
selection plus an aditional offset is never greater than 8f
Or a low voice minus offset never less than 2f.
```

| Vol | EQU | 83h | ; $(+3)$ hi voice |
| :--: | :--: | :--: | :--: |
|  |  |  |  |
|  | EQU | 7Ah | ; (-6) mid voice |
|  |  |  |  |
|  | EQU | 71h | ; (-15) low voice |

; ; ; we converted to a random selection table, but since all voice tables
use the equate= plus some offset, we : . the change in the SAY_0
routine. We always assign voice 3 which is the lowest, and based on
the random power up pitch selection, the ram location 'Rvoice'
holds
the number to add to the voice-offset received from the macro table.

Voice EQU Voice3 ;pitch (choose Voice1, Voice2,
Voice3) (voice2=norm)
; Select Voice3 since it is the lowest and then add the difference to get
; Voice2 or Voice3. Here we assign that difference to an equate to be
; used in the voice table that is randomly selected on power up.
S_voice1 EQU 18 ;Voice3 + 18d = Voice1
S_voice2 EQU 09 ;Voice3 + 09d = Voice2
      S_voice3 EQU 0 ;Voice3 + 00d = Voice3
; Motor speed pulse width ;
; Motor_on = power to motor, Motor_off is none.

Mpulse_on EQU 16 ;
Mpulse_off EQU 16 ;

Cal_pos_fwd EQU 134 ;calibration switch forward direction
Cal_pos_rev EQU 134 ;calibration switch forward direction
; **********************************************************
; **********************************************************
; **********************************************************
; UAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
      ```
;* BANKO user RC $0600 - $7FFA.
;* BANK1 user RC $8000 - $FFFF
;* BANK2 user RC $10000 - $17FFF
;* BANK3 user RC $1A000 - $1FFFF
;*
    VECTORS
;* NMI vector $7FFA / $7FFB
;* RESET vector $7FFC / $7FFD
;* IRQ vector $7FFE / $7FFF
;AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
      

;AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
;AAAAAAAAAAAAAAAAAAAAAAA DATA LATCH PORT_D
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
      ```
Bit3: IE1 A: IE1= 0: Counter clock= external clock from IOC2
Bit2: T1 A = 1, T1= 0: counter clock= CPUCLK/8192
Bit1: IE0 A' T1= 1: counter clock= CPUCLK/6553u
Bit0: T0 AO IE0= 0: Counter clock= external clock from IOC2
                = 1, T0= 0: counter clock= CPUCLK/4
                T0= 1: counter clock= CPUCLK/64
;AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
      ```
    Bit7: I/O 0: Disable ADC; 1: Enable ADC
    Bit6: I/O
    Bit5: I/O
    Bit4: I/O
    Bit3: I/O
    Bit2: I/O
    Bit1: I/O
    Bit0: I/O
;AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
      ; Send a braking pulse to stop motor drift, and this EQU is a decimal number
: that determines how many times through the 2.9 msec loop (how many loops)
; the brake pulse is on. If attempting to make single count jumps, the
; brake pulse needs to be between 26 and 30 . For any jump greater than 10
; braking between 22 and 80 is acceptable. ( Long jumps are not critical
; but short jump will begin to oscillate if braking is too great.)
; 60 long \\& 20 short work at 3.6 v and no pulse width
Drift_long EQU 60 ; number times thru intt before clearing pulse Drift_shect EQU 25 ;
; *****************************************************************
; set this with a number from 0 - 255 to determine timeout of all sensors
; for the sequential increments. If it times out the table pointer ; goes back to the start, else each trigger increments through the table.
; NOTE: this time includes the motor/speech execution time !!!
Global_time EQU $16 ; 1=742 \\mathrm{mSEC} ; ; 255=18.3$ seconds
; *****************************************************************
; This determines how long Firby waits with no sensor activity, then
; calls the Bored_table for a random speech selection.
; Us a number between 1 \\& 255. Should probably not be less than 10.
; SHOULD BE > 10 SEC TO ALLOW TIME FOR TRAINING OF SENSORS
Bored_seld EQU $40 ; 1=742 \\mathrm{mSEC} ; ; 255=189.3$ seconds
; *****************************************************************
; Each sensor has a sequential random sp. : which must equal 16.
; Each sensor has a different assignment.
; The tables are formatted with the first X assignments random
; and the remaining as sequential.
Seq_front EQU 8
Ran_front EQU 8
Seq_back EQU 9
Ran_back EQU 7
Seq_tilt EQU 10
Ran_tilt EQU 6
Seq_invert EQU 8
Ran_invert EQU 8
Seq sound EQU 0
Ran_sound EQU 16
      | Seq_light | EQU | 0 |
| :-- | :-- | :-- |
| Ran_light | EQU | 16 |
| Seq_feed | EQU | 8 |
| Ran_feed | EQU | 8 |
| Seq_wake | EQU | 0 |
| Ran_wake | EQU | 16 |
| Seq_bored | EQU | 7 |
| Ran_bored | EQU | 9 |
| Seq_hunger | EQU | 5 |
| Ran_hunger | EQU | 11 |
| Seq_sick | EQU | 4 |
| Ran_sick | EQU | 12 |

; rev furbl1ja
; Each sensor also determines how often it is random or sequential
; as in $50 / 50$ or $60 / 40$ etc.
; These entries are subtracted from the random nurber generated
; and determine the split. (the larger here, the more likely sequential pick)

| Tilt_split | EQU | 80 h |  |
| :-- | :-- | :-- | :-- |
| Invert_split |  | EQU | 80 h |
| Front_split EQU | 80 h |  |  |
| Back_split EQU | 80 h |  |  |
| Feed_split EQU | 80 h |  |  |
| Sound_split EQU | 80 h |  |  |
| Light_split EQU | 80 h |  |  |
| Bored_split EQU | 80 h |  |  |
| Hunger_split |  | EQU | 80 h |
| Sick_split EQU | 80 h |  |  |


| Random_age EQU 30 h ; at any age, below this number when a random number is picked will cause him to pull from the age 1 table. More Furbish.

Learn_chg EQU 31 ; amount to inc or dec training of words
Food EQU 20 h ; amount to increase 'Hungry' for each feeding
Need_food EQU 80 h ; below this starts complaining about hunger
Sick_reff EQU 60 h ; below this starts complaining about sickness
Real1y_sick EQU 20 h ; below this only complains about sickness
Max_sick EQU 80 h ; cant go below this when really sick
Hungry_dec EQU 01 ; subtract $X$ amount for each sensor trigger
Sick_dec EQU 01 ; subtract $X$ amount for each sensor trigger
Nt_word EQU FEH ; turn speech word active off
Nt_last EQU FBH ; bit 2 off - last word sent to TI
      | Notor_led_timer | EQU | A0h | ;how long after motion done led on for IR |
| :--: | :--: | :--: | :--: |
| Mot_speed_cnt | EQU | Alh | ; motor speed test |
| Mot_optc_cnt | EQU | A2h | ; * |
| Cal_switch_cnt | EQU | A3h | ;used to eliminate noisy reads |
| motorstoped eq | A4h | ; times wheel count when stopping |  |
| Drift_counter | EQU | A5h | ; decides how much braking pulse to apply |
| Mili_sec EQU | A6h | ;used in calc pot position by timer |  |
| Cycle_timer EQU | A7h | ; bypasses intt port c updates to motor |  |
| Sensor_timer | EQU | A8h | ;times between sensor trigger |
| Bored_timer EQU | A9h | ; time with no activity to random speech |  |
| Invrt_count EQU | AAh | ; which speech/motor call is next |  |
| Tilt_count EQU | ABh | ; which speech/motor call is next |  |
| Tchfrnt_count | EQU | ACh | ; which speech/motor call is next |
| Tchbck_count | EQU | ADh | ; which speech/motor call is next |
| Feed_count EQU | AEh | ; which speech/motor call is next |  |
| Last_IR | EQU | AFh | ; last IR sample data to compare to next |
| Wait_time EQU | B0h | ;used in IRQ to create 2.8 mSec timers |  |
| Light_timer EQU | B1h | ; Light sensu. routine |  |
| Light_count EQU | B2h | ; which speech/motor call is next |  |
| Light_reff EQU | B3h | ; holds previous sample |  |
| Sound_timer EQU | B4h | ; time to set new reff level |  |
| Sound_count EQU | B5h | ; which speech/motor call is next |  |
| Milisec_flag | EQU | B6h | ; set every 742 miliseconds |
| Macro_Lo | E7U | ; table pointer |  |
| Macro_Hi EQU | B8h | ; | * |
| Egg_cnt | EQU | B9h | ; easter egg table count pointer |
| ; *************** | * | K | ; Eo |


| NCEL_Lo | EQU | BAh | ; |
| :-- | :-- | :-- | :-- |
| NCEL_HI | EQU | BBh | ; |
| BIT_CT | EQU | BCh | ; |

;
| Prev_random EQU | BEh | ; prevents random number twice in a row |
| :-- | :-- | :-- |
| Bored_count EQU | BPh | ; sequential selection for bored table |
| TEMP5 EQU | C0h | ; general use also used for wake up |
| Temp_ID2 EQU | C1h | ; use in sensor training routines |
| Temp_ID | EQU | C2h ; use in sensor training routines |
| Learn_temp EQU | C3h | ; use in sensor training routines |
|  |  |  |
| Req_macro_lo | EQU | C4h ; holds last call to see if sleep or IR req |
| Req_macro_hi | EQU | C5h ; |
|  |  |  |
| Sickr_count EQU | C6h | ; sequential counter for sick speech table |
| Hungr_count EQU | C7h | ; sequential counter for hunger speech table |
      | \\begin{tabular}{l} 
Motor_pulse 2 \\\\
\\hline | EQU | C8h | :motor pulse timer \\\\
\\hline \\end{tabular} |  |
| :--: | :--: | :--: | :--: | :--: |
|  |  |  |  |  |
| Stat_0 |  | Equ | C9h | : System status |
| Want_name | EQU | 01 H | ; bit | $0=$ set forces system to say Furpy's name |
| Lt_prev_dn | EQU | 02 H | ; bit | $0=$ done flag for quick light changes |
| Init_motor | EQU | 04 H | ; bit | $1=$ on wakeup do motor speed/batt test |
| Init_Mspeed | EQU | 08 H | ; bit | $3=2 \\mathrm{nd}$ part of motor speed test |
| Train_Bk_prev |  | EQU | 10 H | ; bit $4=$ set when 2 back sw hit in a row |
| Sav_new_name |  | EQU | 20 H | ; bit $5=$ only happens on cold boot |
| REV_dark_sleep |  | EQU | 40 H | ; bit $6=$ set -dark level sends to sleep |
| Dark_sleep_prev |  | EQU | 80 H | ; bit $7=$ if set on wake up thend |


| \\begin{tabular}{l} 
Stat_1 \\\\
Word_activ \\\\
Say_activ \\\\
Word_end \\\\
Word_term \\\\
Up_light \\\\
Snd_reff \\\\
Half_age \\\\
Random_sel \\\\
\\hline \\end{tabular} 
\\begin{tabular}{l} 
Stat_2 \\\\
Motor_actv \\\\
Motor_fwd \\\\
Motor_seek \\\\
Bside_dn \\\\
Binvrt_dn \\\\
Tchft_dn \\\\
Tchbk_dn \\\\
Macro_actv \\\\
\\hline \\end{tabular} 
\\begin{tabular}{l} 
Motor_activ \\\\
Motor_fwd \\\\
Motor_seek \\\\
Bside_dn \\\\
Binvrt_dn \\\\
Tchft_dn \\\\
Tchbk_dn \\\\
Macro_actv \\\\
\\hline \\end{tabular} 
\\begin{tabular}{l} 
Motor_activ \\\\
Motor_fwd \\\\
Motor_seek \\\\
Bside_dn \\\\
Binvrt_dn \\\\
Tchft_dn \\\\
Tchbk_dn \\\\
Macro_actv \\\\
\\hline \\end{tabular} 
\\begin{tabular}{l} 
Stat_3 \\\\
Light_stat \\\\
Feed_dn \\\\
Sound_stat \\\\
IRQ_dn \\\\
Lt_reff \\\\
Motor_on \\\\
M_forward \\\\
M_reverse \\\\
\\hline \\end{tabular} 
\\begin{tabular}{l} 
Stat_4 \\\\
Do_end \\\\
Do_light_brt \\\\
Do_light_dim \\\\
Do_tummy \\\\
Do_back
\\end{tabular} 
\\begin{tabular}{l} 
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\
EQU \\\\

      ; is 1-16 for the sesnor table macro list and a ram for count which ; determines how often to call the learned word.
; *** DO NOT CHANGE ORDER----- RAM adrs by Xreg offset

| Tilt_learned | EQU | DAh | ; which word trained | 1 |
| :--: | :--: | :--: | :--: | :--: |
| Tilt_lrn_cnt | EQU | DBh | ; count determines how often called | 2 |
| Feed_learned | EQU | DCh | ; which word trained | 3 |
| Feed_lrn_cnt | EQU | DSh | ; count determines how often called | 4 |
| Light_learned | EQU | DEh | ; which word trained | 5 |
| Light_lrn_cnt | EQU | DFh | ; count determine now often called | 6 |
| Dark_learned | EQU | E0h | ; which word trained | 7 |
| Dark_lrn_cnt | EQU | E1h | ; count determines how often called | 8 |
| Front_learned | EQU | E2h | ; which word trained | 9 |
| Front_lrn_cnt | EQU | E3h | ; count determines how often called | 10 |
| Sound_learned | EQU | E4h | ; which word trained | 11 |
| Sound_lrn_cnt | EQU | E5h | ; count determines how often called | 12 |
| Wake_learned | EQU | E6h | ; which word trained | 13 |
| Wake_lrn_cnt | EQU | E7h | ; count determines how often called | 14 |
| Invert_learned | EQU | E8h | ; which word trained | 15 |
| Invert_lrn_cnt | EQU | E9h | ; count determines how often called | 16 |

; next is equates defining which ram to use for each sensor
; according to the sensor ram defined above. (compare to numbers above)

; ********* Need to allow stack growth down ( EAh- FFH ) *********
      

ORG 0600 H
RESET:

Include
Wake2.asm ;asm file
;******** end Tracker
; For power on test, WE only clear ram to E9h and use EAh for a
; messenger to the warm boot routine. We always clear ram and initialize
; registers on power up, but if it is a warm boot then read E\"PRCM
; and setup ram locations. Location EAH is set or cleared duri. ; power
up
; and then the stack can use it during normal run.
; Clear RAM to 00 H
LDA $\\quad \\# 00 \\mathrm{H} \\quad$; data for fill
LDX $\\quad \\# \\mathrm{E} 9 \\mathrm{H} \\quad$; start at ram location
RAMClear:
STA $00, \\mathrm{X} ;$ base 00, offset x
DEX ; next ram location
CPX \\#7FH ; check for end
BNE RAMClear ; branch, not finished
; fill done
      Main:

| Initio: |  |  |
| :--: | :--: | :--: |
| LDA | \\#01 | :turn DAC on |
| STA | DAC_ctrl | : DAC control |
| LDA | \\#Port_def | : set direction control |
| STA | Ports_dir | :load reg |
| LDA | \\#Con_def | : set configuration |
| STA | Ports_con | :load reg |
| LDA | \\#00 | : set for bank 0 |
| STA | Bank | : set it |
| LDA | \\#00H | :disable wakeup control |
| STA | Wake_up | : |
| LDA | \\#00h | :disable sleep control |
| STA | Sleep | : set dont care |
| LDA | \\#Intt_df1t | : Initialize timers, etc. |
| STA | Interrupts | :load reg |
| LDA | \\#00H | : set timer mode |
| STA | TMA_CON | : set reg |
| LDA | \\#TimeA_low | :get preset timer for interrupts |
| STA | TMA_LSB | :load |
| LDA | \\#TimeA_hi | : get hi byte for preset |
| STA | TMA_MSB | :load it |
| LDA | \\#TimeB_low | : get preset timer for interrupts |
| STA | TMB_LSB | :load |
| LDA | \\#TimeB_hi | : get hi byte for preset |
| STA | TMB_MSB | :load it |
| LDA | \\#C0h | : preset status for motors off |
| STA | Stat_3 | : |
| LDA | \\#00H | : init ports |
| STA | Port_A | : output |
| LDA | \\#33H | : init ports |
| STA | Port_B_Image | : ram image |
| STA | Port_B | : output |
| LDA | \\#0FH | : init ports |
| STA | Port_C | : output |
| LDA | \\#DOH | : init ports |
| STA | Port_D_Image | : ram image |
| STA | Port_D | : output |
| LDA | \\#FFh | : milisec timer reload value |
| STA | Mili_sec | : also preset IRC timer |
| CLI |  | : Enable IRQ |
      JER Kick JRQ ; wait for internust to restart
JSR TI_reset ; go init TI (uses 'Cycle_timer')
: Preset motor speed, assuming mid battey life, we set the pulse width so that the motor wont be running at 6 volts and burn out. We then predict what the pulse width should be for any voltage.

LDA \\#Mpulse_on ; preset motor speed
LDA \\#11
STA Mon_len ; set motor on pulse timing
LDA \\#05
STA Moff_len ; set motor off pulse timing
:EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
: \"Diagnostics and calibration Routine
:EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
Include
Diag7.asm ; asm file
: ***** Only called by diagnostic speech routines ********
: Be sure to set 'MACRO_HI' and all calls are in that 128 byte block.
Diag macro:
STA Macro_Lo ; save lo byte of Macro table entry
LDA \\#0b8h ; \\#90h ; \\#ex offset to adrs. 400 added
to diag call
CLC
ADC Macro_Lo ; add in offset
STA Macro_Lo ; update
LDA \\#01 ; get hi byte ours $400=190 \\mathrm{~h}$
STA Macro_Hi ; save hi byte of Macro table entry
JSR Get macro ; go start motor/speech
JSR Notrdy ; Do / get status for speech and motor
RTS ;yo!
: Enter with Areg holding how many 30 mili second delay cycles
Half delay:
STA TEMPI ; save timer
Half d2:
LDA \\#10 ; set $1 / 2$ sec
(y * 2.9 mSec )
STA Cycle_timer ; set it
Half_d3:
LDA Cycle_timer ; ck if done
BNE Half_d3 ; loop
DEC TEMPI ; ; loop
BNE Half_d2 ; loop
RTS ; done
      Test_byp: ;We assume diagnostic only runs on coldboot

| LDA | \\#FFh | ; initialize word training variable |
| :-- | :-- | :-- |
| STA | Temp_ID |  |
| LDA | \\#FFh | ; |
| STA | Hungry_counter | ; preset furby's health |
| STA | Sick_counter |  |

; We sit here and wait for tilt to go away, and just keep incrementing
; countar until it does. This becomes the new random generator seed.
Init_rnd:
INC TEMP1 ; random counter
LDA Port_D ; get switches
AND \\#03 ;check tilt \\& invert sw
BNE Init_rnd ; loop til gone
LDA TEMP1 ; get new seed
STA Spcl_seed1 ; stuff it
STA Seed_1 ; also load for cold boot
; Use feed sw to generate a better random number

| JSR | Get_feed | ; go test sensor |
| :-- | :-- | :-- |
| LDA | Stat_4 | ; get system |
| AND | \\#Do_feed | ; ck sw |
| BNE | Feed_rnd ${ }^{1}$ | ; if feed sw then cold boot |
| JMP | End_coldinit | ; else do warm boot |

Feed_rnd:
INC TEMP1 ; random counter
LDA Stat_4 ; system
AND \\#DFh ; clear any prev feed sw senses
STA Stat_4 ; update
JSR Get_feed ; go test sensor
LDA Stat_4 ; get system
AND \\#Do_feed ; ck sw
BNE Feed_rnd ; wait for feed to go away
LDA TEMP1 ; get new seed
STA Spcl_seed1 ; stuff it
STA Seed_1 ; also load for cold boot
; IF this is a cold boot, reset command then clear EEPROM and
; chose a new name and voice.
Do_cold_boot:
LDA \\#00
STA Warm_cold ; flag cold boot
      ```
    LDA Stat_0 ; system
    CRA #Say_new_name ;make system say new name
STA Stat_0 ;
;****** NOTE : : : : :
; VOICE AND NAME SLECTION MUST HAPPEN BEFORE EEPRON WRITE OR
; THEY WILL ALWAYS COME UP 00 because ram just got cleared!!!!!!
; Random voice selection here
LDA #80h ;ge random/sequential split
STA IN_DAT ; save for random routine
LDX #00 ;make sure only gives random
LDA #10h ;get number of random selections
JSR Ran_seq ; go get random selection
TAX
LDA Voice_table,X ;get new voice
STA Rvoice ; set new voice pitch
; On power up or reset, Furby must go select a new name . . . ahw how
```

```
    LSR Random ;get 32 possible
    AND #1Fh ;get 32 possible
STA Name ;set new name pointer
JER Do_EE_write ;write the EEPROM
```

End_coldinit:
EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
; * 'Special initialization prior to normal run mode
; * Jump to Warm_boot when portD wakes us up
EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
;
Warm_boot: ;no mal ;tart when Port_D wakes us up.
JSR S_EEF; M_READ ; read data to ram
;Eprom_read_byp:
; If light osc fails, or too dark and that sends us to sleep, we
; set 'Dark_sl=wp_prev' and save it in EEPROM in 'Seed_2'.
; when the sleep routine executes, (00 01 based on this bit)
; When we wake up we recover this bit and it becomes the previous done
; flag back in 'Stat_0', so that if the osc is
      ; still dark or failed, Furby wont go back to sleep.

| LDA | Seed_2 | ; from EEPROM |
| :-- | :-- | :-- |
| BEQ | No_prevsleep | ; jump if none |
| LDA | Stat_0 | ; system |
| ORA | \\#Dark_sleep_prev | ; prev done |
| STA | Stat_0 | ;update |

No_prevsleep:

LDA Spcl_seed1 ; recover start up random number
STA Seed_1 ; set generator
; Pot_timeL2 is save in ram through sleep mode and then reloaded
; Pot_timeL which is the working register for the motor position.
This allows startup routines to clear ram without forgetting the
; last motor position.
LDA Pot_timeL2 ; get current count
STA Pot_ imeL ; save in motor routine counter
; Get age and make sure it is not greater than 3 (age4)
LDA Age ; get current age
AND \\#83h ;preserve bit 7 which is 9th age counter bit
; ; ; ; and insure age not $>3$
STA Age ; set system
; *********************
LDA \\#Bored_reld ; reset timer
STA Bored_timer ;
LDA \\#03 ; set timer
STA Last JR ; timer stops IR from hearing own IR xmit
JSR Get_light ; go get light level sample
LDA TEMP1 ; get new count
STA Light_reff ;update system

LDA Warm_cold ; decide if warm or cold boot
CMP \\#1ih ;ck for warm boot
BEQ No_zero ; jump if is
      | LDA | \\#00 | ; point to macro 0 (SENDS TO SLEEP POSITION) |
| :--: | :--: | :--: |
| STA | Macro_Lo |  |
| STA | Macro_Hi |  |
| JSR | Get_macro | ; go start motor/speech |
| JSR | Notrdy | ; Do / get status for speech and motor |

No_zero:

| LDA | \\#11 | ; preset motor speed |
| :-- | :-- | :-- |
| STA | Mon_len | ; set motor on pulse timing |
| LLA | \\#05 | ; set motor to $3 / 4$ speed for speed test |
| STA | Moff_len | ; set motor off pulse timing |


| LDA | \\#00 | ;clear all system sensor requests |
| :-- | :-- | :-- |
| STA | Stat_4 | ;update |

; Currently uses 4 tables, one for each age.
LDA Stat_0 ; system
ORA \\#Init_motor :flag motor to do speed test
ORA \\#Init_Mspeed ; 2nd part of test
STA Stat_0 ;update
; Do wake up routine :

      ```
;EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII!
;' 'IDLE Routine
;EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII!
;
```

Idle:
; Idle routine is the time slice task master (TSTM) ugh!
; We must call each routine and interlese with a call to speech
; to insure we never miss a TI request for data.
JSR Notrdy ;Do / get status for speech and motor
; THIS bit is set when light sensor is darker than 'Dark_sleep'

| LDA | Stat_0 | ; system |
| :-- | :-- | :-- |
| AND | \\#REQ_dark_sleep | ;ck for req |
| BEQ | No_dark_req ; jump if not |  |
| LDA | Stat_0 | ; system |
| AND | \\#BPh | ; kill req |
| STA | Stat_0 | ;update |
| LDA | \\#A6h | ; sleep macro |
| STA | Macro_Lo |  |
| LDA | \\#00h | ; sleep macro |
| STA | Macro_Hi | ; |
| JMP | Start_macro | ; go say it |

No_dark_req:
; When any sensor or timer calls the \"start_macro\" routine, the
; Macro_Lo \\& Macr_Hi are saved. Everyone jumps back to Idle and when
; speech/motor routines are finished, this routine will look at the
; macros that were used and execute another function if a match is
found.
; Checks for his name first, then any IR to send, and finally, the sleep
; commands. The temp macro buffers are cleared before

```
Spcl_Name1:
    LDX #00 ; offset
Spcl_Name2:
    LDA Ck_Name_table,X ;ck lo byte
    CMP #FFh ;ck for end of table (note 255 cant execute)
    BEQ Spcl_IR1 ;done if is
    CMP Req_macro_lo ;ck against last speech request
    BNE Not_Name2 ; jump if not
    INX ; to hi byte
    LDA Ck_Name_table,X ;ck hi byte
    CMP Req_macro_hi ;ck against last speech request
```
      Spcl_macro1:
LDX \\#00 ; offset
Spcl_sleep1:
LDA sleepy_table, X ;ck lo byte
CMP \\#FFh ; ck for end of table (note 255 cant execute)
BEQ Ck_macro_dn ; done if is
CMP Req macro_lo ;ck against last speech request
BNE Not_sleepy2 ; jump if not
INX ; to hi byte
LDA sleepy_table, X ; ck hi byte
CMP Req macro_hi ; ck against last speech request
BNE Not_sleepy: ; jump if not
LDA \\#00 ; clear macro pointers for wake up
STA Req macro_lo
STA Req macro_hi
;mod F-rels2 ;
Before going to sleep send sleep cmnd to all others.
LDA \\#15 ;
STA TEMP2 ; xmit temp ram
LDA \\#FDh ; TI command for IR xmit
STA TEMP1 ;
JSR Xmit_TI ; go send it
; need to wait $>600$ milisec before going to sleep because we arent using ;busy flags from TI and need to make sure it is done transmitting the ;I.R. code, the sleep routine kills the TI and it would never send the cmnd.

LDA \\#25 ;how many 30 milisec cycles to call
JSR Half_delay ; do 30 milisec delay cycles
;end mod
JMP GoToSleep ; nity-night
Not_sleepy2:
INX ;
Not_sleepy3:
INX ;
JMP Spcl_sleep1 ; loop til done
Ck macro_dn:
LDA \\#00 ; clear macro pointers for wake up
STA Req macro_lo
STA Req macro_hi
JMP Test_new_name ; on to task master
; ; ; ; ; ; SLEEP TABLE \\& IR table ..... MOVE TO INCLUDE FILE LATER
Sleepy_table:
DW 91 ;hangout
DW 166 ; wake up
DW 167 ; wake up
DW 168 ; wake up
DW 169 ; wake up
      | DW | 258 | ; Back sw |
| :--: | :--: | :--: |
| DW | 259 | ; Back sw |
| DW | 260 | ; Back sw |
| DW | 403 | ; IR |
| DW | 413 | ; IR |
| DW | 429 | ; IR |
| DB | FFh,FFh | ; FF FF is table terminator |


| Iramit_table: |  |  |
| :--: | :--: | :--: |
| DW |  | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 13 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 17 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 19 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 26 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 29 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 33 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 34 | ; trigger macro |
| DE | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 44 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 45 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 48 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 50 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 55 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 60 | ; trigger macro |
| DB | 00 | ; which IR command to call ( 0 - 0f) |
| DW | 149 | ; from rooster wake up |
| DB | 00 | ; |
| DW | 352 | ; trigger macro |
| DB | 01 | ; which IR command to call ( 0 - 0f) |
| DW | 363 | ; trigger macro |
| DB | 01 | ; which IR command to call ( 0 - 0f) |
| DW | 393 | ; trigger macro |
| DB | 01 | ; which IR command to call ( 0 - 0f) |
| DW | 248 | ; trigger macro |
| DB | 02 | ; which IR command to call ( 0 - 0f) |
| DW | 313 | ; trigger macro |
| DB | 02 | ; which IR command to call ( 0 - 0f) |
| DW | 86 | ; trigger macro |
| DB | 03 | ; which IR command to call ( 0 - 0f) |
| DW | 93 | ; trigger macro |
| DB | 03 | ; which IR command to call ( 0 - 0f) |
| DW | 339 | ; trigger macro |
      | DB | 03 | ; which IR command to call ( 0 - 0f ) |
| :--: | :--: | :--: |
| DW | 344 | ; trigger macro |
| DB | 03 | ; which IR command to call ( 0 - 0f ) |
| DW | 351 | ; trigger macro |
| DB | 03 | ; which IR command to call ( 0 - 0f ) |
| DW | 404 | ; trigger macro |
| DB | 04 | ; which IR command to call ( 0 - 0f ) |
| DW | 405 | ; trigger macro |
| DB | 04 | ; which IR command to call ( 0 - 0f ) |
| DW | 293 | ; trigger macro |
| DB | 05 | ; which IR command to call ( 0 - 0f ) |
| DW | 394 | ; trigger macro |
| DB | 05 | ; which IR command to call ( 0 - 0f ) |
| DW | 406 | ; trigger macro |
| DB | 05 | ; which IR command to call ( 0 - 0f ) |
| DW | 414 | ; trigger macro |
| DB | 05 | ; which IR command to call ( 0 - 0f ) |
| DW | 422 | ; trigger macro |
| DB | 05 | ; which IR command to call ( 0 - 0f ) |
| DW | 395 | ; trigger macro |
| DB | 06 | ; which IR command to call ( 0 - 0f ) |
| DW | 421 | ; trigger macro |
| DB | 06 | ; which IR command to call ( 0 - 0f ) |
| DW | 423 | ; trigger macro |
| DB | 06 | ; which IR command to call ( 0 - 0f ) |
| DW | 296 | ; trigger macro |
| DB | 07 | ; which IR command to call ( 0 - 0f ) |
| DW | 415 | ; trigger macro |
| DB | 07 | ; which IR command to call ( 0 - 0f ) |
| DW | 416 | ; trigger macro |
| DB | 07 | ; which IR command to call ( 0 - 0f ) |
| DW | 288 | ; trigger macro |
| DB | 08 | ; which IR command to call ( 0 - 0f ) |
| DW | 11 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 12 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 27 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 42 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 57 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 235 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 236 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 237 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 238 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 261 | ; trigger macro |
| DB | 09 | ; which IR command to call ( 0 - 0f ) |
| DW | 262 | ; trigger macro |
      | DB | 09 | ; | which IR command to call ( 0 - 0f ) |
| :--: | :--: | :--: | :--: |
| DW | 396 | ; trigger macro |  |
| DB | 09 | ; | which IR command to call ( 0 - 0f ) |
| DW | 409 | ; trigger macro |  |
| DB | 09 | ; | which IR command to call ( 0 - 0f ) |
| DW | 399 | ; trigger macro |  |
| DB | 10 | ; | which IR command to call ( 0 - 0f ) |
| DW | 407 | ; trigger macro |  |
| DB | 10 | ; | which IR command to call ( 0 - 0f ) |
| DW | 408 | ; trigger macro |  |
| DB | 10 | ; | which IR command to call ( 0 - 0f ) |
| DW | 272 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 273 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 274 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 275 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 400 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 418 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 425 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 426 | ; trigger macro |  |
| DB | 11 | ; | which IR command to call ( 0 - 0f ) |
| DW | 336 | ; trigger macro |  |
| DB | 12 | ; | which IR command to call ( 0 - 0f ) |
| DW | 342 | ; trigger macro |  |
| DB | 12 | ; | which IR command to call ( 0 - 0f ) |
| DW | 401 | ; trigger macro |  |
| DB | 12 | ; | which IR command to call ( 0 - 0f ) |
| DW | 92 | ; trigger macro |  |
| DB | 13 | ; | which IR command to call ( 0 - 0f ) |
| DW | 411 | ; trigger macro |  |
| DB | 13 | ; | which IR command to call ( 0 - 0f ) |
| DW | 419 | ; trigger macro |  |
| DB | 13 | ; | which IR command to call ( 0 - 0f ) |
| DW | 427 | ; trigger macro |  |
| DB | 13 | ; | which IR command to call ( 0 - (f) ) |
| DW | 291 | ; trigger macro |  |
| DB | 14 | ; | which IR command to call ( 0 - 0f ) |
| DW | 402 | ; trigger macro |  |
| DB | 14 | ; | which IR command to call ( 0 - 0f ) |
| DW | 412 | ; trigger macro |  |
| DB | 14 | ; | which IR command to call ( 0 - 0f ) |
| DW | 428 | ; trigger macro |  |
| DB | 14 | ; | which IR command to call ( 0 - 0f ) |
| DW | 256 | ; trigger macro |  |
| DB | 15 | ; | which IR command to call ( 0 - 0f ) |
| DW | 257 | ; trigger macro |  |
| DB | 15 | ; | which IR command to call ( 0 - 0f ) |
| DW | 420 | ; trigger macro |  |
      DB 15 :which IR command to call ( 0 - 0f )
;mod F-rels2 ; send sleep if recv sleep on IR
DW 403 ; trigger macro
DB 15 :which IR command to call ( 0 - 0f )
DW 413 ; trigger uacro
DB 15 :which IR command to call ( 0 - 0f )
; end mod
DB FFh,FFh ;FF FF is table terminator

Ck_Name_table:
DW 97
DW 248
DW 393
DW 414
DW 149
DW 305
DW 404
DW 421
DB FFh,FFh ; FF FF is table terminator
; Say name
Test_new_name:

| LDA | Stat_0 | ; system |
| :-- | :-- | :-- |
| AND | \\#Say_new_name | ; make system say new name |
| BEQ | Nosayname | ; bypass it clear |
| LDA | Stat_0 |  |
| AND | \\#DFh | ; kill req for startup new name |
| STA | Stat_0 | ; update |
| LDA | Name | ; current setting for table offset |
| CLC |  |  |
| ROL | A | $: 2$ 's comp |
| TAX |  |  |
| LDA | Name_table,X | ; get 10 byte |
| STA | Macro_Lo | ; save 10 byte of Macro table entry |
| INX |  |  |
| LDA | Name_table,X | ; get hi byte |
| STA | Macro_Hi | ; save hi byte of Macro table entry |
| JSR | Get_macro | ; go start motor/speech |
| JSR | Notrdy | ; Do / get status for speech and motor |

Nosayname:
; ***** below routines run at 742 mSec loops
; Timer B sets 'Millisec_flag' each 742 milliseconds
      

```
; none active
;;
; Task 0 : scans all active requests from sensors looking for a trigger.
; If any are set then scan through the game select tables for each game
; looking for a match, and increment the counter each time a succesive
; match is found. If one is not in sequence, then that counter is reset
to
; zero. Since all counters are independent, then the first one to
completion
; wins and all others are zeroed.
; All sensor triggers are in one status byte so we can create a number
; based on who has been triggered (we ignore the I.R. sensor).
; The following bits are in Stat_4 and are set when they are triggered
; by the individual sensor routines :
; 00 = none
; 01 = Loud sound
; 02 = Light change brighter
; 04 = Light change darker
; 08 = Front tummy switch
; 10 = Back switch
; 20 = Feed switch
; 40 = Tilt switch
```
      ; $8^{\\circ}=$ Invert switch
; We assi 1 a single bit per game or egg senario. Each time a
; sensor :s triggered, we increment the counter and test all eggs for
; a match. If a particular sensor doesn match, then set its
disqualified
; bit and move on. If at any time all bits are set, then clear counter to
; zero and start over. When a table gets an FF then that egg is executed.
; Each time a sensor is triggered, the system timer is reset. This timer ; called 'Sensor_timer'is reset with 'Global_time' equate. This timer is also
; used for the random sequential selection of sensor responses. If this
; timer goes to zero before an egg is complete, ie, Furby has not been played
; with, then clear all disqualified bits and counters.
; Currently there are 24 possible eggs. (3 bytes)
;Qualify1:
;DQ_fortune EQU 01 ;bit $0=$ fortune teller
;DQ_rap EQU 02 ;bit $1=$ rap song
;DQ_hide EQU 04 ;bit $2=$ hide and seek
;DQ_simon EQU 08 ;bit $3=$ simon says
;DQ_burp EQU 10 ;bit $4=$ burp attack
;DQ_name EQU 20 ;bit $5=$ say name
;DQ_twinkle EQU 40 ;bit $6=$ sing song
;DQ_rooste EQU 80 ;bit $7=$ rooster-love you
;Qualify2: ; ; ; ; removed due to lack of RAM
bit $0=$
bit $1=$
bit $2=$
bit $3=$
bit $4=$
bit $5=$
bit $6=$
bit $7=$
; Test triggers here
Ck game:
LDA Sensor_timer ;ck if no action for a while
LDA Bored_timer ;ck if no action for a while
BNE Ck_gamactv ; jump if system active
JSR Clear_games ; go reset all other triggers and game pointers
Ck_gamactv:
LDA Qualify1 ;test if all are disqualified
CMP \\#FFh ; compare activ bits only
BNE Ck_anysens ; jump if some or all still active
LDA Qualify2 ; test if all are disqualified
CMP \\#00h ; compare activ bits only
BNE Ck_anysens ; jump if some or all still active
JSR Clear_games ; go reset all other triggers and game pointers
Ck_anysens:
LDA Stat_4 ;ck if any sensor is triggered
BNE Ck_gaml ; go ck games if any set
JMP Ck_bored ; bypass if none
      | STA | Qualify1 | ;update system |
| :--: | :--: | :--: |
| JMP | Ck_gam7 | ;check next egg |
| Ck_gam6a: |  |  |
| LDA | Name_egg+1,X | ; get current data +1 to see if end of egg |
| CMP | \\#FFh | ; test if end of table and start of game |
| BNE | Ck_gam7 | ; jump if not at end |
| JSR | Clear_games | ; go reset all other triggers and game pointers |
| LDA | Game_1 | ; get system |
| ORA | \\#Name_mode | ; start game mode |
| STA | Game_1 | ;update |
| LDA | \\#00 | ; clear all pointers |
| STA | Stat_5 | ; system |
| JMP | Idle | ; done |

Ck_gam7: ; twinkle song
LDA Qualify1 ;update game qualification
AND \\#DQ_twinkle ;check if dis-qualified bit
BNE Ck_gam8 ; bail out if is
LDA Twinkle_egg, X ; get current data
AND Stat_4 ; compare against sensor trigger
BNE Ck_gam7a ; if set then good compare
LDA Qualify1 ;update game qualification
ORA \\#DQ_twinkle ; set dis-qualified bit
STA Qualify1 ;update system
JMP Ck_gam8 ; check next egg
Ck_gam7a:
LDA Twinkle_egg+1,X ; get current data +1 to see if end of egg
CMP \\#FFh ; test if end of table and start of game
BNE Ck_gam8 ; jump if not at end
JSR Clear_games ; go reset all other triggers and game pointers
LDA Game_1 ; get system
ORA \\#Twinkle mode ; start game mode
STA Game_1 ;update
LDA \\#00 ; clear all pointers
STA Stat_5 ; system
JMP Idle ; done
Ck_gam8: ; roos' or loves you
LDA Qualify1 ;update game qualification
AND \\#DQ_rooster ; check if dis-qualified bit
BNE Ck_gam9 ; bail out if is
LDA Rooster_egg,X ; get current data
AND Stat_4 ; compare against sensor trigger
BNE Ck_gam8a ; if set then good compare
LDA Qualify1 ;update game qualification
ORA \\#DQ_rooster ; set dis-qualified bit
STA Qualify1 ;update system
JMP Ck_gam9 ; check next egg
Ck_gam8a:
LDA Rooster_egg+1,X ; get current data +1 to see if end of egg
CMP \\#FFh ; test if end of table and start of game
BNE Ck_gam9 ; jump if not at end
JSR Clear_games ; go reset all other triggers and game pointers
LDA Game_1 ; get system
ORA \\#Rooster_mode ; start game mode
STA Game_1 ;update
LDA \\#00 ; clear all pointers
STA Stat_5 ; system
JMP Idle ; done
      # Ck gam9: 


      DB 20h, 20h, 20h, 10h, FFh ; feed,f oed,feed,back
Name_egg:
DB 08h, 08h, 08h, 10h, FFh ; frnt, frnt, frnt, back
Twinkle_egg:
DB 01h, 01h, 01h, 10h, FFh ; and, and, and, back
Rooster_egg:
DB 04h, 04h, 04h, 10h, FFh ; light, light, light, back
;
; Normal task scan of sensc:s and timers.
Ck_bored:
LDA Bored_timer ;ck if bored ... $=0$
BNE Ck_tski ; jump if not bored
; Currently uses 4 tables, one for each age.
LDA \\#Bored_split ; get random/sequential split
STA IN_DAT ; save for random routine
LDX \\#Seq_bored ; get number of sequential selections
LDA \\#Ran_bored ; get number of randoms
JSR Ran_seq ; go decide random/sequential
BLS Bored_ran ; Random mode when carry SET
LDX Bored_count ; save current
INC Bored_count ; if not then next table entry
LDA Bored_count ; get
CLC
SBC \\#Seq_bored-1 ;ck if > assignment
BCC Bored_side ; jump if <
LDA \\#00 ; reset to 1st entry of sequential
STA Bored_count ;
Bored_s'de:
; current count
Bored_ran:
JSR Decid_age ; do age calculation for table entry
LDX TEMPO ; age offset
LDA Bored_S1,X ; get new sound/word
STA Macro_Lo ; save lo byte of Macro table entry
INX
LDA Bored_S1,X ; get new sound/word
STA Macro_Hi ; save hi byte of Macro table entry
JMP Start_macro ; go set group/table pointer for motor \\& spch
;
Ck_tski:
LDA Task_ptr
CMP \\#01 ; decide which
BNE Ck_tsk4 ; jump if not
JMP CR_tilt ; Ck ball switch side sense
Ck_tsk4:
CMP \\#02 ; c\\&cide which
BNE Ck_tsk5 ; jump if not
      BNE NMM_out ; wait til 0
LDA Drift_rev ;
BNE NMM_out ; wait til 0
; Set a timer and ck counter 'motorstopad' (incremented with wheel count)
; to see if it changed. When it stops changing then the motor has stopped.
LDA motorstopad ;ck for 0
BNE NMM_out ; wait till 0
LDA TEMP1 ; get last motor count
CMP Pot_timeL ; ck if changed
BEQ Motor_done ; jump if same (motor finally stopped)
LDA Pot_timeL ; get current
STA TEMP1 ;
LDA \\#15 ; reset timer (8)
STA motorstopad ;
JMP NMM_out ; wait another cycle
Motor_done:
LDA Cycle_timer ; get step timer
BNE NMM_out ; wait til 0
STA Drift_counter ; use as a temp register
JSR Motor_data ; get data
LDA \\#00
STA TEMP1 ; reset
LDA Motor_lo ; get data (use for lbyte table (DB)).
CMP \\#FFh ; is it table end (dont inc off end)
BNE Motor_pause ; more
LDA Stat_2 ; get system
AND \\#Motor_ntseek ; clear seek flag
STA Stat_2 ; update system
NMM_out:
JMP Endtask_2 ; seek complete
Motor_pause:
LDA Motor_lo ; check for pause request on this step (00)
BNE More_motor ; more
JMP Motor_killend ; set cycle timer and :ait for next motor
step
; To initialize the motor call table, the originator loads 'Which_motor' ; with the pointer and calls 'Decide_motor'.

Ck_Macro:
JSR Next_macro ; get data
STA Which_motor ; save motor seek pointer
JSR Next_macro ; get data
STA Mgroup ; save high byte
CMP \\#00h ; check for end of macro
BNE Got_macro ; do it if not 0
LDA Which_motor ; ck lo byte for 0
CMP \\#00h ; check for end of macro
      ; increment, based on new direction, to compensate for the slot which ; will be counted twice.

| LDA | Motor_lo | ; get data |
| :--: | :--: | :--: |
| CMP | Pot_timeL | ; ck for same |
| BNE | Tst_fwdmore | ; jump if not 0 |
| LDA | Stat_2 | ; get system |
| AND | \\#Motor_inactv | ; clear activ flag |
| STA | Stat_2 | ; update system |
| JMP | Endtask_2 | ; bail out |
| Tst_fwdmore: |  |  |
| CLC |  |  |
| SBC | Pot_timeL | ; get current position |
| BCC | Go_rev | ; if borrow then dec command |

Go_fwd:
LDA
*.
AND \\#Pos_sen
BEQ Go_fwd2
LDA Stat_2
AND \\#Motor_fwd
BNE Go_fwd2
DEC
; get I R detector
; bypass if sensor is over slot in wheel ; get system
; get direction motor was last headed
;if set then new direction is same as last ; compensate for counter direction reversal

Go_fwd2:
LDA Stat_2
; get system
ORA \\#Motor_fwd
; set = motor fwd (inc)
ORA \\#Motor_actv ; set motor in motion
STA Stat_2
; update system
LDA Stat_3
; get current status
ORA \\#Motor_off ; turn both motors off
AND \\#Motor_fwds ; move motor in fwd dir
JMP End_rev ; go finish port setup
Go_rev:
LDA
ORA
; get I R detectir
AND \\#Pos_sen
BEQ Go_rev2
; bypass if sensor is over slot in wheel
LDA Stat_2
; get system
AND \\#Motor_fwd ; get direction motor was last headed
BEQ Go_rev2 ; if clr then new direction is same as last INC Pot_timeL2 ; compensate for counter direction revercal

Go_rev2:
LDA Stat_2
; get system
AND \\#Motor_rev ; clear fwd flag
ORA \\#Motor_actv ; set motor in motion
STA Stat_2
; update system
LDA Stat_3
; get current status
ORA \\#Motor_off ; turn both motors off
AND \\#Motor_revs ; move motor in rev dir
End_rev:
STA Stat_3
JMP Endtask_2 ; done
Do_motor:
; f(1) (1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(1)(
      ; jmp Byp_motorS3

| LDA | Stat_0 | ; system |
| :-- | :-- | :-- |
| AND | \\#Init_Mspeed | ; ck if motor to do speed test |
| BEQ | Byp_motorS3 ; only runs on wake up |  |
| LDA | Stat_0 | ; system |
| AND | \\#Init_motor ; ck if motor to do speed test |  |
| BEQ | Byp_motorS2 ; only runs on wake up |  |
| LDA | Stat_0 | ; system |
| AND | \\#Nt_Init_motor | ; done |
| STA | Stat_0 | ; update |
| LDA | \\#00 | ; reset opto speed counter |
| STA | Mot_opto_cnt | ; setit |
| LDA | \\#Opto_spd_reld | ; get timer value for speed test |
| STA | Mot_speed_cnt | ; set it |

Byp_motorS2:
LDA Mot_speed_cnt ;get timer
BNE Byp_motorS3 ; do nothing if $>0$


Byp_motorS3:
;11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
      ```
; DB 58,60,60,60,60,60,60,60,60,60,60
; DB 20,22,24,27,30,32,34,36,38
; DB 46,48,50,52,54,56,58,60,60,60,60,60
    DB 25,26,27,28,30,32,34,36,38,42,\\&5
    DB 48,51,54,57,60,60,60,60,60,60,60
;
On wake up when the motor moves from position 10 to 134, we
; time it and increment a counter which is used to access this table
; and get the motor on pulse value.
; Refer to power up preset pulse width for table pointers
Motor_speed:
    DB Mpulse_on,Mpulse_on,Mpulse_on
    DB Mpulse_on,Mpulse_on,Mpulse_on
    DB Mpulse_on,Mpulse_on,Mpulse_on
    DB Mpulse_on,Mpulse_on,Mpulse_on ;f,10
    DB Mpulse_on,Mpulse_on,Mpulse_on
    DB Mpulse_on,Mpulse_on,Mpulse_on
    DB Mpulse_on,Mpulse_on,Mpulse_on-1
    DB Mpulse_on-2,Mpulse_on-3,Mpulse_on-4 ;1b,1c
    DB Mpulse_on-5,Mpulse_on-5,Mpulse_on-6
    DB Mpulse_on-7,Mpulse_on-8,Mpulse_on-9
    DB Mpulse_on-9,Mpulse_on-9,Mpulse_on-9
    DB Mpulse_on-9,Mpulse_on-9,Mpulse_on-9
    DB Mpulse_on-9,Mpulse_on-9,Mpulse_on-9
    DB Mpulse_on-9,Mpulse_on-9,Mpulse_on-9
DB Mpulse_on-9,Mpulse_on-9,Mpulse_on-9
```

;
; This finds the 16 bit adrs of the table and points the motor

      

Get_macro:
; Motor noise is triggering sound sensor hardware, so this sets the ; previously sound done flag, and the system will not respond to the ; sound sensor until the sound trigger line goes low and clears prev done.

| LDA | Stat_3 | ; system |
| :-- | :-- | :-- |
| ORA | \\#Sound_stat | ; set prev dn |
| STA | Stat_3 | ; set prev dn |

;------------------- end sound flag
      INC Age_counter ;rolls over tc inc age
BNE Same_age ; jump if no roll over

; AGE INCREMENT uses bit 7 to double age counter
LDA Age ;get bit 7 - set = counter rolled over twice
AND \\#80h ;get bit 7
BNE Roll_age ;bit 7 set so inc age
LDA Age
ORA \\#80h ; set bit 7 for next counter roll over
STA Age ;update
JMP Same_age ; done

Roll_age:
INC Age ; just grew up some
LDA Age
AND \\#07h ; clear bit 7
STA Age
CLC
SBC \\#03 ; make sure it isnt > 3 (0-3 age)
BCC Same_age ; jump if <4
LDA \\#03 ; max age
STA Age ;
Same_age:
;------------------- end age

| LDA | Stat_2 | ; system |
| :-- | :-- | :-- |
| ORA | \\#Macro_actv | ; flag request |
| STA | Stat_2 | ; update |
| CLC |  | ; do speech |
| ROL | Macro_Lo | ; move hi bit to carry \\& get 2's offset |
| ROL | Macro_Hi | ; move carry into one of four group ptr |

LDA Macro_Lo ; offset ptr
LDA Macro_Hi ; get current group pointer
CMP \\#03 ; is it table group 4
BEQ Dec_macro4 ; jump if is
CMP \\#02 ; is it table group 3
BEQ Dec_macro3 ; jump i: is
CMP \\#01 ; is it table group 2
BEQ Dec_macro2 ; jump if is
Dec macro1: ; table group 1
LDA Macro_grp1,X ; get lo pointer
STA Macro_Lo ; working buffer
INX ; $\\mathrm{X}+1$
LDA Macro_grp1, X ; get hi pointer
JMP Dec_macro_end ; go finish load
Dec macro2:
LDA Macro_grp2, X ; get lo pointer
STA Macro_Lo ; working buffer
INX ; $\\mathrm{X}+1$
;
LDA Macro_grp2, X ; get hi pointer
JMP Dec_macro_end ; go finish load
Dec macro3:
LDA Macro_grp3, X ; get lo pointer
STA Macro_Lo ; working buffer
INX ; $\\mathrm{X}+1$
      JSR Clear_all_gam
LDA \\#Bored_reld ;reset bored timer
STA Bored_timer ;
LDA Name ; current setting for table offset
CLC
ROL A ; 2's comp
TAX
LDA Name_table, X ; get lo byte
STA Macro_Lo ;save lo byte of Macro table entry
INX
LDA Name_table, X ; get hi byte
STA Macro_Hi ;save hi byte of Macro table entry
JMP Start_macro ;go set group/table pointer for motor \\& spch
;Twinkle song egg
; When song is complete, if both front and back switches are pressed
; we goto deep sleep. That means only the invert can wake us up, not
; the invert switch.
Twinklsnd_lo EQU \\#D5h ;using macro 469
Twinklsnd_hi EQU \\#01h ;
Sleep_lo EQU \\#A6h ;using macro 166 (before going to sleep)
Sleep_hi EQU \\#00h ;
Game_twinkle:
JSR Clear_all_gam
LDA \\#03 ; song counter
STA HCEL_LO ; set
Gtwnk:
DEC HCEL_LO ; -1
LDA Stat_2 ;Get system clear done flags
AND \\#Not_tch_ft ; clear previously inverted flag
AND \\#Not_tch_bk ; clear previously inverted flag
STA Stat_2 ;update
LDA \\#Bored_reld ;reset bored timer
STA Bored_timer ;
LDA \\#Twinklsnd_lo ; get macro lo byte
STA Macro_Lo ;save lo byte of Macro table entry
LDA \\#Twinklsnd_hi ; get macro hi byte
STA Macro_Hi ;save hi byte of Macro table entry
JSR Get_macro ;go start motor/speech
JSR Notify ;Do / get status for speech and motor
JSR Test_all_sens ; get status
JSR Test_all_sens ; get status 2nd time for debounce
LDA Stat_4 ; switch status
AND \\#18h ; isolate front and back switches
CMP \\#18h ;
BEQ Start_sleep ; if both switches pressed, goto sleep
LDA HCEL_LO ; get song loop counter
BNE Gtwnk ; loop
      JMP Idle
; not so egg complete
Start_sleep:
LDA \\#Sleep_lo ;get macro lo byte
STA Macro_Lo ; save lo byte of Macro table entry
LDA \\#Sleep_hi ;get macro hi byte
STA Macro_Hi ; save hi byte of Macro table entry
JSR Get_macro ; go start motor/speech
JSR Notrdy ;Do / get status for speech and motor
LDA \\#11h ; set deep sleep mode
STA Deep_sleep ;
JMP GoToSleep ; nity-night
; Rooster loves you egg
Roostersnd_lo EQU \\#D4h ;using macro 468
Roostersnd_hi EQU \\#01h ;

Game_rooster:
JSR Clear_all_gam
LDA \\#Bored_reld ; reset bored timer
STA Bored_timer ;
LDA \\#Roostersnd_lo ; get macro lo byte
STA Macro_Lo ; save lo byte of Macro table entry
LDA \\#Roostersnd_hi ; get macro hi byte
STA Macro_Hi ; save hi byte of Macro table entry
JMP Start_ma cro ; go set group/table pointer for motor \\& spch
; If a game requires sensor input without triggering the normal
; sensor cycle for speech, then this rtn will check all sensors for
; change and the calling game can check for the appropriate trigger
; DO NOT USE I.R. SENSOR SINCE ITS RAM LOCATIONS ARE USED IN GAMES
Test_all_sens:
JSR Get_back ;
JSR Get_Tilt ;
JSR Get_invert ;
JSR Get_front ;
JSR Get_light ;
JSR Get_sound ;
JSR Get_feed ;
RTS ; back to game
; **** Side all switch triggers when ball falls off center and I/O goes
      ```
hi.
CK_tilt: ;tilt sensor
    JSR Get_Tilt ;go ck for sensor trigger
    BCS Normal_tilt ;go fini normal spch/motor table
    JMP Idle ;no request
Get_Tilt: ;this is the subroutine entry point.
    LDA Port_D ;get I/O
    AND *Ball_side ;ck if we tilted on side
    BNE Do_bside ; jump if hi
    LDA Stat_2 ;Get system
    AND #Not_bside ;clear previously on side flag
    STA Stat_2 ;update
Side_out: \\int
    CLC ;clear indicates no request
    RTS
Do_bside:
    LDA Stat_2 ; system
    AND #Bside_dn ;ck if previously done
    BNE Side_out ; jump if was
    LDA Stat_2 ;get system
    ORA #Bside_dn ;flag set, only execute once
    STA Stat_2 ;update system
    LDA Stat_4 ;game mode status
    ORA #Do_tilt ;flag sensor is active
    STA Stat_4 ;update
    SEC ; carry set indicates sensor is triggered
RTS
Normal_tilt: ; Idle rtn jumps here to complete speech/motor table
;;;;;;; also for testing, when tilt is triggered, it resets all
    ; easter egg routines to allow easy entry of eggs.
JSR Clear_all_gam ;
JSR Life ; go tweek health/hungry counters
BCS More_tilt ; if clear then do sensor else bail
JMP Idle ;done
More_tilt:
```

```
; * * * * * * * * * * * * * * * * * * * * * * * * * * *
    LDA #Tilt_split ;get random/sequential split
    STA IN_DAT ; save for random routine
    LDX #Seq_tilt ; get how many sequential selections
    LDA #Ran_tilt ; get number of random alections
JSR Ran_seq ; go decide random/sequential
```
      

Normal_invert:

JSR Life ;go tweek health/hungry counters
BCS More_invert ;if clear then do sensor else bail
JMP Idle ; done
More_invert:

| LDA | \\#Invert_split | ; get random/sequential split |
| :-- | :-- | :-- |
| STA | IN_DAT | ; save for random routine |

LDX \\#Seq_invert ; get how many sequential selections
LDA \\#Ran_invert ; get number of random alections
JSR Ran_seq ; go decide random/sequential
LDX Sensor_timer ; get current for craining subroutine
BCS Invert_rnd ; Random mode when carry SET
LDA Sensor_timer ;ck if timed out since last action
B\\&Q Invrt_reset ;yep
LDA Invrt_count ; save current
STA BIT_CT ;temp store
INC Invrt_count ; if not then next table entry
LDA Invrt_count ; get
      STA DAC2 ; clear feed sw enable
; LDA Stat_3 ; get system
; AND \\#Feed_dn ;ck if prev done
; BNE Feed_out ; jump if was
; LDA Stat_3 ; get system
; ; Elag set, only execute once
; ; Iagdate system
; ; Iagate system
; ; game mode status
; flag sensor is active
; ; update
SEC ; set when sensor is triggered
RTS
Normal_feed: ; enter here to complete speech/motor
; health table calls here and decision for which speech pattern
LDA \\#Food ; each feeding increments hunger counter
CLC
ADC Hungry_counter ; feed him!
BCC Feeding_dn ; jump if no roll over
LDA \\#FEh ; max count
Feeding_dn:
STA Hungry_counter ; update
;;;;; JSR Life ; go finish sick/hungry speech
LDA \\#Feed_split ; get random/sequential split
STA IN_DAT ; save for random routine
LDX \\#Seq_feed ; get how many sequential selections
LDA \\#Ran_feed ; get random assignment
JSR Ran_seq ; go decide random/sequential
LDX Sensor_timer ; get current for training subroutine
BCS Feedrand ; Random mode when carry set
LDA Sensor_timer ; ck if timed out since last action
BEQ Feed_reset ; yep
LDA Feed_count ; save current
STA BIT_CT ; temp store
INC Feed_count ; if not then next table entry
LDA Feed_count ; get
CLC
SBC \\#Seq_feed-1 ; ck if > assignment
BCC Feed_sut ; jump if <
LDA \\#Seq_feed-1 ; don! inc off end
STA Feed_count ;
JMP Feed_set ; do it
Feed_reset:
      

Normal_light:
; below routines are jumped to by light exec if > reff

JSR Life ; go tweek heelth/hungry counters
BCS More_light ; if clear then do sensor else bail
JMP Idle ;done
More_light:
; get random/sequential split
STA IN_DAT ; save for random routine
      we should never get here so bail back to idle and this will also prevent system lockup when no clk

| LDA | \\#250 | ; never allow roll over |
| :-- | :-- | :-- |
| STA | TEMP1 | $; \\quad$ |
|  |  |  |
| CLI |  | ;re-enable interrupt |
| JSR | Kick JRQ | ; wait fuc motor R/C to start working again |
| LDA | TEMP1 | ; get count |
| CLC |  | ;clear |
| SBC | \\#05 | ;is diff > 5 |
| BCC | No_snd | ;bail out if not |
|  |  |  |
| LDA | Stat_3 | ; system |
| AND | \\#Sound_stat | ; ck for prev done |
| BNE | No_snd2 | ; wait till quiet |
|  |  |  |
| LDA | Stat_3 | ; system |
| ORA | \\#Sound_stat | ; |
| STA | Stat_3 | ; set prev dn |
|  |  |  |
| LDA | Stat_4 |  |
| ORA | \\#Do_snd | ; set indicating change > reff level |
| STA | Stat_4 | ; |
|  |  |  |
| SEC |  | ; carry se: indicates no change |
| RTS |  |  |

No_snd:
LDA Stat_3 ; get system
AND \\#Nt_snd_stat ; clear prev dn
STA Stat_3 ; update
No_snd2:
CLC ; carry clear indicates no sound
RTS ; done
Normal_sound:
; below routines are jumped to if sound pulse detected
; go tweek health/hungry counters
;if clear then do sensor else bail
;done

More_sound:
LDA \\#Sound_split ;get random/sequential split
STA IN_DAT ; save for random itine
LDA \\#Seq_sound ; get how many sequential selections
LDA \\#Ran_sound ; number of random selections
JSR Ran_seq ; go decide random/sequential
      

```
    STA IN_DAT ;save decision
    LDA #Sound_ID ;which ram location for learned word count
    (offset)
    JSR Start_learn ; go record training info
    LDA IN_DAT ; get back word to speak
    JSR Decid_age ; do age calculation for table entry
    LDX TEMPO ; age offset
    LDA Sound_S1,X ; get lo byte
    STA Macro_Lo ; save lo byte of Macro table entry
    INX ;
    LDA Sound_S1,X ; get hi byte
    STA Macro_Hi ; save hi byte of Macro table entry
    JMP Start_macro ; go set group/table pointer for motor & spch
```

; SENSOR TRAINING
; Training for each sensor is set up here and the decision if the
      learned
; word should be played or not.
; Temp_ID hold the ram offset for the last sensor of the learned word.
; Temp_ID2 hold the ram offset for the current sensor of the learned word.
; IN_DAT holds the current word the sensor chose, and will be loaded with
; the learned word instead if the sensor count > the random number that was
; just sampled, ie., force learned word to play.
; ****
; If the sensor timer is at 0 when entering here, then the LEARN_TEMP
; ram location is cleared, else the current learned word is loaded. If
; the learned word is 0 then all entries are cleared.
; When entering, check sensor timer and bail if 0 . THen test if this is
; the back switch and if so then move the current sensor to previous
sensor
; ram and increment the counter.
; If this is not the back switch, then get previous sensor ram counter and
; decrement it. THen move all current sensor information to previous and
; return to caller.
; Because of training difficulties, we now need two back touches to
; increment training counters. If only one occurs then the normal
decreme:t
; happens. This double back touch helps to prevent accidentally training
; with a new macro by hitting the back sw when it is not the macro you
; have been working with.
start_learn:
STA Temp_ID2 ; sensor ram location of counter (current sensor)
LDA Temp_ID2 ;get current sensor ID
CMP \\#EEh ;EF= this is the back switch (special)
BNE Not_BCK ; jumpif not
CPX \\#00 ;ck if sensor timer timed out
BNE Learn_update ; jump if is back switch and not timed out
Not_BCK:
LDA Temp_ID ;get previous sensor ram offset
CMP \\#EEh ;ck if last was back sw
BEQ Not_learned ; jump if no sensor prev
LDX Temp_ID ;get prev:ous sensor ram offset
LDA Tilt_learned, X ; get learned word counter from ram
CMP Learn_temp ; compare with last word
BNE Do_lrn2 ; bail out if different
LDA Tilt_lrn_cnt, X ;prev sensor counter +offset to current
sensor
CLC
SBC \\#Learn_chg ; dec learned word counter since not back sw
STA Tilt_lrn_cnt, X ;update
BCS Do_lrn2 ; jump if > \\#Learn_chg
BPL Do_lrn2 ; jump if not negative (rolled over)
LDA \\#00
STA Tilt_lrn_cnt, X ; set to zero, no roll over
      ; on 1st cycle of new learn, we set counter $1 / 2$ way ..... (chicken)

;
; When IRQ gets turned off, and then restarted, we wait two complete cycle to insure the motor R/C pulses are back in sync.
Kick IRQ:
LDA Stat_3 ;get system
AND \\#Nt IRQdn ;clear IRQ occured status
STA Stat_3 ;update system
LDX \\#03 ; loop counter
Kick2:
LDA Stat_3 ;system
AND \\#IRQ_dn ;ck if IRQ occured
BEQ Kick2 ;wait till IRQ happens
LDA Stat_3 ;get system
AND \\#Nt IRQdn ;clear IRQ occured status
STA Stat_3 ;update system
DEX Kick2 ; 1000 til done
RTS ;is done
; EEPROM READ/WRITE
; Read \\& write subroutines
;
      Enter with 'TEMPO' holding adrs of 0-63. Areg holds lo byte and Xreg holds hi byte. If carry is clear then it was succesfull, if carry is set the write failed.

MODIFIED eeprom, load lo byte in temp1 and hi byte in temp2 and call EEWRIT2.

| LDA | $\\# 00$ | ;use DAC output to put TI in reset |
| :-- | :-- | :-- |
| STA | DAC1 | ; |
| SEI |  | ;turn IRQ off |

LDA $\\quad \\# 00 \\quad$;EEPROM adrs to write data to
STA Sgroup ; save adrs
LDA \\#13 ; number of ram adrs to transfer (x/2)
STA Which_delay ; save
LDA $\\quad \\# 00 \\quad$; Xreg offset
STA Which_motor ; save
Need one read cycle before a write to wake up EEPROM
LDX Which_motor ; eeprom address to read from
JSR EEREAD ; get data (wakes up eeprom)

Write_loop:

| LDA | Sgroup | ; get next EEPROM adrs |
| :-- | :-- | :-- |
| STA | TEMPO | ; buffer |
| LDX | Which_motor | ; ram source |
| LDA | Age,X | ; lo byte (data byte \\#1) |
| STA | TEMP1 | ; save data bytes |
| INC | Which_motor | ; |
| INX |  |  |
| LDA | Age,X | ; |
| STA | TEMP2 | ; hi byte (data byte \\#2) |
| JSR | EEWRIT2 | ; send em |
| BCS | EEfail | ; jump if bad |

INC Sgroup ; 0-63 EEPROM adrs next
INC Sgroup ; 0-63 EEPROM adrs next (eeprom writes 2 bytes)

INC Which_motor ; next adrs
DEC Which_delay ; how many to send
BNE Write_loop ; send some more
RTS ; done
READ EEPROM HERE AND SETUP RAM
S_EEPROM_READ:
Xreg is the adrs 0-63, system returns lo byte in Areg \\& hi byte in Xreg.
on call: $X=$ EEPROM data address (0-63)
on return: ACC = EEPROM data (low byte) (also in TEMPO)
$X=$ EEPROM data (high byte) (also in TEMP1)
      # $* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
      STA Port_A image ;save image RTS
;
;
;
Read data 16-bit data word from EEPROM at specified address
on call: $X=$ EEPROM data address (0-63)
on return: ACC = EEPROM data (low byte)
$X=$ EEPROM data (high byte)
stack usage: 2
RAM usage: TEMPO
;
;
EEREAD:
STX TEMPO ;store data addr
JSR EEENA ; turn on CS
SEC ;send start bit
JSR OUTBIT ;
SEC
JSR OUTBIT ;
CLC
JSR OUTBIT ;
LDX \\#6
ROL TEMPO
ROL TEMPO
; :init addr bit count
; align MS addr bit in bit 7
; : shift address bit into carry
;send it to EEPROM
; bump bit counter
; and repeat until done
LDX \\#16
; init data bit count
LDA \\#0
STA TEMPO
; init data bit accumulators
STA TEMP1
;
EERDO4:
JSR TOOCLK ; toggle clock for next bit
LDA \\#020H ; test data bit (B.5) from EEPROM
BIT Port_B
BNE EERDO8
;
CLC
JMP EERD10
;
EERDO8:
SEC
;
EERD10:
ROL TEMPO
ROL TEMP1
; :create data bit into 16-bit
; accumulator
; bump bit counter
      ```
    BNE EERDO4 ; ; ; and repeat until done
JSR EEDIS ; turn off CS and return
LDA TEMPO ; ret w/data byte in ACC
LDX TEMP1 ; ; and X regs
RTS
;
; ****************************************************************
; Issue ERASE/WRITE ENABLE or DISABLE instruction to EEPROM
; (instruction = 1001100000)
; on call: --
; on return: --
; stack usage: 2
RAM usage: TEMP3
;
***
EEWEN:
    LDA #0FFH ; ;set up enable inst
    JMP EEWEO2
EEWDS:
    LDA #000H ; ; set up disable inst
EEWEO2:
    STA TEMP3 ; save instruction
    JSR EEENA ; turn on CS
    SEC ; send start bit
    JSR OUTBIT ; ;
    CLC ; send ENA/DIS opcode (00)
    JSR OUTBIT ; ;
    CLC ; ;
    JSR OUTBIT ; ;
    LDX #6 ; init instr bit count
EEWEO4:
    ROL TEMP3 ; shift instruction bit into carry
    JSR OUTBIT ; ;send it to EEPROM
    DEX ; bump bit counter
    BNE EEWEO4 ; ; ; and repeat until done
RTS
;
;
; Write data byte to EEPROM at specified address
; on call: TEMPO = EEPROM data address (0-63)
    _ _ ACC = data to be written (low byte)
        X = data to be written (high byte)
    on return: C = 0 on successful write cycle
        C = 1 on write cycle time out
    stack usage: 4
```
      RAM usage: TEMPO, TEMP1, TEMP2

      RTS
;

;
; Subroutine creates sensor table entry for the selected age.
; One table for each age.
; Enter with Acc holding the 1-16 table selection.
; Exit with Acc \\& Temp0 holding the offset 0-FF of the 1-4 age entry.
; Special condition where we have only two tables instead of 4
; (where each table is called based on age), if the \"half_age\" bit is
; set then ages $1 \\& 2$ call table 1 and ages $3 \\& 4$ call table 7.
Decid_age:
STA TEMPO ; save 0-0f selection
LDA Stat_1 ; system
AND \\#Half_age ; test if this is a special 2 table select
BEQ Decid_normal ; jump if not
LDA Stat_1
AND \\#Nt_halff_age ; clear req
STA Stat_1 ; update system
LDA Age ;
AND \\#03h ; get rid of bit 7 (9th counter bit )
CLC
SBC \\#01 ; actual age is $0-3$, test if $<2$
BCC Dec_age1 ; choose age 1 ( actually 0 here)
JMP Spcl_age2 ; choose age 2 ( actually 1 here)
Decid_normal:
; ; ; mod TestR3a.... 25\\% of time chore age1 to add more furbish after
; ; ; he is age 4.
JSR Random ; get a number
CLC
SBC \\#Random_age ; below this level selects age 1
BCS Nospcl_age ; jump if >
LDA \\#00 ; set age 1
JMP Do_age ; go do it
; ; ; end mod
Nospcl_age:
LDA Age ; get current
AND \\#03h ; get rid of bit 7 (9th counter bit )
CMP \\#03 ; is it age 4
BNE Dec_age3 ; jump if not
LDA \\#96 ; point to 4th field
JMP Do_age ; finish load from table
      Life:
; Each FEED trigger increments the HUNGRY counter by (EQU = FOOD).
;Hungry >80 (Need_food) + Sick >CO (Really_sick) = normal sensor
;Hungry >80 (Need_food) + Sick <CO (Really_sick) = random SICK/SENSOR
;Hungry <80 (Need_food) + Sick >CO (Really_sick) = random HUNGRY/SENSOR
;Hungry <80 (Need_food) + Sick <CO (Really_sick) = random
HUNGRY/SICK/SENSOR
;Hungry <60 (Sick_reff) + Sick <CO (Really_sick) = random HUNGRY/SICK
;Hungry $>60$ then each sensor motion increments Sick
;Hungry $<60$ then each sensor motion decrements Sick
; When the system does a cold boot, we set HUNGRY \\& SICK to FFh.....
; When returning from here, carry is set if sensor should execute
; normal routine, and cleared if sensor should do nothing.
;REFF only
;Hingry_counter
; Sick_counter
;Food EQU 20h ;amount to in:rease 'Hungry' for each feeding
;Need_food EQU 80h ; below this starts complaining about hunger
;Sick_reff EQU 60h ; below this starts complaining about sickness
; Real1y_sick EQU C0h ; below this only complains about sickness
;Hungry_dec EQU 01 ;subtract X amount for each sensor trigger
;Sick_dec EQU 01 ;subtract X amount for each sensor trigger
;Max_sick EQU see EQU

LDA Hungry_counter ; current
;mod F-re1s2 ;
; CLC
SEC
; end mod
SEC \\#Hungry_dec ;-X for each trigger
BCS frst_life ; jump if not neg
LDA \\#00 ; reset
fret_life:
STA Hungry_counter ; get count
CLC
SEC \\#Sick_reff ;ck if getting sick
BCS Sick_inc ; jump if not sick
LDA Sick_counter ; current
;mod F-re1s2 ;
; CLC
SEC
; end mod
;mod testr3a
; SBC \\#Sick_dec ;-X for each trigger
BCS frst_sick ; jump if not neg
      JER Ran_seq 0 decide random/sequential BCS Sick_ra: ;Ran: :ode when carry SET

LDA Sensor_timer ;ck if timed out since last action BEQ Sick_reset ;yep
INC Sickr_count ;if not then next table entry
LDA Sickr_count ;get
CLC
SBC \\#Seq_sick-1 ;ck if > assignment
BCC Sick_side ; jump if <
LDA \\#Seq_sick-1 ; dont inc off end
STA Sickr_count ;
JMP Sick_side ; do it
Sick_reset:
LDA \\#00 ;reset to 1st entry of sequenrial
STA Sickr_count ;
Sick_side:
LDA \\#Global_time ;get timer reset value
STA Sensor_timer ;reset it
LDA Sickr_count ;get current pointer to tables
Sick_ran:
JSR Decid_age ; do age calculation for table entry
LDX TEMPO ; age offset
LDA Sick_S1,X ;get lo byte
STA Macro_lo ;save lo byte of Macro table entry
IMX
LDA Sick_S1,X ;get hi byte
STA Macro_Hi ; arrive hi byte of Macro table entry
JSR Get_macro ; go start motor/speech
JSR :olrdy ;Do / get status for speech and motor
CLC ; tells sensor to do nothing
RTS
;
;
;
;
GoToSleep:
; save light sensor fail or sleep command in 'Seed_2' into EEPROM

; EEPROM WRITE
      ; Enter with 'TEMPO' holding adrs of 0-63. Areg holds lo byte and ; Xreg holds hi byte. If carry is clear then it was succesfull, if ; carry is set the write failed.
; MODIFIED eaprom, load lo byte in temp1 and + byte in temp2
; and call EEWRIT2.

| LDA | $\\# 00$ | ; use DAC output to put TI in reset |
| :-- | :-- | :-- |
| STA | DAC1 |  |
| SEI |  | ; turn IRQ off |

LDA \\#00 ;EEPROM adrs to write data o
STA Sgroup ; save adrs
LDA \\#13 ; number of ram adrs to transfi (x/2)
STA Which_delay ; save
LDA \\#00 ; Xreg offset
STA Which_motor ; save
; Need one read cycle before a write to wake up EEPROM
LDX Which_motor ; eeprom address to read from
JSR EEREAD ; get data (wakes up eaprom)

IWrite_loop:

| LDA | Sgroup | ; get next EEPROM adrs |
| :-- | :-- | :-- |
| STA | TEMPO | ; buffer |
| LDX | Which_motor | ; ram source |
| LDA | Age,X | ; lo byte (data byte \\#1) |
| STA | TEMP1 | ; save data bytes |
| INC | Which_motor | ; |
| INX |  |  |
| LDA | Age, X | ; |
| STA | TEMP2 | ; hi byte (data byte \\#2) |
| JSR | EEWRIT2 | ; send em |
| BCS | EEfall | ; jump if bad |
| INC | Sgroup | ; 0-63 EEPROM adrs next |
| INC | Sgroup | ; 0-63 EEPROM adrs next (eeprom writes 2 |

bytes)
INC Which_motor ; next adrs
DEC Which_delay ; how many to send
BNE IWrite_loop ; send some more
;
GoToSleep_2:

Include
Sleep.asm ;
; EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
; \"Interrupt Sulroutines
; EIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
      ; *********** CAUTION ************
; Any ram location written outside of IRQ can only be read in the IRQ,
; likewise if written in the IRQ, then can only be read outside the IRQ.
; THIS WILL PREVENT DATA CORRRUPTION.

NMI :
RTI
;Not used

IRQ:
PHA ;push acc on stack
PHP ;push cpu status on stack
; ****** timer $A=166$ uSEC ********
CKTimerA:
; LDA Interrupts ;get who did it
; AND $\\# 20 \\mathrm{H} \\quad$; test for timerA
; BNE Do_ta ; jump if is
; JMP Ck_timerB ;
;Do_ta:
;***** timer $B=700$ uSEC ******
CK_timerB:
LDA Interrupts ;get status again
AND $\\# 10 \\mathrm{H} \\quad$; test for timer B
BNE Do_timeB ; jump if request true
JMP Intt_false ; bypass all if not
; also changed TimerB relacd value from \\#10h to 00 in EQU
Do_timeB:
; RE-CALIBRATE SWITCH for motor position
; This counter must meet a threshold to decide if the
; calposition ratch is really engaged.
LDA Purt_C ;get I/O
AND \\#Motor_cal ; 10 when limit hit
BNE No_cal_sw ; no position switch found
INC Cal_switch_cnt ; inc each time found low
BNE Cal_noroll ; jump if dldnt roll over (stoppeu on sw tch)
LDA \\#31 ; max count
STA Cal_switch_cnt ;
Cal_noroll:
LDA Cal_switch_cnt ;
CLC
SBC \\#30 ; ck if enough counts
BCC No_lim_etp ; jump if not enough
LDA \\#Cal_pos_fwd ; force value
STA Pot_timeL2 ; reset both
      JMP No Jim stp ; done
No_cal_sw:
LDA \\#00 ; clear count if hi
STA Cal switch cnt ; update

No_lim_stp:
LDA Wait time ; 4 times thru loop $=2.9 \\mathrm{mSec}$
BNE WTa ; $>0$
LDA \\#04 ; counter reset
STA Wait_time ; reload
JMP Timer norm ;
WTa: DEC Wait_time ;
JMP TimerB_dn ; bypass timers until done
Timer norm:
;******** Below routines run at 2.9 mSec
LDA Mot speed cnt ; ck for active
BEQ No spd m ; jump if not
DEC Mot speed cnt ; -1
No spd_m:
LDA motorstoped ; motor drift timer
BEQ No_mstop ; jump if done
DEC motorstoped ; -1
No_mstop:
LDA Motor_led_timer ; Motor_led timer * 742 mSec
BEQ TimeB1 ; jump if done
DEC Motor_led_timer ; -1
TimeB1:
LJA Cycle_timer ; 2.9mSec timer * cycle reload
EaQ TimeB2 ; jump if done
DEC Cycle_timer ; -1
TimeB2:
;m LDA Motor_pulse ; 2.9mSec timer * Motor_pulse
m BEQ TimeB3 ; jump if done
;m DEC Motor_ pulse ; -1
TimeB3:
DEC Mili sec ; -1 \\& allow rollover
BNE TimerB_dn ; wait for rollover ( $2.9 \\mathrm{mS} * 256=742 \\mathrm{mSec}$ )
INC Mili sec flag ; tell task rtn to decrement timers
TimerB dn:
;******** We could test all interrupts here as needed
; Ck2Khz:
; Ck500hz:
; Ck60hz:
;******** Check motor position - IR slot in wheel sensor
      This version does two reads to eliminate noise and sets a done flag to prevent multiple counts. It also reads twice when no slot is present to clear the done flag.

| LDA | Port_C |  | ;get I/O |
| :--: | :--: | :--: | :--: |
| AND | \\#Pos_sen | ; ck position sensor |  |
| BNE | Clr_pos | ; jump if no : | ,ger |
| LDA | Port_C |  | ; get I/O |
| AND | \\#Pos_sen | ; READ $2 x$ to prevent noise trigger |  |
| BNE | Clr_pos | ; jump if no IR trigger |  |
| LDA | Slot_vote | ; get prev cycle |  |
| BEQ | Pc_done | ; bail if prev counted |  |
| LDA | \\#00 | ; |  |
| STA | Slot_vote | ; set ram to 0. (faster than setting a bit) |  |
| JMP | Force_int | ; go count slot |  |

Clr_pos:
LDA Port_C ;get I/O
AND \\#Pos_sen ; READ $2 x$ to prevent noise trigger
BEQ Pc_done ; not 2 equal reads so bypass this cycle
STA Slot_vote ; set ram to 1. (faster than setting a bit)
JMP Pc_done ;

# ExtportC: 

| JMP | Intt_false ; this should be turned off |
| :--: | :--: |
| LDA | Interrupts ; get status again |
| AND | \\#01H ; test for port C bit 1 rising edge |
| BEQ | Pc_done ; jump if not |

Force_int:
LDA Port_D_Image ; system
AND \\#Motor_led ; ck if position I.R. led is on
BEQ Pc_done ; jump if not off
LDA Stat_2 ; get system
AND \\#Motor_fwd ; if set then FWD else REV
BEQ Cnt_rev ; jump if clr
INC Pot_timeL2 ; sensor counter
CLC
LDA Pot_timeL2 ; current
SBC \\#207 ; ck for > 207
BCC Updt_cnt ; jump if not
LDA \\#00 ; roll over
STA Pot_timeL2 ;
JMP Updt_cnt ;
Cnt_rev:
DEC Pot_timeL2 ; -1
CLC
LDA \\#208 ; max count
Pot_timeL2 ; ck for negative ( >207)
S Updt_cnt ; jump if not
Cnt_c
LDA \\#207 ; when neg roll over to max count
STA Pot_timeL2 ;
Updt_cnt:
INC Drift_counter ; to be used for braking pulse
      JMP Intt motor end
M_drft_R2:
DEC Drift_rev ;-1
LDA Port_D_Image ;get system
ORA \\#Motor_off ;turn both motors off
JMP Intt motor end
Intt_motor:
LDA Stat_3
AND \\#C0h ; get motor command bits
STA Intt_Temp ;sav motor direction
:___ Furby1? .. move motor pulse width to interrupt routine
LDA Motor_pulse1 ; get on time
BEQ Intmotor1 ; jump if 0
DEC Motor_pulse1 ;-1
JMP Intmotor_dn ;exit (dent change Intt_temp if on)
Intmotor1:
LDA Motor_pulse2 ; get off time
BEQ Intmotor2 ; got reset timer
DEC Motor_pulse2 ; -1
LDA \\#C0h ; shut motor off
STA Intt_Temp ;
JMP Intmotor_dn ;exit
Intmotor2:
LDA Mon_len ; reset on time
STA Motor_pulse1 ;
LDA Moff_len ;reset off time
STA Motor_pulse2 ;
Intmotor_dn
:---- end motor pulse width
LDA Port_D_Image ; get system
AND \\#3Fh ; clear motor direction bits
CLC
ADC Intt_Temp ; put in motor commands
Intt_motor_end:
STA Port_D_Image ; update system
; at Tracker
EOR \\#\\%11000000 ; ; Tracker add invert motor drivers
; end Tracker
STA Port_D ; output
Intt_done: ;gc ral curn
LDA Stat_3 ; systs
ORA \\#IRQ_dn ; flag :tem IRQ occured
STA Stat_3 ; upiat
Intt_false:
LDA \\#00H ; clear ail intts first
STA Interrupts ;
LDA \\#Intt_dflt ; get default for interrupt reg
STA Interrupts ; set reg \\& clear intt flag
PLP ; recover CPU
      PLA
;recover ACC
RTI
;reset interrupt
; communication protocal with the TI is:
FF is a no action command. (used as end of speech command)
FE sets the command data mode and the TI expects two additional data bytes to complete the string. ( 3 TOTAL)
ALL OTHERS (0-FD) ARE CONSIDERED START OF A SPEECH WOOD !
Command data structure is BYTE 1 + BYTE 2 + BYTE 3
; BYTE 1 is always FE
; Command 1
; BYTE $2=$ FE is pitch table control;
BYTE $3=$ bit 7 set = subtract value from current course value
; clr = add value to current course value
bit 6 set $=$ select music pitch table
; clr = select normal speech pitch table
bit $0-5$ value to change course value (no change $=0$ )
; Command 2
; BYTE $2=$ FD is Infrared transmit cmnd
; BYTE $3=$ Is the I.R. code to send ( 0 - 0Fh only )
; Command 3
; BYTE $2=$ FC is the speech speed control
BYTE $3=$ a value of $0-255$ where 2 Eh is normal speed.
; Enter subroutine with TEMP1 = command byte (1st)
; TEMP2 = data byte (2nd)
Xmit_TI:
LDA \\#FEh ; tells TI command da:a to follow
JSR Spch_more ; out data
LDA TEMPi ; command code
JSR Spch_more ; out data
LDA TEMP2 ; data to send
JSR Spch_more ; out data
RTS ; done
; There is an entry for each bank of speech and only the words in that ; bank are in the list. THIS is a subroutine call.
; The first time thru, we call SAY_x and as long as WORD_ACTIV or SAY_ACTIV
; is set we call DO_NEXTSENT until saysent is done.
; There are 4 groups of 128 pointers in each group. This gives 512
      saysents.
; 1. Enter with 'Which_word' holding 0-12\" and 'Sgroup' for the 1 of 4 tables
; which points to two byte adrs of a saysent. These two bytes are
; lcoded into Saysent_lo \\& Saysent_hi.
; 2. Dat s shuffled to the TI according to the BUSY/REQ line
; Currently we have 167 speech words or sounds in ROM. Words 1 - 12
; are in bank 0 and 13 - 122 are in bank 1 \\& 123 - 167 in bank 2.
Say_0:

      Xney_say:

| LDX | \\#00 | ;no offsett |
| :-- | :-- | :-- |
| LDA | (Saysent_lo,X) | ;get data @ 16 bit adrs |

CLC
ADC Rvoice ;adjut to voice selected on power up
STA TEMP2 ;save new speech pitch
LDA \\#FEh ;command for TI to except pitch data
STA TEMP1
; The math routine converts the value to 00 for 80 and
; if 0 then subtracts from 80 to get the minus ver: a of 00
; ie, if number is 70 then TI gets sent $10(-1$
LDA TEMP2 ;get voice with offsett
BRI No_voice_chg ;if $>80$ then no char
LDA \\#80h ;remove offsett if <80
CLC
SBC TEMP2 ; kill offset
STA TEMP2 ;update
No_voice_chg:
JSR Xmit_TI ;send it to TI
Do_nextsent:
Frst_say:
INC Saysent_lo ; next saysent pointer
BNE Scnd_say ; jump if no roll over
INC Saysent_hi ; +1
Scnd_say:
LDX \\#00 ;no offsett
LDA (Saysent_lo,X) ;get data @ 16 bit adrs
CM? \\#FFH ;check for end
BEQ Say_end ; done
LDA (Saysent_lo,X) ;get data @ 16 bit adrs
STA Which_word ;
Wtest:
CLC
SBC \\#12 ;ck if in bank 1
BCS Get_group1 ; jump if is
Get_group0:
LDA \\#00 ; set bank
STA Bank_ptr ; Bank number
CLC ; clear carry
LDA Which_word ;get word
ROL A ;2's offsett
TAX ;load offset to Xreg
LDA Word_group0,X ;get lo pointer
STA Word_lo ;save
INX ;X+1
LDA Word_group0,X ;get hi pointer
STA Word_hi ;save
JMP Word_fini ; go do it
Get_group1:
LDA Which_word ; selection
CLC
SBC \\#122 ;ck if in bank 2
BCS Get_group2 ; jump if is
      | WAKE2 |  |
| :--: | :--: |
| $1$ | adds deep sleep mide. If 'Deep_sleep'=11h then tilt will not |
| 1 | wake us up. only invert. |

Power up reset decision for three types of startup:
1. Powerup with feed switch zeros ram \\& EEPROM, \\& calls 10-200-10 macro.
2. Power up from battery change wont clear EEPROM but calls 10-200-10 macro.
3. Wake up from Port_D clears ram and jumps directly to startup. No macro.

| SEI |  | :interrupts off |
| :--: | :--: | :--: |
| LDX | \\#COH | :startup setting |
| STX | Interrupts | :disable Watch Dog |
| LDX | \\#FFH | :Reset stack pointer address \\$0FFH |
| TXS |  |  |
| LDX | \\#0 |  |
| LDA | Wake_up | : Get the information from h: rdware to check |
| STA | TEMP5 | :whether reset is from power up or wakeup |
| STX | Wake_up | :disable wakeup immediately, this action can :stop the reset occupied by another changed on :portD, so once the program can execute to :this line then chip will not be reset due to :port changed again |
| AND | \\#\\#00000001 | :mask the rest of bit and just check the port |
| BEC | Power_battery | :jump to power up initial if not port D |

Need to debounce tilt and invert since they are very unstable

| Ck_wakeup: |  |  |
| :--: | :--: | :--: |
| LDA | \\#00 | :clear |
| STA | TEMP1 | : |
| STA | TEMP2 | : |
| LDX | \\#FFh | :loop counter |
| Dbnc_lp: |  |  |
| LDA | Port_D |  |
| AND | \\#01 | :ck tilt sw |
| BEQ | Dbmc_lp2 | : jump if not tilt |
| INC | TEMP1 | : switch counter |
| Dbnc_lp2: |  |  |
| LDA | Port_D |  |
| AND | \\#02 | :ck invert sw |
| BEQ | Dbmc_lp3 | : jump if not invert |
| INC | TEMP2 | : switch counter |
| Dbnc_lp3: |  |  |
| DEX |  | : -1 loop count |
| BNE | Dbnc_lp | : loop |
| LDA | Deep_sleep | : decide if normal or deep sleep |
| CM | \\#11h |  |
| BEQ | Dbmc_lp4 | : if deep sleep then only test invert |
| LDA | TEMP1 | : get tilt count |
| BEQ | Dbnc_lp4 | : jump if 0 |
| CLC |  |  |
| SBC | \\#. | : min count to insure not noise |
| BCS | Power_Port_D | : jump if > min |
      Danc_lp4:


I Verify that Port_D is no longer changing before going to sleep. If not, the CPU will lock up without setting the low power mode. Before we exit here when count is less than minimum count, we must be sure Port_D is not changing. If we jump to sleep routine when it is not stable, the sleep routine will wait forever to be stable which causes Furby appear to be locked up.

| LDA | \\#00 |  |
| :-- | :-- | :-- |
| STA | TEMP1 |  |
| LDA | Port_D | :counter |
| Test_sleep: |  |  |
| CHP | Port_D | :check if changed |
| BNE | Ck_wakeup | :start over if did |
| DEC | TEMP1 | $:-1$ counter |
| BNE | Test_sleep | :loop |
| TMP | GoToSleep_2 | :otherwise, just goto sleep again |

Power_Port_D:
LDA \\#11h
:signal port D wakeup
STA Warm_cold
JMP L_PowerOnInitial
Power_battery:
LDA \\#05h
:signal battery wakeup
STA Warm_cold
L_PowerOnInitial:
LDA \\#00
:clear deep sleep command
STA Deep_sleep
      ```
; MCDS :
; LIGHT3.asm
; Add test to light counter so that if the oscillator
; fails, the system will ignore light sensor and keep running.
; Light4
; When goes to complete dark and hits the 'Dark_sleep' level
; and stays there until the reff level updates, at that point
; we send Furby to sleep.
; Light5 (used in F-RELS2 )
; Change detection of light threshold to preven: false or continuens trigger.
Bright EQU 15 ;light sensor trigger > reff level (Hon)
Dim EQU 15 ;Light sensor trigger < reff level (Hon)
Shift_reff EQU 10 ;max count to set or clear prev done flag
Dark_sleep EQU BOh ; when timer A hi =0f and timer A low
; is = to this EQU then send him to sleep
```

; The CDS light sensor generates a square wave of 500 hz to 24 kHz based on
; light brightness. We can loop on the sense line and count time for the
; 10 period to determine if light has changed and compare it to previous
; samples. This also determines going lighter or darker. We also set a timer
; so that if someone holds their hand over the sensor and we announce it.
; if the change isnt stable for 10 second, we ignore the change back to the
; previous state. If it does exis for > 10 seconds, then it becomes the
; new sample to compare against on the next cycle.
; In order to announce light change, the system must have a consistent
; count > 'Shift_ieff'.
; If a previous reff has been set then the 'Up_light' bit is set to
; look for counts greater than the reff. The system passes through the
; light routine 'Shift_reff' times. If it is consistently greater than
; the reff level, we get a speech trigger. If any single pass is less
; than the reff, the counter is set back to zero. This scenario also
; is obeyed when the trigger goes away, ie remove your hand, and the system
; counts down to zero. ('Up_light' bit is cleared) If during this time any
; trigger greater than reff occurs, the count is set back to max.
; This should prevent false triggers.

Get_light: ; alt entiy for diagnostics
; This uses timer A to get a count from the 10 period of the clk

| SEI |  | : interrupts off |
| :-- | :-- | :-- |
| LDA | \\#0C0H | ;disable timer, clock, ext ints, |
| STA | Interrupts | ; \\& watchdog; select IRQ int. |
| LDA | \\#000H | ; set timer A for timer mode |
| STA | TMA_CON | ; |
      # Light5.asm 

| LDA | \\#000H | ;re-start timer A |
| :--: | :--: | :--: |
| STA | TMA_LSB | ; |
| LDA | \\#000H | ; now CPUCLK; was \\#010H = CPUCLK/4 (Hon) |
| STA | TMA_MSB | ; |
| Ck_1ght2: |  | ; test for dead light osc |
| LDA | TMA_MSB | ; get timer |
| AND | \\#0Fh | ; ck for > 0E |
| CMP | \\#0Fh | ; ck for > OE |
| BNE | Ck_1t2a | ; jump if not |
| LDA | TMA_LSB | ; get 10 byte |
| CLC |  |  |
| SBC | \\#E0h | ; ck for > ;msb+1sb =0FE0) |
| BCC | Ck_1t2a | ; jump if not |
| JMP | L1ight_fail | ; bail out if > |
| Ck_1t2a: |  |  |
| LDA | Port_D | ; get I/O |
| AND | \\#Light_in | ; ck light clk is hi |
| BEQ | Ck_1ght2 | ; wait for it to go hi |
| LDA | \\#000H | ;re-start timer A |
| STA | TMA_LSB | ; |
| LDA | \\#C0H | ; now CPUCLK; was \\#010H = CPUCLK/4 (Hon) |
| STA | A_MSB | ; |
| Ck_1ght3: |  |  |
| LDA | TMA_MSB | ; test for dead light osc |
| AN | \\#0Fh | ; get timer |
| Chy | \\#0Fh | ; ck for > OE |
| BNE | Ck_1t3a | ; jump if not |
| LDA | TMA_LSB | ; get 10 byte |
| CLC |  |  |
| SBC | \\#E0h | ; ck for > (msb+1sb =0FE0) |
| BCS | Light_fail | ; bail out if > |
| Ck_1t3a: |  |  |
| LDA | Port_D | ; get I/O |
| AND | \\#Light_in | ; ck light clk is 10 |
| BNE | Ck_1ght3 | ; wait for it to go 10 to insure the clk edge |
| Ck_1ght4: |  |  |
| LDA | \\#000H | ;re-start timer A |
| CTA | TMA_LSB | ; |
| LDA | \\#000H | ; now CPUCLK; was \\#010H = CPUCLK/4 (Hon) |
| STA | TMA_MSB | ; |
| Ck_1ght4a: |  |  |
| LDA | Port_D | ; get I/O |
| AND | \\#Light_in | ; ck if still 10 |
| BEQ | Ck_1ght4a | ; loop till hi |
| ; Timer A holds count for 10 period of clk |  |  |
| Light4cmp: |  |  |
| LDA | TMA_MSB | ; get timer high byte |
| AND | \\#00FH | ; mask out high nybble |
| STA | TEMF2 | ; and save it |
| LDA | TMA_LSB | ; get timer low byte |
| STA | TEMF1 | ; and save it |
| LDA | TMA_MSB | ; get timer A high byte anain |
      # Light5.asm 

| LDA | Stat_1 | : system |
| :--: | :--: | :--: |
| AND | \\#Up_1ight | :ck if incrmnt mode |
| BNE | Rat_whltup | : jump if incrmnt mode |
| LDA | \\#Shift_reff | : set to max |
| STA | Light_shift | : |
| JMP | No_lt_todo |  |
| Rat_whltup: |  |  |
| INC | Light_shift | $: 11$ |
| LDA | Light_shift | :get counter |
| CLC |  |  |
| SBC | \\#Shift_reff | :ck if > max reff count |
| BCC | No_lt_todo | :jump if < max count |
| LDA | \\#Shift_reff | :reset to max |
| STA | Light_shift | : |
| LDA | Stat_0 | : system |
| AND | \\#Lt_prev_dn | :check if previously done |
| BNE | New_ltreff | : jump if was |
| LDA | Stat_0 | : system |
| ORA | \\#Lt_prev_dn | : set previously done |
| STA | Stat_0 | :update |
| LDA | Stat_1 | : system |
| : AND | \\#EPh | : set sytem to shift decrmnt mode |
| STA | Stat_1 | : uj fate |
| LDA | \\#Light_reload | :reset for next trigger |
| STA | Light_timer | : set it |
| JMP | Do_ltchg | :go announce it |
| New_ltreff: |  |  |
| LDA | Light_timer | : get current |
| BNE | No_lt_todo | :nothing to do |
| LDA | TEMP1 | : get new count |
| STA | Light_reff | :update system |
| -DA | Stat_1 | : system |
| AND | \\#EPh | : set sytem to shift decrmnt mode |
| STA | Stat_1 | :update |
| LDA | TEMP1 | : get current value |
| CLC |  |  |
| SBC | \\#Dark_sleep | :ck if > sleep level |
| BCS | Ck_drk | : jump if > |
| LDA | Stat_0 | : system |
| AND | \\#7Ph | : kill prev done |
| STA | Stat_0 | :update |
| JMP | Kill_ltrf | : |
| Ck_drk: |  |  |
| LDA | Stat_0 | : system |
| AND | \\#D rk_sleep_ prev | : ck if this was already done |
| BNE | Kill_ltrf | :jump if was |
| LDA | Stat_0 | : system |
| ORA | \\#REQ_dark_sleep | : set it |
| ORA | \\#Dark_sleep_ prev | : set also |
| STA | Stat_0 | :update |

Kill_ltrf:
      Light5.asm

|  | LDA | Stat_0 | ; system |
| :--: | :--: | :--: | :--: |
| 1 | AND | \\#Lt_prev_dn | ; check if previously done |
| 1 | BEQ | No_lt_‘ado | ; jump if clear |
|  | LDA | Light_shift | ; get shift counter |
|  | BEQ | Kill_shift | ; jump if went zero last time |
|  | LDA | Stat_1 | ; system |
|  | AND | \\#Up_light | ;ck if incrmnt mode |
|  | BEQ | Rot_shttdn | ; jump if decrmnt mode |
|  | LDA | \\#00 | ; set to min |
|  | STA | Light_shift | ; |
|  | JMP | No_lt_todo | ; |
| Rst_shttdn: |  |  |  |
|  | DEC | Light_shift | ;-1 |
|  | JMP | No_lt_todo | ; done |
| Kill_shift: |  |  |  |
|  | LDA | Stat_0 | ; system |
|  | AND | \\#FDH | ;clears Lt_prev_dn |
|  | STA | Stat_0 | ;update |
|  | LDA | Stat_1 | ; system |
|  | ORA | \\#Up_light | ; prepare to incrmnt 'Light_shift' |
|  | STA | Stat_1 | ;update |

No_lt_todo:
SEC
RTS
; carry set indicates no light change
; carry set
; carry set indicates no light change

| ; | alert system to start speech |  |
| :--: | :--: | :--: |
| Do_ltchg: |  |  |
|  | LDA | Stat_3 |
|  | AND | \\#Light_stat |
|  | BNE | LT_ref_brt |
|  | LDA | Stat_4 |
|  | ORA | \\#Do_1ght_dim |
|  | JMP | Ltref_egg |
| LT_ref_brt: |  |  |
|  | LDA | Stat_4 |
|  | ORA | \\#Do_1ght_brt |
| Ltref_egg: |  |  |
|  | STA | Stat_4 |
|  | CLC |  |
|  | RTS |  |

; system
; ck if went light or dark
; went brighter if set
; get system
; set indicating change < reff level
;
; set indicating change > reff level
; update egg info
; carry clear indicates light > reff
; done
      Diag7.asm
;F:IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
      # Diag7.asm 


      | CLI |  | ; enablu IRQ |  |
| :--: | :--: | :--: | :--: |
| JSR | Kick JRQ | ; wait for time: | s-sync |
| JSR | TI_reset | ; clear TI from |  |
| T.A | Feed_count | ; get 10 byte of macro to call |  |
| JSR | Diag macro | ; go send motor/speech |  |
| Diag2b: | ; Speaker tone | I.R. xmit |  |
|  | Port_C | ; get I/O |  |
| AND | \\#Touch_bck | ; wait for switch |  |
| BNL | Diag2c | ; go check if next test is requesting |  |
| LDA | \\#1 | ; hi beep for start of test |  |
| JSR | Piag macro | ; go send motor/speech |  |
| Diag2blp: |  |  |  |
| LDA | Port_C |  |  |
| AND | \\#Touch_bck |  |  |
| BEQ | Diag2blp |  |  |
| Diag2b1: |  |  |  |
| LDA | \\#04 | ; send long tone (1k sinewave) |  |
| JSR | Diag macro | ; go send motor/speech |  |
| LDA | Port_C | ; |  |
| AND | \\#Touch_bck | ; mask for back switch |  |
| BNE | Diag2b1 | ; loop until back switch pressed |  |
| Xmit_lp: |  |  |  |
| LDA | \\#01 | ; beep |  |
| JSR | Diag macro | ; go send motor/speech |  |
| LDA | Port_C | ; |  |
| AND | \\#Touch_bck | ; mask for back switch |  |
| BNE | Xmit_lp | ; loop until back switch pressed |  |
| LDA | \\#05h | ; send '5' to I.R. xmiter |  |
| STA | TEMP2 | ; |  |
| LDA | \\#F1h | ; send command I.R. to TI |  |
| STA | TEMP1 | ; |  |
| JSR | Xmit_TI | ; send it |  |
| dumb: | LDA | Port_C | ; get I/O |
|  | AND | \\#Touch_bck | ; wait for switch |
|  | BNE | dumb | ; wait for back to be pressed |
| dumber: | LDA | Port_C | ; get I/O |
|  | AND | \\#Touch_frnt | ; ck switch |
|  | BEQ | Next_1 |  |
|  | JMP | Xmit_lp |  |
| Next_1: | LDA | \\#2 | ; hi beep for start of test |
|  | JSR | Diag macro | ; go send motor/speech |
|  | LDA | Port_C | ; get I/O |
|  | AND | \\#0Ch | ; ck for front and back switches made |
|  | BEQ | Next_1 | ; if both not 10 then bail out else start diag |
|  | JMP | New_top |  |
| ; Full test starts here |  |  |  |
| Diag2c: | LDA | Port_D | ; get I/O |
|  | AND | \\#Ball invert | ; wait for switch |
|  | BNE | Diag2d | ; onward if key pressed |
      # Diag7.asm 

JMP Diag2a ; loop back to top if none
Diag2d:
LDA \\#01 ; hi beep for start of test
JSR Diag_macro ; go send motor/speech
; FULL TEST MODE

DiagF1:
LDA
STA
DiagFla:
LDA
AND
BNE
TC
BNE
LDA
JSR
DiagF2:
LDA
AND
CMP
BEQ
LDA
JSR
; wait for no tilt to start full diag
\\#Dwait_tilt ; set delay to be sure no tilts
\\#Dwait_tilt
; set delay to be sure no tilts
\\#T
;
Port_D
\\#3
DiagF1
\\#3
; pass beep
; go send motor/speech
; test tilt 45 deg
Port_C
\\#00001100b
\\#0CN
DiagF22
; fail beep
Diag_macro
DiagF23:
LDA
AND
BEQ
DiagF23
; fail beep
Diag_macro
DiagF23:
LDA
AND
BEQ
DiagF2
; get I/O
; ck for tilt switch (hi = tilted)
wait for tilt
; get I/O
; ck if invert sw made
; jump to error if so
; get I/O
; get front \\& back
; must be hi else error
; if hi then pass
DiagF2a:
LDA \\#3
; fail beep
JSR Diag_macro
; go send motor/speech
JMP DiagF2
; loop till no error
DiagF2b:
LDA \\#2
; pass beep
JSR Diag_macro
; go send motor/speech
DiagF2c:
; wait for no tilt before continuing
      # Diag7.asm 

| LDA | Port_C |
| :-- | :-- |
| AND | \\#Touch_bck |
| BEQ | DiagF3b |

LDA Port_D
:get I/O
AND
:ck for tilt switch (hi = tilted)
: ck for tilt switch (hi = tilted)
BNE DiagF2c
wait for no tilt
(DANGER
LDA
: 3
AND
BEQ
: 3
: test back switch
: 1
: 1
: 1
: 1
DiagF3:
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3
: 3

      DiagF4a2:
LDA
AND
BEQ
LDA
AND
ORA
STA
LDA
ORA
AND
AND
STA

Port_C
$\\square$
DiagF4a2
Stat_2
$\\square$
$\\square$
$\\square$
Port_C
$\\square$
$\\square$
$\\square$

Port_C
$\\square$
$\\square$

|  |  |  |
| :-- | :-- | :-- |
|  |  |  |

get I/O wait for front to clear
;ck switch
;if pressed then wait for release
; get system
; clear fwd flag
; set motor in motion
;update system
; get current status
; turn both motors off
; move motor in rev dir
; get I/O wait for front
;ck switch
; got it
; loop till found
; Send motor end to end and stop on cal sw, else error
DiagF4a4:
LDA
ORA
STA
LDA
LDA
JSR
; get current status
; turn both motors off
;update
; get system
; clear activ flag
; update system
; start motor test
; go
; set delay for motor to stop
; $A$ * half sec delay
; get I/O
; to when hit
;no position switch found
; pass beep
; go send it
; done
DiagF4b:
LDA
JSR
Diag macro
; fail beep
; go send it
; send motor to mouth open for feed sw test
Port_C ; get I/O
; wait for switch
DiagF5
; loop
LDA
; feed position
JSR
Diag macro
; send it
DiagF6:
; ck for feed sw, all other sw = error
; Remember to test invert before setting feed sw test, else conflict.
LDA
STA
LDA
AND
CMP
BNE

## $\\# 00$

STA
Port_C
$\\square$
$\\square$
; clear feed sw enable
Port_C
; get I/O
; ck for front and back switches made
; ck both are clear
; wait till are
      # Diag7.asm 

| LDA | Port_D | ; get I/O |
| :--: | :--: | :--: |
| AND | \\#03 | ; ck fuc tilt and invert |
| BNL | DiagF a | ; if either hi then wait till clear |
| JMP | DiagF6b | ; jump when all clear |
| DiagF6a: |  |  |
| LDA | \\#3 | ; fail beep when any other switch made |
| JSR | Diag macro | ; send it |
| JMP | DiagF6 | ; loop |
| DiagF6b: |  |  |
| ; mod diag6 ; inc random number seeds until feed switch down |  |  |
| INC | Seed_1 | ; create random based on switches |
| LDA | TMA LSB | ; get timer A also (should be unknown) |
| STA | Seed_2 | ; save it |
| ;end mod |  |  |
| LDA | \\#FFh | ; turn DAC2 on to enaule feed switch |
| STA | DAC2 | ; out |
| LDA | Port_D | ; get I/O |
| AND | \\#Ball_invert | ; ck if feed switch closed |
| BEQ | DiagF6 | ; loop until switch closed |
| LDA | \\#00 |  |
| STA | DAC2 | ; clear feed sw enable |
| LDA | \\#7 | ; pass beep |
| JSR | Diag macro | ; go send motor/speech |
| DiagF7: | ; Light sensor test |  |
| ; mod to compensate for new light sense routine |  |  |
| LDA | \\#00 | ; clear light timer to force new reff cycle |
| STA | Light timer | ; set it |
| LDA | Stat_3 | ; get system |
| ORA | \\#Lt reff | ; make this pass a new light reff |
| STA | Stat_3 | ; update |
| JSR | Get_1ight | ; go get light level, establish 1st level |
| LDA | Stat_4 | ; |
| AND | \\#Nt_do_lt_dim | ; clear indicating change > reff level |
| STA | Stat_4 | ; update system |
| JSR | Get_1ight | ; go get light level sample |
| LDA | TEMPI | ; get new count |
| STA | Light_reff | ; update system |

Diag7a:
JSR Get_1ight ; go get again and test for lower level
LDA Stat_4 ; get system
AND \\#Do_1ght_dim ; check if went dimmer
BEQ DiagF7a ; loop if no change
LDA \\#8 ;pass beep and motor motion
JSR Diag macro ; send it
DiagF8:
; Sound sensor test
LDA \\#OO
; clear sound timer to force new reff cycle
STA Sound_timer ; set
LDA Stat_1 ; get system again
ORA \\#End_reff ; make this pass a new sound reff
      | STA | Stat_1 | ;update |
| :--: | :--: | :--: |
| JER | Get_sound | ;go get light level, establish 1st level |
| LDA | Stat_4 | ; |
| AND | \\#Nt_do_and | ;clear indicating change > reff level |
| STA | Stat_4 | ;update system |
| DiagF8a: |  |  |
| JER | Get_sound | ;go get again and test for lower level |
| LDA | Stat_4 | ;get system |
| ANJ | \\#Do_and | ; check if went louder |
| BEQ | DiagF8a | ; loop if no change |
| LDA | \\#9 | ;pass beep and motor motion |
| JER | Diag_macro | ; send it |
| DiagF9: | ;wait for I.R. data received |  |
| LDX | \\#10 | ; ;Tracker change, orginal is 100 |
| DiagF9al: |  |  |
| LDA | \\#1 |  |
| JER | Half_delay |  |
| DEX |  |  |
| BNE | DiagF9al |  |
| JER | D_IR_test | ;go ck for data |
| BCC | DiagF9 | ; ; loop until data receive |
| CMP | \\#A5H | ; is it the expected data |
| BNE | DiagF9a | ; jump if wrong data |
| LDA | \\#1 | ; pass beep and motor motion |
| JER | Diag_macro | ; send it |
| JMP | DiagF10 | ; done |
| DiagF9a: |  |  |
| LDA | \\#3 | ; fail beep and motor motion |
| JER | Diag_macro | ; send it |
| DiagF10: | ; all tests complete, send to sleep mode |  |
| LDA | \\#10 | ; |
| JER | Half_delay | ; |
| LDA | \\#10 | ; put furby in sleep postion |
| JER | Diag_macro | ; send it |
| ; Clear RAM to 00 H |  |  |
| ; we dont clear Seed_1 or Seed_2 since they are randomized at startup. |  |  |
| LDA | \\#00H | ; data for fill |
| LDX | \\#D7h | ; start at ram location |
| Clear: |  |  |
| STA | 00,X | ; base 00, offset $x$ |
| DEX |  | ; next ram location |
| CPX | \\#7FH | ; check for end |
| BNE | Clear | ; branch, not finished |

Random voice selection here
LDA \\#80h
; get random/sequential split
      # Diag7.asm 

| STA | IN_DAT | ;save for random routine |
| :--: | :--: | :--: |
| LDX | \\#00 | ;make sure only gives random |
| LDA | \\#10h | ; get number of random selections |
| JSR | Ran_seq | ;go get random selection |
| TAX |  |  |
| LDA | Voice_table, X | ; get new voice |
| STA | Rvoice | ; set new voice pitch |

On power up or reset, Furby must go select a new name , . ., ahw how cute.

| JSR | Random |  |
| :-- | :-- | :-- |
| AND | \\#1Fh | ; get 32 possible |
| STA | Name | ; set new name pointer |


| LDA | \\#FFh | ; insure not hungry or sick |
| :-- | :-- | :-- |
| STA | 'ungry_counter | ; max not hungry |
| STA | ick_counter | ; Max not sick |

; Clear training on all sensors

| LDA | \\#00 |
| :-- | :-- |
| STA | iw.p_ID |
| STA | Temp_ID2 |
| STA | Tilt_learned |
| STA | Tilt_lrn_cnt |
| STA | Feed_learned |
| STA | Feed_lrn_cnt |
| STA | Light_learned |
| STA | Light_lrn_cnt |
| STA | Dark_learned |
| STA | Dark_lrn_cnt |
| STA | Front_learned |
| STA | Front_lrn_cnt |
| STA | Sound_learned |
| STA | Sound_lrn_cnt |
| STA | Wake_learned |
| STA | Wake_lrn_cnt |
| STA | Invert_learned |
| STA | Invert_lrn_cnt |
| JMP | GoToSleep | ; write ee memory YO |
      Diag7.asm
      Furby27.inc ; ; change twinkle egg song to one pass in macro

; Lowered voice $+10$, voice +9 to voice +8
; Wayne's mods:
; Furby5b.inc = add voice selection table
; Dave's
; added feed (mouth open)
; $170,171,173,174,175,182,183,190,191,194$
; mod fo; ir
; NOW 24 NAMES
; TABLES
; FRONT
; PORTUNE
; o-too-mah
; HANGOUT
; delay
; FEED
; WAKE
; HUNGER
; INVERT
; BACK
; SICK
; LIGHT
; DARK
; SOUND
; TILT
; IR
; FURBY SAYS
; 435,436
; 95,96,97
; 439
; Diagnostic
; Names
; 451,452
; Names
; 454
; 455
; 456
; 457
; 458
; 459
; 460
; NEW EASTER EGGS
; 468
; 469
; 470
; 471
; 472
; 46

MACRO
$2-64$
$65-83$
84
$85-101$
102
$103-145$
$146-169$
$170-201$
202-238
239-275
276-292
293-307
308-331
332-351
352-392
393-429
430-434
435,436
437,438
95,96,97
439
440-450
451,452
453
454
455
456
457
458
459
460
461
462
463
464
465
466
467
; DODLE DO, ME LOVE YOU
; SING A SONG
; BURB ATTACK
; furby says win sound
; furby says lose sound
      

# ; Sensor tables 

; Each sensor has 4 speech/motor tables based on age 1-4, of 16 entries each.
; These tables are 16 bit entries, the user enters as a decimal 1-511
; **** '00' is illegal ****
; This number calls the MACRO tables to get specific speech and motor
; tables. MACRO tables chain together multiple motor and speech tables.
; The first 8 entries of speech is random selections and
; the second 8 entries is sequential.
; one of three voice pitc: selections, randomly load table and
; table is randomly called on power up to select a new voice.
; THIS gives a number added to voice 3 to create which voice will be
      | Table | Description | Page |
|--------|-----------------------------|-------|
|  | **Voice_table:** | 352 |
|  | **DB** | 353 |
|  | **DB** | 354 |
|  | **DB** | 352 |
|  | **DB** | 355 |
|  | **DB** | 356 |
|  | **DB** | 357 |
|  | **DB** | 358 |
|  | **DW** | 359 |
|  | **DW** | 360 |
|  | **DW** | 361 |
|  | **DW** | 362 |
|  | **DW** | 363 |
|  | **DW** | 364 |
|  | **DW** | 365 |
| **Tilt_S2:** | **DW** | 366 |
|  | **DW** | 367 |
|  | **DW** | 366 |
|  | **DW** | 355 |
|  | **DW** | 368 |
|  | **DW** | 357 |
|  | **DW** | 369 |
|  | **DW** | 370 |
|  | **DW** | 359 |
|  | **DW** | 360 |
|  | **DW** | 371 |
|  | **DW** | 372 |
|  | **DW** | 373 |
|  | **DW** | 374 |
|  | **DW** | 355 |
|  | **DW** | 375 |
| **Tilt_S3:** | **DW** | 366 |
|  | **DW** | 355 |

; **#1** AGE 1
; **#2** AGE 1
; **#3** AGE 1
; **#4** AGE 1
; **#5** AGE 1
; **#6** AGE 1
; **#7** AGE 1
; **#8** AGE 1
; **#9** AGE 1
; **#10** AGE 1
; **#11** AGE 1
; **#12** AGE 1
; **#13** AGE 1
; **#14** AGE 1
; **#15** AGE 1
; **#16** AGE 1

; **#1** AGE 2
; **#2** AGE 2
; **#3** AGE 2
; **#4** AGE 2
; **#5** AGE 2
; **#6** AGE 2
; **#7** AGE 2
; **#8** AGE 2
; **#9** AGE 2
; **#10** AGE 2
; **#11** AGE 2
; **#12** AGE 2
; **#13** AGE 2
; **#14** AGE 2
; **#15** AGE 2
; **#16** AGE 2

; **#1** AGE 3
; **#2** AGE 3
      
; SNITCH FOR DO SOUND) is
Sound_S1: DW 332
DW 333
DW 334
DW 335
DW 336
DW 337
DW 338
DW 339
DW 332
DW 333
DW 334
      | DW | 335 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | 
      | DO HUBGER |  |  |
| :--: | :--: | :--: |
|  |  |  |
| Hunger_S1: |  |  |
| DW | 170 | : \\#1 AGE 1 |
| DW | 173 | : \\#2 AGE 1 |
| DW | 176 | : \\#3 AGE 1 |
| DW | 180 | : \\#4 AGE 1 |
| DW | 182 | : \\#5 AGE 1 |
| DW | 173 | : \\#6 AGE 1 |
| DW | 165 | : \\#7 AGE 1 |
| DW | 189 | : \\#8 AGE 1 |
| DW | 193 | : \\#9 AGE 1 |
| DW | 194 | : \\#10 AGE 1 |
| DW | 173 | : \\#11 AGE 1 |
| DW | 195 | : \\#12 AGE 1 |
| DW | 189 | : \\#13 AGE 1 |
| DW | 193 | : \\#14 AGE 1 |
| DW | 194 | : \\#15 AGE 1 |
| DW | 199 | : \\#16 AGE 1 |
| Hunger_S2: |  |  |
| DW | 171 | : \\#1 AGE 2 |
| DW | 174 | : \\#2 AGE 2 |
| DW | 177 | : \\#3 AGE 2 |
| DW | 181 | : \\#4 AGE 2 |
| DW | 183 | : \\#5 AGE 2 |
| DW | 174 | : \\#6 AGE 2 |
| DW | 186 | : \\#7 AGE 2 |
| DW | 190 | : \\#8 AGE 2 |
| DW | 193 | : \\#9 AGE 2 |
| DW | 194 | : \\#10 AGE 2 |
| DW | 174 | : \\#11 AGE 2 |
| DW | 196 | : \\#12 AGE 2 |
| DW | 190 | : \\#13 AGE 2 |
| DW | 193 | : \\#14 AGE 2 |
| DW | 194 | : \\#15 AGE 2 |
| DW | 200 | : \\#16 AGE 2 |
| Hunger_S3: |  |  |
| DW | 172 | : \\#1 AGE 3 |
| DW | 174 | : \\#2 AGE 3 |
| DW | 178 | : \\#3 AGE 3 |
| DW | 181 | : \\#4 AGE 3 |
| DW | 184 | : \\#5 AGE 3 |
| DW | 175 | : \\#6 AGE 3 |
| DW | 187 | : \\#7 AGE 3 |
| DW | 191 | : \\#8 AGE 3 |
| DW | 193 | : \\#9 AGE 3 |
| DW | 173 | : \\#10 AGE 3 |
| DW | 175 | : \\#11 AGE 3 |
| DW | 197 | : \\#12 AGE 3 |
| DW | 191 | : \\#13 AGE 3 |
| DW | 193 | : \\#14 AGE 3 |
| DW | 193 | : \\#15 AGE 3 |
| DW | 200 | : \\#16 AGE 3 |
| Hunger_84: |  |  |
| DW | 171 | : \\#1 AGE 4 |
| DW | 175 | : \\#2 AGE 4 |
      

MACRO 65-83,SAY 62-78
Fortyes_81:

| DW | 065 |
| :-- | :-- |
| DW | 066 |
| DS | 067 |
| DW | 068 |
| DW | 069 |
| DW | 070 |
| DW | 071 |
| DW | 072 |
| DW | 073 |
| DW | 074 |
| DW | 075 |
| DW | 076 |
| DW | 077 |
| DW | 078 |
| DW | 079 |
| DW | 080 |

Fortyes_S2:

| DW | 081 |
| :-- | :-- |
| DW | 082 |
| DW | 083 |
| DW | 065 |
| DW | 066 |
| DW | 067 |
| DW | 068 |
| DW | 069 |
| DW | 070 |
| DW | 071 |
| DW | 072 |
| DW | 073 |
| DW | 074 |
| DW | 075 |
| DW | 076 |
| DW | 077 |

; END PORTUNE
; END GEORGE 07/04/98
;
      ;touch front sensor table
;GEORGE 07/03/98 MACRO 2-64;SAY 1-61
Tfrnt_S1: DW 002
DW 003
DW 004
DW 005
DW 006
DW 007
DW 008
DW 0 9
DW 10
DW 11
DW 12
DW 013
DW 014
DW 015
DW 016
DW 017
Tfrnt_S2: DW 018
DW 019
DW 020
DW 021
DW 022
DW 023
DW 024
DW 025
DW 026
DW 027
DW 028
DW 029
DW 030
DW 031
DW 032
DW 033
; \\#1 AGE 1
; \\#2 AGE 1
; \\#3 AGE 1
; \\#4 AGE 1
; \\#5 AGE 1
; \\#6 AGE 1
; \\#7 AGE 1
; \\#8 AGE 1
; \\#9 AGE 1
; \\#10 AGE 1
; \\#11 AGE 1
; \\#12 AGE 1
; \\#13 AGE 1
; \\#14 AGE 1
; \\#15 AGE 2
; \\#16 AGE 2

Tfrnt_S3: DW 034
DW 035
DW 036
DW 037
DW 038
DW 039
DW 040
DW 041
DW 002
DW 042
DW 043
DW 044
DW 045
DW 046
DW 047
DW 048
Tfrnt_S4: DW 049
DW 050
DW 051
DW 052
DW 053
DW 054
DW 055

; \\#1 AGE 3
; \\#2 AGE 3
; \\#3 AGE 3
; \\#4 AGE 3
; \\#5 AGE 3
; \\#6 AGE 3
; \\#7 AGE 3
; 025 ; \\#8 AGE 3
; \\#9 AGE 3
; \\#10 AGE 3
; \\#11 AGE 3
; \\#12 AGE 3
; \\#13 AGE 3
; \\#14 AGE 3
; \\#15 AGE 3
; \\#16 AGE 3

; \\#1 AGE 4
; \\#2 AGE 4
; \\#3 AGE 4
; \\#4 AGE 4
; \\#5 AGE 4
; \\#6 AGE 4
; \\#7 AGE 4
      | DW | 056 |
| :-- | :-- |
| DW | 057 |
| DW | 058 |
| DW | 059 |
| DW | 060 |
| DW | 061 |
| DW | 062 |
| DW | 063 |
| DW | 064 |
|  |  |

; END GEORGE 07/03/98
; 8 AGE 4
; 99 AGE 4
; 10 AGE 4
; 11 AGE 4
; 12 AGE 4
; 13 AGE 4
; 14 AGE 4
; 15 AGE 4
; 16 AGE 4

Feed_s2:

| DW | 118 |
| :-- | :-- |
| DW | 119 |
| DW | 120 |
| DW | 121 |
| DW | 122 |
| DW | 123 |
| DW | 124 |
| DW | 125 |
| DW | 126 |
| DW | 127 |
| DW | 128 |
| DW | 113 |
| DW | 114 |
| DW | 111 |
| DW | 129 |
| DW | 116 |

Feed_S3:

| DW | 118 |
| :-- | :-- |
| DW | 130 |
| DW | 131 |
| DW | 132 |
| DW | 122 |

; 1 AGE 1
; 2 AGE 1
; 3 AGE 1
; 4 AGE 1
; 5 AGE 1
; 6 AGE 1
; 7 AGE 1
; 8 AGE 1
; 9 AGE 1
; 10 AGE 1
; 11 AGE 1
; 12 AGE 1
; 13 AGE 1
; 14 AGE 1
; 15 AGE 1
; 16 AGE 1
      | DW | 107 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | 
      GEORGE 07/07/98
; INVERT
; Ball invert sensor table

|  |  |  |  |  |  |
| :--: | :--: | :--: | :--: | :--: | :--: |
| In | DW | 202 |  | \\#1 | AGE 1 |
|  | DW | 203 |  | \\#2 | AGE 1 |
|  | DW | 206 |  | \\#3 | AGE 1 |
|  | DW | 208 |  | \\#4 | AGE 1 |
|  | DW | 212 |  | \\#5 | AGE 1 |
|  | DW | 213 |  | \\#6 | AGE 1 |
|  | DW | 217 |  | \\#7 | AGE 1 |
|  | DW | 219 |  | \\#8 | AGE 1 |
|  | DW | 220 |  | \\#9 | AGE 1 |
|  | DW | 224 |  | \\#10 | AGE 1 |
|  | DW | 228 |  | \\#11 | AGE 1 |
|  | DW | 232 |  | \\#12 | AGE 1 |
|  | DW | 234 |  | \\#13 | AGE 1 |
|  | DW | 232 |  | \\#14 | AGE 1 |
|  | DW | 234 |  | \\#15 | AGE 1 |
|  | DW | 235 |  | \\#16 | AGE 1 |
|  |  |  |  |  |  |
| In | DW | 202 |  | \\#1 | AGE 2 |
|  | DW | 203 |  | \\#2 | AGE 2 |
|  | DW | 207 |  | \\#3 | AGE 2 |
|  | DW | 209 |  | \\#4 | AGE 2 |
|  | DW | 212 |  | \\#5 | AGE 2 |
|  | DW | 214 |  | \\#6 | AGE 2 |
|  | DW | 217 |  | \\#7 | AGE 2 |
|  | DW | 219 |  | \\#8 | AGE 2 |
|  | DW | 221 |  | \\#9 | AGE 2 |
|  | DW | 225 |  | \\#10 | AGE 2 |
|  | DW | 229 |  | \\#11 | AGE 2 |
|  | DW | 232 |  | \\#12 | AGE 2 |
|  | DW | 234 |  | \\#13 | AGE 2 |
|  | DW | 232 |  | \\#14 | AGE 2 |
|  | DW | 234 |  | \\#15 | AGE 2 |
|  | DW | 236 |  | \\#16 | AGE 2 |
|  |  |  |  |  |  |
| In | DW | 202 |  | \\#1 | AGE 3 |
|  | DW | 204 |  | \\#2 | AGE 3 |
|  | DW | 207 |  | \\#3 | AGE 3 |
|  | DW | 210 |  | \\#4 | AGE 3 |
|  | DW | 212 |  | \\#5 | AGE 3 |
|  | DW | 215 |  | \\#6 | AGE 3 |
|  | DW | 218 |  | \\#7 | A4. 3 |
|  | DW | 219 |  | \\#8 | AGE 3 |
|  | DW | 222 |  | \\#9 | AGE 3 |
|  | DW | 226 |  | \\#10 | AGE 3 |
|  | DW | 230 |  | \\#11 | AGE 3 |
|  | DW | 232 |  | \\#12 | AGE 3 |
|  | DW | 234 |  | \\#13 | AGE 3 |
|  | DW | 232 |  | \\#14 | AGE 3 |
|  | DW | 234 |  | \\#15 | AGE 3 |
|  | DW | 237 |  | \\#16 | AGE 3 |
|  |  |  |  |  |  |
| In | DW | 202 |  | \\#1 | AGE 4 |
|  | DW | 205 |  | \\#2 | AGE 4 |
|  | DW | 207 |  | \\#3 | AGE 4 |
|  | DW | 211 |  | \\#4 | AGE 4 |
      | $D^{21}$ | 212 |  | 5 | AGE 4 |
| :--: | :--: | :--: | :--: | :--: |
| D-1 | 216 |  | 6 | AGE 4 |
| DW | 218 |  | 7 | AGE 4 |
| DW | 219 |  | 8 | AGE 4 |
| DW | 223 |  | 9 | AGE 4 |
| DW | 227 |  | 10 | AGE 4 |
| DW | 231 |  | 11 | AGE 4 |
| DW | 233 |  | 12 | AGE 4 |
| DW | 231 |  | 13 | AGE 4 |
| DW | 233 |  | 14 | AGE 4 |
| DW | 234 |  | 15 | AGE 4 |
| DW | 238 |  | 16 | AGE 4 |

;GEORGE 07/07/98
; BACK
; touch back sensor table

      | ; DO LIGHT DARKER |  |  |  |  |
| :--: | :--: | :--: | :--: | :--: |
| Dark_S1: | DW | 308 |  | 1 AGE 1 |
|  | DW | 309 |  | 2 AGE 1 |
|  | DW | 310 |  | 3 AGE 1 |
|  | DW | 311 |  | 4 AGE 1 |
|  | DW | 312 |  | 5 AGE 1 |
|  | DW | 313 |  | 6 AGE 1 |
|  | DW | 314 |  | 7 AGE 1 |
|  | DW | 315 |  | 8 AGE 1 |
|  | DW | 308 |  | 9 AGE : |
|  | DW | 309 |  | 10 AGE 1 |
|  | DW | 310 |  | 11 AGE 1 |
|  | DW | 311 |  | 12 AGE 1 |
|  | DW | 312 |  | 13 AGE 1 |
|  | DW | 313 |  | 14 AGE 1 |
|  | DW | 314 |  | 15 AGE 1 |
|  | DW | 315 |  | 16 AGE 1 |
| Dark_S2: |  |  |  |  |
|  | DW | 316 |  | 1 AGE 2 |
|  | DW | 317 |  | 2 AGE 2 |
|  | DW | 318 |  | 3 AGE 2 |
|  | DW | 319 |  | 4 AGE 2 |
|  | DW | 311 |  | 5 AGE 2 |
|  | DW | 319 |  | 6 AGE 2 |
|  | DW | 313 |  | 7 AGE 2 |
|  | DW | 320 |  | 8 AGE 2 |
|  | DW | 315 |  | 9 AGE 2 |
|  | DW | 316 |  | 10 AGE 2 |
|  | DW | 317 |  | 11 AGE 2 |
|  | DW | 318 |  | 12 AGE 2 |
|  | DW | 311 |  | 13 AGE 2 |
|  | DW | 319 |  | 14 AGE 2 |
|  | DW | 313 |  | 15 AGE 2 |
|  | DW | 320 |  | 16 AGE 2 |
|  | DW | 315 |  | 17 AGE 3 |
| Dark_S3: | DW | 321 |  | 2 AGE 3 |
|  | DW | 322 |  | 3 AGE 3 |
|  | DW | 323 |  | 4 AGE 3 |
|  | DW | 311 |  | 5 AGE 3 |
|  | DW | 319 |  | 6 AGE 3 |
|  | DW | 313 |  | 7 AGE 3 |
|  | DW | 324 |  | 8 AGE 3 |
|  | DW | 325 |  | 9 AGE 3 |
|  | DW | 321 |  | 10 AGE 3 |
|  | DW | 322 |  | 11 AGE 3 |
|  | DW | 323 |  | 12 AGE 3 |
|  | DW | 311 |  | 13 AGE 3 |
|  | DW | 319 |  | 14 AGE 3 |
|  | DW | 313 |  | 15 AGE 3 |
|  | DW | 324 |  | 16 AGE 3 |
|  | DW | 325 |  | 16 AGE 3 |
| Dark_S4: | DW | 326 |  | 1 AGE 4 |
|  | DW | 327 |  | 2 AGE 4 |
|  | DW | 328 |  | 3 AGE 4 |
|  | DW | 311 |  | 4 AGE 4 |
|  | DW | 329 |  | 5 AGE 4 |
|  | DW | 313 |  | 6 AGE 4 |
|  | DW | 330 |  | 7 AGE 4 |
      | DW | 000 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | 
      Macro_grp2: : points into macro tables

DW Tbl2_Macro128
DW Tbl2_Macro129, Tbl2_Macro130, Tbl2_Macro131, Tbl2_Macro132, Tbl2_Macro 133
DW Tbl2_Macro134, Tbl2_Macro135, Tbl2_Macro136, Tbl2_Macro137, Tbl2_Macro 138
DW Tbl2_Macro139, Tbl2_Macro140, Tbl2_Macro141, Tbl2_Macro142, Tbl2_Macro 143
DW Tbl2_Macro144, Tbl2_Macro145, Tbl2_Macro146, Tbl2_Macro147, Tbl2_Macro 148
DW Tbl2_Macro149, Tbl2_Macro150, Tbl2_Macro151, Tbl2_Macro152, Tbl2_Macro 153
DW Tbl2_Macro154, Tbl2_Macro155, Tbl2_Macro156, Tbl2_Macro157, Tbl2_Macro 158
DW Tbl2_Macro159, Tbl2_Macro160, Tbl2_Macro161, Tbl2_Macro162, Tbl2_Macro 163
DW Tbl2_Macro164, Tbl2_Macro165, Tbl2_Macro166, Tbl2_Macro167, Tbl2_Macro 168
DW Tbl2_Macro169, Tbl2_Macro170, Tbl2_Macro171, Tbl2_Macro172, Tbl2_Macro 173
DW Tbl2_Macro174, Tbl2_Macro175, Tbl2_Macro176, Tbl2_Macro177, Tbl2_Macro 178
DW Tbl2_Macro179, Tbl2_Macro180, Tbl2_Macro181, Tbl2_Macro182, Tbl2_Macro 183
DW Tbl2_Macro184, Tbl2_Macro185, Tbl2_Macro186, Tbl2_Macro187, Tbl2_Macro 188
DW Tbl2_Macro189, Tbl2_Macro190, Tbl2_Macro191, Tbl2_Macro192, Tbl2_Macro 193
DW Tbl2_Macro194, Tbl2_Macro195, Tbl2_Macro196, Tbl2_Macro197, Tbl2_Macro 198
DW Tbl2_Macro199, Tbl2_Macro200, Tbl2_Macro201, Tbl2_Macro202, Tbl2_Macro 203
DW Tbl2_Macro204, Tbl2_Macro205, Tbl2_Macro206, Tbl2_Macro207, Tbl2_Macro 208
DW Tbl2_Macro209, Tbl2_Macro210, Tbl2_Macro211, Tbl2_Macro212, Tbl2_Macro 213
DW Tbl2_Macro214, Tbl2_Macro215, Tbl2_Macro216, Tbl2_Macro217, Tbl2_Macro 218
DW Tbl2_Macro219, Tbl2_Macro220, Tbl2_Macro221, Tbl2_Macro222, Tbl2_Macro 223
DW Tbl2_Macro224, Tbl2_Macro225, Tbl2_Macro226, Tbl2_Macro227, Tbl2_Macro 228
DW Tbl2_Macro229, Tbl2_Macro230, Tbl2_Macro231, Tbl2_Macro232, Tbl2_Macro 233
DW Tbl2_Macro234, Tbl2_Macro235, Tbl2_Macro236, Tbl2_Macro237, Tbl2_Macro 238
DW Tbl2_Macro239, Tbl2_Macro240, Tbl2_Macro241, Tbl2_Macro242, Tbl2_Macro 243
DW Tbl2_Macro244, Tbl2_Macro245, Tbl2_Macro246, Tbl2_Macro247, Tbl2_Macro 248
DW Tbl2_Macro249, Tbl2_Macro250, Tbl2_Macro251, Tbl2_Macro252, Tbl2_Macro 253
DW Tbl2_Macro254, Tbl2_Macro255
Macro_grp3: ; points into macro tables
      DW
DW
DW
261
DW
266
DW
271
DW
276
DW
281
DW
286
DW
291
DW
296
DW
301
DW
306
DW
311
DW
316
DW
321
DW
326
DW
331
DW
336
DW
341
DW
346
DW
351
DW
356
DW
361
DW
366
DW
371
DW
376
DW
381
DW
Macro_grp4: ; points into macro tables
DW Tbl4_Macro384
DW Tbl4_Macro385, Tbl4_Macro386, Tbl4_Macro387, Tbl4_Macro388, Tbl4_Macro
389
DW Tbl4_Macro390, Tbl4_Macro391, Tbl4_Macro392, Tbl4_Macro393, Tbl4_Macro
      ; The first group of numbers is the speech/motor table value.
; The last line is the terminator of 00 . ( 00 so 'DB' takes 1 less byte)
; ex: $1=$ will call the saysent 1 and the motor table 1.
Tb11_Macro0:

; (MIDDLE)
;
; put sounds and motions together
; DW 5 (first sound and motion, in this case \"5\")
; DW 3 (next sound and motion, in this case \"3\")
; DW 00 ( end of sequence)
Tb11_Macro1:

; GECROE 07/03/98
Tb11_Macro2:

; ; FRONT SEQ1AGE1

Tb11_Macro4:

Tb11_Macro5:

; ; FRONT SEQ4AGE1

      ;END GEORGE 07/03/98
; GEORGE 07/04/98
;START FORTUNE
Tbl1_Macro65:
DW 062
DW 051 ; 72 ;FORTUNE 1
DW 00 ; end
Tbl1_Macro66:
DW 003
DW 063 ;FORTUNE 2
DW 003
DW 00 ; end
Tbl1_Macro67:
DW 090 ; 94
DW 064
DW 063 ;FORTUNE 3
DW 00 ; end
Tbl1_Macro68:
DW 065 ;FORTUNE 4
DW 063
DW 00 ; end
Tbl1_Macro69:
DW 067 ; MODIFIED FOR NAME DMH
; DW 068
DW 053
DW 066 ;FORTUNE 5
DW 063
DW 00 ; end
Tbl1_Macro70:
DW 069 ;FORTUNE 6
DW 070
DW 00 ; end
Tbl1_Macro71:
DW 067
DW 068
DW 071
DW 073
DW 072
DW 00 ; end
Tbl1_Macro72:
DW 074 ;FORTUNE 8
DW 00 ; end
Tbl1_Macro73:
DW 074 ;FORTUNE 9
DW 063
DW 00 ; end
Tbl1_Macro74:
      ; END INVERT
; GEORGE 07/07/98
; BACK
Tbl2_Macro239: ;BACKSG ;SGDONF
DW 193
DW 193
DW 00 ; end
;
Tbl2_Macro240: ;SGDONE
DW 193
DW 194
DW 195
DW 00 ; end
;
Tbl2_Macro241: ;SGDONE
DW 193
DW 196
DW 195
DW 00 ; end
;
Tbl2_Macro242: ;SGDONE
DW 193
DW 194
DW 197
DW 00 ; end
;
Tbl2_Macro243: ;SGDONE
DW 193
DW 196
DW 197
DW 00 ; end
;
Tbl2_Macro244: ;SGDONE
DW 198
DW 199
DW 200
DW 201
DW 00 ; end
;
Tbl2_Macro245: ;SGDONE
DW 198
DW 199
DW 202
DW 201
DW 00 ; end
;
Tbl2_Macro246: ;SGDONE
DW 198
DW 199
DW 200
DW 184 ; 148 ; 212
DW 00 ; end
;
Tbl2_Macro247: ;SGDONE
DW 198
DW 199
DW 202
DW 184 ; 148 ; 212
DW 00 ; end
      ```
; ; ; ; ; ; SAYSENT pointer tables (128 max per table ---- 255 tables max)
Spch_grp1:
    DW Tbl1_say000
    DW
Tbl1_say001,Tbl1_say002,Tbl1_say003,Tbl1_say004,Tbl1_say005
    DW
Tbl1_say006,Tbl1_say007,Tbl1_say008,Tbl1_say009,Tbl1_say010
    DW
Tbl1_say011,Tbl1_say012,Tbl1_say013,Tbl1_say014,Tbl1_say015
    DW
Tbl1_say016,Tbl1_say017,Tbl1_say018,Tbl1_say015,Tbl1_say020
    DW
Tbl1_say021,Tbl1_say022,Tbl1_say023,Tbl1_say024,Tbl1_say025
    DW
Tbl1_say026,Tbl1_say027,Tbl1_say028,Tbl1_say029,Tbl1_say030
    DW
Tbl1_say031,Tbl1_say032,Tbl1_say033,Tbl1_say034,Tbl1_say035
    DW
Tbl1_say036,Tbl1_say037,Tbl1_say038,Tbl1_say039,Tbl1_say040
    DW
Tbl1_say041,Tbl1_say042,Tbl1_say043,Tbl1_say044,Tbl1_say045
    DW
Tbl1_say046,Tbl1_say047,Tbl1_say048,Tbl1_say049,Tbl1_say050
    DW
Tbl1_say051,Tbl1_say052,Tbl1_say053,Tbl1_say054,Tbl1_say055
    DW
Tbl1_say056,Tbl1_say .7,Tbl1_say058,Tbl1_say059,Tbl1_say060
    DW
Tbl1_say061,Tbl1_sa,062,Tbl1_say063,T _say064,Tbl1_say065
    DW
Tbl1_say066,Tbl1_say067,Tbl1_say068,Tbl1_say069,Tbl1_say070
    DW
Tbl1_say071,Tbl1_say072,Tbl1_say073,Tbl1_say074,Tbl1_say075
    DW
Tbl1_say076,Tbl1_say077,Tbl1_say078,Tbl1_say079,Tbl1_say080
    DW
Tbl1_say081,Tbl1_say082,Tbl1_say083,Tbl1_say084,Tbl1_say085
    DW
Tbl1_say086,Tbl1_say087,Tbl1_say088,Tbl1_say089,Tbl1_say090
    DW
Tbl1_say091,Tbl1_say092,Tbl1_say093,Tbl1_say094,Tbl1_say095
    DW Tbl1_say096,Tbl1_say097,Tbl1_say098,Tbl1_say099
    DW
Tbl1_say100,Tbl1_say101,Tbl1_say102,Tbl1_say103,Tbl1_say104
    DW Tbl1_say105,Tbl1_say106,Tbl1_say107,Tbl1_say108,Tbl1_say109
    DW Tbl1_say110,Tbl1_say111,Tbl1_say112,Tbl1_say113,Tbl1_say114
    DW Tbl1_say115,Tbl1_say116,Tbl1_say117,Tbl1_say118,Tbl1_say119
    DW Tbl1_say120,Tbl1_say121,Tbl1_say122,Tbl1_say123,Tbl1_say124
    DW Tbl1_say125,Tbl1_say126,Tbl1_say127
```
      | DW | Tb12_say128 |
| :--: | :--: |
| DW | Tb12_say129, Tb12_say130, Tb12_say131, Tb12_say132, Tb12_say133 |
| DW | Tb12_say134, Tb12_say135, Tb12_say136, Tb12_say137, Tb12_say138 |
| DW | Tb12_say139, Tb12_say140, Tb12_say141, Tb12_say142, Tb12_say143 |
| DW | Tb12_say144, Tb12_say145, Tb12_say146, Tb12_say147, Tb12_say148 |
| DW | Tb12_say149, Tb12_say150, Tb12_say151, Tb12_say152, Tb12_say153 |
| DW | Tb12_say154, Tb12_say155, Tb12_say156, Tb12_say157, Tb12_say158 |
| DW | Tb12_say159, Tb12_say160, Tb12_say161, Tb12_say'62, Tb12_say163 |
| DW | Tb12_say164, Tb12_say165, Tb12_say166, Tb12_say 67, Tb12_say168 |
| DW | Tb12_say169, Tb12_say170, Tb12_say171, Tb12_say172, Tb12_say173 |
| DW | Tb12_say174, Tb12_say175, Tb12_say176, Tb12_say177, Tb12_say178 |
| DW | Tb12_say179, Tb12_say180, Tb12_say181, Tb12_say182, Tb12_say183 |
| DW | Tb12_say184, Tb12_say185, Tb12_say186, Tb12_say187, Tb12_say188 |
| DW | Tb12_say189, Tb12_say190, Tb12_say191, Tb12_say192, Tb12_say193 |
| DW | Tb12_say194, Tb12_say195, Tb12_say196, Tb12_say197, Tb12_say198 |
| DW | Tb12_say199, Tb12_say200, Tb12_say201, Tb12_say202, Tb12_say203 |
| DW | Tb12_say204, Tb12_say205, Tb12_say206, Tb12_say207, Tb12_say208 |
| DW | Tb12_say209, Tb12_say210, Tb12_say211, Tb12_say212, Tb12_say213 |
| DW | Tb12_say214, Tb12_say215, Tb12_say216, Tb12_say217, Tb12_say218 |
| DW | Tb12_say219, Tb12_say220, Tb12_say221, Tb12_say222, Tb12_say223 |
| DW | Tb12_say224, Tb12_say225, Tb12_say226, Tb12_say227, Tb12_say228 |
| DW | Tb12_say229, Tb12_say230, Tb12_say231, Tb12_say232, Tb12_say233 |
| DW | Tb12_say234, Tb12_say235, Tb12_say236, Tb12_say237, Tb12_say238 |
| DW | Tb12_say239, Tb12_say240, Tb12_say241, Tb12_say242, Tb12_say243 |
| DW | Tb12_say244, Tb12_say245, Tb12_say246, Tb12_say247, Tb12_say248 |
| DW | Tb12_say249, Tb12_say250, Tb12_say251, Tb12_say252, Tb12_say253 |
| DW | Tb12_say254, Tb12_say255 |


| DW | Tb13_say256 |
| :--: | :--: |
| DW | Tb13_say257, Tb13_say258, Tb13_say259, Tb13_say260, Tb13_say261 |
| DW | Tb13_say262, Tb13_say263, Tb13_say264, Tb13_say265, Tb13_say266 |
| DW | Tb13_say267, Tb13_say268, Tb13_say269, Tb13_say270, Tb13_say271 |
| Dv | Tb13_say272, Tb13_say273, Tb13_say274, Tb13_say275, Tb13_say276 |
| DW | Tb13_say277, Tb13_say278, Tb13_say279, Tb13_say280, Tb13 -ay281 |
| DW | Tb13_say282, Tb13_say283, Tb13_say284, Tb13_say285, Tb13 -ay286 |
| DW | Tb13_say287, Tb13_say288, Tb13_say289, Tb13_say290, Tb13_say291 |
| DW | Tb13_say292, Tb13_say293, Tb13_say294, Tb13_say295, Tb13_say296 |
| DW | Tb13_say297, Tb13_say298, Tb13_say299, Tb13_say300, Tb13_say301 |
| DW | Tb13_say302, Tb13_say303, Tb13_say304, Tb13_say305, Tb13_say306 |
| DW | Tb13_say307, Tb13_say308, Tb13_say309, Tb13_say310, Tb13_say311 |
| DW | Tb13_say312, Tb13_say313, Tb13_say314, Tb13_say315, Tb13_say316 |
| DW | Tb13_say317, Tb13_say318, Tb13_say319, Tb13_say320, Tb13_say321 |
| DW | Tb13_say322, Tb13_say323, Tb13_say324, Tb13_say325, Tb13_say326 |
| DW | Tb13_say327, Tb13_say328, Tb13 say329, Tb13_say330, Tb13_say331 |
| DW | Tb13_say332, Tb13_say333, Tb1 -say334, Tb13_say335, Tb13_say336 |
| DW | Tb13_say337, Tb13_say338, Tb1 -say339, Tb13_say340, Tb13_say341 |
| DW | Tb13_say342, Tb13_say343, Tb1 -say344, Tb13_say345, Tb13_say346 |
| DW | Tb13_say347, Tb13_say348, Tb13_say349, Tb13_say350, Tb13_say351 |
| DW | Tb13_say352, Tb13_say353, Tb13_say354, Tb13_say355, Tb13_say356 |
| DW | Tb13_say357, Tb13_say358, Tb13_say359, Tb13_say360, Tb13_say361 |
| DW | Tb13_say362, Tb13_say363, Tb13_say364, Tb13_say365, Tb13_say366 |
| DW | Tb13_say367, Tb13_say368, Tb13_say369, Tb13_say370, Tb13_say371 |
| DW | Tb13_say372, Tb13_say373, Tb13_say374, Tb13_say375, Tb13_say376 |
| DW | Tb13_say377, Tb13_say378, Tb13_say379, Tb13_say380, Tb13_say381 |
      
; ALL SPEECH SAYSENT START HERE ; ; ; ; ; ; ; ;
; ; Saysent grnups for Tbl 1
; The first line of each group is the speech speed command.
; This is a number from 40 - 55 where 46 is stand d speed
;
; The next line is PITCH control which works as follows:
; Actual numeric value for TI pitch control
; bit 7 set = subtract value from current course value
; clr = add value to current course value
; bit 6 set = select music pitch table
; clr = select normal speech pitch table
; bit 0-5 value to change course value (no change $=0$ )
      ```
8Fh ; hi voice (8f is very squeeeeke) (8F=143)
81h ; one step higher than normal use range 81-8F (129-143)
00 ; normal voice
01 ; one step lower than normal
2fh ; 10 voice ( very low) use range 01-7F (01-47)
```

A math routine in 'say_0' converts the value for + or -
if $<80$ then subtracts from 80 to get the minus version of 90
ie, if number is 70 then TI gets 10 (which is -10 )
If number is 80 or $>80$ then get sent literal as positive.
NOTE: MAX POSITIVE IS 8B
MAX NEGATIVE is 2F ( 80h - 2Fh or 51h)
8Bh is hi voice (8f is very squeeeeke)
2Fh 10 voice ( very low)
When entering changes, 'Voice' holds the current pitch for Furby and it is modified by adding or subtracting a pitch change : : :
ex: Voice +8 increases the pitch from the current voice by 8
ex: Voice-10 decreases the pitch from the current voice by 10
The next group of entries are the speech words.
The last line is the terminator of 'FF'
(BOTTOM)
1 is very fast
46 is average
255 is very slow
DB 46 (speed of speech)
DB 123 (do sound 123)
DB 43 (do sound 43)
DB FFH
FITCH PROGRAMKING RANGE:
Voice +8 (highest)
Voice-20 (lowest)
Tbl1_say060:
DB 46
DB Voice
DB 163
DB FFH
; GEORGE 07/03/98
Tbl1_say001:
; dON START SEQ1 AGE1
DB
$46 ;$ sp sech speed
$169,162,162,164,149$; DONE 1FRONT SEQ1
DB FFH ; end
Tbl1_say002:
      DB 52 : speech speed
DB Voice+8 ; system pitch setting
DB $117,59 \\quad$; DONE 1FRONT SEQ2 age1
DB FFH ; end
Tbl1_say003:
DB 46
; speech speed
DB Voice-4 ; system pitch setting
DB $118 \\quad$; 1 front seq3 - seq4-part1-SEQ7PART2
DB FFH ; end
Tbl1_say004:
DB 46
; speech speed
DB Voice ; system pitch setting
DB $62,22,85 \\quad$; 1 front seq3 part2
DB FFH ; end
Tbl1_say005:
DB 50
; speech speed
DB Voice+8 ; system pitch setting
DB $58,39 \\quad$; 1 front seq4 part 2
DB FFH ; end
Tbl1_say006:
DB 46
; speech speed
DB Voice ; pitch control
DB $162,162,99,117$; seq5 age1 front part of seq6
DB FFH ; end
Tbl1_say007:
DB 55
; speech speed
DB Voice: 8 ; system pitch setting
DB $156 \\quad$; seq6 age1 front back part
DB FFH ; en: 4
Tbl1_say008:
DB 46
; speech speed
DB Voice ; pitch control
DB $162,162,99,10,39 \\quad$; SEQ7 FRONT AGE1 ADD SAY 003
DB FFH ; end
Tbl1_say009:
DB 46
; speech speed
DB Voice ; system pitch setting
DB $99,99,145 \\quad$; SEQ8 FRONT AGE1
DB FFH ; end
Tbl1_say010:
DB 46
; speech speed
DB Voice ; system pitch setting
DB $98 \\quad$; seq9 FRONT AGE1
DB FFH ; end
Tbl1_say011:
DB 30
; speech speed
DB Voice+8 ; system pitch setting
DB $96,165,165,165,129,149$; seq10 FRONT AGE1 ADD SAY20
DB FFH ; end
Tbl1_say012:
      ```
Tb11_say078:
    DB 50
    DB Voice+7
    DB 49
    DB FFH ; end
;END SAY FORTUNE
;END GEORGE 07/04/98
;START HANGOUT
;GEORGE 07/04/98
Tb11_say079:
    DB 56
    DB Voice+8
    DB 110
DUM) AGE1
    DB FFH ; end
Tb11_say080:
    DB 60
    DB Voice+8
    DB 109
    DB FFH ; end
Tb11_say081:
    DB 56
    DB Voice+8
    DB Voice+8
    DB 116
DB FFH ; end
Tb11_say082:
    DB 46
    DB Voice+7
    DB 113
    DB FFH ; end
Tb11_say083:
    DB 53
    DB Voice+5
    DB 162,114,162,114
    DB FFH ; end
Tb11_say084:
    DB 46
    DB Voice+8
    DB 115
    DB FFH ; end
Tb11_say085:
    DB 60
    DB Voice+5
    DB 126,163 ;SEQ4 HANGING (LA LA)
    DB FFH ; end
Tb11_say086:
    DB 56
    DB Voice+5
    DB 127
    DB FFH ; end
```
      Tbl1_say113:
DB 58 :speech speed
DB Voice+7 : system pitch setting
DB 23 ;SEQ2 FEED AGE1 (DO MOH)
DB FFH ; end
Tbl1_say114:
DB $\\quad 58 \\quad$ speech speed
DB Voice : system pitch setting
DB 79 ; TOH-DYE
DB FFH ; end
Tbl1_say115:
DB 46 :speech speed
DB Voice : system pitch setting
DB 97 ;BURF
DB FFH ; end
Tbl1_say116:
DB 46 :speech speed
DB Voice : system pitch setting
DB 140 ;SIGH
DB FFH ; end
Tbl1_say117:
DB 46 :speech speed
DB Voice : system pitch setting
DB 10 ;BOO
DB FFH ; end
Tbl1_say118:
DB 46 :speech speed
DB Voice : system pitch setting
DB 85 ;WAH
DB FFH end
Tbl1_say119:
DB $\\quad 60 \\quad:$ speech speed
DB Voice+8 : system pitch setting
DB 80 ;TOH-LOO
DB FFH ; end
Tbl1_say120:
DB 46 :speech speed
DB Voice+8 : system pitch setting ;A TAY
DB 7
DB FFH ; end
Tbl1_say121:
DB 46 :speech speed
DB Voice : system pitch setting
DB 33 ;SEQ1 FEED AGE2 HUNGRY
DB FFH ; end
143 SAME AS TBL1_SAY072
; Tbl2_say143:
DB 46 ; speech speed
DB Voice : system pitch setting
DB 28
; :SEQ2 FEED AGE3 (GOOD)
DB FFH ; end
      DB 46 ; speech speed
DB Voice ; system pitch setting
DB $52,48,81,152$;S7 A2 TILT (with say/m5) js
DB FFH ; end
Tb13_say328:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB 155
; S8 A2 TILT (with say/m5) js
DB FFH ; end
Tb13_say329:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB $52,57 ;$ S11 A2 TILT (with say/m2) js
DB FFH ; end
Tb13_say330:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB $158,60,80 ;$ S12 A2 TILT js
DB FFH ; end
Tb13_say331:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB $163,156 ;$ S13 A2 TILT (with say/m5) js
DB FFH ; end
Tb13_say332:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB $8,22,85$;S14 A2 TILT js
DB FFH ; end
Tb13_say333:
DB 46 ; speech speed
DB Voice ; pitch control
DB $154,118,163,145,165,162,118$;S16 A2/S14 A3/S14 A4
TILT js
DB FFH ; end
Tb13_say334:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB 159
; S3 A3 TILT js
DB FFH ; end
Tb13_say335:
DB 46 ; speech speed
DB Voice ; pitch control
DB 83,1
; S4 A3/S4 A4 TILT (with say/m26) js
DB FFH ; end
Tb13_say336:
DB 46 ; speech speed
DB Voice ; system pitch setting
DB $155,52,62,85$;S5 A3 TILT js
DB FFH ; end
      DIALOGUE

      ```
Tb14_say438:
Tb14_say439:
Tb14_say440:
Tb14_say441:
Tb14_say442:
Tb14_say443:
Tb14_say444:
Tb14_say445:
Tb14_say446:
Tb14_say447:
Tb14_say448:
Tb14_say449:
Tb14_say450:
Tb14_say451:
Tb14_say452:
Tb14_say453:
Tb14_say454:
Tb14_say455:
Tb14_say456:
Tb14_say457:
Tb14_say458:
Tb14_say459:
Tb14_say460:
Tb14_say461:
Tb14_say462:
Tb14_say463:
Tb14_say464:
Tb14_say465:
Tb14_say466:
Tb14_say467:
```
      ```
Tbl4_say468:
Tbl4_say469:
Tbl4_say470:
Tbl4_say471:
Tbl4_say472:
Tbl4_say473:
Tbl4_say474:
Tbl4_say475:
Tbl4_say476:
Tbl4_say477:
Tbl4_say478:
Tbl4_say479:
Tbl4_say480:
Tbl4_say481:
Tbl4_say482:
Tbl4_say483:
Tbl4_say484:
Tbl4_say485:
Tbl4_say486:
Tbl4_say487:
Tbl4_say488:
Tbl4_say489:
Tbl4_say490:
Tbl4_say491:
Tbl4_say492:
Tbl4_say493:
Tbl4_say494:
Tbl4_say495:
Tbl4_say496:
Tbl4_say497:
```
      ```
Tbl4_say498:
Tbl4_say499:
Tbl4_say500:
Tbl4_say501:
Tbl4_say502:
Tbl4_say503:
Tbl4_say504:
Tbl4_say505:
Tbl4_say506:
Tbl4_say507:
Tbl4_say508:
Tbl4_say509:
Tbl4_say510:
Tbl4_say511:
; ON POWER UP, UNTIL WAKE-UP TABLE INSTALLED (Dave)
    DB 46 ; speech speed
        DB Voice
        DB 165
        DB FFH ; end
;
; **************************************************************
; Motor tables
; Offsett pointer :
Motor_grpl:
DW Tbl1_M000
DW Tbl1_M001,Tbl1_M002,Tbl1_M003,Tbl1_M004,Tbl1_M005
DW Tbl1_M006,Tbl1_M007,Tbl1_M008,Tbl1_M009,Tbl1_M010
DW Tbl1_M011, Tbl1_M012,Tbl1_M013, Tbl1_M014, Tbl1_M015
DW Tbl1_M016,Tbl1_M017,Tbl1_M018,Tbl1_M019,Tbl1_M020
DW Tbl1_M021, Tbl1_M022,Tbl1_M023,Tbl1_M024,Tbl1_M025
DW Tbl1_M026,Tbl1_M027,Tbl1_M028,Tbl1_M029,Tbl1_M030
DW Tbl1_M031,Tbl1_M032,Tbl1_M033,Tbl1_M034,Tbl1_M035
DW Tbl1_M036,Tbl1_M037,Tbl1_M038,Tbl1_M039,Tbl1_M040
DW Tbl1_M041, Tbl1_M042,Tbl1_M043,Tbl1_M044,Tbl1_M045
DW Tbl1_M046,Tbl1_M047,Tbl1_M048,Tbl1_M049,Tbl1_M050
DW Tbl1_M051,Tbl1_M052,Tbl1_M053,Tbl1_M054,Tbl1_M055
DW Tbl1_M056,Tbl1_M057,Tbl1_M058,Tbl1_M059,Tbl1_M060
```
      ; 'FF' or '255' is the end of table command.
; TABLES WITH ENDING STEP NOT WITHIN REQUIRED RANGE (10-20), (132,136)
;M94,M127,M131,M139,M140,M143,M146
; WITH DUPLICATE STEPS PUT CONSECUTIVELY
;M187,M193,M219,M220,M229,M237,M241,M242
;M250,M310,M321,M369
Tb11_M000:
DB 50 ; motor delay between steps
DB $10,135$
DB FFH ; end
; GEORGE 07/03/98
Tb11_M001:
$\\begin{array}{ll}\\text { DB } & 1 \\\\ \\text { DB } & 190,133\\end{array}$
DB FFH
Tb11_M002:
DB
1
DB
190,145,138,120,145,133,147,133
$\\begin{array}{ll}\\text { DB } & \\text { FFH } \\\\ \\text { DB } & \\text { FFH }\\end{array} ;$ end
Tb11_M003:
DB
10
DB
100,190,160,100,133 ; CONNECTED M22 ; dON START
SEQ3 AGE1
DB
145,160,0,0,0,160
DB
FFH ; end
Tb11_M004:
DB
1
SEQ3 AGE1
DB
;
Tb11_M005:
DB
170,130,90,100,133 ; DONE conected m22 seq4 age1
DB
; motor delay between steps
DB
10 ; motor delay between steps
DB
FFH ; end
Tb11_M006:
DB
10 ; motor delay between steps
DB
150,200,0,0,150,133 ; seq5 front1 age1
$\\begin{array}{ll}\\text { DB } & \\text { FFH } \\\\ \\text { DB } & \\text { FFH }\\end{array} ;$ end
Tb11_M007:
DB
1
; motor delay between steps
DB
120,150,133 ; SEQ6 FRONT1 AGE1 HORSE LAUGH
DB
FFH ; end
Tb11_M008:
DB
10
; motor delay between steps
DB
150,200,150,170,133 ; SEQ7 FRONT AGE1
      DB FFH ; end
Tbl4_M389:
DB 90 ; motor delay between steps
DB $150,0,130,0,100,0,133$; YANN
DB FFH ; end
; DANGER SLEEP
Tbl4_M390:
DB 90 ; motor delay between steps
DB $0,0,0,85,30,0,20,0,85,30,0,20,0,85,30,0,20,0,85,10$
DB FFH ; end
;END GEORGE 07/09/98
;END IR
; FURBY SAYS: (LIGHT) DMH
Tbl4_M391:
DB 10 ; motor delay between steps
DB $110,133 ; \\quad ;$ LIGHT (furby says)
DB $110,120,133 ; \\quad ;$ LIGHT (furby says)
DB FFH ; end
Tbl4_M392:
DB
DB
DB
$150,0,0,0,115,0,0,0,0,133$
DB FFH ; end
Tbl4_M393:
DB
DB
DB
$30 ;$ motor delay between steps
$150,0,0,0,115,0,0,0,0,133$
DB FFH ; end
Tbl4_M394:
DB
DB
DB
DB
Tbl4_M395:
DB
DB
DB
DB
Tbl4_M396:
DB
DB
DB
$120,130,120,133$; ME ME
DB FFH ; end
Tbl4_M397:
DB
1
; motor delay between steps
DB
$115,130,110,133$; DO MOH
$120,130,120,133$; ME ME
$115,130,110,133$; DO MOH
$120,130,110,133$; TOH LOO
$115,130,110,133$; TOH LOO
Tbl4_M398:
DB
1
; end
      ```
Tbl4_M470:
Tbl4_M471:
Tbl4_M472:
Tbl4_M473:
Tbl4_M474:
Tbl4_M475:
Tbl4_M476:
Tbl4_M477:
Tbl4_M478:
Tbl4_M479:
Tbl4_M480:
Tbl4_M481:
Tbl4_M482:
Tbl4_M483:
Tbl4_M484:
Tbl4_M485:
Tbl4_M486:
Tbl4_M487:
Tbl4_M488:
Tbl4_M489:
Tbl4_M490:
Tbl4_M491:
Tbl4_M492:
Tbl4_M493:
Tbl4_M494:
Tbl4_M495:
Tbl4_M496:
Tbl4_M497:
Tbl4_M498:
Tbl4_M499:
```
      ```
Tbl4_M500:
Tbl4_M501:
Tbl4_M502
Tbl4_M503:
Tbl4_M504:
Tbl4_M505:
Tbl4_M506:
Tbl4_M507:
Tbl4_M508:
Tbl4_M509:
Tbl4_M510:
    DB
                            10
                            ; motor delay between steps
                            ; 10
                            ; 10
                            ; 10
                            ;end
Tbl4_M511:
    DB
                            10
                            ; motor delay between steps
                            ; 10
                            ;end
```
