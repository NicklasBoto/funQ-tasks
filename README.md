# funQ programming tasks

Make sure that `funQ` is installed on your system. For example:

```
$ funq --version
0.1.0.0
```

The tasks are described in seperate markdown documents. Go through the tasks one by one and write down your solution as a `funQ` program in the solutions folder. After you are done, run the `test` script and submit this repository as a compressed folder. Below is a basic description of the language, its type system, and some basic quantum computing. Documentation on the CLI-tool can be displayed by executing 

```
$ funq --help
```

and the commands of the interactive environment are listed by the command `:?`.

## Basic syntax

funQ uses Haskell-style syntax, and should feel familiar to any Haskell programmer.

### Top level definitions
```hs
-- constants
b : Bit
b = 0

-- functions
id : Bit -o Bit
id x = x
```

### Anonymous functions / lambdas
funQ uses church-style lambdas, which means that the variables introduced by lambdas must be explicitly typed.
```hs
\x : Bit . x 
```

### Literals
The only literals that currently exist are the bits `0` and `1`, as well as the unit value `*`.

```hs
mybit : Bit
mybit = 1

unit : T
unit = *
```
### Products / tuples
Tuples are created by parenthesis. N-tuples with be treated like right nested 2-tuples.
```hs
(0,1,*) == (0,(1,*))
```

### Let expressions
Tuples can be pattern matched by `let` expressions.

```hs
tup : Bit >< Bit
tup = (0,1)

one : Bit
one = let (a,b) = tup in b
```

### Ternary opertor / if-then-else
Basic logic can be performed by the `if` expression. It accepts a bit as predicate and its cases must be of the same type (technically they must only have a common supertype).

```hs
and : Bit -o Bit -o Bit
and x y = if x then y else 0
```

## Type system
`funQ` uses an affine type system. This means that variables in the language can be used at most once (that is, 0 or 1 times). We will refer to this property as *linearity*, a variable that upholds linearity is called *linear*. Up to this point all variables in the code examples have been linear (convince yourself of this). This property is integral to the language as it ensures that values of type `QBit` cannot be duplicated (in accordance with the no-cloning property). It is possible to construct non-linear values in the language as well, but only for expressions that do not duplicate qubits. Non-linear values (or *duplicable* values) are denoted by the type level prefix operator `!`, pronounced **bang**. A linear bit would therefore be typed as `Bit`, while a duplicable bit would be typed as `!Bit`. Jump into the `funQ` interactive environment and look at the types of some expressions (for example `0`, `*`, `new`, `measure`, and `H`).

For example, the following code would break the linearity constraint and will produce a type error.

```hs
\x : Bit . (x,x)
```

Think about why this code fails, and try to fix it (this is the same as task 1).

### Subtyping
`funQ` features subtyping of duplicity types. Intuitively this means that a duplicable type can be used where a linear type is required (since if it can be used many times, it can be used 0 or 1 times). For example look at the input type of `new` and the type of `0`, then check the type of their application. However, if a function requires a duplicable type a linear type cannot be passed.

### Note on special duplicable types
Look up the type of the function `measure`.

```hs
measure : !(QBit -o !Bit)
```

Why does the actual function type need to be duplicable? Well, it doesn't! As with all other types, a linear function type would mean that the function could only be called once. This would of course be very cumbersome, and since `measure` does not duplicate any quantum data its type can be duplicable (but not its input type).

A special property of product types (the type of tuples) is that the bangs of the type arguments are factored out. For example, if you create a tuple of two duplicable bits you would get `!(QBit >< QBit)` and not `!QBit >< !QBit`.


### Qubits
Qubits are represented internally by their *state vectors*. State vectors represent the probabilities of a qubit to be measured to a certain value. For example, `new 0` creates the vector `[1,0]`, meaning a qubit with a 100 % probability of collapsing to 0 (`new 1` would be `[0,1]`). 

### Gates
Gates are always typed in all caps. `funQ` ships with many gates but the only ones needed for these tasks are the following. Please consult [this webpage](https://en.wikipedia.org/wiki/Quantum_logic_gate), or experiment in the interactive environment, if you are unsure what a gate does.

#### Hadamard
![hadamard](https://raw.githubusercontent.com/NicklasBoto/funQ/main/docsImages/h.PNG)
```hs
H : !(QBit -o QBit)
```

The hadamard gate puts a qubit in perfect superposition

#### Pauli-X
![paulix](https://raw.githubusercontent.com/NicklasBoto/funQ/main/docsImages/x.PNG)
```hs
X : !(QBit -o QBit)
```

The Pauli-X gate acts like a classical NOT. Flipping the probabilities of a state vector.

#### Pauli-Z
![pauliz](https://raw.githubusercontent.com/NicklasBoto/funQ/main/docsImages/z.PNG)
```hs
Z : !(QBit -o QBit)
```

The Pauli-Z gate negates the second element in the state vector, mapping `|0>` to `|0>`, and `|1>` to `-|1>`.

#### CNOT
![cnot](https://raw.githubusercontent.com/NicklasBoto/funQ/main/docsImages/cnot.PNG)
```hs
CNOT : !((QBit >< QBit) -o QBit >< QBit)
```

The CNOT (controlled not) gate is a 2-qubit gate. It therefore accepts a 2-tuples as its input argument. This gate is a Pauli-X gate acting on the target qubit (second one in the tuple), controlled by the control qubit (first one).

#### Circuits

Quantum circuits are a series of lines representing qubits, and gates acting on them.

![epr](https://upload.wikimedia.org/wikipedia/commons/b/bb/H_CNOTGate.png)

This circuit for example creates two qubits in the zero state, performs hadamard on the first one then performs a controlled not with the first qubit as control and the second as target.
