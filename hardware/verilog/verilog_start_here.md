## Verilog

### Setup

Setup your project folder with the following folders:

- hdl: your VHDL or verilog codes
- simulation: ModelSim projects
- synthesis: Quartus projects to FPGA
- testbench

This section specifically deals with Setting up Verilog code on Altera DE2 board. The Software used is Quartus II 64-Bit Version 13.0.1 Build 232 0/6/12/2013 SJ Web Edition. As of September 2024, the latest version available for this specific version is here: https://www.intel.com/content/www/us/en/software-kit/666221/intel-quartus-ii-web-edition-design-software-version-13-1-for-windows.html

- Device: Cyclone II
- Chip: EP2C35F672C6N
- Pin Assignment:
  - SW[0]: PIN_N26
  - SW[1]: PIN_N25
  - LEDR[0]: PIN_AE23

We are going to use SW[0] and SW[1] of the board to turn on LEDR[0] whenever one of the switch is flipped on, but not both. Here's the verilog code to program:

```verilog
module light(x1,x2, f);
	input x1, x2;
	output f;

	assign f=(x1&~x2)|(~x1&x2);
endmodule
```

### ModelSim Simulation

Make sure you setup the project with the folder structure mentioned in the beginning. Then, open up ModelSim click Simulate > Runtime Options > Default > Default radix:binary to display the output in binary. Once that's set, no need to revisit this again unless you decides to change to hexadecimal.

Go to File > New > Project, and enter in the name. Click on browse and set the directory to simulation. Set the library name, but default of "work" is okay. Click on "Reference Library Mapping" and then OK.

A pop-up window should appear. Click "Add items to the project" and "Add Existing File". Browse to /hdl folder and click all verilog files and click "Open". Set the project to "ALWAYS click Reference from current location", and OK.

Compile the codes by Compile > Compile All, then once everything is okay, Simulate > Start Simulation. A new window will appear and find the name of the library you named when creating a new project. Default was work, so expand the work library. Clikc on the testbench file you included and click OK

By default you should see sim tab on the first left section of the project, but if not open up the sim tab located at the bottom of the section. You'll notice that in the Object section (right of the sim) all the internal and external signals displayed. You can select all of them by dragging the name of the testbench from the sim tab to the wave, or select and drag the specific waves you want to analyze to the waveform window.

Go to simulate > run > run-all, and on the waveform window press "f" to zoom full.

#### Error handling on ModelSim

- If you do not see the waveforms of signals you expect:
  - Simulate > Restart
  - Check All
  - Ok
  - Simualte > Run > Run-All
- If you make changes to hdl file in a separate editor, sometimes the system will give you a popup window saying "Warning! File modified outside of source editor". If so, click on "reload"
  - Click on the project tab on the bottom of the left window
  - You should see "?" staus for testbench file
  - Compile > Compile Out of Date
  - Compile > Compile All
  - See if the status is green
  - Simulate > Run > Run-All

### SystemVerilog Type definition

In SystemVerilog the basic types can be separated into two types: two state values, and four state values.

- Four State Value: four state values are - 0 and 1, x for unknown, and z for high impedance
  When Simulation kernel starts, the four state types usually start with x as unknown, then gets assigned to whichever value that programmer assigns afterwards. Problem with four state types is that it requires two more state, x and z than the counterpart two state, and in a large system, designers usually opt for two states intead because of this overhead.

### Port Specification

Usually when module is declared, you would set the order of the parameters in the higher module the same as the lower module's order. For example, say we have a simple mux:

```verilog
module mux
  (input logic a, b, sel,
  output logic f);

  assign f = (sel) ? a : b;
endmodule: mux
```

we know from above that the inputs has to be on the first three parameter, and the output has to be placed on the last parameter. So when this is called in the higher order module

```verilog
module datapath;
  logic sel, in0, in1, result;

  mux a(in0,in1,sel,result);
endmodule:datapath
```

inputs has to match the order of the lower module. There are other ways to do this, and few of them are: setting **named port**, using **all name match** and **some name match**.

#### Named port

Instead of ordering them, we can assign the parameters to match the name of the lower module using "." and the lower module name, and parentheses for parameter used in higher module, and not worryh about the orders.

```verilog
module byNamePort;
  logic s,in1,in0,out;

  mux b(.a(in0), .sel(s), .b(in1), .f(out))
```

#### All name match

If all the names in higher order matches the lower module's parameter, we can use the '.' with the wildcard "\*".

```verilog
module allNamesMatch
  logic a, b, sel, f;

  mux c(.*);
```

#### Some name match

If most of them matches the lower module's port but some of them don't, wee can use named port for those, and use all name match for the rest.

```verilog
module someNamesMatch
  logic in0, in1, sel, f;

  mux d(.a(in0), .b(in1), .*);
```

### Enumeration and Type Definitions

Enumeration provides controleld method for introducing named constants. They are very similar to programming language's enumeration type. For example

```verilog
enum {CHEVY,FORD,TOYOTA} car1, car2;
```

says car1 and car2 can only have those three values. We can also assign values to the enum type so that it would be easier and less error prone when looking up the values. For example,

```verilog
enum logic[2:0] {
  ADD = 3'b100,
  SUB = 3'b010,
  AND = 3'b001,
  OR = 3'b110,
  XOR = 3'b001
} op;
```

here the enum name is op, and will only consist of those five operations. More common approach to enum is using typedef to create a new type definition. Following up the enum example above, we can create a new type and then use it in our module. Here's a long, but detailed version of how typedef enum would be used

```verilog
typedef enum logic[2:0]{
  ADD = 3'b100,
  SUB = 3'b010,
  AND = 3'b001,
  OR = 3'b110,
  XOR = 3'b110
} aluInst_t;

module alu
  (input logic [7:0] a, b,
  output logic [7:0] result,
  input aluInst_t op);

  always_comb
    unique case(op)
      ADD: result = a + b;
      SUB: result = a - b;
      AND: result = a & b;
      OR: result = a | b;
      XOR: result = a ^ b;
    endcase
endmodule: alu
```

Enums have methods that allow you to help other task such as printing name of the enum type, but also create loops. You can use `<enum name>.<method>` to access these methods. Here are some of the examples available for enums:

- name: provides string that can be printed as part of $monitor or $display
- first: returns first value of the first enumeration label
- next: returns the value of the next enumeration label in the list
- last: returns the value of the last enumeration label in the list

### Defintions

- module hierarchy: modules instantiated within other modules
- Scalar variables: single-bit variable
- Vectors variables: multi-bit variable
- bit select: only one bit of vector is selected. For example, A[0] selects the first bit from right

```

```
