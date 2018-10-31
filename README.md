# An Abstract Stack Based Approach to Verified Compositional Compilation to Machine Code

This is the website hosting the artifact associated with our POPL'19 submission.

You will find our source code in [this archive](compcert-new-popl19-32bits.zip).

## Requirements

The compiler is basd on CompCert. You can find the user manual of CompCert [here](http://compcert.inria.fr/man/).

The development is known to compile with the following software:
- Menhir v.20170712 or later
- Coq v8.6.1
- OCaml v4.02.3

## Instructions

Download the source code and unzip it to a directory.

The easiest way to setup the development environment is to use the [Nix package manager](https://nixos.org/nix/). We provide a configuration file (`default.nix`) for a nix environment. Simply run `nix-shell` in the directory will install all the dependent pacakages and enter into a shell for compiling our source code.

You can also setup the environment using your preferred method (such as OPAM).

To compile our development, first configure the target architecture:

  <pre>./configure x86_32-linux</pre>
  
This will configure the development for the 32-bit x86 architecture. Note that generally speaking our approach for machine code generation is architecture independent. We are limited to the 32-bit x86 architecture because the [RockSalt](https://github.com/gangtan/CPUmodels) instruction encoder we use for this project only supports x86_32. 

Then generate the dependencies:

  <pre>make depend</pre>
  
Finally compile the whole development

  <pre>make -j</pre>
  
This will compile CompCertMC (which Stack-Aware CompCert is a part of) and Stack-Aware CompCertX.

This takes approximately 30-40 minutes depending on your machine. Note that the compilation of RockSalt encoder consumes a lot of memory. To avoid running out of memory, we recommand to have at least 16GB memory (We use a workstation with 64GB memory).

## Usage

The Stack-Aware CompCert is a part of CompCertMC which on its own works in the same way as the original CompCert. CompCertMC supports compilation to ELF files. Stack-Aware CompCertX supports contextual compilation, which will replace CompCertX in the [CertiKOS](http://flint.cs.yale.edu/certikos/) project.

The CompCertMC compiler is verified up to the MC language which is very close to machine code. Further compiler to machine code (ELF_X88_32 format) is supported through an unverified process. For now we only support encoding of a subset of CompCert assembly instructions and the compilation to an ELF executable (not object) files. Moreover, the call to external functions are not supported because it would require us to implement a linker to link with the standard C libraries (we are still trying to figure out how to implement such a linker). For the purpose of demonstration, we have set up some examples in `c_examples`. As an example, you can compile and run the following program for computing fibonacci numbers.

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

The `-machinecode` flag is passed to CompCertMC for generating machine code. The resulting ELF file is `fib.o`. The last echo command outputs the most recent return value, which is the 12th fibonacci number. 

The following is another example of compiling and running a program that computes factorials.

<pre>
$ cat fac.c 

int fac(int n)
{
  int v = 0;

  if (n == 0) {
    v = 1;
  } else {
    v = n * fac(n-1);
  }

  return v;
}

int main()
{
  return fac(6);
}

$ ../ccomp -timings -machinecode fac.c 
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
   0.01s  Generation of RockSalt program
   0.03s  Total compilation time

$ chmod +x ./fac.o 
$ ./fac.o 
$ echo $?
208

</pre>

Here we use the `-timings` flag to show the compilation passes went through. Note that the correct result is 720. We got 208 because return value is truncated to a byte.

Our most non-trivial example is an algorithm for solving the N-queen problem, which will return the number of solutions (See the table at [this wiki page](https://en.wikipedia.org/wiki/Eight_queens_puzzle) to find out the number of solutions).  

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

For reference, you can browse the original CompCert source code [here](http://compcert.inria.fr/doc/index.html).

We also export our code as [browsable HTML files](./doc/).

We have made the following changes to the origianl CompCert for our compilers.

### Stack-Aware CompCert

* Extension of the Memory Model with an Abstract Stack

The abstract stack is defined in [StackADT.v](doc/StackADT.html#stack). The definitions of stages, frames and abstract blocks described in the paper can be found in this file. Lower in this file are the definition of [stack injection](doc/StackADT.html#stack_inject).

Then in [Memimpl.v](doc/Memimpl.html#mem'), the CompCert memory model is enriched with a stack and some invariants on this stack, described as the `stack_inv` predicate. The new operations [record](doc/Memimpl.html#Mem.record_stack_blocks), [push_stage](doc/Memimpl.html#Mem.push_new_stage) and [pop_stage](doc/Memimpl.html#Mem.unrecord_stack_block) are defined here.

* Instrumentation of Semantics

The semantics of every language is instrumented to insert stack operations. Let us have a look for example at the semantics for [Cminor](doc/Cminor.html#step_internal_function). The rule `step_internal_function` records a new frame, based on the size of the stack block. The rule `step_return` pops the current stage and the `step_call` rule pushes a new stage.

Similar changes can be observed in the semantics of all languages except Mach and Asm: [RTL](doc/RTL.html), [Clight](doc/Clight.html) for instance.

* Proofs of preservation of stack access policy and stack consumption

The stack access policy is stated as [the `stack_access` predicate](doc/StackADT.html#stack_access). It is preserved under injection, as shown [here](doc/StackADT.html#stack_access_inj_gen).

The size of the stack is defined [here](doc/StackADT.html#size_stack). For most compiler passes, the stacks are in the [`stack_equiv`](doc/StackADT.html#stack_equiv) relation, implying they are of the same size.

For Inlining, the stacks are in the [`inline_sizes`](doc/Inliningproof.html#inline_sizes) relation, meaning a number of stages in the source correspond to one frame in the target stack. A [corollary of this relation](doc/Inliningproof.html#inline_sizes_size_stack) is that the target stack is not larget than the source stack.

Similarly for Tailcall recognition: the relation is [`tc_sizes`](doc/Tailcallproof.v#tc_sizes).

### CompCertMC

CompCert MC extends Stack-Aware CompCert with the following phases (all the files listed below are in the folder `x86` and are new additions we made to CompCert):

* Merging stack blocks into a single stack. We have designed an alternative semantics for CompCert assembly (we call `RawAsm`) which operates on a single stack that merges the original stack blocks.

  - [RawAsm.v](doc/RawAsm.html) contains the semantics of assembly using a single stack
  - [RawAsmproof.v](doc/RawAsmproof.html) contains the simulation proof from CompCert Asm to RawAsm
  
* Elimination of pseudo instructions. This pass eleminates pseudo instructions in CompCert assembly that do not correspond to actual machine instructions, including `Pallocframe`, `Pfreeframe` and `Pload_parent_pointer`.

   - [RealAsm.v](doc/RealAsm.html) is an alternate semantics for assembly, where the responsibility of pushing the return address at a call site is transferred from the callee (in CompCert Asm) to the caller (as in regular x86 assembly), similarly for function return, and pseudo instructions are updated accordingly.
   - [RealAsmproof2.v](doc/RealAsmproof2.html) contains a proof that the RealAsm semantics simulates the RawAsm semantics, i.e. every behaviour of the RealAsm semantics is also a behaviour of the RawAsm semantics.
   - [PseudoInstructions.v](doc/PseudoInstructions.html) is a compiler pass from RealAsm to RealAsm, where pseudo-instructions are eliminated. 
   - [PseudoInstructionsproof.v](doc/PseudoInstructionsproof.html) contains the correctness proof of the pseudo-instructions elimination pass.
   
* Layout the code and data in a flat memory. In this step, we concatenate the blocks for the global variables into a continous data segment and the instructions in global functions into a continous code segment.

   - [Segment.v](doc/Segment.html) defines segments for code and data
   - [FlatAsmBuiltin.v](doc/FlatAsmBuiltin.html) contains builtin definitions for FlatAsm
   - [FlatAsmProgram.v](doc/FlatAsmProgram.html) contains the definition of functions, instructions and programs for assembly with a flat memroy space
   - [FlatAsmGlobenv.v](doc/FlatAsmGlobenv.html) contains the definition of global environments for `FlatAsm` programs. We define a new version of global environment because the origianl CompCert assumes functions and data reside in separate blocks, which is no longer true for `FlatAsm` programs.
   - [FlatAsm.v](doc/FlatAsm.html) defines an assembly language that operates on the flat code and data segments. 
   - [FlatAsmgen.v](doc/FlatAsmgen.html) defines the generation of `FlatAsm` programs from `RealAsm` programs.
   - [FlatAsmgenproof.v](doc/FlatAsmgenproof.html) contains the simulation proof for `FlatAsmgen`.
   - [FlatAsmSep.v](doc/FlatAsmSep.html) defines the linking operators for `FlatAsm` programs and proves separate compilation of `FlatAsm` programs.
   
* Resolution of IDs. We transform FlatAsm programs by replacing the absolute addressing with relative addressing.

   - [MC.v](doc/MC.html) defines a language where all the labels are replaced by relative offsets and control flow operates by using those offsets
   - [MCgen.v](doc/MCgen.html) defines the generation from `FlatAsm` programs to `MC` programs
   - [MCgenproof.v](doc/MCgenproof.html) contains the simulation proof for `MCgen`
   - [MCSep.v](doc/MCSep.html) defines the linking operators for `MC` programs and proves separate compilation of `MC` programs.
   
* The correcntess proof of the compilation chain to MC

  We have linked the simulation proofs for all the passes to form the correcntess proofs of the compilation chain down to MC. They are the following theorems in [`driver/Compiler.v`](doc/Compiler.html).
  
  - [c_semantic_preservation_mc](doc/Compiler.html#c_semantic_preservation_mc) is the correcntess of compilation of whole programs
  - [separate_transf_c_program_correct_mc](doc/Compiler.html#separate_transf_c_program_correct_mc) is the correctness of separate compilation
 
* Generation of RockSalt Asm programs (not verified). This makes use of the RockSalt x86 machine model (in the directory `cpu_models`).
 
  - [RockSaltAsm.v](doc/RockSaltAsm.html) defines the assembly language based on RockSalt
  - [RockSaltAsmGen.v](doc/RockSaltAsmGen.html) generates `RockSaltAsm` programs from `MC` programs
  
* Generation of ELF files (not verified)

  - RSAsmToElf.ml is an OCaml program that writes `RockSaltAsm` programs into an ELF_x86_32 executable file.
 
### Stack-Aware CompCertX
 
 The `compcertx` folder contains new files for the contextual compilation aspect of this work.
 Each language semantics has an `X` version in this folder, where the semantics are redefined so that they do not denote the meaning of a main program without arguments but rather the execution of an arbitrary function in a program, starting from an arbitrary memory state and register state (when applicable). Look for example at [ClightX](doc/ClightX.html) or [RTLX](doc/RTLX.html). The `step` relation is kept but the initial and final states are redefined.
 
 The correctness proofs in `*proofX.v` are mostly reused from the classic CompCert version, with some adjustments for the new initial and final states.
 
 The final theorem for CompCertX is found in [SeparateCompilerproof.v](doc/SeparateCompilerproof.html).
 
