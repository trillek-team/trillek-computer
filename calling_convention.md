Calling convention for RC3200 CPU
=================================
Version 0.2 (WIP)

- %r0 to %r3 holds th argument values passed to a subrutine, and also holds 
  the results returned from a subrutine.
- Subsequent arguments are passed on the stack. Function arguments of a 
  procedural language are pushed in reversed order that are declared.
- %r4 to %r29 are used for local variables. Callee subrutine or function must 
  preserve it,
- %r30 (%bp) register must be preserved being pushed to the stack before the 
  extra arguments. %r30 takes the value of %r31 (%sp) after pushing the extra
  arguments.
- Calle function/subrutine can use %bp + n to read extra arguments, and %bp - n
  for local variables.

Example:

    int callee(int a, int b, int c, int d, int e) {
        int tmp = a + b + c + d + e;
        return tmp;
    {
     
    int caller(void)
    {
            register int ret = 0;
     
            ret = callee(1, 2, 3, 4, 5);
            ret += 5;
            return ret;
    }

Produces this (notice that is not optimiced code for clarity) :

    caller:
            push    %r4             ; Prologue. Preserves %r4 in the stack

            push    %bp             ; Code that preserves %bp and pass the
            set     %r0, 1          ; arguments
            set     %r1, 2
            set     %r2, 3
            set     %r3, 4
            push    5               ; Fifth argument to the stack
            cpy     %bp, %sp        ; BP points to the last argument if is any
                                    ; or to the old BP value
            
            call    callee           
            sub     %sp, 4, %sp     ; Code that recovers BP value
            pop     %bp
            
            add     %r4, %r0, 5     ; ret += 5

            cpy     %r4, %r0        ; Epilogue. Sets %r0 to return value and
            pop     %r4             ; restores used %r4
            ret


    calle:
            push    %r4             ; Prologue. Preserves %r4 in the stack
            push    %r5             ; Prologue. Preserves %r5 in the stack

            add     %r0, %r1, %r4   ; tmp = a + b
            add     %r4, %r2, %r4   ; tmp += c
            add     %r4, %r3, %r4   ; tmp += d

            load    %r5, %bp        ; Reads e and puts in a local var
            add     %r4, %r5, %r4   ; tmp += e

            cpy     %r4 , %r0       ; Epilogue. Sets %r0 to the return value
            pop     %r5             ; restores used %r5
            pop     %r4             ; restores used %r4
            ret

### FAQ

#### Why arguments in reverse order ?
Reverse order of arguments in stacks allow to access each argument by the same declaration order reading at [BP + n] were n:

        n = (Argument number - 5) * 4 

 
For example, to read argument 5, 6 and 7:

    LOAD    %r5, %bp                 ; %r5 = Fith argument
    LOAD    %r6, %bp + 4             ; %r6 = Sixth argument
    LOAD    %r7, %bp + 8             ; %r6 = Seventh argument

#### Where I put local vars if i noit have enought registers
If you exaust the registers %r4 to %r29 for local vars, the calle function can
use the stack to store local vars. Only need, in the end of prologue, to 
substract to %sp a value to give space in the stack and restore the %sp value 
in the epilogue.

For example, a 32 bit interger var in the stack :

    ; Prologue
    ... Gets parametes from the stack
    SUB     %sp, 4, %sp              ; We move the stack pointer down and we
                                     ; create space to a 32 bit interger value
                                     ; in the Stack
    ... Preserve used registers %r5 to %r29

    ... code of the funtion
    STORE   %bp - 4 , %r5            ; Example of writing to the local var in stack
    LOAD    %r5, %bp - 4             ; Example of reading the local var in stack
    ... code of the function

    ; Epilogue
    ... Sets result values
    ... Restores used register %r5 to %r29
    ADD     %sp, 4, %sp              ; We restores the Stack pointer value
                                     ; local vars i nthe stack aren forgot
    RET
