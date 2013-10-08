Calling convention for RC3200 CPU
---------------------------------

  - r0 to r3 holds th argument values passed to a subrutine, and also holds the results returned from a subrutine.
  - Subsequent arguments are passed on the stack. Function arguments of a procedural language are pushed in reverse order.
  - r4 to r29 are used for local variables. Callee subrutine or function must preserve it,
  - r30 (BP) register must be preserved being pushed to the stack before the extra arguments. r30 takes the value of r31 (SP) after pushing r30 to the stack.
  - Calle function/subrutine can use BP + n to read extra arguments

Example:

    int callee(int, int, int, int, int);
     
    int caller(void)
    {
            register int ret = 0;
     
            ret = callee(1, 2, 3, 4, 5);
            ret += 5;
            return ret;
    }

Produces this :

    caller:
            push    r4              ; Prologue. Preserves r4 in the stack

            push    BP              ; Code that preserves BP and pass the
            cpy     BP, SP          ; arguments
            set     r0, 1
            set     r1, 2
            set     r2, 3
            set     r3, 4
            push    5               ; Fith argument tothe stack
            
            call    callee           
            sub     SP, 4, SP       ; Code that recovers BP value
            pop     BP
            
            add     r4, 5, r0

            cpy     r0, r4          ; Epilogue. Sets r0 to return value and
            pop     r4              ; restores used r4
            ret



Calling convention for RC1600 CPU
---------------------------------

  - r0 to r3 holds th argument values passed to a subrutine, and also holds the results returned from a subrutine.
  - Subsequent arguments are passed on the stack. Function arguments of a procedural language are pushed in reverse order.
  - r4 to r13 are used for local variables. Callee subrutine or function must preserve it,
  - r14 (BP) register must be preserved being pushed to the stack before the extra arguments. In the prologue, r14 takes the value of r15 (SP) after pushing r14 to the stack.
  - Calle function/subrutine can use BP + n to read extra arguments

Example:

    int callee(int, int, int, int, int);
     
    int caller(void)
    {
            register int ret = 0;
     
            ret = callee(1, 2, 3, 4, 5);
            ret += 5;
            return ret;
    }

Produces this :

    caller:
            push    r4              ; Prologue. Preserves r4 in the stack

            push    BP              ; Code that preserves BP and pass the
            cpy     BP, SP          ; arguments
            set     r0, 1
            set     r1, 2
            set     r2, 3
            set     r3, 4
            push    5               ; Fith argument tothe stack
            
            call    callee           
            sub     SP, 4, SP       ; Code that recovers BP value
            pop     BP
            
            cpy     r4, r0
            add     r4, 5

            cpy     r0, r4          ; Epilogue. Sets r0 to return value and
            pop     r4              ; restores used r4
            ret

Calling convention for T-32 CPU
-------------------------------

  - A and B registers holds the argument values passed to a subrutine, and also holds the results returned from a subrutine.
  - Subsequent arguments are passed on the stack. Function arguments of a procedural language are pushed in reverse order.
  - C to K are used for local variables. Callee subrutine or function must preserve it,
  - BP register must be preserved being pushed to the stack before the extra arguments. BP takes the value of SP after pushing BP to the stack.
  - Calle function/subrutine can use BP + n to read extra arguments

Example:

    int callee(int, int, int, int, int);
     
    int caller(void)
    {
            register int ret = 0;
     
            ret = callee(1, 2, 3, 4, 5);
            ret += 5;
            return ret;
    }

Produces this :

    caller:
            set     PUSH, C         ; Prologue. Preserves C in the stack

            set     PUSH, BP        ; Code that preserves BP and pass the
            set     BP, SP          ; arguments
            set     A, 1
            set     B, 2
            set     PUSH, 5         ; Extra arguments in reverse order
            set     PUSH, 4
            set     PUSH, 3
            
            call    callee           
            sub     SP, 4*3         ; Code that recovers BP value
            set     BP, POP
            
            set     C, A
            add     C, 5

            set     A, C            ; Epilogue. Sets A to return value and
            set     C, POP          ; restores used C register
            ret


### FAQ

#### Why arguments in reverse order ?
Reverse order of arguments in stacks allow to access each argument by the same declaration order reading at [BP - n] were n:

- T-32: n = (Argument number - 3) * 4
- RC3200:  n = (Argument number - 5) * 4 
- RC1600:  n = (Argument number - 5) * 2 

 
For example, to read argument 5:

RC3200/RC1600 :

    LOAD    BP , r5
    
T-32 :

    SET     Y, [BP - 8]



