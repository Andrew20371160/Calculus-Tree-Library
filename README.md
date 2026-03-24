# Calculus Tree Library

A C++ library for preprocessing, building, manipulating, and evaluating mathematical expression trees. It supports symbolic differentiation, numerical integration, simplification, variable substitution, and common vector-calculus operators (gradient, divergence, curl, Laplacian).

## Features

- **Expression parsing**
  - Recursive descent parser with operator precedence
  - Full parenthesis support
  - **Implicit multiplication** (e.g., `2x` → `2*x`, `(x+y)(z+5)` → `(x+y)*(z+5)`)
- **Symbolic differentiation**
  - Single-variable and multivariable differentiation
  - Chain, product, quotient, and power rules
  - Built-in rules for standard functions (trig, inverse trig, hyperbolic, log/ln/exp, etc.)
- **Numerical evaluation**
  - Substitute variables at runtime
  - Templated `DataType` (commonly `long double`)
- **Numerical integration**
  - Simpson’s **1/3** and **3/8** rules
- **Vector calculus**
  - `gradient`, `divergence`, `curl` (3D), `laplacian`
- **Expression manipulation**
  - Simplification, variable exchange/substitution
  - Save/load expressions
  - Random equivalence testing
- **Optional complex mode**
  - Enable `COMPLEX_MODE` for `std::complex<long double>` support and complex constants/functions

## Repository layout (high level)

- `calculus_tree.h/.cpp` — main `calculus_tree<DataType>` class (parsing, evaluation, diff, integration, vector calculus)
- `node.h/.cpp` — expression tree node + structural operations
- `preprocessor.h/.cpp` — expression validation + implicit operator insertion
- `functions_and_known_constants.h/.cpp/.tpp` — supported functions/constants + evaluation helpers
- `DOCUMENTATION.MD` — full documentation and API details

## Requirements

- **C++11 or later**
- Standard headers used include: `<iostream>`, `<string>`, `<vector>`, `<cmath>`, `<complex>`, `<limits>`, `<fstream>`, etc.

## Getting started

### 1) Include the library
Add the `.h/.cpp/.tpp` files to your project, then:

```cpp
#include "calculus_tree.h"
```

### 2) Create and evaluate an expression

```cpp
#include <iostream>
#include <vector>
#include <string>
#include "calculus_tree.h"

int main() {
    calculus_tree<long double> f("x^2 + 3*x + 2");

    std::vector<std::string> vars = {"x", "5"};
    long double result = f.evaluate_at(vars);

    std::cout << "f(5) = " << result << "\n";     // expected: 52
    std::cout << "expr = " << f.expression() << "\n";
}
```

**Variable substitution format:** `{"var1","value1","var2","value2",...}`

## Symbolic differentiation

```cpp
#include <iostream>
#include <vector>
#include <string>
#include "calculus_tree.h"

int main() {
    calculus_tree<long double> f("sin(x) * x^2");
    auto df = f.diff_with("x");

    std::cout << "f(x)  = " << f  << "\n";
    std::cout << "f'(x) = " << df << "\n";

    std::vector<std::string> at = {"x", "3.14159"};
    std::cout << "f'(pi) ≈ " << df.evaluate_at(at) << "\n";
}
```

## Vector calculus

### Gradient

```cpp
#include <iostream>
#include <vector>
#include <string>
#include "calculus_tree.h"

int main() {
    calculus_tree<long double> f("x^2 + y^2");

    std::vector<std::string> vars = {"x", "y"};
    auto grad = f.gradient(vars);

    std::cout << "df/dx = " << grad[0] << "\n"; // 2*x
    std::cout << "df/dy = " << grad[1] << "\n"; // 2*y

    std::vector<std::string> point = {"x","3","y","4"};
    std::cout << "grad at (3,4) = ("
              << grad[0].evaluate_at(point) << ", "
              << grad[1].evaluate_at(point) << ")\n";
}
```

### Laplacian

```cpp
#include <iostream>
#include <vector>
#include <string>
#include "calculus_tree.h"

int main() {
    calculus_tree<long double> f("x^2 + y^2 + z^2");
    std::vector<std::string> vars = {"x","y","z"};

    auto lap = f.laplacian(vars);
    std::cout << "∇²f = " << lap << "\n"; // expected: 6
}
```

## Numerical integration (Simpson’s rule)

```cpp
#include <iostream>
#include <vector>
#include <string>
#include "calculus_tree.h"

int main() {
    calculus_tree<long double> f("x^2");

    long double approx = f.simpson_rule_1_3(
        "x",     // integration variable
        0.0L,    // start
        10.0L,   // end
        1000,    // intervals
        0,       // traversal_array_size (0 = auto)
        {""}     // other variables (optional)
    );

    std::cout << "Integral ≈ " << approx << "\n"; // exact: 1000/3 ≈ 333.33...
}
```

## Simplification & substitution

```cpp
#include <iostream>
#include "calculus_tree.h"

int main() {
    calculus_tree<long double> t("0 + x");
    t.simplify();
    std::cout << t << "\n"; // x

    auto u = t.exchange("x", "2*y"); // replace x with (2*y)
    std::cout << u << "\n";          // 2*y
}
```

## Supported operators / functions / constants

### Operators
`+  -  *  /  ^` and parentheses `(` `)`.

### Common functions
Includes (non-exhaustive): `sin`, `cos`, `tan`, `sec`, `csc`, `cotan`, `asin`, `acos`, `atan`, `sinh`, `cosh`, `tanh`, `asinh`, `acosh`, `atanh`, `sqrt`, `abs`, `ln`, `exp`, and `log` / `logN` forms (e.g., `log2(x)`).

### Constants
`pi`, `e`, `inf`, `nan` (and `i` in complex mode).

See **DOCUMENTATION.MD** for the full list and derivative rules.

## Complex mode (optional)

To use complex numbers:
1. Enable `COMPLEX_MODE` in `functions_and_known_constants.h` (as described in `DOCUMENTATION.MD`)
2. Use `calculus_tree<std::complex<long double>>`

Example:
```cpp
calculus_tree<std::complex<long double>> z("sin(x) + 5*i");
```

## Notes / Error handling

- The preprocessor validates expressions and inserts implicit operators.
- On syntax errors, the preprocessor may return `"0"` and print messages to `std::cout` (see `DOCUMENTATION.MD`).

## Documentation
- Full documentation: `DOCUMENTATION.MD`
