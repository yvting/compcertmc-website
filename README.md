# An Abstract Stack Based Approach to Verified Compositional Compilation to Machine Code

This is the website hosting the artifact associated with our POPL'19 submission.

You will find our source code in [this archive](popl19-artifact.tar.gz).

### Requirements

The compiler is basd on CompCert. You can find the user manual of CompCert at [here](http://compcert.inria.fr/man/)

The development is known to compile with the following software:
- Menhir v.20170712 or later
- Coq v8.6.1
- OCaml v4.02.3

### Instructions

Download the source code and untar it to a directory.

The easiest way to setup the development environment is to use the Nix package manager. We already provide a configuration file for a nix environment. Simply run 'nix-shell' in the directory will install all the dependent pacakages and enter into a shell for compiling our source code.

You can also setup the environment using your preferred method (such as OPAM).

To compile our development, first configure the target architecture:

  <pre>./configure x86_32-linux</pre>
  
This will configure the development for the 32-bit x86 architecture. Note that our approach for machine code generation is architecture independent. We are limited to the 32-bit x86 architecture because the RockSalt instruction encoder only supports x86_32. 

Then generate the dependencies:

  <pre>make depend</pre>
  
Finally compile the whole development

  <pre>make -j</pre>
  
This will compile CompCertMC (which Stack-Aware CompCert is a part of) and Stack-Aware CompCertX.

This takes approximately 30-40 minutes depending on your machine. Note that the compilation of RockSalt encoder consumes a lot of memory. We recommand to have at least 16GB memory (We use a workstation with 64GB memory). Otherwise, the compilation may run out of memory.

### Usage

The Compiler is verified up to the MC language which is very close to machine code. Further compiler to machine code (ELF 32) is supported through an unverified process. For now we only support encoding of a subset of CompCert assembly instructions and the compilation to an ELF executable (not object) files. Moreover, the call to external functions are not supported because it would require us to implement a linker to link with the standard C libraries (which we have not time to do yet). For demonstration purpose, we have set up some examples in `c_examples`. 

<pre>
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
</pre>
