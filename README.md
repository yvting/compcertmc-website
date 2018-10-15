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

### Structure of the proofs

For reference, you can browse the original CompCert source code at [here](http://compcert.inria.fr/doc/index.html).

We have made the following changes to the origianl CompCert for our compilers

## Stack-Aware CompCert (For Pierre to finish)

* Addition of Abstract Stack to Memory Model 

* Instrumentation of Semantics

* Proofs of preservation of stack access policy and stack consumption

## CompCertMC

CompCert MC extends Stack-Aware CompCert with the following phases (all the files listed below are new additions we made to CompCert):

* Merging stack blocks into a single stack. We have designed an alternative semantics for CompCert assembly (we call `RawAsm`) which operates on a single stack that merges the original stack blocks.

  - `RawAsm.v` contains the semantics of assembly using a single stack
  - `RawAsmproof.v` contains the simulation proof from CompCert Asm to RawAsm
  
* Elimination of Pseudo Instructions  (For Pierre to finish)

   - `PseudoInstructions.v`
   - `PseudoInstructionsproof.v`
   - `RealAsm.v`
   - `RealAsmproof2.v`
   
* Layout the code and data in a flat memory. In this step, we concatenate the blocks for the global variables into a continous data segment and the instructions in global functions in a continous code segment.

   - `FlatAsmBuiltin.v` contains buildin definitions for FlatAsm
   - `FlatAsmProgram.v` contains the definition of functions, instructions and programs for assembly with a flat memroy space
   - `FlatAsmGlobenv.v` contains the definition of global environments for `FlatAsm` programs. We define a new version of global environment because the origianl CompCert assumes functions and data reside in separate blocks, which is no longer true for `FlatAsm` programs.
   - `FlatAsm.v` defines am assembly language that operates on the flat code and data segments. 
   - `FlatAsmgen.v` defines the generation of `FlatAsm` programs from `RealAsm` programs.
   - `FlatAsmgenproof.v` contains the simulation proof for `FlatAsmgen`.
   - `FlatAsmSep.v` defines the linking operators for `FlatAsm` programs and proves separate compilation of `FlatAsm` programs.
   
* Resolution of IDs. We transform FlatAsm programs by replacing the absolution addressing to relative addressing.

   - `MC.v` defines a language where all the labels are replaced by relative offsets and control flow operates by using those offsets
   - `MCgen.v` defines the generation from `FlatAsm` programs to `MC` programs
   - `MCgenproof.v` contains the simulation proof for `MCgen`
   - `MCSep.v` defines the linking operators for `MC` programs and proves separate compilation of `MC` programs.
   - 
 overall
 * The Correcntess proofs of the compilation chain to MC
 
 * Generation of RockSalt Asm programs (not verified)
 
 * Generation of ELF files
 
 ## Stack-Aware CompCertX (For Pierre to finish)
 
