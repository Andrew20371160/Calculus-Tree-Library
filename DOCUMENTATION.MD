# Calculus Tree Library - Full Documentation

## Overview

The Calculus Tree Library is a comprehensive C++ library for building, manipulating, and evaluating mathematical expression trees. It supports symbolic differentiation, numerical integration, vector calculus operations (gradient, curl, divergence, laplacian), and flexible expression evaluation with support for custom variables and constants.

**Language:** C++ (100%)

---

## Table of Contents

1. [Core Components](#core-components)
2. [Class Reference](#class-reference)
3. [Usage Examples](#usage-examples)
4. [Architecture](#architecture)
5. [Supported Functions and Constants](#supported-functions-and-constants)
6. [Features](#features)

---

## Core Components

### 1. **node.h / node.cpp** - Expression Tree Node

The `node` class abstracts linking operations for binary tree nodes used in expression representation.

#### Node Structure

```cpp
class node {
  private:
    std::string symbol;        // Stores: numbers, variable names, or operators
    node *parent;              // Pointer to parent node
    node *left;                // Left child (first operand/function argument)
    node *right;               // Right child (second operand)
};
```

#### Key Methods

**Tree Structure Operations:**

| Method | Purpose |
|--------|---------|
| `append_child(const std::string&)` | Add a child to current node (fills left, then right) |
| `append_parent(const std::string&)` | Create a new parent node and make current node its child |
| `append_next(const std::string&)` | Add a sibling (right child to parent) |
| `disconnect_self()` | Disconnect from parent (preserves children) |
| `exchange_parent(const std::string&)` | Replace parent with new node in tree hierarchy |

**Node Pointer Variants:** All operations above have versions accepting `node*&` parameters for tree reorganization.

#### Node Construction

```cpp
node(const std::string& _symbol);
```

---

### 2. **calculus_tree.h / calculus_tree.cpp** - Main Expression Tree Class

The primary user-facing class for expression tree operations.

#### Template Declaration

```cpp
template<typename DataType>
class calculus_tree { ... };
```

The `DataType` parameter allows flexibility:
- `calculus_tree<long double>` - Standard floating-point arithmetic
- `calculus_tree<complex<long double>>` - Complex number support (enable COMPLEX_MODE)

#### Constructors and Assignment

```cpp
// Default constructor
calculus_tree();

// From expression string
calculus_tree(const std::string &expression);

// Copy constructor
calculus_tree(const calculus_tree&);

// Destructor
~calculus_tree();

// Assignment operators
calculus_tree& operator=(const calculus_tree&);
template<typename input_datatype>
calculus_tree& operator=(const input_datatype&);
```

#### Arithmetic Operators

```cpp
calculus_tree operator+(const calculus_tree&) const;  // Addition
calculus_tree operator-(const calculus_tree&) const;  // Subtraction
calculus_tree operator*(const calculus_tree&) const;  // Multiplication
calculus_tree operator/(const calculus_tree&) const;  // Division
calculus_tree operator^(const calculus_tree&) const;  // Power/Exponentiation
```

#### Expression Management

```cpp
void set_exp(const std::string &expression);         // Set new expression
bool remove_tree();                                    // Clear tree
bool load(const std::string &filePath);              // Load from file
bool save(const std::string &filePath);              // Save to file
std::string expression(node*ptr = NULL) const;       // Get expression as string
void print(node*ptr = NULL) const;                   // Print to console
```

#### Evaluation Functions

```cpp
// Evaluate with variable substitution
// Example: vector<string> vars = {"x", "5", "y", "3.14"};
DataType evaluate_at(std::vector<std::string> vars_equal = {""});
```

**Variable Format:** Variables and values alternate in the vector: `{var1, val1, var2, val2, ...}`

#### Differentiation and Vector Calculus

```cpp
// Single-variable differentiation
calculus_tree diff_with(const std::string &variable);

// Gradient: returns vector of partial derivatives
std::vector<calculus_tree<DataType>> gradient(
    const std::vector<std::string>&ind_vars);

// Curl: 3D vector field → 3D vector field
std::vector<calculus_tree<DataType>> curl(
    const std::vector<calculus_tree<DataType>>&gradient_field,
    const std::vector<std::string>&independent_variables);

// Divergence: Vector field → Scalar
calculus_tree divergence(
    const std::vector<calculus_tree<DataType>>&gradient_field,
    const std::vector<std::string>&independent_variables);

// Laplacian: ∇²f = fxx + fyy + fzz
calculus_tree laplacian(const std::vector<std::string>&ind_vars);
```

#### Numerical Integration

```cpp
// Simpson's 1/3 Rule
// Parameters:
//   variable: integration variable
//   beg, end: integration bounds
//   intervals_count: number of subintervals
//   traversal_array_size: (optional) array size for recursive calls (auto-adjusted if 0)
//   variables_and_values: other variables and their values
DataType simpson_rule_1_3(
    const string&variable,
    const DataType beg,
    const DataType end,
    const unsigned int intervals_count,
    unsigned int traversal_array_size=0,
    vector<string> variables_and_values={""}) const;

// Simpson's 3/8 Rule (for cases where interval is divisible by 3)
DataType simpson_rule_3_8(
    const string&variable,
    const DataType beg,
    const DataType end,
    const unsigned int intervals_count,
    unsigned int traversal_array_size=0,
    vector<string> variables_and_values={""}) const;
```

**Balancing Strategy:** For large intervals (e.g., 1,000,000), the tree is traversed multiple times with smaller array sizes to manage stack usage vs. computation balance.

#### Table Generation

```cpp
// Generate evaluation table across variable ranges
// Parameters:
//   variables_and_values: {var1, initial_val1, var2, initial_val2, ...}
//   fx_size: number of evaluations per variable
//   step_size: increment for each variable
//   traversal_array_size: (optional) recursive call array size
vector<DataType> table(
    vector<string>& variables_and_values,
    const unsigned int &fx_size,
    const vector<DataType> &step_size,
    unsigned int traversal_array_size=0) const;
```

**Preconditions:**
- `variables_and_values.size()` must be even
- `fx_size > 0`
- `step_size.size() == variables_and_values.size() / 2`

#### Variable and Simplification

```cpp
// Get independent variables in expression
std::vector<std::string> independent_variables();

// Variable substitution/exchange
// Example: exchange("x", "2*y") replaces all x with (2*y)
calculus_tree<DataType> exchange(
    const string&old_variable,
    const string&new_var) const;

// Simplify expression
// Evaluates constant functions, eliminates 0s and 1s, etc.
void simplify();

// Random equivalence testing
bool random_equivalence(calculus_tree&, unsigned int trial_count);
```

---

### 3. **preprocessor.h / preprocessor.cpp** - Expression Validation and Preprocessing

Handles tokenization, validation, and automatic insertion of implicit operators.

#### Preprocessing Rules

The preprocessor enforces mathematical expression rules:

1. Opening brackets must match closing brackets
2. Implicit multiplication: `2x`, `xsin(x)`, `x(2)`, `x2` → insert `*`
3. Variable names: `xy` is a single variable, not `x*y`
4. Functions must be followed by `(`: `sin(x)`, not `sinx`
5. Operand/operator distinction
6. Adjacent expressions: `(x+y)(z+5)` → `(x+y)*(z+5)`
7. Sign operators: `+` and `-` can start expressions

#### Token Types

```cpp
enum {
    ERROR=-1,           // Invalid token
    VAR_CONST,          // Variable or constant
    FUNCTION,           // Known function
    OPERATOR,           // +, -, *, /, ^
    OPEN_BRACKET,       // (
    CLOSE_BRACKET       // )
};
```

#### Validation Methods

| Method | Purpose |
|--------|---------|
| `prepare_exp(const std::string&)` | Main preprocessing function |
| `skip_spaces()` | Consume whitespace |
| `preprocess_extract()` | Extract next token |
| `token_type()` | Identify token type |
| `valid_*_token()` | Context-specific validation |

#### Usage

```cpp
preprocessor processor;
std::string cleaned_exp = processor.prepare_exp("2x+sin(x)");
// Result: "2*x+sin(x)"
```

---

### 4. **functions_and_known_constants.h / functions_and_known_constants.cpp** - Function and Constant Definitions

Defines supported mathematical functions and constants, with extensibility for custom additions.

#### Supported Functions (20 in normal mode, 21 with COMPLEX_MODE)

| Category | Functions |
|----------|-----------|
| **Logarithmic** | `log` (any base, e.g., `log2`), `ln`, `exp` |
| **Algebraic** | `sqrt`, `abs` |
| **Trigonometric** | `sin`, `cos`, `tan`, `sec`, `csc`, `cotan` |
| **Inverse Trig** | `asin`, `acos`, `atan` |
| **Hyperbolic** | `sinh`, `cosh`, `tanh`, `asinh`, `acosh`, `atanh` |
| **Complex** | `img` (imaginary unit, COMPLEX_MODE only) |

#### Supported Constants

**Normal Mode:**
- `pi` (π)
- `e` (Euler's number)
- `inf` (infinity)
- `nan` (not a number)

**Complex Mode (additional):**
- `i` (imaginary unit)

#### Usage

```cpp
// Enable complex mode (uncomment in header)
// #define COMPLEX_MODE 1

calculus_tree<complex<long double>> tree("sin(x) + 5*i");
```

#### Helper Functions

```cpp
// Check if token is a known function
int is_known_function(const std::string &expression, unsigned int pos);

// Check if character is operator
bool is_op(const std::string &expression, unsigned int pos);

// Check if string is a valid number
bool is_num(const std::string &var);

// Check if token is a known constant
int is_known_constant(const std::string &var, unsigned int pos);

// Evaluate function at a value
template<typename DataType>
DataType evaluate_function(const int fn, const DataType var, const DataType base);

// Evaluate known constant
template<typename DataType>
DataType evaluate_constant(const std::string&symbol);

// Evaluate binary operators
template<typename DataType>
DataType evaluate_operator(char op, const DataType&left_operand, const DataType&right_operand);
```

#### Extension Guide

To add a custom function or constant:

1. Update `functions_count` or `constants_count` in both COMPLEX_MODE and normal sections
2. Add enum value to `functions_codes` or `constants_codes`
3. Add string representation to `known_functions[]` or `known_constants[]`
4. Add evaluation logic in `functions_and_known_constants.tpp` (template file)
5. Add differentiation logic in `calculus_tree.cpp` in the `diff_function()` method

---

## Class Reference

### calculus_tree<DataType>

#### Member Variables (Private)

```cpp
preprocessor processor;        // Expression preprocessor
node *root;                   // Root of expression tree
```

#### Private Methods (Parsing)

```cpp
node* parse_parenthese(const std::string&, unsigned int&);
node* parse_function(const std::string&, unsigned int&);
node* parse_block(const std::string&, unsigned int&);
node* create_tree(const std::string&, unsigned int&);
int precedence(const std::string&, unsigned int);
void var_op_func(const std::string&, node*&, node*&, node*&);
std::string extract(const std::string&, unsigned int&);
```

These methods implement recursive descent parsing with operator precedence handling.

#### Private Methods (Evaluation)

```cpp
DataType evaluate(node*, const std::vector<std::string>&) const;
std::string eval_extract(const std::string&, unsigned int&);
bool is_function_tree(node*&) const;      // Left child only, no right
bool is_constant_tree(node*) const;       // No children
bool is_op_tree(node*) const;             // Both children present
```

#### Private Methods (Differentiation)

```cpp
std::string diff(node*, const std::string &var);
std::string diff_function(const int fn, node*, const std::string &var);
std::string diff_op(node*, const std::string &var);
std::string diff_plus_minus(node*, const std::string &var);
std::string diff_mult(node*, const std::string &var);
std::string diff_div(node*, const std::string &var);
std::string simplify_add(const std::string&, const std::string&);
std::string simplify_sub(const std::string&, const std::string&);
std::string simplify_mult(const std::string&, const std::string&);
std::string simplify_div(const std::string&, const std::string&);
```

---

## Usage Examples

### Example 1: Basic Expression Creation and Evaluation

```cpp
#include "calculus_tree.h"
using namespace std;

int main() {
    // Create expression
    calculus_tree<long double> tree("x^2 + 3*x + 2");
    
    // Evaluate at x=5
    vector<string> vars = {"x", "5"};
    long double result = tree.evaluate_at(vars);
    cout << "Result: " << result << endl;  // Output: 52
    
    // Print expression
    cout << "Expression: " << tree.expression() << endl;
    
    return 0;
}
```

### Example 2: Symbolic Differentiation

```cpp
int main() {
    calculus_tree<long double> f("sin(x) * x^2");
    
    // Differentiate with respect to x
    calculus_tree<long double> df = f.diff_with("x");
    
    cout << "f(x) = " << f << endl;
    cout << "f'(x) = " << df << endl;
    
    // Evaluate derivative at x=pi
    vector<string> vars = {"x", "3.14159"};
    long double slope = df.evaluate_at(vars);
    cout << "f'(π) ≈ " << slope << endl;
    
    return 0;
}
```

### Example 3: Gradient and Vector Calculus

```cpp
int main() {
    // Function: f(x,y) = x^2 + y^2
    calculus_tree<long double> f("x^2 + y^2");
    
    // Compute gradient ∇f = (∂f/∂x, ∂f/∂y)
    vector<string> vars = {"x", "y"};
    vector<calculus_tree<long double>> grad = f.gradient(vars);
    
    cout << "∂f/∂x = " << grad[0] << endl;  // 2*x
    cout << "∂f/∂y = " << grad[1] << endl;  // 2*y
    
    // Evaluate gradient at (3, 4)
    vector<string> point = {"x", "3", "y", "4"};
    long double dfx = grad[0].evaluate_at(point);
    long double dfy = grad[1].evaluate_at(point);
    cout << "∇f(3,4) = (" << dfx << ", " << dfy << ")" << endl;
    
    return 0;
}
```

### Example 4: Numerical Integration

```cpp
int main() {
    // Integrate x^2 from 0 to 10
    calculus_tree<long double> f("x^2");
    
    // Simpson's 1/3 rule with 1000 intervals
    long double integral = f.simpson_rule_1_3(
        "x",                    // variable of integration
        0.0,                    // start
        10.0,                   // end
        1000,                   // intervals
        0,                      // auto-adjust traversal size
        {""}                    // no other variables
    );
    
    cout << "∫₀¹⁰ x² dx ≈ " << integral << endl;
    // Exact value: 1000/3 ≈ 333.33
    
    return 0;
}
```

### Example 5: Simplification

```cpp
int main() {
    calculus_tree<long double> tree1("0 + x");
    tree1.simplify();
    cout << tree1 << endl;  // Output: x
    
    calculus_tree<long double> tree2("(x+1)*0");
    tree2.simplify();
    cout << tree2 << endl;  // Output: 0
    
    return 0;
}
```

### Example 6: Complex Numbers (if COMPLEX_MODE enabled)

```cpp
#include "calculus_tree.h"
using namespace std;

int main() {
    calculus_tree<complex<long double>> z("5 + 3*i");
    
    cout << "z = " << z << endl;
    
    // Differentiate (result in complex mode)
    auto dz = z.diff_with("i");
    
    return 0;
}
```

### Example 7: Laplacian

```cpp
int main() {
    // Function: f(x,y,z) = x^2 + y^2 + z^2
    calculus_tree<long double> f("x^2 + y^2 + z^2");
    
    // Laplacian: ∇²f = ∂²f/∂x² + ∂²f/∂y² + ∂²f/∂z²
    vector<string> vars = {"x", "y", "z"};
    calculus_tree<long double> laplacian_f = f.laplacian(vars);
    
    cout << "∇²f = " << laplacian_f << endl;  // Output: 6
    
    return 0;
}
```

---

## Architecture

### Expression Tree Structure

Mathematical expressions are represented as binary trees:

```
       +
      / \
     ^   3
    / \
   x   2
```

Represents: `x^2 + 3`

**Node Classification:**
- **Leaf nodes:** Variables or constants (no children)
- **Function nodes:** Parent holds function name, left child is argument, no right child
- **Operator nodes:** Both children present, root is the operator

### Parsing Strategy

1. **Tokenization:** Extract operands, operators, and parentheses
2. **Preprocessing:** Validate and insert implicit operators
3. **Recursive Descent:** Parse with operator precedence rules
   - Higher precedence (^) binds tighter
   - Equal precedence (*, /) associate left-to-right
   - Lower precedence (+, -) handled last

### Differentiation Algorithm

Uses symbolic differentiation rules:

- **Power Rule:** d/dx[x^n] = n*x^(n-1)
- **Product Rule:** d/dx[u*v] = u'*v + u*v'
- **Quotient Rule:** d/dx[u/v] = (u'*v - u*v')/v²
- **Chain Rule:** d/dx[f(g(x))] = f'(g(x))*g'(x)
- **Function Rules:** Each function has its differentiation rule

---

## Supported Functions and Constants

### Complete Function List

| Function | Description | Derivative |
|----------|-------------|-----------|
| `log(x)` | Base-10 logarithm | 1/(x*ln(10)) |
| `logN(x)` | Base-N logarithm (e.g., `log2`) | 1/(x*ln(N)) |
| `ln(x)` | Natural logarithm | 1/x |
| `exp(x)` | e^x | e^x |
| `sqrt(x)` | Square root | 1/(2*sqrt(x)) |
| `abs(x)` | Absolute value | sign(x) |
| `sin(x)` | Sine | cos(x) |
| `cos(x)` | Cosine | -sin(x) |
| `tan(x)` | Tangent | sec²(x) |
| `sec(x)` | Secant | sec(x)*tan(x) |
| `csc(x)` | Cosecant | -csc(x)*cotan(x) |
| `cotan(x)` | Cotangent | -csc²(x) |
| `asin(x)` | Inverse sine | 1/sqrt(1-x²) |
| `acos(x)` | Inverse cosine | -1/sqrt(1-x²) |
| `atan(x)` | Inverse tangent | 1/(1+x²) |
| `sinh(x)` | Hyperbolic sine | cosh(x) |
| `cosh(x)` | Hyperbolic cosine | sinh(x) |
| `tanh(x)` | Hyperbolic tangent | sech²(x) |
| `asinh(x)` | Inverse hyperbolic sine | 1/sqrt(x²+1) |
| `acosh(x)` | Inverse hyperbolic cosine | 1/sqrt(x²-1) |
| `atanh(x)` | Inverse hyperbolic tangent | 1/(1-x²) |
| `img(x)` | Imaginary (COMPLEX_MODE) | i*img'(x) |

### Supported Constants

| Constant | Value | Use |
|----------|-------|-----|
| `pi` | π ≈ 3.14159 | Trigonometric calculations |
| `e` | e ≈ 2.71828 | Exponential/Logarithmic |
| `i` | √(-1) | Complex numbers (COMPLEX_MODE) |
| `inf` | ∞ | Undefined operations |
| `nan` | Not a Number | Invalid results |

---

## Features

### ✅ Implemented Features

1. **Expression Parsing**
   - Recursive descent parser with operator precedence
   - Automatic implicit multiplication
   - Full parenthesis support

2. **Symbolic Differentiation**
   - Single and multivariable
   - Chain rule, product rule, quotient rule
   - All standard functions included

3. **Numerical Evaluation**
   - Variable substitution
   - Support for all functions and constants
   - Templated data type (long double, complex, etc.)

4. **Numerical Integration**
   - Simpson's 1/3 rule
   - Simpson's 3/8 rule
   - Balancing strategy for large intervals

5. **Vector Calculus**
   - Gradient computation
   - Divergence
   - Curl (3D)
   - Laplacian

6. **Expression Manipulation**
   - Tree copying
   - Variable substitution/exchange
   - Simplification
   - Save/load from files

7. **Complex Numbers**
   - Optional complex mode
   - Support for `i` constant and `img()` function

8. **Extensibility**
   - Easy addition of custom functions
   - Easy addition of custom constants
   - Template-based evaluation

---

## Technical Specifications

### Global Constant

```cpp
const long double threshold = 10e-6;
```

Used for floating-point comparisons and edge cases (e.g., 0^0 handling).

### Supported Operators

- **Addition:** `+`
- **Subtraction:** `-`
- **Multiplication:** `*`
- **Division:** `/`
- **Exponentiation:** `^`
- **Parentheses:** `(` `)`

### Precedence (Highest to Lowest)

1. Functions, Parentheses
2. Exponentiation `^`
3. Multiplication `*`, Division `/`
4. Addition `+`, Subtraction `-`

### Error Handling

The preprocessor returns `"0"` (zero expression) on syntax errors and prints error messages to cout. Users should validate expressions before use.

---

## Compiler Requirements

- **C++ Standard:** C++11 or later
- **Headers Required:**
  - `<iostream>`, `<string>`, `<vector>`, `<queue>`, `<set>`, `<list>`
  - `<fstream>`, `<cmath>`, `<complex>`, `<limits>`

---

## Conclusion

The Calculus Tree Library provides a powerful, flexible toolkit for mathematical expression manipulation. Its template-based design, comprehensive function support, and symbolic differentiation capabilities make it suitable for scientific computing, educational applications, and mathematical modeling.

```
