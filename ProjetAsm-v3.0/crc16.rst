ASxxxx Assembler V01.70 + NoICE + SDCC mods + Flat24 Feb-1999  (Intel 8051), page 1.



                              1 ;-----------------------------------------------------------------------------------------------------------
                              2 ; crc16.asm    Calcul du crc16
                              3 ; Sébastien Lefevre                                                            mar 08 jun 2004 12:49:36 CEST 
                              4 ;-----------------------------------------------------------------------------------------------------------
                              5 	.module  Crc16
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
                              8 ; Calcul du CRC16 de la trame adressée par R0
                              9 ;-----------------------------------------------------------------------------------------------------------
                             10 	.area CSEG	(CODE)
   01E8                      11 _calcul_crc16:
   01E8 7C FF                12 	mov	R4,#0xFF            ; poids fort crc16
   01EA 7D FF                13 	mov	R5,#0xFF            ; poids faible crc16
                             14 
   01EC E8                   15         mov     A,R0                ; début de trame
   01ED 24 03                16 	add     A,#IndexTrame_taille; zone TAILLE
   01EF F9                   17 	mov     R1,A
   01F0 E7                   18 	mov     A,@R1               ; recuperation de la taille à processer
   01F1 24 06                19 	add     A,#TAILLE_ENTETE    ; + taille entete
   01F3 C3                   20         clr C                       ; Carry = 0 pour la soustraction
   01F4 94 02                21 	subb    A,#0x02             ; on ne calcule pas le CRC du CRC !!
   01F6 FB                   22 	mov     R3,A                ; R3 = compteur d'octet
                             23 
   01F7                      24 _boucle_octet:
                             25                                	; debut de boucle
   01F7 ED                   26 	mov	A,R5		; poids faible crc 16
   01F8 66                   27 	xrl	A,@R0		; xor octet en cours de codage
   01F9 FD                   28 	mov	R5,A
                             29 
   01FA 7A 08                30 	mov	R2,#0x08        ; boucle bit
   01FC                      31 _boucle_bit:
   01FC EC                   32 	mov	A,R4		; poids fort crc
   01FD C3                   33 	clr	C		; Carry = 0
   01FE 13                   34 	rrc	A		; rotate right
   01FF FC                   35 	mov	R4,A		; save
   0200 ED                   36 	mov	A,R5		; au tour du poids faible
   0201 13                   37 	rrc	A		; rotate
   0202 FD                   38 	mov	R5,A		; save
   0203 50 08                39 	jnc	_boucle_bit_fin
                             40 
   0205 EC                   41 	mov 	A,R4		; Xor si retenue = 1
   0206 64 A0                42 	xrl	A,#0xA0
   0208 FC                   43 	mov	R4,A
   0209 ED                   44 	mov	A,R5
   020A 64 01                45 	xrl	A,#0x01
   020C FD                   46 	mov	R5,A
   020D                      47 _boucle_bit_fin:
   020D DA ED                48         djnz    R2,_boucle_bit
                             49 
   020F 08                   50 	inc	R0       	; on avance dans la trame
   0210 DB E5                51 	djnz    R3,_boucle_octet
   0212 22                   52         ret
                             53 ;-----------------------------------------------------------------------------------------------------------
                             54 
