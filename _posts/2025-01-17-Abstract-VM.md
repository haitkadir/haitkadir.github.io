---
title: Abstract VM
date: 2025-01-17 10:00:00 +0000
categories: [Projects, AbstractVM]
tags: [C++, Virtual Machine, Assembly, Stack]
---

## Abstract VM: A Stack-Based Virtual Machine in C++

Abstract VM is a C++ project designed to create a simple stack-based virtual machine capable of interpreting and executing arithmetic programs written in a minimalistic assembly-like language. This article explores the assembly language syntax, interface design, and implementation details.

---

### Features of Abstract VM

- **Stack-Based Architecture**: Operates with a LIFO (Last-In-First-Out) stack for arithmetic and program control operations.
- **Multiple Operand Types**: Supports `int8`, `int16`, `int32`, `float`, and `double` data types for calculations.
- **Custom Assembly Language**: Runs user-defined programs with simple, human-readable instructions.
- **Error Handling**: Manages errors like invalid instructions, stack underflow, division by zero, and missing program termination.
- **Dynamic Operand Factory**: Utilizes a factory pattern to create operands of different types dynamically.

---

### Assembly Language Overview

Abstract VM executes programs written in its own assembly language. Each program consists of sequential instructions, with one instruction per line. Comments start with `;` and continue to the end of the line.

#### Example Program

```shell
; A simple program to add two numbers  
push int32(42)  
push int32(33)  
add  
dump  
exit  
```

#### Supported Instructions

- `push v`: Pushes a value `v` onto the stack. Example: `push int32(42)`.
- `pop`: Removes the topmost value from the stack.
- `dump`: Displays all stack values from top to bottom without altering the stack.
- `assert v`: Verifies that the topmost value matches `v`. Halts execution on failure.
- `add`, `sub`, `mul`, `div`, `mod`: Perform arithmetic on the top two values. Example: `add` adds the top two values.
- `print`: Converts the topmost `int8` value to an ASCII character and displays it.
- `exit`: Terminates the program.

#### Error Handling

Abstract VM identifies and handles the following errors:

- Lexical or syntactic errors in the program.
- Invalid instructions or unsupported operations.
- Stack underflow (e.g., `pop` or arithmetic on an empty stack).
- Division or modulo by zero.
- Overflow or underflow during calculations.
- Missing `exit` instruction in the program.

---

### Implementation Details

#### Stack Design

Abstract VM uses a stack to store operands, represented as pointers to objects of the `IOperand` interface. This design ensures flexibility and strict type safety during operations.

#### Operand Types

The machine supports the following types:

- **Int8**, **Int16**, **Int32**: Represent signed integers of different bit-widths.
- **Float**, **Double**: Represent single-precision and double-precision floating-point numbers.

#### Operand Precision

When performing operations between operands of different types, the result is cast to the more precise type. The precision hierarchy is:  
`Int8` < `Int16` < `Int32` < `Float` < `Double`.

#### Operand Factory

Abstract VM uses a factory pattern to dynamically create operands. The factory uses private methods for each type, selected based on an enumeration value.

```cpp
IOperand const * createOperand(eOperandType type, std::string const & value) const;
```

Depending on the type, it calls one of these methods:

```cpp
- createInt8(std::string const & value)  
- createInt16(std::string const & value)  
- createInt32(std::string const & value)  
- createFloat(std::string const & value)  
- createDouble(std::string const & value)  
```

#### IOperand Interface

The `IOperand` interface defines the behavior of all operand types. It ensures that all operands support arithmetic operations and type inspection.

```cpp
class IOperand {  
public:  
    virtual int getPrecision(void) const = 0; // Return the precision (type) of the instance  
    virtual eOperandType getType(void) const = 0; // Return the type of the instance  

    virtual IOperand const * operator+(IOperand const & rhs) const = 0; // Addition  
    virtual IOperand const * operator-(IOperand const & rhs) const = 0; // Subtraction  
    virtual IOperand const * operator*(IOperand const & rhs) const = 0; // Multiplication  
    virtual IOperand const * operator/(IOperand const & rhs) const = 0; // Division  
    virtual IOperand const * operator%(IOperand const & rhs) const = 0; // Modulus  

    virtual std::string const & toString(void) const = 0; // String representation of the operand  

    virtual ~IOperand(void) {} // Virtual destructor  
};  
```

---

### Grammar of the Assembly Language

The syntax of Abstract VM’s assembly language is formalized as follows:

- **Instruction Format**: INSTRUCTION VALUE

- **Value Types**:
  - int8(n)
  - int16(n)
  - int32(n)
  - float(z)
  - double(z)
- **Example Program**:

```shell
    push int32(42)  
    push float(3.14)  
    mul  
    dump  
    exit  
```

---

### Execution Examples

- **Running from Standard Input**:

```shell
    $ ./avm  
    push int32(42)  
    push int32(33)  
    add  
    dump  
    exit  
    ;;  

    75  
```

- **Running from a File**:

```shell
    $ cat program.avm  
    push int32(42)  
    push float(3.14)  
    add  
    dump  
    exit  
```

```shell
    $ ./avm program.avm  
    45.14  
```

---

### Check out the project here:

- [AbstracVM](https://github.com/haitkadir/AbstractVM)

---

### Conclusion

Abstract VM is a compact yet powerful project that introduces key concepts in virtual machine design, stack-based computation, and assembly language interpretation. It provides an excellent learning opportunity for mastering C++ features like interfaces, dynamic object creation, and error handling.

Feel free to share your experience with this project or suggest improvements!
