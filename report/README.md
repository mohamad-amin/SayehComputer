# Design
Here we describe how we designed the details of our SayehComputer VHDL project.
## Pre Steps
well hello.
### Designing optional instructions
As described in page 6 of the project description (SAYEH Instructions), for some of the new operations we 
have to design some new mnemonic and bit vector representation. The next table is a representation of this design.

| Mnemonic        | Bits (15:0)           | RTL notation  |
| --- | :---: | --- |
| **xor** (xor registers) | `0000-10-11-0-XXX-D-S` | Rd **<=** Rd **xor** RS |
| **twc** (2s complement) | `0000-10-11-1-XXX-D-S` | Rd **<=** **~** Rs **+** 1 |
| **rnd** (generate random 16bit) | `0000-11-00-0-XXX-D-XX` | Rd **<=** random 16bit |
| **sqr** (calculate square root) | `0000-11-00-1-XXX-D-S` | Rd **<=** **sqrt(** Rs **)**|
| **div** (calculate division) | `0000-11-01-0-XXX-D-S` | Rd **<=** Rs **/** Rd |
| **sin** (calculate sinus) | `0000-11-01-1-XXX-D-S` | Rd **<=** **sin(** Rs **)**|
| **cos** (calculate co-sinus) | `0000-11-10-0-XXX-D-S` | Rd **<=** **cos(** Rs **)** |
| **tan** (calculate tangent) | `0000-11-10-1-XXX-D-S` | Rd **<=** **tan(** Rs **)** |
| **cot** (calculate co-tangent) | `0000-11-11-0-XXX-D-S` | Rd **<=** **cot(** Rs **)** |

Here `D` and `S`  are addresses of two registers (`Rd` and `Rs`) that we will be working on.

### Designing the `ControlWord`
Control word is a bit vector that the `Controller` uses to fill the signals of each signal in our CPU to execute an instruction. In each clock a specific `ControlWord` is used to perform an operation in the process of executing an instruction. 
We first define the bit vector representations for each component's signals and and then design the `ControlWord` using these newly-defined bit vectors. So starting from the components:

#### ALU Flags 
 We have `CSet`, `CReset`, `ZSet` and `SRload`. As only one of these signals will be high during an operation we use a **3Bit** bit vector to represent these signals. This 3Bit vector shows that:

| Bit Vector (1:0) | High Signal |
| --- | --- |
| `000` | CSet |
| `001` | CReset |
| `010` | ZSet |
| `011` | SRload |
| `100` | Nothing |

#### Arithmetic Unit
We have `AandB`, `AorB`, `NotB` , `AaddB` , `AsubB` , `AmulB` , `AcmpB` , `ShrB` , `ShlB`, `AxorB` , `Random` , `SqrB` , `AdivB` , `SinB` , `CosB` , `TanB` and `CotB` . As only one of these signals will be high during an operation we use a **5Bit** bit vector to represent these signals. This 5Bit vector shows that:

| Bit Vector (4:0) | High Signal |
| --- | --- |
| `00000` | AandB |
| `00001` | AorB |
| `00010` |  NotB |
| `00011` | Aadd |
| `00100` | AsubB |
| `00101` | AmulB |
| `00110` | AcmpB |
| `00111` | ShrB |
| `01000` | ShlB |
| `01001` | AxorB |
| `01010` | Random |
| `01011` | SqrB |
| `01101` | AdivB |
| `01110` | SinB |
| `01111` | CosB |
| `10000` | TanB |
| `10001` | CotB |

#### Address Logic
We have `ResetPC` , `PCplus1` , `PCplus0` , `R0plus1` and `R0plus0` . As only one of these signals will be high during an operation we use a **3Bit** bit vector to represent these signals. This 3Bit vector shows that:

| Bit Vector (2:0) | High Signal |
| --- | --- |
| `000` | ResetP |
| `001` | PCplus1 |
| `010` | PCplus0 |
| `011` | R0plus1 |
| `100` | R0plus0 |
| `101` | Nothing |

#### Register File
We have `RFLwrite` and `RFHwrite` . As only one of these signals will be high during an operation we use a **2Bit** to represent these signals. This 2Bit shows that:

| Bit | High Signal |
| --- | --- |
| `00` | RFLwrite |
| `01` | RFHwrite |
| `10` | Nothing |


#### Memory
We have `ReadMem` and `WriteMem` signals. So as only one of these control signals should be high at one clock, we use a **2Bit** representation for this:

| Bit | High Signal | 
| --- | --- |
| `00` | ReadMem |
| `01` | WriteMem |
| `10` | Nothing |

#### Window Pointer
We have `WPadd` and `WPreset` signals. So as only one of these control signals should be high at one clock, we use a **2Bit** representation for this:

| Bit | High Signal | 
| --- | --- |
| `00` | WPreset |
| `01` | WPadd |
| `10` | Nothing |

#### Other One-Bit signals
Here is a list of other signals that don't belong to a specific component but should be handled in control unit. These signals can't be encoded into smaller bit vectors because many of them can be high at once. In the following table you can see a list of all these signals and a small description of what they do if they're set to `1` (except `MemDataReady`).

| Signal | Description | 
| --- | --- |
| `Address_on_Databus `| Puts the address from `Address Logic` on the `Databus`  |
| `Rs_on_AddressUnitRSide` | Puts the data of `Rs` register on the `RSide` port of `Address Unit` |
| `Rd_on_AddressUnitRSide` | Puts the data of `Rd` register on the `RSide` port of `Address Unit` |
| `ALUout_on_Databus` | Puts the output of `ALU` on the `Databus`|
| `RFRight_on_OpndBus` | Puts the data of the `Rs` register on the `OpndBus` |
| `IR_on_LOpndBus` | Put the  |
| `IR_on_HOpndBus` | Hello |
| `MemDataReady` | When set to `1`, shows that the requested data from memory is available to put on the `Databus` |
| `Shadow` | Selects either `IR[11:8]` (or `IR[3:0]` when set to `0`) as the `Register File`'s address inputs |
| `IRload` | Puts data of the `Databus` into the `IR` register |

#### Defining the Control Word
So each signal has now been defined using a bit vector or a single bit as described above and it's time to merge these bits to create **a unique bit vector named control word** in our `control unit` to perform each operation.
We should have:

- **3** bits for the  `ALU Flag`
- **5** bits for the `Arithmetic Unit`
- **3** bits for the `Address logic`
- **2** bit for the `Register File`
- **2** bit for the `Memory`
- **2** bit for the `Window Pointer`
- And finally **10** bits for other one-bit signals that don't belong to a specific component.

**Note:** in control word these ten bits are the last ones and come in bit vector one by one (i.e. first bit in these ten bits corresponds to `Address_on_Databus` and the last bit corresponds to `IRload`)

So the `ControlWord` must have **27** bits and is each of it's bits are defined in the above list. A simple `ControlWord` would look like this:
```
000-00000-000-00-00-00-0000000000
```

### Designing process of each instruction
Here we describe how each instruction is going to be executed in the CPU using the datapath and the available components in the SAYEH computer. Here we describe and list what should be done (the operations) after decoding each instruction to execute it. 

#### No Operation `nop`

#### Halt Operation `hlt`

#### Set Zero Flag `nop`

#### Clear Zero Flag `nop`

#### Set Carry Flag `nop`

#### Clear Window Pointer `nop`

#### **Move Register** `mvr` (0001-D-S)
We need to clocks to execute this operation. In the first clock we move the data of the `Rs` register to `DataBus` and in the second clock we move the data in the `DataBus` to the `Rd` register. So in the first clock we:

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


#### **Load Addressed** `lda` (0010-D-S)
We need to clocks to execute this operation. In the first clock we move the address of the `Rs` register to `DataBus` and in the second clock we move the data in the `DataBus` to the `Rd` register. So in the first clock we:

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
