## SystemVerilog Simulation

SystemVerilog has simulation kernel that keeps the time and update if there are any changes. The internal time that the kernel keeps is called **virtual time**, and the kernel has table of events in a list and fetches those events whenever virtual time is set to that specific time. The events in the list are called **updated events**.

When the simulation starts, it sets the virtual time to 0 and all the logic variables to "x", indicating that it's unknown that that time. It then continuously retrieves update for the CURRENT time, which then updates the simulation's model states (i.e. state of gates is the value of its output) and fansout its output to other model state and see if there are any changes. Once the model gates are updated, it then executes these model states and see if there are any changes to the module's output. The kernel then moves to the next event list and continues the same pattern until there are no more events left.

The _gate execution model_ occurs when primitve gates' input change occurs, and the gate model executes its function possibly producing a new output value. If there is a new output value, it is scheduled in the event list to appear, possibly with some time delay. When the simulation reaches the future time, simulation state is updated and the new value is propagated to gates on its output and those gates are evaluated. This continues until all outputs have settled to their final values.