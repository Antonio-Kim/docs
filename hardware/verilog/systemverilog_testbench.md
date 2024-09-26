## SystemVerilog TestBench

### Introduction

The module that we want to test is called Design-Under-Test. In testbenches, it consists DUT, testbench programs at adds values to input and outputs. The testbench also includes clock module for synchroncity, and assertion module to check whether the program has correctly outputted the values that are expected. Lastly, testbench includes functional coverage to check how many assertion has passed or not, and alert the user if there are too many failures. In this section, we'll focus mostly on testbench program, and other section will include both assertion and functional coverages.

### Testbench for FSM

When testing FSM of DUT, two combinational functions must be tested: does the machine transit to correct state correctly under all input combination, and for each state, does the machine exhibit the correct outputs under all input combinations.

#### Clock and Reset

Every testbench requires clock and reset, and this has to be declared outside testbench's program. Here's a simple clock that alternative forever.

```verilog
logic clock;
initial begin
	clock = 0;
	forever #5 clock = ~clock;
end
```

This initializes clock as 0, then after 5 units, clock turns into 0 and this goes forever, as stated on "forever" syntax. Instead of being forever, we can also specific timing end

```verilog
bit clock;
initial begin
	clock = 0;
	repeat (1000) #5 clock = ~clock;
	$finish;
end
```

The repeat statement is a loop that repeats the statement following it the given number of times. In this case, clock is inverted 1000 times caussing 500 state changes. We can so add reset to the clock to initialize module.

```verilog
bit clk, rst;
initial begin
	clk = 0;
	rst = 0;
	#1 rst = 1;
	#4 forever #5 clk = ~clk;
end
```

Clock and reset are both initialized to 0 at time 0. After delay of 1, reset is set to 1, and at time 5, the forever loop begins, which means that at time 1 positive edge of clock would occur.

### Using $moniter for debugging FSM

$monitor is more reliable approach to debugging FSM than $display because $monitor output all the changed variable at the end of the specified time. When writing test cases, we may not know all the internal variable of DUTs, and we would most likely want to test the state of the DUTs in our test benches. To do so, we can access the states using the dot operator as we have done on other modules. Here's a simple code that demonstrates this

```verilog
module top;
	...
	design dut(.*);
	TBench tb(.*);

	initial $monitor($time, " Current state = %s", dut.state.name);
endmodule: top

program Tbench (port declarations)
...
endmodule: Tbench

module design(port declaration);
	enum logic[4:0]{ RESET, A, B, C,...} state;
	...
endmodule: design
```

The most import part is how the top module is accessing dut's states, which are RESET,A,B,C,... The method .name is used to get the name of the state at the current. We may want to move the $monitor inside the Testbench instead of having it in the top module. To do so, we do the following inside the TBench:

```verilog
initial $monitor($time, " Current state= %s", top.dut.state.name);
```

Notice how we use top instead dut this time. When resolving hierarchical name, the parser looks for the first part of the name in the current module, and if it cannot find it, it looks up the module hierarchy to find it. Hiearchical name allows access to any variables in the project. However, be careful this; we want to use only for debugging, not for accessing ports.

### Classes

Somewhat similar to object-oriented programming languages, SystemVerilog also have a class. Here's an example

```verilog
class testVectors;
	rand bit [4:0] fiveBits;
	randc logic [8:0] nineBits;
endclass
```

Some explanation is needed with variables here. The "rand" indicate that the variable fiveBits will have random values between 0-31, and the 31 is due to maximum 5 bits inside the array. The nineBit will have a range between 0-512. Another subtle difference between rand and randc is with replacement and without replacement. That is, rand could have repeating values between 0-31, whereas randc will not have repeating values between 0-512. We can instantiate our class with `new` syntax. Note that the example above is constructor without any parameters, so the `new` syntax can be used without any values

```verilog
bit [4:0] outputA;
logic [8:0] outputB;
bit condition, clk;
textVectors tv = new;
```

If a class had empty contructor but would want some sort of values assigned or function displayed, it would have empty contructor with them inside like this:

```verilog
class testVectors;
	rand bit [4:0] fiveBits;
	randc logic [8:0] nineBits;

	function new()
		$display("Hello, world!");
	endfunction
endclass
```

and then intializing the class will require empty contructor to work:

```verilog
bit [4:0] outputA;
logic [8:0] outputB;
bit condition, clk;
textVectors tv = new();
```
