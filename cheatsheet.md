TR3200 SHEET
============

Cheat sheet that compares assembly of TR3200 v0.8 vs DCPU-16 v1.7 

## DCPU-16 - TR3200 Registers equivalence:
    
- A   -> %r0
- B   -> %r1
- C   -> %r2
- X   -> %r3
- Y   -> %r4
- Z   -> %r5
- I   -> %r6
- J   -> %r7
- SP  -> %sp = %r31
- PC  -> %pc
- EX  -> CF bit in %flags and %y for multiplications

%r8 to %r13 and %r16 to %r26 could be uses as temporal variable without penalty
%r30 = %bp is asociated to stack allocation and function arguments, but can be 
used as a regular register.

## Intruction syntax

The usual format : 

		INTR Op1, Op2, Op3 

Follow this logic:
		
- Op1 : Were to put the result
- Op2 and Op3 : Input values

## Addresing modes and ALU operations comparation

    DCPU-16                                                                  TR3200
    -------------------------------------------------------------------------------
    SET  [A], B                                                       LOAD %r0, %r1
    
    SET  A, [B]                                                      STORE %r1, %r0
    
    SET  PUSH, C                                                           PUSH %r2

    SET  C, POP                                                             POP %r2

    SET  D, PICK 3                                                LOAD %sp + 3, %r3

    ; D = [SP - 3]
    SET  D, SP                                                    LOAD %sp - 3, %r3
    SUB  D, 3
    SET  D, [D]                                                

    SET  A, [B + 100]                                           LOAD %r1 + 100, %r0
    
    ; a = [C + X]
    ADD  C, X                                                   LOAD %r2 + %r3, %r0
    SET  A, [C]                                          

    SET  A, 0xCAFE                                                  MOV %r0, 0xCAFE
    
    SET  PC, label                                                        JMP label

    IFE  10, I                                                         IFEQ %r6, 10

    IFL  10, I                                                          IFG %r6, 10

    JSR  label                                                           CALL label


    ; B = B + A
    ADD B, A                                                      ADD %r1, %r1, %r0
                                               or ADD %r1, %r0 (pseudo instruction)
    
    ; C = B + A
    SET C, B                                                   ADD %r2, %r1, %r0
    ADD C, A

    ; *B = *A
    SET [B], [A]                                                     LOAD %r10, %r0
                                                                    STORE %r1, %r10

    ; *(B++) = *(A++)
    STI [I], [J]                                                     LOAD %r10, %r7
                                                                    STORE %r6, %r10
                                                                    ADD %r6, 1, %r6
                                                                    ADD %r7, 1, %r7
