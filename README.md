## An Abstract Stack Based Approach to Verified Compositional Compilation to Machine Code

This is the website hosting the artifact associated with our POPL'19 submission.

You will find our source code in [this archive](compcertmc.zip).

### Requirements
The development is known to compile with the following software:
- Menhir v.20170712 or later
- Coq v8.6.1
- OCaml v4.02.3

### Instructions
To compile our development, first configure the target architecture:

  ./configure x86_32-linux
  
This will configure the development for the 32-bit x86 architecture. (We are limited to the 32-bit x86 architecture because of the RockSalt instruction encoder.)
Then generate the dependencies:

  make depend
  
Finally compile the whole development

  make -j
  
This takes approximately 15-25 minutes depending on your machine.

### Usage

There is a number of examples in `c_examples`.

  $ cd c_examples
  $ cat fib.c
    int fib (int n)
    {
      if (n <= 2) 
        return 1;
      else 
        return fib(n-1) + fib(n-2);
    }
    
    int n = 12;
    
    int main() 
    {
      return fib(n);
    }
  
  $ ../ccomp -machinecode fib.c
  $ ./fib.o 
  $ echo $?
    144
