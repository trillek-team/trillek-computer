Calling conventions for TR3200 CPU and DCPU-16E
===============================================
Version 0.2 (WIP)


The key word "CALLER" is the function or code that is making the call, the key word "CALLEE" is the function being called

## TR3200

- %r0 to %r3 holds argument values passed to a subroutine. CALLER code MUST assume that these values will not be preserved.
- %r0 will be used to the return value if there is any. If the return value is a 64 bit value, then %r1:%r0 will be the 64 bit return value.
- Subsequent arguments are passed on the stack. Function arguments of a 
  procedural language are pushed in reversed order that are declared.
- %r4 to %r10 are used for local variables. CALLEE subroutine or function MUST preserve it.
- %y, %ia and %flags special registers must be preserved.
- %bp register MUST be preserved being pushed to the stack before the extra arguments by the CALLER. %bp takes the value of %sp after pushing the extra arguments.
- CALLEE function/subroutine can use %bp + n to read extra arguments, and %bp - n
  for local variable. Where **n = (number of parameter - 5) * 4**.

Example:

    int CALLEE(int a, int b, int c, int d, int e) {
        int tmp = a + b + c + d + e;
        return tmp;
    {
     
    int CALLER(void)
    {
            register int ret = 0;
     
            ret = callee(1, 2, 3, 4, 5);
            ret += 5;
            return ret;
    }

Produces this (notice that is not optimized code for clarity) :

    CALLER:
            PUSH    %r4             ; Prologue. Preserves %r4 in the stack

            PUSH    %bp             ; Code that preserves %bp and pass the
            MOV     %r0, 1          ; arguments
            MOV     %r1, 2
            MOV     %r2, 3
            MOV     %r3, 4
            PUSH    5               ; Fifth argument to the stack
            MOV     %bp, %sp        ; BP points to the last argument if is any
                                    ; or to the old BP value
            
            CALL    CALLEE           
            SUB     %sp, %sp, 4     ; Code that recovers BP value
            POP     %bp
            
            ADD     %r4, %r0, 5     ; ret += 5

            MOV     %r4, %r0        ; Epilogue. Sets %r0 to return value and
            POP     %r4             ; restores used %r4
            RET


    CALLEE:
            PUSH    %r4             ; Prologue. Preserves %r4 in the stack
            PUSH    %r5             ; Prologue. Preserves %r5 in the stack

            ADD     %r0, %r1, %r4   ; tmp = a + b
            ADD     %r4, %r2, %r4   ; tmp += c
            ADD     %r4, %r3, %r4   ; tmp += d

            LOAD    %r5, %bp        ; Reads e and puts in a local var
            ADD     %r4, %r5, %r4   ; tmp += e

            MOV     %r4 , %r0       ; Epilogue. Sets %r0 to the return value
            POP     %r5             ; restores used %r5
            POP     %r4             ; restores used %r4
            ret

### FAQ

#### Why arguments in reverse order ?
Reverse order of arguments in stacks allow to access each argument by the same declaration order reading at [BP + n] were n:

        n = (Argument number - 5) * 4 

 
For example, to read argument 5, 6 and 7:

    LOAD    %r5, %bp                 ; %r5 = Fith argument
    LOAD    %r6, %bp + 4             ; %r6 = Sixth argument
    LOAD    %r7, %bp + 8             ; %r6 = Seventh argument

#### Where I put local vars if I not have enough registers
If you exhaust the registers %r4 to %r10 for local vars, the CALLEE function can
use the stack to store local vars. Only need, in the end of prologue, to 
subtract to %sp a value to give space in the stack and restore the %sp value 
in the epilogue.

For example, a 32 bit integer var in the stack :

    ; Prologue
    ... Gets parameters from the stack
    SUB     %sp, %sp, 4              ; We move the stack pointer down and we
                                     ; create space to a 32 bit integer value
                                     ; in the Stack
    ... Preserve used registers %r5 to %r10

    ... code of the function
    STORE   %bp - 4 , %r5            ; Example of writing to the local var in stack
    LOAD    %r5, %bp - 4             ; Example of reading the local var in stack
    ... code of the function

    ; Epilogue
    ... Sets result values
    ... Restores used register %r5 to %r10
    ADD     %sp, %sp, 4              ; We restore the Stack pointer value
                                     ; local vars in the stack are forgot
    RET

## DCPU-16E

- A and B registers holds argument values passed to a subroutine. CALLER code MUST assume that these values will not be preserved.
- A will be used to the return value if there is any. If the return value is a 64 bit value, then %B:%A will be the 64 bit return value.
- Subsequent arguments are passed on the stack. Function arguments of a 
  procedural language are pushed in reversed order that are declared.
- X, Y, Z, I, and J are used for local variables. CALLEE subroutine or function MUST preserve it.
- EX and IA special registers must be preserved.
- C register MUST be preserved being pushed to the stack before the extra arguments by the CALLER. C takes the value of SP after pushing the extra arguments. (C will be used as Base Pointer)
- CALLEE function/subroutine can use C + n to read extra arguments, and %C - n
  for local variable. Where **n = (number of parameter - 3) * 2**.
