# Homework 1

Listen to the [Zero Knowledge](https://zeroknowledge.fm/) podcast featuring [Celestia](https://zeroknowledge.fm/268-2/). This will serve as an introduction and complement the topics of:

- Modular Blockchains
- Data Availability

# Homework 2

Discuss in your teams the following topics:

1. Why is client diversity important for Ethereum?
2. How would you describe the following concepts and how do you rank them in terms of their importance to you?
   - TPS and transaction cost
   - Finality
   - Privacy
   - Decentralisation
   - Security


# Homework 3

## ZkEVM Set Up Environment

In preparation for the development tasks we will be doing, we need to setup a development environment.

1. Install the Rust toolchain. See [Instructions](https://rustup.rs/)

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
2. Install a browser wallet if you don't already have one.

## Zustlings for zkEVM
We have some Rust exercises for you to work through. These are in the bootcamp [repo](https://github.com/ExtropyIO/zkEVMBootcamp). You can either clone this locally or set up a codespace.

### Installing Zustlings
1. Make sure you have rust installed
2. In the rust directory run `cargo install --force --path .`


### Doing exercises

- In the rust directory run `zustlings homework n` where `n` is the number of the homework you are doing, e.g `zustlings homework 3`.

- You will need to open another terminal so that you can edit the files. When you start, it will start with the code in `homework3/variables/variable.rs`. As you can see it does not compile. You need to make the change to the code in this file so that it will compile. When you save the file, it will recompile and show you if there are still errors.

- Once you have finished a particular file remove the comment `// I AM NOT DONE` and you will be taken to the next file that you need to fix.


# Homework 4

Some Modular Arithmetic

1. Working with the following set of Integers S = {0,1,2,3,4,5,6}
    - What is a) 4 + 4
    - b) 3 x 5
    - c) what is the inverse of 3 under
    - i) addition
    - ii) multiplication

2. For S = {0,1,2,3,4,5,6}
    - Can we consider 'S' and the operation '+' to be a group?

3. What is -13 mod 5?

4. Polynomials

    For the polynomial 23 - 22 + 4x - 12
    - Find a positive root
    - What is the degree of this polynomial?