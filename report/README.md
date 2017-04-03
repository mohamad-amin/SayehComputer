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
| `000` | Nothing |
| `001` | CReset |
| `010` | ZSet |
| `011` | SRload |
| `100` | CSet |

#### Arithmetic Unit
We have `AandB`, `AorB`, `NotB` , `AaddB` , `AsubB` , `AmulB` , `AcmpB` , `ShrB` , `ShlB`, `AxorB` , `Random` , `SqrB` , `AdivB` , `SinB` , `CosB` , `TanB` and `CotB` . As only one of these signals will be high during an operation we use a **5Bit** bit vector to represent these signals. This 5Bit vector shows that:

| Bit Vector (4:0) | High Signal |
| --- | --- |
| `00000` | AandB |
| `00001` | AorB |
| `00010` | NotB |
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
| `10010` | B15to0 |

#### Address Logic
We have `ResetPC` , `PCplus1` , `PCplus0` , `R0plus1` and `R0plus0` . As only one of these signals will be high during an operation we use a **3Bit** bit vector to represent these signals. This 3Bit vector shows that:

| Bit Vector (2:0) | High Signal |
| --- | --- |
| `000` | Nothing |
| `001` | PCplus1 |
| `010` | PCplus0 |
| `011` | R0plus1 |
| `100` | R0plus0 |
| `101` | ResetPC |

#### Register File
We have `RFLwrite` and `RFHwrite` . As only one of these signals will be high during an operation we use a **2Bit** to represent these signals. This 2Bit shows that:

| Bit | High Signal |
| --- | --- |
| `00` | Nothing |
| `01` | RFHwrite |
| `10` | RFLwrite |


#### Memory
We have `ReadMem` and `WriteMem` signals. So as only one of these control signals should be high at one clock, we use a **2Bit** representation for this:

| Bit | High Signal | 
| --- | --- |
| `00` | Nothing |
| `01` | WriteMem |
| `10` | ReadMem |

#### Window Pointer
We have `WPadd` and `WPreset` signals. So as only one of these control signals should be high at one clock, we use a **2Bit** representation for this:

| Bit | High Signal | 
| --- | --- |
| `00` | Nothing |
| `01` | WPadd |
| `10` | WPreset |

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
| `EnablePC` | Regular enable flag for the `PC` register |
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
- And finally **11** bits for other one-bit signals that don't belong to a specific component.

**Note:** in control word these ten bits are the last ones and come in bit vector one by one (i.e. first bit in these ten bits corresponds to `Address_on_Databus` and the last bit corresponds to `IRload`)

So the `ControlWord` must have **27** bits and is each of it's bits are defined in the above list. A simple `ControlWord` would look like this:
```
000-00000-000-00-00-00-00000000000
```


### Designing how instructions are executed (Finite State Machine)
At first, `PC` is initialized to **20** where is the start of the instructions that will be performed by this computer. Then we follow these steps (**synchronous with positive clock edges**) to execute each instruction (actually, these are the first steps of our FSM that is going to work in our Control Unit to manage our CPU) :

1. The `ControlWord` is set to `Control Word here` to allow the instruction to be loaded on the `Databus` from `Memory`. 
2. We wait (**asynchronous from clock pulses**) until `MemDataReady` is one and then set the `ControlWord` to `Control Word Here` to make the `IR` register load the instruction.
3. We decode the `IR`'s data in the Control Unit to see if it is two 8-bit instructions or one 16-bit instruction. If it was one 16-bit instruction, we move to the step 4, otherwise we move to the 6th step.
4. The controller provides the corresponding `ControlWord` (one or more, depending on the instruction) based on the instruction (that is discussed in the next section) and some operations is performed based on the instruction (Actually, here we provide the corresponding state for the instruction and here we decode the instruction to execute and then we move to that state as the current state and the rest of executing the instruction is decided in that state).
5. After completion of the instruction's execution, the controller provides `Control Word here` as the `ControlWord` to increment the `PC` register by `1` and to move to the next instructions (we return to the first state (*number `1`*), as you might've gussed). 
6. Now that we know the data of the `IR` register consists of two 8-bit instructions **and** we're going to execute the instruction in `IR[15:8]`. **For deciding that we should execute the low 8-bits of `IR` data or the high 8-bits**, we use a flag named `ShadowHigh` flag that at first is initialized to `1`; if the flag is `1` we know that we should execute `IR[15:8]` and if it's `0` we know that we should execute `IR[7:0]`. The controller provides the corresponding `ControlWord` **as in state 4** to execute the first instruction.
**Important Note:** We modify the `ShadowHigh` flag and set it as `ShadowHigh <= ~ShadowHigh` **after executing each 8-bit instruction**. 
7. The controller provides the `ControlWord` as in step 6, **but** this time the decoding process is based on `IR[7:0]` (because of `ShadowHigh = 0` flag) to execute the next (AKA shadow) instruction in our 16-bit data.
8. After completion of the both instruction's execution, the controller provides `Control Word here` as the `ControlWord` to increment the `PC` register by `1` and to move to the next instructions (again we return to the first state (*number `1`*). 
 
**Important Note:** In instructions that are made up of two 8-bit instructions, the `shadow` flag is as the same as the `ShadowHigh` flag. In 16-bit instructions though, the `shadow` flag is always set to `0`.
### Designing process of each instruction
Here we describe how each instruction is going to be executed in the CPU using the datapath and the available components in the SAYEH computer. Here we describe and list what should be done (the operations) after decoding each instruction to execute it. 

#### **No Operation** `nop` (0000-00-00)
We don't need to set any `ControlWord` here as we won't perform any specific operation.
So we just go to the next  step and perform the next operations.
#### Halt Operation `hlt` (0000-00-01)

#### **Set Zero Flag** `szf`(0000-00-10)
The controller should set the `ZSet` flag of `ALU` to `1` to Set the zero flag. So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.  

#### **Clear Zero Flag** `czf`(0000-00-11)
The controller should set the `ZReset` flag of `ALU` to `1` to clear the zero flag.  So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.  

#### **Set Carry Flag** `scf`(0000-01-00)
The controller should set the `CSet` flag of `ALU` to `1` to Set the zero flag. So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.  

#### **Clear Carry Flag** `scf`(0000-01-01)
The controller should set the `CReset` flag of `ALU` to `1` to Reset the zero flag. So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.  

#### **Clear Window Pointer** `cwp`(0000-01-10)
The controller should set the `WPReset` flag of `WP` to `1` to clear the window pointer. So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.  

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
We need two clocks to execute this operation. In the first clock we move the address of the `Rs` register to `DataBus` and in the second clock we move the data in the `DataBus` to the `Rd` register. So in the first clock we:

#### **Store Addressed** `sta`(0011-D-S)
We need two clocks to execute operation. In the first clock the address of `(Rd)` must be put on the `AddressLogic` wire and indicate to the `address` signal of our `memory` component and then in the second clock the data of `Rs` register should be put in the `memory` with the indicated `address`.

So for the first clock the control unit:

- Sets `Rd_on_AddressUnitRSide` to `1` to put the data of `Rd` register on the `RSide` wire of `AddressLogic`.
- Sets `EnablePC` to `1` to prevent contents of the `PC` to get replaced by the data of `Rd` register.
- Sets the `Rplus0` signal of `AddressLogic` to `1` to set the data of the `RSide` wire as `AddressUnit`'s output.

So the  `ControlWord` for the first clock would look like this:
```
Control Word here
```

And in the next clock the control unit:

- Sets `B15to0` flag of `ALU` to `1` to put `Rs` register's data on `ALU` output.
- Sets `ALUout_on_Databus` to `1` to  put `ALU`'s output on the `Databus`.
- Sets `WriteMem` flag of `Memory` to `1` to write the `Databus`'s data to `memory`.

So the `ControlWord` for the second clock is:
```
Control Word here
```

#### Input From Port `inp` (0100-D-S)
TODO

#### Output From Port `oup` (0101-D-S)
TODO

#### **AND Registers** `and`(0110-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `AandB` flag of the `ALU` to `1` to `AND` the data of `Rs` and `Rd` reigisters.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### **OR Registers** `orr`(0111-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `AorB` flag of the `ALU` to `1` to `OR` the data of `Rs` and `Rd` reigisters.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```


#### **NOT Register** `not`(1000-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `NotB` flag of the `ALU` to `1` to `Not` the data of `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```
.

#### **Shift Left** `shl`(1001-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `ShlB` flag of the `ALU` to `1` to `Shift Left` the data of `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```


#### **Shift Right** `shr`(1010-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `ShrB` flag of the `ALU` to `1` to `Shift Right` the data of `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```


#### **Add Registers** `add`(1011-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `AaddB` flag of the `ALU` to `1` to `Add` the data of `Rs` and `Rd` reigisters.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```


#### **Subtract Registers** `sub`(1100-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `AsubB` flag of the `ALU` to `1` to `Subtract` the data of `Rs` and `Rd` reigisters.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### Multiply Registers `mul` (1101-D-S)
Todo	
#### Compare`cmp`(1110-D-S)
The controller should set the `AcmpB` flag of `ALU` to `1` to compare Rs and Rd  .So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.

#### Move Immediate Low `mil` (1111-D-00-I)

#### Move Immediate High `mih`(1111-D-01-I)

#### Save PC `spc` (1111-D-10-I)

#### Jump Addressed `jpa` (1111-D-11-I)
 
#### Jump Relative `nop`

#### Branch If Zero `nop`

#### Branch If Carry `nop`

#### Add Window Pointer`nop`

#### **XOR Registers** `xor`(0000-10-11-0-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `AxorB` flag of the `ALU` to `1` to `XOR` the data of `Rs` and `Rd` reigisters.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```


#### **2s Complement** `twc`(0000-10-11-1-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `B2sComplement` flag of the `ALU` to `1` to `2s Complement` the data of `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### **Generate Random** `rnd`(0000-11-00-0-XXX-D-XX)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `rand` flag of the `ALU` to `1` to Generate `Random` number .
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### **Calculate Square Root** `sqr`(0000-11-00-1-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `sqrB` flag of the `ALU` to `1` to `Square` the data of `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### **Divide Registers** `div`(0000-11-01-0-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `AdivB` flag of the `ALU` to `1` to `Divide` the data of `Rs` and `Rd` reigisters.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

####  **Calculate Sinus** `sin`(0000-11-01-1-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `sinB` flag of the `ALU` to `1` to calculate `Sinus` of the `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### **Calculate Co-Sinus** `cos`(0000-11-10-0-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `cosB` flag of the `ALU` to `1` to calculate `Co-Sinus` of the `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```

#### **Calculate Tangent** `tan`(0000-11-10-1-XXX-D-S)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `tanB` flag of the `ALU` to `1` to calculate `Tangent` of the `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```
#### **Calculate Co-Tangent** `cot`(0000-11-11-0-XXX-D-S	)
We need three clocks to execute this operation:
In the first clock, the control unit:

- Sets `cotB` flag of the `ALU` to `1` to calculate `Co-Tangent` of the `Rs` reigister.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.

So the `ControlWord` for the first clock is:
```
Control Word here
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
Control Word here
```
