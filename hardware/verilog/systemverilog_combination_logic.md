## Combination Logic in SystemVerilog

Two main approaches to describing combination logic:

1. **always_comb**: used like a continuous looping statement
2. **assign**: similar to always_comb but often used for one-line logic expression

So when do you use **always_comb** and when do you use **assign**? assign is used for simple one-line statement, and always_comb is used for more complex behaviour. You can have multiple line of assign statements via comma, but they are not executed procedurally like always_comb. Here's an example:

```verilog
module compare
	(output logic eq,neq,
	input logic[3:0] value);

	assign neq = ~eq, eq = (value == 0);
endmodule:compare
```

- When using if-else statement, you need to make sure that the output always have some output rather conditional, when input changes. Otherwise, it becomes stateful and is not valid in combinational circuits. For example

```verilog
module notCombinational
	(input logic[3:0] a, b,
	output logic[3:0] sum,
	input logic hold);

	always_comb
		if(~hold)
			sum=a+b;
endmodule:notCombinational
```

the output sum changes only when hold is not true. If it had else statement and the sum had some sort of output with input changes, then it would be combinational.

**NB**: one popular style to ensure this is that when writing always_comb with conditionals, you first write the output with either 0 or 1, then write the procedural statements to set it to the other value.

Switch statement in SystemVerilog is similar to other programming languages

```verilog
module basicCase
	(input logic a, b, c,
	output logic f);

	always_comb
		case({a,b,c})
			3'b000: f=0;
			3'b001: f=0;
			3'b010: f=0;
			3'b100: f=0;
			default: f= 1;
		endcase
endmodule:basicCase
```

### Don't care situations

Most of the time you do not want to implement don't care situations, and have the code handle these situation separately. But there are cases where this is necessary. For example, there may be cases where when we use switch statements, we don't add all the cases. We would normally implement default cases as described above, but there are cases where there's no default. If so, we have couple of options: unique cases and casez. Unique cases work similar to regular switch statements, but we add "unique" syntax

```verilog
module aluUnique
	(input logic [7:0] a, b,
	output logic [7:0] result,
	input logic [2:0] op);

	always_comb
		unique case(op)
			3'b100: result = a + b;
			3'b010: result = a - b;
			3'b001: result = a & b;
			3'b110: result = a | b;
			3'b011: result = a ^ b;
		endcase
endmodule: aluUnique
```

Another option is using casez syntax, which allows "?" as don't care. For example, there might be a whole row or column in K-map that will have one value, or possibly a coverable block in the map will have the samle value. A "?" is used in this situation as a wildcard specification, meaing that either 1 or 0 can be substituded for each "?" and t he same statement will be executed.

```verilog
module simpleKmap
	(input logic a,b,c,
	output logic f);

	always_comb
		casez({a,b,c})
			3'b1??: f=0;
			3'b01?: f=1;
			3'b000: f=1;
			3'b?01: f=0;
		endcase
endmodule:simpleKmap
```

There also might be a case where you might have don't care values, but you would want one of the case to be priortized before. Then we use "priority casez" to handle this situation. For example say we have the following code:

```verilog
module pcase
	(output bit out,
	input bit a,b);

	always_comb
		priority casez({a,b})
			2'b1?:out=1'b0;
			2'b?0:out=1'b0;
			2'b?1:out=1'b1;
		endcase
endmodule:pcase
```

You'll notice that there might be a conflict when input of a and b is 2'b11 on normal case. But with priority casez, you'll have 1'b0 as the output instead of 1'b1. If the case were switched, then the output would be 1'b1 instead of 1'b0.

### Writing Testbenches for Combinational Logic

- **$monitor(...)**: prints snapshot of values at the end of simulation time
- **$display(...)**: provide information about a model as it is executing
- **$strobe(...)**: similar to display, but it prints at the end of current time.

These statements are usually placed in the beginning of the always block. The format that's used is usually `$display(<print_string>, <list of variables>);`.
