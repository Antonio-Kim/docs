## Verilog

### Setup

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
