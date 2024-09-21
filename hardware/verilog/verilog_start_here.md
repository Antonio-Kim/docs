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

### Defintions

- module hierarchy: modules instantiated within other modules
- Scalar variables: single-bit variable
- Vectors variables: multi-bit variable
- bit select: only one bit of vector is selected. For example, A[0] selects the first bit from right
