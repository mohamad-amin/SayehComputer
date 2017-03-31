# Design
Here we describe how we designed the details of our SayehComputer VHDL project.
## Pre Steps
well hello.
### Design optional instructions
As described in page 6 of the project description (SAYEH Instructions), for some of the new operations we 
have to design some new mnemonic and bit vector representation. The next table is a representation of this design.

| Mnemonic        | Bits (15:0)           | RTL notation  |
| --- | :---: | --- |
| **xor** (xor operation) | 0000-01-11-D-S | Rd **<=** Rd **xor** RS |
| **twc** (2s complement) | 0000-10-00-D-S | Rd **<=** **~** Rs **+** 1 |
| **rnd** (generate random 16bit) | 0000-10-01-D | Rd **<=** random 16bit |
| **sqr** (calculate square root) | 0000-10-10-D-S | Rd **<=** **sqrt(** Rs **)** |
| **div** (calculate division) | 0000-10-11-D-S | Rd **<=** Rs **/** Rd |
| **sin** (calculate sinus) | 0000-11-00-D-S | Rd **<=** **sin(** Rs **)**|
| **cos** (calculate co-sinus) | 0000-11-01-D-S | Rd **<=** **cos(** Rs **)** |
| **tan** (calculate tangent) | 0000-11-10-D-S | Rd **<=** **tan(** Rs **)** |
| **cot** (calculate co-tangent) | 0000-11-11-D-S | Rd **<=** **cot(** Rs **)** |
