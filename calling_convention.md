Calling convention for RC3200 CPU
---------------------------------

  - %r0 to %r3 holds th argument values passed to a subrutine, and also holds the results returned from a subrutine.
  - Subsequent arguments are passed on the stack. Function arguments of a procedural language are pushed in reverse order.
  - %r4 to %r29 are used for local variables. Callee subrutine or function must preserve it,
  - %r30 (%bp) register must be preserved being pushed to the stack before the extra arguments. %r30 takes the value of %r31 (%sp) after pushing %r30 to the stack.
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
            push    %r4             ; Prologue. Preserves %r4 in the stack

            push    %bp             ; Code that preserves %bp and pass the
            cpy     %bp, %sp        ; arguments
            set     %r0, 1
            set     %r1, 2
            set     %r2, 3
            set     %r3, 4
            push    5               ; Fifth argument to the stack
            
            call    callee           
            sub     %sp, 4, %sp     ; Code that recovers BP value
            pop     %bp
            
            add     %r4, 5, r0

            cpy     %r0, %r4        ; Epilogue. Sets %r0 to return value and
            pop     %r4             ; restores used %r4
            ret


### FAQ

#### Why arguments in reverse order ?
Reverse order of arguments in stacks allow to access each argument by the same declaration order reading at [BP - n] were n:

        n = (Argument number - 5) * 4 

 
For example, to read argument 5 and 6:

    LOAD    %bp, %r5                 ; %r5 = Fith argument
    SUB     %r10, 4, %bp             ; Stores %bp - 4 in %r10
    LOAD    %r10, %r6                ; %r6 = Sixth argument
    

