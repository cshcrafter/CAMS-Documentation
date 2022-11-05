### Background

The CAMS architecture provides users with a robust framework to effectively execute complex actions.

Program information is conveyed to the hardware primarily using sequences of named items†, the reason for doing so being twofold. For one thing, items are the fastest to interpret††. The principles of contraption hypertransfer work greatly to our advantage, allowing for 10 items per second of read throughput and O(1) decoding time. Secondly, named items are the optimal choice for user readability. While the items could in theory be named using any desired scheme, I tended to follow the general conventions of Assembly.

While the overall architecture remains the same across all applications, there can be some ambiguity in the instruction namespace. These applications are not mutually exclusive, however some names are subject to be changed in cases of combining applications (Ex: Universal 3D printing requiring both Storage system parts and kinetic hardware). 

Hopefully this channel can provide a decent overview of how CAMS functions under the hood. Enjoy!

**Important note**: When writing CAMS code I use spaces to denote separate items (Ex: `START Crafters ;` is expressed as three items each named `START`, `Crafters`, and `;` respectively).



† In the case of an eventual self-replicating machine items may have to be unnamed as the process is currently not automatable. This would lead to the use of a more traditional opcode model in which discrete amounts of unnamed items would be decoded into program information to be executed. Alternatively, items may be ditched entirely in favour of create redstone for easier code replication. Time will tell...
†† In the future it may be less efficient to keep items in cases involving recursion, as jump instructions would take O(N) time to execute instead of O(1) in the case of more traditional program memory structures. For now, items remain the fastest and easiest to work with.

### General Syntax

In general, instructions follow the form:
`<Operation> <Arguments> ;`
where
- `<Operation>` specifies what is going to happen and contains the necessary global timing / control for said action
- `Arguments` are amounts, values, and addresses passed to the system to aid in operation
- `;` signals the end of a line and triggers execution

Flags can be used to temporarily pause execution in cases where a certain condition needs to be met before proceeding. Follows the form:
`Flag<condition>`
where said item sets a latch connected to the input funnel of the program reader until the condition is reached, resetting the latch and allowing the program to continue - _this item can be inserted at the end of any instruction to pause after said line

Jumps can be used whenever one wants to skip forward to a label in the code†. Takes the form:
`JUMP <Label> ;`
where
- `JUMP` regulates the global functionality of the instruction being carried out
- `<Label>` is an item that acts as a marker that the system will search for - _this item can be inserted at the beginning of any instriction one wants to jump to_
- `;` End of line

Similarly, subroutines can be initialized and called which act as distinct subtasks to the 'main' process. This can be used to repeat segments of code, implement loops, etc. Calling a subroutine follows the syntax:
`CALL <Subroutine Label> ;`
where
- `CALL` regulates the global functionality of the instruction being carried out. Executes similarly to a jump. **Note: See [[CAMS Program Syntax#Storage Systems / Processing Automation |Storage Systems / Processing Automation]] for how execution differs from a normal jump when using that functionality**
- `<Subroutine Label>` acts similarly to a jump label, acting as a marker that indicates the beginning of the subroutine being called.
- `;` End of line
Returning from a given subroutine is done as such:
`RETURN <Return Label> ;`
where
- `RETURN` regulates the global functionality of the instruction being carried out. Executes similarly to a jump. **Note: See [[CAMS Program Syntax#Storage Systems / Processing Automation |Storage Systems / Processing Automation]] for how execution differs from a normal jump when using that functionality**
- `<Return Label` acts similarly to a jump label, acting as a marker that indicates to where the system will return - _Can be a pointer, see below_
- `;` End of line
Importantly, arguments††† can be passed into subroutines by setting pointers accordingly before calling the routine, making them reusable. Takes the form:
`SETPOINTER <Pointer Value> <Instruction Item> ;`
where
- `SETPOINTER` regulates execution of the instruction, suppressing `<Instruction Item>` from being executed and instead using it as an argument
- `<Pointer Value>` is a globally-accessible item that is functionally equivalent to calling the designated `<Instruction Item>` - _has a fixed number of possible values_††††
- `<Instruction Item>` is the code item being pointed to

Signal strength values can be stored directly to memory to be used elswhere in the code. Follows the form:
`SETVALUE <Memory Location> <Value> ;`
where
- `SETVALUE` globally controls the instruction's execution
- `<Memory Location>` indicates to where the system will write the value
- `<Value>` is a signal strength value (one hexadecimal digit), either originating from an attached inventory or elsewhere in the memory
- `;` End of line

Finally, conditional operations will executee one of two instructions based on the result of the condition. Analagous to an if-else statement. Follows the form:
`CHECKCOND <value> <conditional operator> <value> <inst. to run if true> , <inst. to run if false> ;`
where
- `CHECKCOND` manages the global functionality of the instruction
- `<value>` in either case can be any accessible signal strength value, either a direct comparator output or a stored hex value in memory
- `<conditional operator>` indicates the comparison being executed - _can either assume `>` or `=`
- `<inst. to run if true>` and `<inst. to run if false>` can be any other instruction type (except another nested `CHECKCOND`) - _does not require a `;` at the end of each statement, but rather a single `;` at the end of the entire line_
- `,` acts as a separator between the two instructions being run††.
Note: If you don't need to execute a statement if the condition evaluates to false, one can simply write `CHECKCOND <value> <conditional operator> <inst. to run if true> ;`
- `;` End of line



† This is currently achieved by reading forward (and overflowing) until the desired label is detected. While this technique is less efficient, the benefits of an item-based architecture make this worth doing.
†† `,` is implemented by having two separate physical 'slots' where instruction data can be written to, with the item acting as a multiplexer between them. The evaluation of the condition will reset whatever is written in either the 'true' slot or the 'false' slot depending on the result.
††† Any I/O items, Timer values, and Label values can be used, but Instruction items cannot.
†††† Pointer values will often be labelled as `Arg0`-`ArgN`, but a special label will be designated to subroutine return labels, which will take the form `ReturnLabel`. The function of either pointer type is entirely the same, only differing in name to make the return label distinct.

### Storage Systems / Processing Automation

CAMS Storage/Processing systems use a contraption hypertransfer bus structure to allow for universal movement of items / fluids.

The first and arguably most important instruction is the `MOVE` instruction, which allows a specified amount of items or fluid to be moved from one place to another. Follows the form:
`MOVE <Amount> <Inputs> <Outputs> ;`
where
- `MOVE` regulates the global timing and control of the movement process.
- `<Amount>` informs the duration the hyperclock will be enabled to the provided output (in the case of items) or pump duration (in the case of fluids)
- `<Inputs>` inform the system of which inventories to assemble to the bus – _can be a list_
- `<Outputs>` inform the system of what location to which the hyperclock should be enabled – _can be a list_
- `;` End of line

Items and Fluids can also be temporarily stored for use in a series of 'registers'. While the hardware is much the same as a typical storage module, they have no filter (nor any dynamic filtering hardware) and thus can accept any item or fluid type respectively. Importantly, these register 'values' are pushed to a stack whenever a subroutine is called and popped when returning.

### Kinetic Hardware

This is a placeholder!
### Code Snippets / Example Programs

This is a placeholder!
