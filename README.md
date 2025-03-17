# ClarityFuzzer (experimental)

## Motivation
Inspired by the research done by the [EF/CF team](https://github.com/uni-due-syssec/efcf-framework), we introduce ClarityFuzzer: am extremely fast, malleable, Fuzzer for Clarity contracts.

## How it works
Clarity contracts are first "transpiled" opcode by opcode into C++. The resulting program is functionally equivalent to the Clarity contracts, but has the speed and tooling availability advantages of a tried and tested compiled language like C++.\
We insert the transpiled contracts into a template that provides blockchain context (you may think of this as a Virtual Machine created dynamically, that runs only a predefined set of programs defined by the user) and a fuzzing harness setup to run [AFL++](https://github.com/AFLplusplus/AFLplusplus/tree/stable). This immediately gives us two very desirable things when fuzzing:
- Accurate coverage feedback, and
- Complete control over the context in which the contracts run (available contracts for cross calls and blockchain state -accounts, assets, balances, etc.-).

The fuzzing campaign then proceeds by utilizing AFL to produce random input, interpreting this input as valid input for the functions to be run, and then collecting coverage feedback at the end.\
Generally speaking, input that generates new coverage is weighted positively so as to be used as a basis for mutation more often.

Property checks may be added and written natively in Clarity or C++. These are inserted into the fuzzer at the end of a function as test oracles: any input that failed the property check should produce a log with all necessary data to replicate the finding.


## Getting started

The transpiler needs `python3` installed on your computer. Then, you should install `TreeSitter` and the Clarity grammar Python bindings. To do so, run
```bash
pip install tree-sitter
pip install tree-sitter-clarity
```
Once done, the transpiler should be set up.

In order for the fuzzing side to work, you need to build AFL++. Follow the instructions [here](https://github.com/AFLplusplus/AFLplusplus/blob/stable/docs/INSTALL.md) for details on how to do that according to your architecture.


### Running the transpiler
To run the transpiler and prepare the fuzzing template, you may use
```bash
python3 ClarityTranspiler/ClarityTranspiler.py /path-to-your-contracts/ 0
```

### Starting a fuzzing campaign
To start a fuzzing campaign, run the following command to build the fuzzing target:
```bash
afl-clang-fast -o fuzz_target ClarityFuzzer.cpp -lstdc++
```
Then, you may start your fuzzing campaign with
```bash
afl-fuzz -i seeds -o out -- fuzz_target
```

## Future steps
Currently, opcode support is limited, as this is a _PoC_. In the upcoming months we plan on expanding transpiler opcode support to cover the full Clarity language.\
We are also in the process of streamlining the creation of property checks, to make it easier for the community to use for their own projects.\
A contribution guide will be added once the main property check mechanism is in place.