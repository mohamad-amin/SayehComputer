# Design
Here we describe how we designed the details of our SayehComputer VHDL project.
## Pre Steps
well hello.
### Designing optional instructions
As described in page 6 of the project description (SAYEH Instructions), for some of the new operations we 
have to design some new mnemonic and bit vector representation. The next table is a representation of this design.

| Mnemonic        | Bits (15:0)           | RTL notation  |
| --- | :---: | --- |
| **xor** (xor registers) | `0000-10-11-0-D-S` | Rd **<=** Rd **xor** RS |
| **twc** (2s complement) | `0000-10-11-1-D-S` | Rd **<=** **~** Rs **+** 1 |
| **rnd** (generate random 16bit) | `0000-11-00-0-D` | Rd **<=** random 16bit |
| **sqr** (calculate square root) | `0000-11-00-1-D-S` | Rd **<=** **sqrt(** Rs **)**|
| **div** (calculate division) | `0000-11-01-0-D-S` | Rd **<=** Rs **/** Rd |
| **sin** (calculate sinus) | `0000-11-01-1-D-S` | Rd **<=** **sin(** Rs **)**|
| **cos** (calculate co-sinus) | `0000-11-10-0-D-S` | Rd **<=** **cos(** Rs **)** |
| **tan** (calculate tangent) | `0000-11-10-1-D-S` | Rd **<=** **tan(** Rs **)** |
| **cot** (calculate co-tangent) | `0000-11-11-0-D-S` | Rd **<=** **cot(** Rs **)** |

Here `D` and `S`  are addresses of two registers (`Rd` and `Rs`) that we will be working on.

### Designing process of each instruction
Here we describe how each instruction is going to be executed in the CPU using the datapath and the available components in the SAYEH computer. Here we describe and list what should be done (the operations) after decoding each instruction to execute it. 

#### No Operation `nop`

#### Halt Operation `hlt`

#### Set Zero Flag `nop`

#### Clear Zero Flag `nop`

#### Set Carry Flag `nop`

#### Clear Window Pointer `nop`

#### **Move Register** `mvr` (0001-D-S)
We need to clocks to execute this operation. In the first clock we move the data of the `Rd` register to `DataBus` and in the second clock we move the data in the `DataBus` to the `Rd` register. So in the first clock we:

- Set `shadow` to `1` to let `IROut[11:8]` go to `Right` input of `RegisterFile` as selection bits for `Rd` and `Rs`  registers that are defined by `D` and `S` in instruction bit representation.
- Set `B15to0` and `ALUout_on_Databus` to `1` to send the data of the `Rs` register to the `DataBus` through the `ALU`.

So the `ControlWord` would look like this:
```
Control Word here
```

Then in the second clock we:

- Set `RFLRead` and `RFLWrite` to `1` to make the `RegisterFile` able to read all 16 bit input from `DataBus`. 
- Let `shadow` remain `1` to perform as the previous clock.

And the `ControlWord` for the second clock is:
```
Control Word here
```


#### Load Addressed `nop`

#### Store Addressed `nop`

#### Input From Port `nop`

#### Output From Port `nop`

#### AND Registers `nop`

#### OR Registers `nop`

#### NOT Register `nop`

#### Shift Left `nop`

#### Shift Right `nop`

#### Add Registers `nop`

#### Subtract Registers `nop`

#### Multiply Registers `nop`

#### Compare`nop`

#### OR Registers `nop`

#### NOT Register `nop`

#### Shift Left `nop`

#### Shift Right `nop`

#### Add Registers `nop`

#### Subtract Registers `nop`

#### Multiply Registers `nop`

#### Compare`nop`

#### Move Immediate Low `nop`

#### Move Immediate High `nop`

#### Save PC `nop`

#### Jump Addressed `nop`

#### Jump Relative `nop`

#### Branch If Zero `nop`

#### Branch If Carry `nop`

#### Add Window Pointer`nop`

#### XOR Registers `nop`

#### 2s Complement `nop`

#### Generate Random `nop`

#### Calculate Square Root `nop`

#### Divide Registers`nop`

####  Calculate Sinus`nop`

#### Calculate Co-Sinus `nop`

#### Calculate Tangent `nop`

#### Calculate Co-Tangent `nop`
