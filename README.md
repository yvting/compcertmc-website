# An Abstract Stack Based Approach to Verified Compositional Compilation to Machine Code

This is the website hosting the artifact associated with our POPL'19 submission.

You will find our source code in [this archive](popl19-artifact.tar.gz).

## Requirements

The compiler is basd on CompCert. You can find the user manual of CompCert at [here](http://compcert.inria.fr/man/)

The development is known to compile with the following software:
- Menhir v.20170712 or later
- Coq v8.6.1
- OCaml v4.02.3

## Instructions

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

## Usage

The Compiler is verified up to the MC language which is very close to machine code. Further compiler to machine code (ELF_X88_32 format) is supported through an unverified process. For now we only support encoding of a subset of CompCert assembly instructions and the compilation to an ELF executable (not object) files. Moreover, the call to external functions are not supported because it would require us to implement a linker to link with the standard C libraries (which we have not time to do yet). For demonstration purpose, we have set up some examples in `c_examples`. As an example, you can run the following example of fibonacci numbers.

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
  $ chmod +x fib.o
  $ ./fib.o 
  $ echo $?
    144
</pre>

Note that the the return values will be truncated if they exceeds 256. So sometimes you will see results that are not correct.

The most non-trivial example is an algorithm for solving the N-queen problem, which will return the number of solutions (See the table at [this wiki page](https://en.wikipedia.org/wiki/Eight_queens_puzzle) to find out the number of solutions). The following is an example that computes the number of solutions for 8-queen problem. Here the `-timings` flag is used to show the compilation passes it goes through.

<pre>
$ ../ccomp -timings -machinecode nqueen.c 
   0.00s  Parsing
   0.00s  Elaboration
   0.00s  Emulations
   0.00s  CompCert C generation
   0.00s  Clight generation
   0.00s  Simplification of locals
   0.00s  C#minor generation
   0.00s  Cminor generation
   0.00s  Instruction selection
   0.00s  RTL generation
   0.00s  Tail calls
   0.00s  Inlining
   0.00s  Renumbering
   0.00s  Constant propagation
   0.00s  CSE
   0.00s  Redundancy elimination
   0.00s  Unused globals
   0.00s  Register allocation
   0.00s  Branch tunneling
   0.00s  CFG linearization
   0.00s  Label cleanup
   0.00s  Mach generation
   0.00s  Asm generation
   0.00s  Elimination of pseudo instruction
   0.00s  Generation of FlatAsm
   0.00s  Generation of MC
   0.02s  Generation of RockSalt program
   0.12s  Total compilation time

$ ls
fac.c  fib.c  gcd_alt.c  gcd.c  max.c  nqueen.c  nqueen.o

$ chmod +x nqueen.o 

$ ./nqueen.o 

$ echo $?
92
</pre>


## Structure of the proofs

For reference, you can browse the original CompCert source code at [here](http://compcert.inria.fr/doc/index.html).

We have made the following changes to the origianl CompCert for our compilers

### Stack-Aware CompCert (For Pierre to finish)

* Addition of Abstract Stack to Memory Model 

* Instrumentation of Semantics

* Proofs of preservation of stack access policy and stack consumption

### CompCertMC

CompCert MC extends Stack-Aware CompCert with the following phases (all the files listed below are in the folder `x86` and are new additions we made to CompCert):

* Merging stack blocks into a single stack. We have designed an alternative semantics for CompCert assembly (we call `RawAsm`) which operates on a single stack that merges the original stack blocks.

  - `RawAsm.v` contains the semantics of assembly using a single stack
  - `RawAsmproof.v` contains the simulation proof from CompCert Asm to RawAsm
  
* Elimination of Pseudo Instructions  (For Pierre to finish)

   - `PseudoInstructions.v`
   - `PseudoInstructionsproof.v`
   - `RealAsm.v`
   - `RealAsmproof2.v`
   
* Layout the code and data in a flat memory. In this step, we concatenate the blocks for the global variables into a continous data segment and the instructions in global functions in a continous code segment.

   - `Segment.v` defines segments for code and data
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
   
* The Correcntess proofs of the compilation chain to MC

  We have linked the simulation proofs for all the passes to form the correcntess proofs of the compilation chain down to MC. They are the following theorems in `driver/Compiler.v`.
  
  - `c_semantic_preservation_mc` is the correcntess of compilation of whole programs
  - `separate_transf_c_program_correct_mc` is the correctness of separate compilation
 
* Generation of RockSalt Asm programs (not verified)
 
  - `RockSaltAsm.v` defines the assembly language based on RockSalt
  - `RockSaltAsmGen.v` generates `RockSaltAsm` programs from `MC` programs
  
* Generation of ELF files (not verified)

  - `RSAsmToElf.ml` is a OCaml program that writes `RockSaltAsm` programs into an ELF_x86_32 executable file.
 
 ### Stack-Aware CompCertX (For Pierre to finish)
 
