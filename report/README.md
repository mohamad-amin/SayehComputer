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

| Bit Vector (2:0) | High Signal |
| --- | --- |
| `000` | Nothing |
| `001` | CReset |
| `010` | ZSet |
| `011` | SRload |
| `100` | CSet |
| `101` | ZReset |

#### Arithmetic Unit
We have `AandB`, `AorB`, `NotB` , `AaddB` , `AsubB` , `AmulB` , `AcmpB` , `ShrB` , `ShlB`, `AxorB` , `Random` , `SqrB` , `AdivB` , `SinB` , `CosB` , `TanB`, `CotB`, `TwcB`. As only one of these signals will be high during an operation we use a **5Bit** bit vector to represent these signals. This 5Bit vector shows that:

| Bit Vector (4:0) | High Signal |
| --- | --- |
| `00000` | AandB |
| `00001` | AorB |
| `00010` | NotB |
| `00011` | AaddB |
| `00100` | AsubB |
| `00101` | AmulB |
| `00110` | AcmpB |
| `00111` | ShrB |
| `01000` | ShlB |
| `01001` | AxorB |
| `01010` | Random |
| `01011` | SqrB |
| `01100` | AdivB |
| `01101` | SinB |
| `01110` | CosB |
| `01111` | TanB |
| `10000` | CotB |
| `10001` | B15to0 |
| `10010` | TwcB |

#### Address Logic
We have `ResetPC` , `PCplus1` , `PCplus0` , `R0plus1` and `R0plus0` . As only one of these signals will be high during an operation we use a **3Bit** bit vector to represent these signals. This 3Bit vector shows that:

| Bit Vector (2:0) | High Signal |
| --- | --- |
| `000` | PCplusI |
| `001` | PCplus1 |
| `010` | R0plusI |
| `011` | R0plus0 |
| `100` | ResetPC |
| `others` | PC |

#### Register File
We have `RFLwrite` and `RFHwrite` . As only one of these signals will be high during an operation we use a **2Bit** to represent these signals. This 2Bit shows that:

| Bit Vector (1:0) | High Signal |
| --- | --- |
| `00` | Nothing |
| `01` | RFHwrite |
| `10` | RFLwrite |
| `11` | Write both |


#### Memory
We have `ReadMem` and `WriteMem` signals. So as only one of these control signals should be high at one clock, we use a **2Bit** representation for this:

| Bit Vector (1:0) | High Signal | 
| --- | --- |
| `00` | Nothing |
| `01` | WriteMem |
| `10` | ReadMem |

#### Window Pointer
We have `WPadd` and `WPreset` signals. So as only one of these control signals should be high at one clock, we use a **2Bit** representation for this:

| Bit Vector (1:0) | High Signal | 
| --- | --- |
| `00` | Nothing |
| `01` | WPadd |
| `10` | WPreset |

#### Databus tri-state control signals
These signals select the data to be written on the `Databus`. We use a **2Bit** representation for this:

| Bit Vector (1:0) | High Signal |
| --- | --- |
| `00` | Nothing |
| `01` | ALUout_on_Databus |
| `10` | Address_on_Databus |

#### AddressUnitRSideBus tri-state control signals
These signals select the data to be written on the `AddressUnitRSideBus`. We use a **2Bit** representation for this:

| Bit Vector (1:0) | High Signal |
| --- | --- |
| `00` | Nothing |
| `01` | Rs_on_AddressUnitRSide |
| `10` | Rd_on_AddressUnitRSide |

#### Other One-Bit signals
Here is a list of other signals that don't belong to a specific component but should be handled in control unit. These signals can't be encoded into smaller bit vectors because many of them can be high at once. In the following table you can see a list of all these signals and a small description of what they do if they're set to `1` (except `MemDataReady`).

| Signal | Description | 
| --- | --- |
| `Shadow` | Selects either `IR[11:8]` (or `IR[3:0]` when set to `0`) as the `Register File`'s address inputs |
| `EnablePC` | Regular enable flag for the `PC` register |
| `IRload` | Puts data of the `Databus` into the `IR` register |

#### Defining the Control Word
So each signal has now been defined using a bit vector or a single bit as described above and it's time to merge these bits to create **a unique bit vector named control word** in our `control unit` to perform each operation.
We should have:

- **3** bits for the  `ALU Flag`
- **5** bits for the `Arithmetic Unit`
- **3** bits for the `Address logic`
- **2** bits for the `Register File`
- **2** bits for the `Memory`
- **2** bits for the `Window Pointer`
- **2** bits for the `Databus`
- **2** bits for the `AddressUnitRSideBus`
- And finally **3** bits for other one-bit signals that don't belong to a specific component.

**Note:** in control word these ten bits are the last ones and come in bit vector one by one (i.e. first bit in these ten bits corresponds to `Address_on_Databus` and the last bit corresponds to `IRload`)

So the `ControlWord` must have **24** bits and is each of it's bits are defined in the above list. A simple `ControlWord` would look like this:
```
000-00000-000-00-00-00-00-00-000
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
We don't need to set the `ControlWord` below as we won't perform any specific operation.
So we just go to the next  step and perform the next operations 
```
000-10001-111-00-00-00-00-00-S00
```

#### Halt Operation `hlt` (0000-00-01)
TODO

#### **Set Zero Flag** `szf`(0000-00-10)
The controller should set the `ZSet` flag of `ALU` to `1` to Set the zero flag. So it provides the `ControlWord` below  to perform this operation in one clock.
```
000-10001-010-00-00-00-00-00-S00
```

#### **Clear Zero Flag** `czf`(0000-00-11)
The controller should set the `ZReset` flag of `ALU` to `1` to clear the zero flag.  So it provides the `ControlWord` value `Control Word Here` to perform this operation in one clock.  
```
000-10001-101-00-00-00-00-00-S00
```

#### **Set Carry Flag** `scf`(0000-01-00)
The controller should set the `CSet` flag of `ALU` to `1` to Set the zero flag. So it provides the `ControlWord` below  to perform this operation in one clock.  
```
000-10001-100-00-00-00-00-00-S00
```

#### **Clear Carry Flag** `scf`(0000-01-01)
The controller should set the `CReset` flag of `ALU` to `1` to Reset the zero flag. So it provides the `ControlWord`below  to perform this operation in one clock.  
```
000-10001-001-00-00-00-00-00-S00
```

#### **Clear Window Pointer** `cwp`(0000-01-10)
The controller should set the `WPReset` flag of `WP` to `1` to clear the window pointer. So it provides the `ControlWord` below  to perform this operation in one clock.  
```
000-10001-111-00-00-10-00-00-S00
```

#### **Move Register** `mvr` (0001-D-S)
We need two clocks to execute this operation. In the first clock we move the data of the `Rs` register to `DataBus` and in the second clock we move the data in the `DataBus` to the `Rd` register. So in the first clock we:

- Set `B15to0` and `ALUout_on_Databus` to `1` to send the data of the `Rs` register to the `DataBus` through the `ALU`.

So the `ControlWord` would look like this:
```
000-10001-111-00-00-00-01-00-S00
```

Then in the second clock we:

- Set `RFLRead` and `RFLWrite` to `1` to make the `RegisterFile` able to read all 16 bit input from `DataBus`. 

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Load Addressed** `lda` (0010-D-S)
We need three (effective) clocks to execute operation. In the first clock the address of `(Rs)` must be put on the `AddressLogic` wire and indicate to the `address` signal of our `memory` component and then in the second clock the data of `Rs` register should be put in the `memory` with the indicated `address`.

So for the first clock the control unit:

- Sets `Rs_on_AddressUnitRSide` to `1` to put the data of `Rs` register on the `RSide` wire of `AddressLogic`.
- Sets `EnablePC` to `0` to prevent contents of the `PC` to get replaced by the data of `Rs` register.
- Sets the `Rplus0` signal of `AddressLogic` to `1` to set the data of the `RSide` wire as `AddressUnit`'s output.

So the  `ControlWord` for the first clock would look like this:
```
000-10001-011-00-00-00-00-01-S00
``` 

And in the next clock the control unit:

- Sets `ReadMem` signal of `Memory` to `1` to read data of the given address.
- Waits until `MemDataReady` is `1` so that it could be read onto the `Databus`.

So the  `ControlWord` for the second clock would look like this:
```
000-10001-011-00-10-00-00-01-S00
```
And in the third clock it:

- Sets `RFLwrite` and `RFHwrite` to `1` to write data of the `Databus` to the `Rd` register.

And the last control word is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Store Addressed** `sta` (0011-D-S)
We need one (effective) clock to execute operation. In this clock the address of `(Rd)` must be put on the `AddressLogic` wire and indicate to the `address` signal of our `Memory` component and the data of `Rs` register should be put into the `Memory` with the indicated `address`.

So for the first clock the control unit:

- Sets `Rd_on_AddressUnitRSide` to `1` to put the data of `Rd` register on the `RSide` wire of `AddressLogic`.
- Sets `EnablePC` to `0` to prevent contents of the `PC` to get replaced by the data of `Rd` register.
- Sets the `R0plus0` signal of `AddressLogic` to `1` to set the data of the `RSide` wire as `AddressUnit`'s output.
- Sets `B15to0` flag of `ALU` to `1` to put `Rs` register's data on `ALU` output.
- Sets `ALUout_on_Databus` to `1` to  put `ALU`'s output on the `Databus`.
- Sets `WriteMem` flag of `Memory` to `1` to write the `Databus`'s data to `memory`.

So the  `ControlWord` for the first clock would look like this:
```
000-10001-011-00-01-00-01-10-S00
```
Then we wait until the data is written onto `Memory` (`MemDataReady` maybe).

#### Input From Port `inp` (0100-D-S)
TODO

#### Output From Port `oup` (0101-D-S)
TODO

#### **And Registers** `and` (0110-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AandB` flag of the `ALU` to `1` to `And` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00000-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Or Registers** `orr` (0111-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AorB` flag of the `ALU` to `1` to `Or` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00001-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Not Register** `not` (1000-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `NotB` flag of the `ALU` to `1` to `Not` the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00010-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Shift Left** `shl` (1001-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `ShlB` flag of the `ALU` to `1` to `Shift Left` the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01000-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Shift Right** `shr` (1010-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `ShrB` flag of the `ALU` to `1` to `Shift Right` the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00111-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Add Registers** `add` (1011-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AaddB` flag of the `ALU` to `1` to `Add` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00011-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Subtract Registers** `sub` (1100-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AsubB` flag of the `ALU` to `1` to `Subtract` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00100-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Multiply Registers** `mul` (1101-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AmulB` flag of the `ALU` to `1` to `Multiply` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-00101-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Compare Registers** `cmp` (1110-D-S)
We need one clock to execute this operation:
So the control unit:

- Sets `AcmpB` flag of the `ALU` to `1` to `Compare` the data of `Rs` and `Rd` registers.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` is:
```
011-00110-111-00-00-00-00-00-S00
```

#### **Move Immediate Low** `mil` (1111-D-00-I)
We need one clock to execute the operation. In this clock, the control unit:

- Sets `RFLWrite` to `1`  to write the low 8-bit of `Databus` into the `Rd` register.

So the `ControlWord` for the first clock is:
```
000-10001-111-10-00-00-00-00-S00
```

#### **Move Immediate High** `mih` (1111-D-01-I)
We need one clock to execute the operation. In this clock, the control unit:

- Sets `RFHWrite` to `1`  to write the high 8-bit of `Databus` into the `Rd` register.

So the `ControlWord` for the first clock is:
```
000-10001-111-01-00-00-00-00-S00
```

#### **Save PC** `spc` (1111-D-10-I)
We need one clock to execute this operation.
In this clock, the control:

- Sets `EnablePC` to `0` to prevent `PC` register's data from getting corrupted.
- Sets `PCplusI` to `1` to add `PC` value to immediate value.
- Sets `Address_on_Databus` to `1` to write the result on the `Databus`.
- Sets `RFLwrite` and `RFHwrite` to `1` to write the data on the `Rd` register.

So the `ControlWord` is:
```
000-10001-000-11-00-00-10-00-S00
```

#### **Jump Addressed** `jpa` (1111-D-11-I)
We need one clock to execute this, the control unit:

- Sets `Rd_on_AddressUnitRSide` to `1` to put the data of the `Rd` register on the `RSide` wire of `AddressUnit`.
- Sets `EnablePC` to `1` to load new data to the `PC` register.
- Sets `R0plusI` to `1` to add `RSide` value to the immediate value.

So the `ControlWord` is:
```
000-10001-010-00-00-00-00-10-S10
```
 
#### **Jump Relative** `jpr` (0000-01-11-I)
We need one clock to execute this, the control unit:

- Sets `EnablePC` to `1` to load new data to the `PC` register.
- Sets `PCplusI` to `1` to add `PC` value to the immediate value.

So the `ControlWord` is:
```
000-10001-000-00-00-00-00-00-S10
```
 
#### **Branch If Zero** `brz` (0000-10-00-I)
For this operation, the control unit checks the `Zout` flag of `ALU Flags` and if it was set, it performs a `Jump Relative` operation.

#### **Branch If Carry** `brc` (0000-10-01-I)
For this operation, the control unit checks the `Cout` flag of `ALU Flags` and if it was set, it performs a `Jump Relative` operation.

#### **Add Window Pointer** `awp` (0000-10-10-I)
The controller should set the `WPadd` flag of `WP` to `1` to increment value of `WP`. So it provides the `ControlWord` value below to perform this operation in one clock:
```
000-10001-111-00-00-01-00-00-S00
```

#### **XOR Registers** `xor` (0000-10-11-0-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AxorB` flag of the `ALU` to `1` to `Xor` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01001-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **2s Complement** `twc`(0000-10-11-1-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `TwcB` flag of the `ALU` to `1` to `Calculate 2s Complement` of the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-10010-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Generate Random** `rnd`(0000-11-00-0-XXX-D-XX)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `Rnd` flag of the `ALU` to `1` to `Generate Random Number`.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01010-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Calculate Square Root** `sqr`(0000-11-00-1-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `SqrB` flag of the `ALU` to `1` to `Calculate Square Root` of the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01011-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Divide Registers** `div`(0000-11-01-0-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `AdivB` flag of the `ALU` to `1` to `Divide` the data of `Rs` and `Rd` registers.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01100-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

####  **Calculate Sinus** `sin`(0000-11-01-1-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `SinB` flag of the `ALU` to `1` to `Calculate Sinus` of the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01101-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Calculate Co-Sinus** `cos`(0000-11-10-0-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `CosB` flag of the `ALU` to `1` to `Calculate Co-Sinus` of the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01110-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Calculate Tangent** `tan`(0000-11-10-1-XXX-D-S)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `TanB` flag of the `ALU` to `1` to `Calculate Tangent` of the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-01111-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

#### **Calculate Co-Tangent** `cot`(0000-11-11-0-XXX-D-S	)
We need two clocks to execute this operation:
In the first clock, the control unit:

- Sets `CotB` flag of the `ALU` to `1` to `Calculate Co-Tangent` of the data of `Rs` register.
- Sets `ALUout_on_Databus` to `1` to put the result of `ALU` operation on the `Databus`.
- Sets `SRLoad` of `ALU Flags` to `1` to enable showing flags from `ALU`.

So the `ControlWord` for the first clock is:
```
011-10000-111-00-00-00-01-00-S00
```
And then the control unit:

- Sets `RFLwrite` and `RFHwrite` to `1` to enable writing to the `RegisterFile`.

And the `ControlWord` for the second clock is:
```
000-10001-111-11-00-00-00-00-S00
```

