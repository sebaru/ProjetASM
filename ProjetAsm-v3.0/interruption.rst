ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 1.



                              1 ;-----------------------------------------------------------------------------------------------------------
                              2 ; interruption.asm    Vecteurs d'interruption
                              3 ; Sébastien Lefevre                                                            mar 08 jun 2004 12:49:36 CEST 
                              4 ;-----------------------------------------------------------------------------------------------------------
                              5         .module Interruption
ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 2.



                              6         .include "cst.asm"
                              1 ;-----------------------------------------------------------------------------------------------------------
                              2 ; cst.asm    Definition des constantes prgs
                              3 ; Sébastien Lefevre                                                            mar 08 jun 2004 12:49:36 CEST 
                              4 ;-----------------------------------------------------------------------------------------------------------
                              5 
ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 3.



                              6         .include "conf.asm"                                                       ; Inclusion des constantes 
                    0001      1 ID_ESCLAVE = 1
                    0008      2 NBR_ENTRE_ANA = 8
                    0014      3 NBR_ENTRE_TOR = 20
                    0000      4 NBR_ENTRE_CHOC = 0
                    0000      5 NBR_SORTIE_TOR = 0
                    0000      6 NBR_SORTIE_ANA = 0
ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 4.



                              7 ;----------------------------------------Déclarations des prototypes de fonctions --------------------------
                              8         .globl _start                                                                        ; dans init.asm
                              9         .globl TRAME_RECUE
                             10         .globl RX_BUF
                             11         .globl TX_BUF
                             12         .globl INPUT_BUF
                             13         .globl ENTRE_ANA_BUF
                             14         .globl ID_IDENT
                             15         .globl NBR_INT_CAN
                             16 
                             17         .globl _envoyer_rs232                                                               ; dans rs232.asm
                             18         .globl _recu_rs232
                             19         .globl _rs232_purge_trame
                             20 
                             21         .globl _clignotant_run                                                         ; Dans cligno_run.asm
                             22         .globl _del_trame_on
                             23         .globl _del_trame_off
                             24         .globl _cdg_logiciel
                             25         .globl _init_cdg_logiciel
                             26 
                             27         .globl _calcul_crc16                                                                ; dans crc16.asm
                             28 
                             29         .globl _start_of_conversion                                                           ; dans can.asm
                             30         .globl _recu_can
                             31 	.globl _lance_start_of_conversion
                             32         .globl CAN_PRET
                             33 
                             34         .globl _fct_entre_ana                                                       ; dans fct_entre_ana.asm
                             35 
                             36         .globl _fct_entre_tor                                                       ; dans fct_entre_tor.asm
                             37         .globl _raz_sortie_tor
                             38         .globl _polling_entre_tor
                             39 
                             40         .globl _fct_ident                                                               ; dans fct_ident.asm
                             41 
                             42         .globl _fct_sortie_tor                                                     ; dans fct_sortie_tor.asm
                             43 
                             44 ;---------------------------------------- Constantes communes à tous ---------------------------------------
                    00FF     45 	ID_MAITRE         = 0xFF
                    0003     46 	VERSION_MAJOR     = 0x03
                    0001     47 	VERSION_MINOR     = 0x01
                    0018     48 	TAILLE_BUF        = 24
                    0006     49 	TAILLE_ENTETE     = 6
                    00C0     50 	P4                = 0xC0
                    00C0     51 	P4.0              = 0xC0
                    00C1     52 	P4.1              = 0xC1
                    00C2     53 	P4.2              = 0xC2
                    00C3     54 	P4.3              = 0xC3
                    00C4     55 	P4.4              = 0xC4
                    00C5     56 	P4.5              = 0xC5
                    00C6     57 	P4.6              = 0xC6
                    00C7     58 	P4.7              = 0xC7
                             59 
                    00C4     60 	P5                = 0xC4     ; P5 c'est pas D0 ->>>> a voir !!
                    00FF     61 	T3                = 0xFF
                    00C5     62 	ADCON             = 0xC5
ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 5.



                    00C6     63 	ADCH              = 0xC6
                    00C8     64 	ADCS              = 0xC8
                    00C9     65 	ADCI              = 0xC9
                    00CC     66 	ADC1              = 0xCC
                    00CB     67 	ADC0              = 0xCB
                    00A8     68         IEN0              = 0xA8
                    0003     69 	IndexTrame_taille = 3
                             70 
                    0001     71 	FCT_IDENT         = 0x01
                    0002     72 	FCT_ENTRE_TOR     = 0x02
                    0003     73 	FCT_ENTRE_ANA     = 0x03
                    0004     74 	FCT_SORTIE_TOR    = 0x04
                    00F0     75 	DEBUG_PRENDRE_LIGNE = 0xF0
                    00F1     76 	DEBUG_LIBERE_LIGNE= 0xF1
                    00FF     77 	FCT_PING          = 0xFF
                             78 
                    0000     79 	BANK0             = 0x00
                    0008     80 	BANK1             = 0x08
                    0010     81 	BANK2             = 0x10
                    0018     82 	BANK3             = 0x18
                             83 ;-----------------------------------------------------------------------------------------------------------
ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 6.



                              7 ;-----------------------------------------------------------------------------------------------------------
                              8 ; interrupt vector 
                              9 ;-----------------------------------------------------------------------------------------------------------
                             10 	.area INTSEG    (ABS)
   0000                      11 	.org #0x0000
   0000 02 00 7B             12 _int:	ljmp _start
   0003                      13 	.org #0x0003
   0003 32                   14 	reti                  ; IRQ1   INT0         IE0       0003H
                             15 
                             16 ;-----------------------------------------------------------------------------------------------------------
                             17 ; IRQ 2: interruption timer 0: Purge de la trame RS232
                             18 ;-----------------------------------------------------------------------------------------------------------
   000B                      19 	.org #0x000B          ; IRQ2   TIMER 0      TF0       000BH
   000B 12 01 CE             20         lcall _rs232_purge_trame
   000E 32                   21 	reti        
                             22 
   0013                      23 	.org #0x0013
   0013 32                   24 	reti                  ; IRQ3   INT1         IE1       0013H
                             25 
                             26 ;-----------------------------------------------------------------------------------------------------------
                             27 ; IRQ 4: interruption timer 1: Utilisé pour la communication RS232
                             28 ;-----------------------------------------------------------------------------------------------------------
   001B                      29 	.org #0x001B
   001B 32                   30 	reti                  ; IRQ4   TIMER 1      TF1       001BH
                             31 
                             32 ;-----------------------------------------------------------------------------------------------------------
                             33 ; IRQ 5: interruption ligne rs232
                             34 ;-----------------------------------------------------------------------------------------------------------
   0023                      35 	.org #0x0023
   0023 30 98 03             36         jnb  RI,_end_int_rs232
   0026 12 01 6F             37         lcall _recu_rs232   ; si RI=1 saut et mise à 0 de RI
   0029                      38 _end_int_rs232:
   0029 32                   39         reti
                             40 
   002B                      41 	.org #0x002B
   002B 32                   42 	reti                  ; IRQ6   TIMER 2      TF2+EXF2  002BH
   0033                      43 	.org #0x0033
   0033 32                   44 	reti                  ; IRQ7
   003B                      45 	.org #0x003B
   003B 32                   46 	reti                  ; IRQ8
   0043                      47 	.org #0x0043
   0043 32                   48 	reti                  ; IRQ9
   004B                      49 	.org #0x004B
   004B 32                   50 	reti                  ; IRQ10
                             51 ;-----------------------------------------------------------------------------------------------------------
                             52 ; IRQ 11: interruption C.A.N
                             53 ;-----------------------------------------------------------------------------------------------------------
   0053                      54 	.org #0x0053
   0053 53 C5 EF             55         anl ADCON,#0xEF       ; ADCI = 0
   0056 12 03 39             56         lcall _recu_can       ; si RI=1 saut et mise à 0 de RI
   0059 32                   57         reti
                             58 
   005B                      59 	.org #0x005B
   005B 32                   60 	reti                  ; IRQ12
   0063                      61 	.org #0x0063
   0063 32                   62 	reti                  ; IRQ13
ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 7.



   006B                      63 	.org #0x006B
   006B 32                   64 	reti                  ; IRQ14
   0073                      65 	.org #0x0073
   0073 32                   66 	reti                  ; IRQ15
                             67 ;-----------------------------------------------------------------------------------------------------------
                             68 
