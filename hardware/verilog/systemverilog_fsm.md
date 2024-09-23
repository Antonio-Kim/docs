## FSM in SystemVerilog

### D Flip-Flop

Let's start with D Flip-Flop, which is a fundamental building block to sequential circuit, and consequently, for Finite State Machine. D Flip-Flop typically has an clock, input, and reset as the module input, and then an output. We can write out the D Flip-Flop with the following SystemVerilog code:

```verilog
module dFF
	(input logic clk, d, rst,
	output logic q);

	always_ff@(posedge clk, negedge rst)
		if (~rst)
			q <= 0;
		else q <= d;
endmodule: dFF
```

Little explanation is needed here. Similar to combination, we have always block, but using ff to indicate that it's an always block using flip flop. The "@" symbol indicate that the always block gets executed whenever there's a change in the parenthesis variables. Inside the parenthese we hae clock (clk), and reset(rst), but only when they occur at a specific value. The "posedge" occurs when there's positive/rising edge of clk, and "negedge" occurs when there's negative/falling edge of rst. Thus, whenever rst is equal to 0, reset occurs, whether it would be in negative edge since negedge is conditional here, but also whenever clk becomes positive since the block inside will execute the first if statement, and will see that ~rst == 1 and set q to 0. We can assume here that reset is low-impedence and change occurs whenever reset is 0. Whenever clk is running, whether it would be in same consistent pattern or in different form, when there's positive edge, so long as rst == 1, the output will be its input.

### Steps from FSM to SystemVerilog Code

Whether it will be Moore or Mealy machine, FSM requires procedural steps to go from a problem to SystemVerilog. The small difference between Moore and Mealy machine is that in Mealy machine, the output also includes input of the problem.

When writing a SystemVerilog model to the view of FSM, two combinational functions are written (using always_comb or assign statement) and the sequential update is written using always_ff. The combinational function is typically for output of module, and sequential is written for next state.

To write FSM onto SystemVerilog, the goal is to find the equation used to assign each bit of the next state to internal variables and assigning the equation to the internal variables inside the always_ff. For the output we use combinational, whether it be assign or always_comb. Implicity the internal variables are stored inside the flip-flop as state. Thus, we first start with finding the state for any given problem. We always start with Reset state, and then list all other possible states in the problem. We labeled them and figure out given the input, how each state moves, known as transition expression. Note that there are cases where no input is given and will move without any conditions (time for example). We then draw these into state transition diagrams, including both state and transition expression, and here is where we decide whether to use Moore or Mealy, which will affect the circuit of the FSM.

We have to decide how we will assign each state a value, but also output, known as state encoding. Once state encoding is done, we first create state transition table by combining both current state, the input, and next state using state encoding and input values. For example, the table may look like this:

| s1  | s0  | x   | s1' | s0' |
| --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   |
| 0   | 0   | 1   | 0   | 1   |
| 0   | 1   | x   | 1   | 0   |
| 1   | 0   | 0   | 1   | 0   |
| 1   | 0   | 1   | 0   | 0   |

x is input, s1 and s0 are state encoding of current state, and s1' with s0' are state encoding of the next state. The table explains how each current state with input will transition, hence the name state transition table. The x in the x column indicate it doesn't matter whatever the input is. Also, s1 = 0 s0 = 0 is also the reset state as mentioned above. Based on the table above, we can find the state equation for each state bit shown. For simple case like this, we can use boolean equation

```
s0' = x * ~s1 * ~s0;
s1' = ~s1 * s0 + s1 * ~s0 * ~x;
```

Here \* indicate AND, + indicate OR, and ~ indicate NOT. For more complicated tables, using Karnaugh maps can be used to handle and simplify more difficult states. We then move into output equation based on state. Here is where Mealy and Moore differs: the output table will include input if it is Mealy, and without it in Moore. Say we are using Mealy in this example

| s1  | s0  | x   | y   |
| --- | --- | --- | --- |
| 0   | 0   | 0   | 0   |
| 0   | 0   | 1   | 1   |
| 0   | 1   | x   | 0   |
| 1   | 0   | 0   | 0   |
| 1   | 0   | 1   | 1   |

NB: the output can be more than just one bit, and can have multiple bit as well. But for sake of simplicity, we start with just one bit. Notice that there are only two rows of output, which then turns into the following equation:

```
y = ~s1 * ~s0 * x + s1 * ~s0 * x;
```

You can draw circuit with this equations where Flip-Flop in the middle that includes states, the output of the Flip-Flop indicate current state and input of the Flip-Flop indicate next state. The output is based on the combinational assignment as shown above, and the next state circuit is based on combination of state and input based on the equation writte above. Thus, we can really think of FSM as two combinational circuits, but the flip-flop changes how we view the problem and the solution that comes with it.

Again, we can write SystemVerilog code based on the equations given from state transition table, and output encoding table. Here's how it would be written:

```verilog
module FSM
	(input logic x, clk, rst,
	output logic y);
	// internal state/next state
	logic s1, s0;

	// Sequential logic
	always_ff@(posedge clk, negedge rst)
	begin
		if (~rst) {s1, s0} = 2'b00;
		else begin
			s0 <= x & ~s1 & ~s0;
			s1 <= (~s1 * s0) | (s1 & ~s0 & ~x);
		end
	end

	// Combinational logic
	assign y = (~s1 & ~s0 & x) | (s1 & ~s0 & x);
endmodule: FSM
```

There's few things to note here. One, notice that s1 and s0 are assigned "<=" instead of "=" in the combinational block. This is important distinction because "<=" indicate concurrent/non-blocking assignment, whereas "=" is blocking statement. This means that both s0 and s1 will be assigned at the same time, instead of step by step. Second, notice that we have separated our code into two blocks: sequential and combinational. Most of FSM problems will be divided into something like this. Lastly, the internal state/next state is not part of the parameter and thus declared inside the module, not in the input parameters.
