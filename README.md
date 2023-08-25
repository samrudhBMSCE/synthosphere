# Synthosphere

Synthosphere is an RTL design and logic synthesis competition where the participants can design their own RTL code, test it by running it on iverilog and gtkwave,followed by synthesizing the code using the "Yosys Open Synthesis Suite" on ubuntu OS. The pre synthesis (gtkwave waverform) and the post synthesis (GLS/gate level netlist) are then compared to check for correctness of the logic synthesis. Logic Optimization is also done to make the synthesis more efficient.

# Quick Links

* Files
  1. [RTL Design Files](https://github.com/samrudhBMSCE/synthosphere/tree/main/Design%20Files)
  2. [Test Bench File](https://github.com/samrudhBMSCE/synthosphere/tree/main/Test%20Bench)
  3. [Netlists](https://github.com/samrudhBMSCE/synthosphere/tree/main/netlists)
  4. [Gate Level Simulation](https://github.com/samrudhBMSCE/synthosphere/blob/main/README.md#gate-level-simulation-gls)
  5. [Optimization](https://github.com/samrudhBMSCE/synthosphere/blob/main/README.md#optimization-of-the-rtl)

* Navigation through the report
  1. [Introduction to basic SIMD processor](https://github.com/samrudhBMSCE/synthosphere/blob/main/README.md#basic-simd-single-instructionmultiple-data-processor)
  2. [RTL Files of all modules](https://github.com/samrudhBMSCE/synthosphere/blob/main/README.md#rtl-of-all-modules)
  3. [Simulation using iverilog and gtkwave](https://github.com/samrudhBMSCE/synthosphere/blob/main/README.md#simulation-using-iverilog-and-gtkwave)

# Basic SIMD (Single Instruction,Multiple Data) Processor

The RTL code describes a basic SIMD processor with a pipeline architecture that processes instructions in multiple stages,each represented by a specific state. It performs arithmetic,logic and memory operations based on instructions with the help of the core - which is a 16-bit SIMD ALU which performs 2's complement calculations. The pipelined architecture allows for efficient execution of multiple instructions in parellel stages, enhancing the processor's throughput.
The ALU operation will take two clocks. The first clock cycle will be used to load values into the registers. The second will be for performing the operations. 6-bit opcodes are used to select the functions. The instruction code including the opcode will be 18-bits.
* Architecture
  
  ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/21502382-0515-4826-bdd3-dc856c01ce9f)

  The ALU is embedded into a simple processor based on 5-stage, delay of each stage will be 1 cycle, meeting the delay of ALU, as shown in the figure. The 5 typical stages are IF, ID, EX, MEM and WB, without     
  pipeline. In the stage IF, a 10-bit address will be sent to an instruction Block-RAM (BRAM) to fetch 18-bit instruction. In the stage ID, the instruction will be decoded and some of control registers will be set 
  to control the following stage. In the stage EX, ALU will process data in registers or implement some control commands, e.g. jump. In the stage MEM, if the instruction is “store” or “load”, data would be read 
  from/ written to data BRAM, based on instruction and address. Finally, in the stage WB, data will be written back to register. The pins of clock, reset, address, data and BRAM enable will be exposed on the 
  interface of processor.

* Design of SIMD ALU

  The ALU consists of three SIMD basic computation units: SIMD adder, SIMD multiplier and SIMD shifter. These three components can be reused for the computation of data with different width (4-bit, 8-bit or 16- 
  bit) 
  and can handle all the instructions (instruction set is listed below for reference) in the design specification. Each of them is controlled by three input port, H (for 16-bit), O (for 8-bit) and Q (for
  4-bit), indicating the kind of input data, so that the unit can handle the data properly.

  1. SIMD adder:
     The SIMD adder is implemented based on 4 4-bit adders. To reuse the adder for data with different width, the input signals, H, O and Q, control the forwarding of the carries between the
     adders. Moreover, the adder can also support subtraction, by flipping the second operand and add one to it when the signal sub is set. The data path of the SIMD adder is shown below.
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/473334e0-4c91-4c31-9d48-85c7d4d13b79)

  3. SIMD Shifter:
     The SIMD shifter can also support data with different width. It is based on two 16-bit shifters and necessary overstep-correct logic. To illustrate the implementation of the shifter, the
     example based on logic-left-shift is shown in below figure. The shifter will determine whether the MSB of 4-bit block should be set to 0 or just inherit the value from the LSB from 4-bit block in
     front of it, according to the input signal, H and O.
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/f6006532-4ab5-44d8-90dd-22dda758da55)

  4. SIMD Multiplier:
     The SIMD multiplier is implemented with adders and shifters. To make it support multiplication with different data width, the input signal, H, O and Q, are used to control the input operands of the adders 
     and select corresponding outputs for the target data width. The 1x16x16 multiplication is implemented as a typical multiplier, as the below figure, where adder tree is utilized. The least significant 16 
     bits of the sum is the output of multiplication.
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/8b31b8d3-3ec4-4924-9abe-eda0a46f74aa)
     
     To reuse the structure of 1x16x16 multiplication implemented as above, for the input signal,H, O and Q, are used to select the inputs of the adders, to control the adders to only sum up the range of bits. 
     The 4x4x4 multiplication is implemented as shown in the figure below:
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/96616f1a-9717-45b7-a0a4-062a001c71f9)
     
     where for each adder,just a part of input bit is valid while other bits will be set to 0.
     Similarly, the 2x8x8 multiplication is also done as illustrated below:
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/7af41e4c-ce56-453b-9909-5e2d7b0b3add)

* Register File

  There are registers which are used to store the data for computation.
  16-bit registers: H1, H2, H3, H4
  8-bit registers (3 sets): {O1,O2}, {O3, O4}, {O5, O6}
  4-bit registers (3 sets): {Q1, Q2, Q3, Q4}, {Q5, Q6, Q7, Q8}, {Q9, Q10, Q11, Q12}
  Loop counter (for loop control): LC

* Instruction Set

  ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/c4bb7a42-78c8-4566-9f0d-7fc07b89bfb3)
  ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/46fb08c9-290e-48a7-896a-b9e51093eac6)

# RTL of all modules

  1. [CPUtop.v](https://github.com/samrudhBMSCE/synthosphere/blob/main/Design%20Files/CPUtop.v) : Top Module
  2. [SIMDadd.v](https://github.com/samrudhBMSCE/synthosphere/blob/main/Design%20Files/SIMDadd.v) : SIMD Adder Module
  3. [SIMDmultiply.v](https://github.com/samrudhBMSCE/synthosphere/blob/main/Design%20Files/SIMDmultiply.v) : SIMD Multiply Module
  4. [SIMDshifter.v](https://github.com/samrudhBMSCE/synthosphere/blob/main/Design%20Files/SIMDshifter.v) SIMD Shifter Module
  5. [processor_tb.v](https://github.com/samrudhBMSCE/synthosphere/blob/main/Test%20Bench/processor_tb.v) : Test Bench for top module

# Simulation using iverilog and gtkwave

  * Codelines on ubuntu terminal:
1. Running the .v files on iverilog
   ```
    $ iverilog SIMD.v tb.v
    $ ./a.out
    ```
    Sample Output:

    ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/0e6c525d-ff3d-421f-840c-958e5012e7ee)

2. RTL simulation using gtkwave
   ```
   $ gtkwave SIMD.vcd
   ```
   Waveform:
   
   ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/8b628b7c-f07a-4004-9f82-a1ae0531f492)

# Synthesis using Yosys

  1. Booting Yosys: ``` yosys ```
  2. Reading the library file: ``` >read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib ```
  3. Flatten command ```>flatten``` for ease of synthesis
  4. Synthesizing the modules one by one:
     
     a. SIMDadd.v - ```>show SIMDadd```

     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/31b7883d-5baf-4042-bddb-839215bbb4d3)

     b. SIMDmultiply.v - ```>show SIMDmultiply```

     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/45bb8886-7970-444d-817f-f5181eba0a92)

     c. SIMDshifter.v - ```>show SIMDshifter```

     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/9bb346b5-aa38-45db-8293-a700e3a70a94)

     d. Finally, the top module CPUtop.v - ```>show CPUtop```

     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/6b06a5a9-da92-4fed-9612-a63f3e2ac1f9)

     Note: Since the processing power is limited in the Virtual Machine, the synthesis for the top module was done before declaration of CPUtop.v as the top module and 
     synthesizing it.

  5. Running the synthesis with CPUtop.v as the top module and printing the statistics - ```>synth -top CPUtop``` 
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/995d14ac-83e8-4a4f-a3fb-c2444fdc7936)

  6. Mapping the flipflops to our design - ```>dfflibmap -liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```

     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/fc0c76b0-1c0f-47be-8f57-aeb9592b50ac)

  7. Combining the logic to generate abc results - ```>abc -liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib```
     
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/7706025d-92f7-49ac-b2e8-4263ba17916d)
     ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/7a23861e-d591-4b75-88cc-11c0e458a643)

  8. Dumping the netlist into a file - ```write_verilog -noattr netlist_CPUtop.v```


# Gate Level Simulation (GLS)

  ```$ iverilog netlist_CPUtop.v ../verilog_model/primitives.v ../verilog_model/sky130_fd_sc_hd_edited.v tb.v```
  ```$ ./a.out```
  ```gtkwave CPUtop.vcd```

  ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/41b1c0b0-d035-4892-9717-7469c3a2d418)

  ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/8b628b7c-f07a-4004-9f82-a1ae0531f492)

  The GLS output window is in par with the RTL simulation output as observed above.

# Optimization of the RTL

   1. Booting Yosys: ``` yosys ```
   2. Reading the library file: ``` >read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib ```
   3. Reading the RTL file: ```>read_verilog SIMD.v ```
   4. Optimization: ``` >opt -purge CPUtop```
      
      ![image](https://github.com/samrudhBMSCE/synthosphere/assets/143097746/90dd56cc-8415-4f3b-bf45-d8d5e374f1f5)


  




   

     


     
     
 

     

     



     



  





