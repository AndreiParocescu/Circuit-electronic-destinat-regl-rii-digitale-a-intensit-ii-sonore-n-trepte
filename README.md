# Circuit-electronic-destinat-regl-rii-digitale-a-intensit-ii-sonore-n-trepte
# Proiectarea unui circuit electronic destinat reglării digitale a intensității sonore în trepte și afișarea acestuia
# Cod Assembly
;configuratie pini uC -------------------
;pin7 - - Iesire C (b2) /RA0
;pin6 - Iesire B (b3)/RA1 
;pin5 – unused /RA2 msb
;pin4 - Intrare iDown (buton decrementare )/RA3
;pin3 - Iesire A (b0)/RA4 lsb 
;pin2 - Intrare iUp (buton incrementare)/RA5
;-----------------------------------------
;
 LIST P=PIC12F1822
 INCLUDE "P12F1822.inc"
;
;Setare cuvinte de configurare
;
 __CONFIG _CONFIG1 , _FOSC_INTOSC & _WDTE_OFF & _PWRTE_OFF & _MCLRE_OFF & _CP_OFF & 
_CPD_OFF & _BOREN_OFF & _CLKOUTEN_OFF & _IESO_OFF & _FCMEN_OFF
 __CONFIG _CONFIG2 , _WRT_OFF & _PLLEN_OFF & _STVREN_OFF & _BORV_LO & _LVP_OFF
;
;Definire registrii utilizator
;
Cblock 0x020 ;Begin of RAM
rNiv
rFLAGS
endc
;
#define iUp PORTA,5 
#define iDown PORTA,3
#define fUp rFLAGS,0
#define fDown rFLAGS,1
;
 ORG 0
 GOTO START
;
;rutina de intrerupere --------------------------------------
;
 ORG 4
goto START
;
;start program ====================================
;
START
;
; Define OSC (4MHz, IntOSC)
 BANKSEL OSCCON 
 MOVLW B'11101000' 
 MOVWF OSCCON
;
;PORTA as digital I/O 
BANKSEL ANSELA
clrf ANSELA
;
;define I/O port
 BANKSEL TRISA
 MOVLW B'11101000'
;RA0=C
;RA1=B
;RA2=0
;RA3=iDown
;RA4=A
;RA5=iUp
 MOVWF TRISA
;
;initializare PORTA
BANKSEL PORTA
clrf PORTA
movlw 0x00
movwf rNiv
clrf rFLAGS
;
MAIN
;
movfw rNiv
call Table
movwf PORTA
;verifica buton Up
btfsc iUp ;apasat?
goto MAIN1 ;Nu
;Da
bcf fDown
btfsc fUp ;deja apasat?
goto MAIN ;Da
bsf fUp ;Nu, prima apasare
movlw 0x13
xorwf rNiv,W
btfsc STATUS,Z
goto MAIN
incf rNiv
goto MAIN
;
MAIN1 ;verifica buton Down
bcf fUp
btfsc iDown ;apasat?
goto MAIN2 ;Nu
;Da
btfsc fDown ;deja apasat?
goto MAIN ;Da
bsf fDown ;Nu, prima apasare
movlw 0x0F
xorwf rNiv,W
btfsc STATUS,Z
goto MAIN
decf rNiv
goto MAIN
MAIN2
bcf fDown
goto MAIN
;
Table
ADDWF PCL
retlw 0x00 ;0
retlw 0x10 ;1
retlw 0x02 ;2
retlw 0x12 ;3
retlw 0x01 ;4
retlw 0x11 ;5
retlw 0x03 ;6
retlw 0x13 ;7
;
END
