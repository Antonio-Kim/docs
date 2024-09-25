## Hardware Thread (FSM-D)

Let's extend FSM little further. While there are plenty of cases where FSM is used by itself, more often or not FSM is attached to another module called Data path. **Data Path** is a a module where data flows from registers or memories, to another register/memories after being modified by some combinational logic. This module is controlled by FSM and together they are called **FSM-D**, for Finite State Machine - Data Path. The action of data flowing from register/memories, getting modified by combinational unit, then stored into another registers/memories is called **Register Transfer**. FSM-D is sometimes also called Hardware Thread. The difference between hardware thread and software thread is that hardware thread is always continuous, whereas software thread occurs when data is available.

Hardware thread has few specification required in order to function. They are:

- always synchronized with rest of the computation system to receive input and return result
- Specifies both control logic and data path logic to carry out computation
- has interface that allows outside module to interact with the thread.

### Data Path Components

As mentioned before, Data path consists of various components for computation, mainly combinational components and registers. Here we look at high overview of the components using SystemVerilog. The code should be changed to meet the problem required.

#### ALU

Here's a simple ALU that does the following: ADD, SUB, AND, OR, XOR. In SystemVerilog this should be fairly simple to implement, the only change is that we use typedef enum for operational code

```verilog
typedef enum logic[2:0] {
	ADD=3'b100, SUB=3'b010, AND=3'b001, OR=3'b110, XOR=3'b011
} aluInst_t;

module alu
	(input logic[7:0] a,b,
	output logic[7:0] result,
	input aluInst_t op);

	always_comb
		unique case(op)
			ADD: result = a + b;
			SUB: result = a - b;
			AND: result = a & b;
			OR: result = a | b;
			XOR: result = a ^ b;
		endcase
	end
endmodule: alu
```

#### Registers

Typically registers include clock, reset, but also load indicater as part of input function, including incoming data. Registers also sometimes include single operand such as increment (PC Counter for example), or shift register. Here are examples of both registers, shown in same code:

```verilog
module register #(parameter w = 1)
	(input logic clk, rst, load, incr,
	input logic [w-1:0] dInput,
	output logic [w-1:0] sum);

	always_ff@(posedge clk, negedge rst)
		begin
		if(~rst) sum <= 0;
		else if(load) sum <= dInput;
		else if(incr) sum <= sum + 1;
	end
endmodule: register

module shiftRegister
	(input clk, load, shift,
	input[7:0] d,
	output lowBit);
	logic q;
	assign lowBit = q[0];

	always_ff@(posedge clk)
		if (load) q <= d;
		else if (shift)
			q <= q >> 1;
endmodule: shiftRegister
```

#### Decoder

Decoder has n-bit input that is used to select 2^n options for output. Decoders are often found in datapath as it serves to enable one of several functions to enable. Here's an example of 3-8 decoder in SystemVerilog

```verilog
module decoder
	(input logic [2:0] sel,
	output logic [7:0] oneHot)

	always_comb
		case(sel)
			0: oneHot = 8'b0000_0001;
			1: oneHot = 8'b0000_0010;
			2: oneHot = 8'b0000_0100;
			3: oneHot = 8'b0000_1000;
			4: oneHot = 8'b0001_0000;
			5: oneHot = 8'b0010_0000;
			6: oneHot = 8'b0100_0000;
			7: oneHot = 8'b1000_0000;
		endcase
endmodule: decoder
```

### Memories

NB: Memory is widely used in datapath so pay attention to this section

In SystemVerilog, memories are declared using 2-D array. Note that declaration is slightly different than other programming language

```verilog
bit [7:0] m[255]
logic [7:0] n[255:0]
```

First, notice that the first bracket is the width of the memory. Thus, it would be 8 bit-wide data. Next backet indicate the length of the memory which in this case 256 bit. Second, there are two declaration for memory type here: bit and logic. Remember that in SystemVerilog, they are typically two primitive types: two state and four state. Bit indicate two states - 0 and 1 - and logic indicate four states - 0,1,x, and z. Whichever type you decide to use, remember that bit declarationg of the height does not require colon, whereas logic type requires the range. The example below shows a simple memory array:

```verilog
module memory
	#(parameter Dwidth = 8,
	Words = 256,
	AWidth = $clog2(Words))
	(input bit [Dwidth-1:0] dataIn,
	input bit [AWidth-1:0] address,
	input bit clk, wEnable,
	output bit [Dwidth-1:0] dataOut);

	bit [Dwidth-1:0] m[Words];

	assign dataOut = m[address];
	always_ff@(posedge clk)
		if (wEnable) m[Words] <= dataIn;
endmodule: memory
```

Take note that we used $clog2 in Parameter declaration. $clog2 stands for "ceiling of base 2 log", and this declaration gives AWidth a correct width even for word counts that are not exact power of 2. We can also make our memory little bit more efficient by having one of the port both in and out ports, and this will introduce new type and syntax in SystemVerilog.

```verilog
module busMemory
	#(Parameter DW = 8,
	W = 256,
	AW = $clog2(W))
	(input bit [AW-1:0] addr,
	input logic re, we, clk,
	inout tri [DW-1:0] data);

	logic [DW-1:0] m[W];

	assign data = (re) ? m[addr] : 'bz;

	always @(posedge clk)
		if (we) being
			m[addr] <= data;
		end
endmodule: busMemory
```

First, notice we used inout to declare the port parameter both in and out. Second, since we are using in-out port, it must be defined to be one of the SystemVerilog net types. In this case, we used tri which is either 0,1, or z net type. SystemVerilog has two types of net: wire and tri.

### Interfacing

Let's take it little bit further and expand our abstraction out. We talked about how Hardware threads have Data Path and FSM for controller. Now let's have multiple Hardware threads sharing data between the two. This is common in small to larger scale system. CPU and hard drive could be two of many hardware threads that are talking to each other. Sensors and Process is another example of two threads interfacing together. Thus, you would need standard protocols and interfances so that each threads allows other threads to interact and share data.

Starting with basic concept, usually there's one producers and one or more consumers. However, this does not mean they are producers/consumers forever. There may be, and most likely, threads that are producing and also consuming some sort of data within extended thread trees. Say we start with two threads: thread P and thread C. Thread P produces while Thread C consumes data. How would they communicate? One example would be that Thread C requests data from Thread P, and thread P sends both data and dataReady as a signal that production is completed. Another way is that Thread P sends a message to thread C that data is ready, and thread C sends ok signal ready to receive, then Thread P sends the data. The signal that intialization communication between the two thread is called "signals" or "signal variables". The information send through this signal is called protocol. It's not necessarily 0 or 1 like a status flag, but sequence of action that occurs over time.

One of the most important factor when communicating between two threads is clock synchronicity. Each threads may have same clock frequency and thus requires some sort of agreement between the two threads.

1. A Synchronous signal: both threads share the same clock frequency
2. An Asynchronous signal: two threads do not share the same clock frequency
3. A synchrnoized signal: one thread's clock frequency is passed on through synchronizer flip-flop and sync with other thread's clock frequency.

The only way for a hardware thread to respond reliably to an asynchronous input signal is by synchronizing it, producing a synchronized signal. At that point, a synchronized signal the same as a synchronous signal; the synchronized signal is now part of the clock domain.

not only the timing has to be synchronized, but each thread also need to share similar state, called state-synchroncity. This means that both threads must have a wait state to start communication between the two. The wait state is one state that loops to itself when an input signal is not asserted.

#### Asynchronous Thread Interaction

Here we'll specifically discuss asynchronous thread interaction since it is the most common interaction between threads. A consumer thread can send a request signal for data and the producer can send both data-ready signal and data itself. This process is asynchronous since you do not know when consumer will send request signal, nor required to have same state to make the transfer occur. Here we'll discuss one of the main method of asynchronous thread interaction: **Fully Interlocked Handshake protocol**.

Similar to TCP, there's a communication between two threads where producer and consumers sends signal indicating data is ready, and requesting data, respectively. However, the data is not shared directly; there is a buffer in-between the threads that either reads the data or writes the data depending on who's turn it is to read and write. Consumer sends requests signal, then while the consumer waits, producers goes through multiple states to produce the data, and once it's done it sends the data to the buffer and sends signal to consumer that data is ready. The data transfers goes back and forth between consumer and producer until the interaction ends

There's also a variation to this protocol where there are two buffers instead of one. While producer is placing the data into one buffer, the consumer is reading data that was previously placed by producer on the other buffer, and this goes back and forth. Another variation is instead of buffer, there's a queue that acts as FIFO (first-in, first-out) where producers addes data to the end of the queue while consumer reads the data and pops out the data. Queue is often used in data that are placed or read in bursty manner. We can create a simple queue by implementing the following code with methods:

- empty: indicate there are no items in the queue
- full: indicate that there are no more sipace in the queue
- put: a control poin that tells the queue there is a value on its input to load into the queue
- get: contorl point that tells the queue to remove a value and make it available on the output.

#### Interface

Similar to other object-oriented languages, SystemVerilog allows interface syntax to declare variables and set input and output depending on how declaration is used. Note that you do have to instantiate the interface on the top-level of the design, and like static objects, the ports are shared among these modules that implement interfaces.

Let's use a quick example and description will continue.

```verilog
interface ReqAck
  (input clk);
  logic req, ack;
endinterface: ReqAck

module modA(ReqAck c);
  logic avail;

  always_ff@(posedge c.clk)
    c.ack <= c.req & avail;
  ...
endmodule: modA

module modB(ReqAck b);
  always_ff@(posedge b.clk);
    b.req <= 1;
  ...
endmodule: modB

module top;
  logic clk;
  ReqAck RA(clk);

  modA t1(RA);
  modB t2(RA);
endmodule
```

Interface module is declared at the top, where input clk is common for all module that will share the ReqAck module. The logic req and ack acts like a static variable that will also be share among all modules that uses ReqAck interface. ModA implements that ReqAck interface and calls it c, and each variables of the ReqAck is accessed using the dot operator. At the top module, ReqAck is instantiated and named RA, and it takes in clock as the input. The instantiated interface is then added to the module's input parameter for both modA and modB, and they are also instantiated named t1 and t2 respectively. Note that variables are shared among t1 and t2 so any modification of one variable modifies other variable. Now you might be wondering which one of the parameter is an input and which one is output, and that declaration can be modified using modport (short of modifying port) type, and the variables be either input or output depending on how it's declared and which modport the modules use.

```verilog
interface ReqAck
  (input clk);
  logic req, ack;
  modport G(output req,
  input ack, clk);
  modport H(input req, clk,
  output ack);
endinterface

module modA(ReqAck.H c);
  logic avail;

  always_ff@(posedge c.clk)
    c.ack <= c.req & avail;
  ...
endmodule

module modB(ReqAck.G b);
  always_ff@(posedge b.clk);
    b.req <= 1;
  ...
endmodule

module top;
  logic clk;
  ReqAck RA(clk);

  modA t1(RA.H);
  modB t2(RA.G);
endmodule
```

Modport G sets req as output and ack as input, whereas the previous example did not have any declaration. Together these mean that access to the variables listed in modport G are limited to the direciton specified there. modB at the top level is only allowed to use ack and clk as input and req as an output.

There are two takeaway from the example above: you can view interface as bundle of wires and once the modules uses these interface, they share the wires together. Second, modports can be used to set direction of the wires for specific modules. Note that it's not only the port declaration that are set in the interface. You can also add like always_comb, always_ff, tasks, and also functions
