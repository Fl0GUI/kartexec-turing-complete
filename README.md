# `kartexec.cfg` is Turing complete

This repository is collection of `kartexec.cfg` files that implement Turing machines.

`kartexec.cfg` is a configuration file for [SRB2Kart](https://srb2.org/mods/). It is a kart racing mod based on the 3D Sonic the Hedgehog fangame [Sonic Robo Blast 2](https://srb2.org/), based on a modified version of [Doom Legacy](http://doomlegacy.sourceforge.net/).

![demo](img/demo.gif?raw=true)

The Turing machines are implemented with extensive use of the `alias` command.

`alias <name> <command>: create a shurtcut command that executes other commands` 

What makes this command powerful is that it is possible to nest an alias. However, it is not possible to nest a string in \<command\>, but a workaround (with the use of alias of course) exists. 

There is no support for variables or values whatsoever. But by redefining an alias we change what actions will be taken later on.

## The tape

### Shifting left and right

```
// tape
alias tape_0 "tape_0_shifts"
alias tape_1 "tape_1_shifts"
alias tape_2 "tape_2_shifts"

// shifting
alias ERR "echo ERROR"
alias tape_0_shifts "alias shift_r tape_1; alias shift_l ERR"
alias tape_1_shifts "alias shift_r tape_2; alias shift_l tape_0"
alias tape_2_shifts "alias shift_r ERR   ; alias shift_l tape_1"

// -----------

tape_0 // shift_r is now tape_1
shift_r // shift_r is now tape_2
shift_r // shift_r is now ERR
```

What position of the tape we are on is only determined by the definition of the commands that operate in that position, eg. `shift_l` and `shift_r`. This will later be extended with functionality for reading and writing on the tape. By calling `shift_r` and `shift_l` we redefine those functions.



### Reading and writing the tape

```
// tape 0
alias tape_0 "tape_0_writes ; tape_0_shifts"
alias tape_0_writes "alias write_0 tape_0_write_0 ; alias write_1 tape_0_write_1"

alias  tape_0_write_0 "alias tape_0 _tape_0_write_0"
alias  tape_0_write_1 "alias tape_0 _tape_0_write_1"
alias _tape_0_write_0 "alias read read_0 ; tape_0_shifts ; tape_0_writes"
alias _tape_0_write_1 "alias read read_1 ; tape_0_shifts ; tape_0_writes"

// -----------

tape_0 // there is no read defined yet
write_0 // tape_0 is now "alias read read_0 ; tape_0_shifts ; tape_0_writes"
read // will run read_0
write_1
read // will run read_1
```

Instead of redefining the shift commands, when writing we redefine `tape_0 ` which will redefine `read` for future use. `read_0` and `read_1` will be defined by the state.

This block of aliases will be copied for each `tape_n`

## The state

```
alias stateA  "alias read_0 stateA_read_0  ; alias read_1 stateA_read_1"
alias accept "echo ACCEPT"

alias stateA_read_0 "alias state stateA ; alias write write_1 ; alias shift shift_r"
alias stateA_read_1 "alias state accept ; alias write write_0 ; alias shift shift_l"

// -----------

stateA // current state is stateA

// first step
read_0 // read a 0 of the tape
write // writes a 1 on the current position
state // next state is stateA
shift // shift the tape to the right

// second step
read_1 // read a 1 of the stack
write // writes a 0 on the current position
state // next state is accept
shift // shift the tape to the left
```

The definition of a state is the definition of a  `read` command for each symbol on the tape, a `read` command in it's turn is the definition of what to write, the next state and the direction of a shift. Each `<state>_read_<symbol>` is a transition.

## Running the Turing machine

```
alias initial_tape tape_0
alias inital_state stateA

alias run "read ; write ; state ; shift"

// initialize the tape content
tape_0 ; write_1
tape_1 ; write_0

// initialize the tape position and initial state
initial_tape ; initial_state
run // runs through one step
```

Before running the Turing machine we first need to load our input into the tape. This is easily done with eg. `tape_0 ; write_1`. `tape_0` defines `write_1` and `write_1` will define `read` for later use.

The `run` command is the concatenation of all operations needed for a single step of the Turing machine. By initializing the tape and state we've defined `read`.

In a run step `read` is executed (which is either `read_0` or `read_1`).  `read_0` or `read_1` will define what symbol to write, what the next state is and in what direction to shift.

 `write` will redefine what symbol `read` on the current position will be.

`state` will redefine what action to take upon reading the next symbol.

`shift` will redefine `read` to be either `read_0` or `read_1`.



#### Do you want to write your own Turing machine in a config file? Here's what to do:

First you have to download and install srb2kart from the links given in the beginning of the readme.

Create a `kartexec.cfg` file into the root directory of the game. This is where you write your Turing machine. You can of course copy an existing Turing implementation into this file. After starting the game, hit back-quote to open the console where you can write your commands, ie. `run` or `autorun`.

I suggest starting from an existing config file, change the size of the tape to what you need by copying the definitions and changing the numbers. Then you can define all states and transitions for your Turing machine.

### Credits

Author: Fl_GUI#5136
inspiration: Froil#6092

Froil came up with the original idea of nesting and redefining aliases. I jokingly said that the `kartexec.cfg` file is Turing complete and the ball started rolling.
