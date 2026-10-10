# Complex Numbers

A **complex number** consists of two parts:

- A **real part**
    
- An **imaginary part**
    

It is commonly written as:

        z = a + bi

where:

- `a` = real component
    
- `b` = imaginary component
    
- `i` = imaginary unit, where i2=−1i^2 = -1
    

For example:

```text
3 + 4i
```

has:

```text
Real part      = 3
Imaginary part = 4
```

C provides built-in support for complex arithmetic through the `<complex.h>` header.

---

# Complex Types

C provides three complex floating-point types:

```c
float _Complex
double _Complex
long double _Complex
```

They can also be written using the `_Complex` keyword:

```c
float _Complex z1;
double _Complex z2;
long double _Complex z3;
```

`<complex.h>` provides convenient type names:

```c
complex
```

through the `complex` macro.

For example:

```c
double complex z;
```

is equivalent to:

```c
double _Complex z;
```

---

# Complex Type Precision

The three complex types correspond to different floating-point precisions.

```text
float complex
    ↓
float real part + float imaginary part

double complex
    ↓
double real part + double imaginary part

long double complex
    ↓
long double real part + long double imaginary part
```

For most calculations:

```c
double complex
```

is the commonly used choice.

---

# Defining Complex Numbers

Include `<complex.h>`:

```c
#include <complex.h>
```

Then declare a complex variable:

```c
double complex z;
```

You can also initialize it using real and imaginary components.

```c
double complex z = 3.0 + 4.0 * I;
```

`I` is provided by `<complex.h>` and represents the imaginary unit.

Therefore:

```c
3.0 + 4.0 * I
```

represents:

```text
3 + 4i
```

---

# Assigning Complex Numbers

Complex values can be assigned to other complex variables just like other C values.

```c
double complex a = 3.0 + 4.0 * I;
double complex b;

b = a;
```

After the assignment:

```text
a = 3 + 4i
b = 3 + 4i
```

The complete complex value is copied.

---

# Assigning Real Values

A real value can also be assigned to a complex variable.

```c
double complex z;

z = 5.0;
```

The imaginary component becomes zero:

```text
z = 5 + 0i
```

Similarly:

```c
double complex z = 5.0 + 0.0 * I;
```

represents the same value.

---

# Constructing Complex Numbers

The `<complex.h>` header provides functions for constructing complex values from their components.

The main function is:

```c
double complex CMPLX(double x, double y);
```

For the corresponding floating-point types:

```c
float complex CMPLXF(float x, float y);
long double complex CMPLXL(long double x, long double y);
```

Example:

```c
double complex z = CMPLX(3.0, 4.0);
```

This creates:

```text
z = 3 + 4i
```

---

# Deconstructing Complex Numbers

The real and imaginary components of a complex number can be extracted using:

```c
creal()
cimag()
```

Example:

```c
double complex z = 3.0 + 4.0 * I;

double real = creal(z);
double imaginary = cimag(z);
```

Result:

```text
real      = 3
imaginary = 4
```

There are also type-specific versions:

```c
crealf()
cimagf()

creall()
cimagl()
```

---

# Printing Complex Numbers

C's `printf()` does not provide a dedicated conversion specifier for complex numbers.

Instead, extract the real and imaginary parts and print them separately.

Example:

```c
#include <stdio.h>
#include <complex.h>

int main(void)
{
    double complex z = 3.0 + 4.0 * I;

    printf("Real: %f\n", creal(z));
    printf("Imaginary: %f\n", cimag(z));
}
```

Output:

```text
Real: 3.000000
Imaginary: 4.000000
```

You can also format the value yourself:

```c
printf("%f + %fi\n", creal(z), cimag(z));
```

---

# Complex Arithmetic

C supports normal arithmetic operators directly on complex values.

The basic operators are:

```text
+
-
*
/
```

Example:

```c
double complex a = 2.0 + 3.0 * I;
double complex b = 4.0 + 1.0 * I;

double complex sum  = a + b;
double complex diff = a - b;
double complex prod = a * b;
double complex quot = a / b;
```

The compiler performs the corresponding complex arithmetic.

---

# Complex Addition

For:

```text
(a + bi) + (c + di)
```

the result is:

```text
(a + c) + (b + d)i
```

Example:

```text
(2 + 3i) + (4 + i)
```

produces:

```text
6 + 4i
```

---

# Complex Subtraction

For:

```text
(a + bi) - (c + di)
```

the result is:

```text
(a - c) + (b - d)i
```

Example:

```text
(5 + 4i) - (2 + i)
```

produces:

```text
3 + 3i
```

---

# Complex Multiplication

Complex multiplication follows the normal algebraic rules.

```text
(a + bi)(c + di)
```

produces:

```text
(ac - bd) + (ad + bc)i
```

Example:

```text
(2 + 3i)(4 + i)
```

produces:

```text
8 + 2i + 12i + 3i²
```

Since:

```text
i² = -1
```

the result is:

```text
5 + 14i
```

---

# Complex Division

Division is also supported by the `/` operator:

```c
double complex result = a / b;
```

The compiler handles the complex division according to the rules for complex arithmetic.

---

# Complex Comparisons

Complex numbers have an important limitation when it comes to comparisons.

You **cannot** use relational operators such as:

```c
<
>
<=
>=
```

with complex values.

For example:

```c
double complex a = 2.0 + 3.0 * I;
double complex b = 4.0 + 1.0 * I;

if (a < b) {       // Invalid
}
```

There is no natural ordering such as "less than" between arbitrary complex numbers.

---

# Equality and Inequality

Equality operators can be used:

```c
==
!=
```

Example:

```c
if (a == b)
{
    printf("Equal\n");
}
```

This compares the complex values.

However, remember that floating-point equality can have the usual precision issues associated with floating-point calculations.

---

# Complex Math

The `<complex.h>` header provides mathematical functions specifically designed for complex numbers.

These functions are grouped into several categories:

```text
Trigonometric functions
Exponential functions
Logarithmic functions
Power functions
Absolute-value functions
Manipulation functions
```

Most functions have versions for:

```text
float complex
double complex
long double complex
```

---

# Trigonometric Functions

C provides complex versions of common trigonometric functions:

```c
csin()
ccos()
ctan()
```

Example:

```c
double complex z = 1.0 + 2.0 * I;

double complex result = csin(z);
```

These functions calculate the sine, cosine, or tangent of a complex number.

There are also corresponding functions for other precisions:

```c
csinf()
ccosf()
ctanf()

csinl()
ccosl()
ctanl()
```

---

# Inverse Trigonometric Functions

Complex versions of inverse trigonometric functions are also available:

```c
casin()
cacos()
catan()
```

For example:

```c
double complex result = casin(z);
```

These calculate:

```text
complex arcsine
complex arccosine
complex arctangent
```

---

# Hyperbolic Functions

C also provides complex hyperbolic functions:

```c
csinh()
ccosh()
ctanh()
```

and their inverse forms:

```c
casinh()
cacosh()
catanh()
```

Example:

```c
double complex result = csinh(z);
```

---

# Exponential Functions

The complex exponential function is:

```c
cexp()
```

Example:

```c
double complex z = 1.0 + 2.0 * I;

double complex result = cexp(z);
```

This calculates:

eze^z

for a complex `z`.

There are also:

```c
cexpf()
cexpl()
```

for the other complex floating-point types.

---

# Logarithmic Functions

The complex natural logarithm is provided by:

```c
clog()
```

Example:

```c
double complex result = clog(z);
```

This calculates:

log⁡(z)\log(z)

for a complex value.

The other versions are:

```c
clogf()
clogl()
```

---

# Power Functions

The complex power function is:

```c
cpow()
```

Example:

```c
double complex result = cpow(a, b);
```

This calculates:

aba^b

where both `a` and `b` can be complex numbers.

Corresponding versions:

```c
cpowf()
cpowl()
```

---

# Absolute Value Functions

For a complex number:

z=a+biz = a + bi

its magnitude is:

∣z∣=a2+b2|z| = \sqrt{a^2 + b^2}

C provides:

```c
cabs()
```

Example:

```c
double complex z = 3.0 + 4.0 * I;

double magnitude = cabs(z);
```

Result:

```text
5
```

because:

```text
sqrt(3² + 4²) = 5
```

There are also:

```c
cabsf()
cabsl()
```

---

# Argument of a Complex Number

The angle of a complex number can be obtained using:

```c
carg()
```

Example:

```c
double angle = carg(z);
```

The result represents the phase angle of the complex number in radians.

---

# Conjugate

The complex conjugate of:

```text
a + bi
```

is:

```text
a - bi
```

C provides:

```c
conj()
```

Example:

```c
double complex z = 3.0 + 4.0 * I;

double complex result = conj(z);
```

Result:

```text
3 - 4i
```

There are also:

```c
conjf()
conjl()
```

---

# Manipulation Functions

The `<complex.h>` header provides several functions for examining and manipulating complex values.

Important functions include:

```c
creal()     // obtain real component
cimag()     // obtain imaginary component
cabs()      // obtain magnitude
carg()      // obtain phase angle
conj()      // obtain complex conjugate
```

Example:

```c
double complex z = 3.0 + 4.0 * I;

printf("Real      = %f\n", creal(z));
printf("Imaginary = %f\n", cimag(z));
printf("Magnitude = %f\n", cabs(z));
printf("Angle     = %f\n", carg(z));

double complex conjugate = conj(z);
```

Conceptually:

```text
             z = a + bi
                │
       ┌────────┼────────┐
       │        │        │
       ▼        ▼        ▼
    creal()   cimag()   cabs()
       │        │        │
       ▼        ▼        ▼
       a        b      magnitude

             │
             ▼
           carg()
             │
             ▼
          phase angle

             │
             ▼
           conj()
             │
             ▼
           a - bi
```

---

# Complex Numbers and Memory

From a low-level perspective, a complex floating-point value consists of a real and imaginary component.

Conceptually:

```text
double complex z

+----------------------+
| real component       |
+----------------------+
| imaginary component  |
+----------------------+
```

The exact representation and implementation details are governed by the C standard and implementation.

Do not assume that every compiler must represent every complex object in exactly the same machine-level way.

---

# Reverse Engineering Perspective

Complex numbers are less common in ordinary systems programming than integers, pointers, structures, and floating-point values.

However, they can appear in programs involving:

- Signal processing
    
- Audio processing
    
- Image processing
    
- Scientific computing
    
- Physics and engineering
    
- Fourier transforms
    
- Numerical algorithms
    

When reverse engineering such software, recognizing complex-number operations can help identify the underlying algorithm.

For example, code performing operations equivalent to:

```c
cabs(z)
conj(z)
creal(z)
cimag(z)
```

may indicate that the binary is manipulating complex mathematical data.

At the assembly level, these operations are ultimately implemented using ordinary floating-point instructions and compiler-generated routines.

---

# Mental Model

```text
Complex Number
      │
      ├── Real component
      │
      └── Imaginary component
              │
              ▼
        double complex
              │
       ┌──────┼─────────┐
       │      │         │
       ▼      ▼         ▼
      + - * /        Math functions
                       │
             ┌─────────┼──────────┐
             ▼         ▼          ▼
           Trig     Exp/Log     Power
             │         │          │
             └─────────┼──────────┘
                       ▼
                  Complex result
```

---

# Long-Term Takeaways

- C supports complex arithmetic through `<complex.h>`.
    
- The complex types are:
    

```c
float complex
double complex
long double complex
```

- `I` represents the imaginary unit.
    
- Complex values can be assigned directly to other complex variables.
    
- Real values can be assigned to complex variables, producing a zero imaginary component.
    
- `CMPLX()` can construct a complex value from real and imaginary components.
    
- `creal()` extracts the real component.
    
- `cimag()` extracts the imaginary component.
    
- Complex values support:
    

```text
+  -  *  /
```

- Equality operators `==` and `!=` can be used with complex values.
    
- Relational operators such as `<` and `>` are not defined for complex numbers.
    
- `csin()`, `ccos()`, and `ctan()` provide complex trigonometric functions.
    
- `cexp()` and `clog()` provide complex exponential and logarithmic operations.
    
- `cpow()` performs complex exponentiation.
    
- `cabs()` calculates the magnitude of a complex value.
    
- `carg()` obtains its phase angle.
    
- `conj()` produces the complex conjugate.
    
- Complex-number operations can appear in scientific, signal-processing, and numerical software.
    
- From an RE perspective, recognizing complex arithmetic can help identify mathematical algorithms inside compiled binaries.