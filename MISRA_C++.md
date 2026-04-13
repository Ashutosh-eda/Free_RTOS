# What is MISRA?

- **MISRA**: Motor Industry Software Reliability Association
- **Originally**: UK government initiative to develop guidelines for embedded software in road vehicle electronic systems
- **Purpose**: Promote best practices in safety- and security-related electronic systems and other software-intensive applications
- **Focus**: C and C++ for critical systems
- **Critical systems**: Failure can lead to significant harm, loss of life, injury, or substantial property damage

## Different behaviors in C++

C++ does not always define behavior in the same way. Some actions are fully defined, some depend on the compiler or platform, and some are left unpredictable. MISRA is concerned with these cases because safety-critical code should avoid surprises.

- **Undefined behavior (UB)**: The language gives no guarantees about what happens. Once UB occurs, the result can be anything, including a crash or seemingly random output.
  - Examples: dereferencing `nullptr`, accessing memory outside array bounds.

- **Implementation-defined behavior**: The compiler or platform is allowed to choose a valid behavior, but it must document that choice.
  - Example: the exact size of `int` on a given system.

- **Unspecified behavior**: The program is still valid C++, but the standard does not require the compiler to document which option it chooses.
  - Example: the order in which function arguments are evaluated.

## Problems with C++ in critical systems

C++ is powerful, but that also makes it easy to write code that behaves unpredictably or depends too much on compiler details. In safety-critical software, that is a serious problem.

- **Undefined or unspecified behavior** can lead to results that are unpredictable or impossible to rely on.
- **Feature misuse** happens when programmers use language features in ways that are technically allowed but unsafe, fragile, or easy to get wrong.
- **Language misunderstandings** are common in tricky areas such as implicit type conversions and memory management.
  - For example, a value may be converted automatically in a way the programmer did not expect.
  - Memory mistakes can easily lead to leaks, invalid access, or undefined behavior.
- **Runtime checks are limited** in C++ for things like array bounds and pointer validity, so many errors are not caught automatically when the program runs.

## MISRA guidelines

MISRA guidance is split into different types so teams can understand how strict each recommendation is and how it should be enforced.

- **Directives** are harder to check automatically with tools, so they often need human judgment or process-based review.
- **Rules** are written more concretely, which makes them easier to check using static analysis tools.
- **Categories** describe how strongly a guideline should be followed:
  - **Mandatory**: must be followed, and exceptions are not allowed.
  - **Required**: should be followed, but a documented deviation may be allowed when there is a valid reason.
  - **Advisory**: recommended best practice that should be followed where practical.
- **Enforcement** is typically done using static analysis tools and code reviews to catch violations early.

## Formal deviations

Sometimes a project cannot follow a MISRA guideline exactly. When that happens, the exception should not be informal or hidden. It needs to be handled in a controlled and traceable way.

- If compliance cannot be achieved, the deviation must be recorded and managed systematically.
- A deviation record should include:
  - the guideline that was broken,
  - the reason the deviation was necessary,
  - background or context that explains the situation,
  - and an assessment of the risk introduced.
- Teams also need traceability so they can identify which deviations exist in the codebase.
  - This is often supported with comments or machine-readable markers that make deviations easier to find and audit later.

## Modern C++ practices

MISRA C++:2023 overlaps with several modern C++ best practices. The idea is to avoid language features that can hide bugs or make code harder to reason about.

- **4.1.3**: Avoid code that depends on undefined or critically unspecified behavior. In practice, this means not relying on compiler accidents or situations the language does not fully define.
- **0.1.2**: Always use a function’s return value when the function is meant to produce one. Ignoring the result can hide errors or make the call pointless.
- **0.2.2**: Give named function parameters a real purpose in the function body. If a parameter is unused, that usually suggests dead code, a design issue, or a missed mistake.
- **6.4.1**: Do not let a name declared in a smaller scope hide a name from an outer scope. Shadowing can make code confusing and can cause the wrong variable to be used.
- **11.6.2**: Do not read the value of an object before it has been initialized. Using uninitialized data makes the program’s behavior unreliable.
- **15.1.3**: Constructors and conversion operators that can be called with a single argument should be marked `explicit`. This prevents accidental implicit conversions that can change program behavior in surprising ways.

### Examples

- **4.1.3**
  - Example of undefined behavior: using a macro that expands to `defined`, initializing a 16-bit signed value with a constant that is too large, or shifting a value by too many bits.
  - These are the kinds of cases MathWorks flags as violations because the result is not dependable.

- **0.1.2**
  - If a function returns a status code or computed result, ignoring it can hide a failure.
  - Example:
    ```cpp
    int result = compute_value();
    (void)result; // the value exists, but the program does not use it
    ```

- **0.2.2**
  - If a function takes a parameter but never uses it, that can mean the interface is poorly designed or the parameter was forgotten.
  - Example:
    ```cpp
    void log_event(int level, const char* message) {
      (void)level;
      // message is used, level is not
    }
    ```

- **6.4.1**
  - Shadowing can happen when a local variable uses the same name as one from an outer scope.
  - Example:
    ```cpp
    int value = 10;
    {
      int value = 20; // hides the outer value
    }
    ```

- **11.6.2**
  - Reading an object before it has been initialized can produce garbage data.
  - Example:
    ```cpp
    int count;
    if (count > 0) { // count has not been set yet
      // ...
    }
    ```

- **15.1.3**
  - A single-argument constructor should usually be marked `explicit` so the compiler does not convert values automatically.
  - Example:
    ```cpp
    class Speed {
    public:
      explicit Speed(int v) : value(v) {}
    private:
      int value;
    };
    ```

## Dynamic memory and integer types

### 21.6.1 Dynamic memory should not be used

This rule discourages dynamic memory allocation in critical code because allocation can fail, take unpredictable time, or lead to leaks and fragmentation. MathWorks notes that even implicit allocation through containers can count as a violation if `std::allocator` is used by default.

- Use of `new`, `delete`, `malloc`, `free`, and similar allocation functions is noncompliant.
- Standard containers such as `std::vector` and `std::set` are also problematic when they rely on dynamic allocation.
- Safer alternatives include fixed-size storage such as `std::array`, memory pools, or preallocated buffers.

Example:
```cpp
int* p = new int(42);          // Noncompliant
std::vector<int> v;            // Noncompliant
auto f = std::make_shared<int>(42); // Noncompliant
```

### 21.6.2 Dynamic memory shall be managed automatically

If dynamic memory is used at all, MISRA expects it to be handled through automatic resource management rather than manual `new` and `delete` calls. The goal is to reduce leaks, double frees, and ownership mistakes.

- Avoid direct memory management in application code.
- Prefer smart pointers such as `std::unique_ptr` when ownership is simple.
- Avoid calling `release()` unless there is a very deliberate ownership transfer.
- If manual management is unavoidable, isolate it and wrap it carefully with RAII-style helpers.

Example:
```cpp
auto p = new int(42);                 // Noncompliant
delete p;                             // Noncompliant
auto f = std::make_unique<int>(42);   // Compliant
f.release();                          // Noncompliant
```

### 6.9.2 Standard signed and unsigned integer type names should not be used

This rule discourages using built-in type names like `int`, `short`, and `unsigned long` directly because their sizes vary by compiler and platform. For safety-critical code, that makes assumptions fragile and can hurt portability.

- Prefer fixed-width integer types such as `std::int32_t` and `std::uint16_t`.
- The `main` function return type and `argc` are common exceptions.
- Type aliases are also acceptable when they help centralize the choice of type.

Example:
```cpp
int count = 0;                 // Not preferred
std::int32_t count2 = 0;       // Preferred

using ticks_t = long;          // Alias can be used in some designs
```

## Casts and arrays

### 8.2.2 C-style casts and functional notation casts shall not be used

This rule discourages old-style casts because they are too broad and can hide what kind of conversion is happening. In safety-critical code, it is better to use casts that show intent clearly.

- C-style casts can perform many different conversions without making the type change obvious.
- They can also be confused with constructor calls, which makes code harder to read.
- A cast to `void` is allowed when you intentionally want to ignore a value.
- Prefer `static_cast` or `dynamic_cast` when they fit the situation.

Example:
```cpp
auto i = (std::int16_t)42;     // Noncompliant
auto z = std::int16_t(42);     // This is a constructor call, not a cast
(void)i;                       // Allowed exception
```

### 8.2.5 `reinterpret_cast` shall not be used

`reinterpret_cast` is very dangerous because it lets you force unrelated types into each other. That often leads to undefined behavior or code that depends on machine details.

- Do not use `reinterpret_cast` for arbitrary type punning.
- The one exception is when converting between pointer types that are explicitly allowed by the rule, such as `void*`, character types, `std::byte`, or `std::uintptr_t`-related cases.
- In general, if you feel tempted to use `reinterpret_cast`, the design probably needs a safer approach.

Example:
```cpp
std::int32_t i = 42;
auto pI = reinterpret_cast<std::uint32_t*>(i); // Noncompliant

std::int32_t val = 32;
auto pToVal = reinterpret_cast<std::byte*>(&val); // Compliant by exception
```

### 7.11.2 An array passed as a function argument shall not decay to a pointer

When an array is passed to a function, it often becomes just a pointer, which means the size information is lost. MISRA wants to avoid that because array bounds are important and should not disappear silently.

- Losing array size makes it easier to write out-of-bounds code.
- If a function needs to work with arrays of different sizes, the size should be carried explicitly or the array should be passed by reference.
- String literals are a common exception because their length can be inferred from the terminating `\0`.

Example:
```cpp
void processArray1(std::int32_t arr[42]);      // Noncompliant, decays to pointer
void processArray2(std::int32_t* arr);         // Noncompliant, no size information
void processArray3(std::int32_t (&arr)[42]);   // Compliant, preserves array size

template<std::size_t N>
void processArray4(std::int32_t (&arr)[N]);     // Compliant, preserves array size
```

## String conversion

### 21.2.1 The library functions `atof`, `atoi`, `atol`, and `atoll` from `<cstdlib>` shall not be used

These functions are unsafe because they do not provide strong error handling. If the input cannot be converted, the result may be misleading or undefined for the caller’s purposes. Modern code should use safer conversion utilities instead.

- Prefer `std::stoi`, `std::stod`, or `std::from_chars` depending on the data type and error-handling needs.
- `std::from_chars` is especially useful because it gives explicit success or failure information.

Example:
```cpp
int i = atoi("42");   // Noncompliant
int j = std::stoi("42"); // Compliant

std::int32_t res = 0;
std::string_view str = "42";
auto [p, errc] = std::from_chars(str.data(), str.data() + str.size(), res);
if (errc == std::errc()) {
  std::cout << "Parsed integer: " << res << '\n';
} else {
  std::cerr << "Error parsing integer\n";
}
```

## Globals, character handling, and variadic functions

### 6.7.2 Global variables shall not be used

Global state makes code harder to reason about because anything can change it, sometimes from multiple places at once. In critical systems, that can lead to data races, hidden dependencies, and initialization problems.

- Avoid non-constant globals and non-constant `static` data in global scope.
- Global initialization order can be unpredictable, especially across translation units.
- Shared mutable state is a frequent source of bugs.

Example:
```cpp
std::int32_t currentSpeed = getCurrentSpeed(); // Noncompliant
const std::int32_t prevSpeed{ currentSpeed };   // Noncompliant if dynamic init is involved
```

### 24.5.1 Character handling functions from `<cctype>` and `<cwctype>` shall not be used

Functions like `isalpha`, `isdigit`, and `toupper` can behave incorrectly if the input value is not representable as `unsigned char` or `EOF`. That makes them unsafe in code that must be robust and portable.

- The problem is not the name of the function itself, but the input domain and locale dependence.
- Prefer safer, well-defined alternatives from `<locale>` when character classification is needed.

Example:
```cpp
std::isalpha('a');          // Noncompliant
std::isdigit('1', std::locale{}); // Compliant
```

### 21.10.1 The features of `<cstdarg>` shall not be used

Variadic argument lists are risky because they bypass compile-time type checking. The caller and callee must agree exactly on the argument types, but the compiler cannot verify that.

- `va_list`, `va_start`, `va_arg`, and related functions are not allowed.
- A variadic function can easily become unsafe if arguments are missing, reordered, or read with the wrong type.
- Prefer variadic templates or functions with a fixed parameter list.

Example:
```cpp
void doSomething(va_list args) { /* ... */ } // Noncompliant

template<typename T, typename... Args>
void doSomething(const T& first, const Args&... args) { /* ... */ }
```

## C library functions

### 30.0.1 The C library input/output functions shall not be used

The classic C I/O functions are not a good fit for safety-critical C++ code because they can be error-prone and less type-safe than C++ streams or higher-level APIs.

- Functions like `gets`, `fopen`, `fclose`, `fread`, `fflush`, `printf`, and `scanf` are discouraged.
- Safer C++ alternatives include streams such as `std::cin`, `std::ifstream`, and `std::ofstream`.
- In modern C++, prefer APIs that make resource management and error handling explicit.

Example:
```cpp
printf("Hello, World!\n");        // Noncompliant
std::cout << "Hello, World!" << std::endl; // Compliant
```

### 21.2.3 The library function `system` from `<cstdlib>` shall not be used

`system` is risky because it executes shell commands, which can create security problems and platform-specific behavior. It also hides what really happens from the code reviewer.

- It can be exploited if command input is not tightly controlled.
- Its behavior varies by platform and shell environment.
- Prefer direct APIs or filesystem libraries instead of spawning a shell.

Example:
```cpp
std::system("ls ."); // Noncompliant

for (const auto& entry : std::filesystem::directory_iterator(".")) {
  std::cout << entry.path() << std::endl; // Compliant
}
```

### 21.2.2 The string handling functions from `<cstring>`, `<cstdlib>`, `<cwchar>`, and `<cinttypes>` shall not be used

Classic C string functions are dangerous because they do not track buffer sizes well and can cause out-of-bounds access or missing null-termination bugs.

- Functions like `strcat`, `strcmp`, `strcpy`, `strlen`, and `strtol` are fragile in safety-critical code.
- Prefer `std::string` and `std::string_view` when possible.
- When working with buffers, make the size explicit and use safer copy patterns.

Example:
```cpp
void populate(char* message, std::size_t messageSize) {
  const char* payload = "a message";
  if ((strlen(payload) + 1U) < messageSize) {
    strcpy(message, payload); // Noncompliant style
  }
}

std::string_view payload = "a message"; // Modern alternative
payload.size();
payload.copy(message, payload.size());   // Safer pattern
```

## Storage duration, pragmas, predicates, and exceptions

### 6.7.1 Local variables shall not have static storage duration

This rule discourages local variables that live for the entire program instead of only for the duration of a function call. Static locals can create hidden shared state and make initialization and destruction harder to reason about.

- Static local variables can introduce temporal coupling and data races.
- Their initialization order may be tricky in more complex programs.
- They can make functions behave like stateful components instead of simple computations.

Example:
```cpp
template<typename T>
T& getSingleton() {
  static T instance{}; // Noncompliant
  return instance;
}
```

### 19.6.1 The `#pragma` directive and the `_Pragma` operator should not be used

Pragmas are compiler-specific and reduce portability. They can also hide behavior from readers and tools.

- Avoid compiler directives that are not part of standard, portable C++.
- Prefer standard language constructs and project-wide build configuration instead.
- If you must use a pragma, it should be carefully justified and documented.

Example:
```cpp
// Foo.h
#pragma once   // Noncompliant
#ifndef FOO_H
#define FOO_H
// ...
#endif
```

### 28.3.1 Predicates shall not have persistent side effects

Predicates are functions that return a boolean result. They should not change important shared state while being evaluated, because that makes control flow hard to reason about and can produce surprising results.

- A predicate should tell you something, not secretly change the program.
- Side effects inside a predicate can create unpredictable behavior when algorithms call it multiple times.
- This matters a lot for STL algorithms such as `std::count_if`.

Example:
```cpp
using Samples = std::array<int32_t, 10U>;
int64_t countBadSamples(const Samples& s, int32_t& superBad) {
  return std::count_if(s.begin(), s.end(), [&superBad](int32_t sample) {
    if (sample < 100) { superBad++; } // Noncompliant
    return sample < 0;
  });
}
```

### 18.5.2 Program-terminating functions should not be used

Functions such as `std::abort`, `std::exit`, `std::quick_exit`, and `std::terminate` stop the program abruptly. That can leave resources unreleased and make the system end in an unsafe or incomplete state.

- Destructors may not run, so cleanup may be skipped.
- Files, locks, or external resources can be left in a bad state.
- `assert` is treated differently because it is a debugging aid rather than a runtime termination strategy.

Example:
```cpp
void handleError() {
  std::cerr << "Fatal error occurred, terminating program\n";
  std::exit(EXIT_FAILURE); // Noncompliant
}
```

### 18.3.1 There should be at least one exception handler to catch all otherwise unhandled exceptions

If exceptions are used, the program should have a catch-all handler so that unexpected exceptions do not escape uncontrolled. The goal is to fail in a known way instead of leaving termination behavior to the implementation.

- A `catch (...)` block acts as a safety net for exceptions not handled elsewhere.
- This is especially useful around `main`.
- The specific handling strategy should still distinguish known exceptions from unexpected ones.

Example:
```cpp
int main() {
  try {
    cool_company::Car car{}; // Application code
  }
  catch (const cool_company::NoFuelException& e) { return 1; }
  catch (...) { return 99; } // Catch-all for all other exceptions
  return 0;
}
```

### 8.2.10 Functions shall not call themselves, either directly or indirectly

Recursion is disallowed because it makes worst-case execution harder to predict and can lead to stack overflow. In safety-critical software, bounded and predictable execution is more important than elegant recursive logic.

- Recursive calls can consume stack space in a way that is hard to bound.
- Indirect recursion is just as risky as direct recursion.
- If recursion is truly necessary, it should usually be avoided or formally justified.

Example:
```cpp
int32_t factorial(int32_t n) {
  if (n <= 1) return 1;          // Base case
  return n * factorial(n - 1);   // Noncompliant, recursion
}
```

## Conditionals, enums, and class layout

### 9.4.1 All `if` ... `else if` constructs shall be terminated with an `else` statement

This rule encourages defensive programming. An `else` branch makes it clear that the code has considered every path, even if the final branch is only a fallback or error handler.

- Without a final `else`, it is easier to miss an unhandled case.
- An empty `else` may still be acceptable if it makes the intent explicit.
- For `switch` statements, the similar idea is that a default case should exist.

Example:
```cpp
if (car.getSpeed() > 100) { /* OK */ }
else if (car.getSpeed() < 50) { /* ... */ }
else { /* Compliant, handles all cases */ }
```

### 10.2.1 An enumeration shall be defined with an explicit underlying type

An enum should declare its underlying type so the size and representation are predictable. This improves portability and reduces surprise conversions.

- Unscoped enums can leave the underlying type up to the implementation.
- Scoped enums should usually specify the base type explicitly.
- This is especially useful when the enum values are stored, serialized, or exchanged across interfaces.

Example:
```cpp
enum class Month { Jan = 1, Feb, Mar, Apr, May, Jun }; // Noncompliant
enum class Day : int8_t { Sun = 0, Mon, Tue, Wed, Thu, Fri }; // Compliant
enum class Color { R, G, B }; // Compliant by exception
```

### 14.1.1 Non-static data members should be either all private or all public

Mixing public and private data members usually leads to weaker encapsulation and makes it easier to violate class invariants accidentally. MISRA prefers a clearer design boundary.

- Use member functions to expose controlled access instead of mixing access levels.
- A class with only public data is less protected against misuse.
- A class with all private data is easier to keep consistent internally.

Example:
```cpp
class Car {
public:
  int32_t id{}; // Noncompliant, mixed access
  int32_t getSpeed() const { return speed; }
private:
  int32_t speed{};
};
```

## Conclusion

These are some of the most important MISRA C++ rules I found while reviewing the MathWorks MISRA C++:2023 reference.

All of these rules can also be found on the MathWorks website:

https://in.mathworks.com/help/bugfinder/misra-cpp-2023-rules-and-directives.html

If you want, I can keep adding more rules from the same source and expand this into a fuller study note.
