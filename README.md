# Lorentz-contraction

# Length
## mathematical components adapted for fixed-point arithmetic 

The classical Lorentz contraction formula is:

![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/4fef995f-af14-410d-b35c-f750ace55dc5)


\[ L' = L \sqrt{1 - \left(\frac{v}{c}\right)^2} \]

where:

- \( L' \) is the length as measured by an observer moving at velocity \( v \) relative to the object.
- \( L \) is the proper length of the object (the length of the object in the frame of reference in which the object is at rest).
- \( v \) is the relative velocity between the observer and the moving object.
- \( c \) is the speed of light.

The original formula includes a square root and a division, operations that can be challenging with fixed-point arithmetic due to their non-linear nature and potential for generating results that don't fit in a fixed number of bits. Here's how the program approximates the formula in Forth:

1. **Fixed-Point Representation**: We use a scaling factor to represent real numbers as integers. Here, we've chosen \( 2^{14} \) as the scaling factor, so 1.0 is represented as \( 16384 \) in our fixed-point system. This allows us to represent fractions in the range \( [0, 1) \).

2. **Constants**: The speed of light is approximated to fit within a 16-bit integer after accounting for the fixed-point scaling. Instead of \( 299792458 \), we use \( 29979 \) with the scaling factor already considered.

3. **Velocity Squared**: We calculate \( v^2 \) as follows:
4. 
![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/ea78da87-0791-496f-8aed-6e96e37b85a3)

\[ v'^2 = (v \cdot FIXED\_POINT)^2 = (v \cdot 16384)^2 \]

However, since we're using fixed-point representation, we then need to scale the result back down:

![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/ccdf4fda-b148-4149-b7a5-0b9e5b41546c)

\[ \frac{v'^2}{FIXED\_POINT} = \frac{(v \cdot 16384)^2}{16384} \]

This effectively gives us \( v^2 \) in our fixed-point system.

4. **Beta Factor**: This is ![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/4be04832-9404-4acd-9136-6f9bd495de99)
\( \left(\frac{v}{c}\right)^2 \), which in fixed-point is represented as:

![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/bace6090-7ff6-446c-ac9a-02409668b8e3)

\[ \text{beta'} = \frac{v'^2}{(c \cdot FIXED\_POINT)^2} \]

Again, after calculating \( v'^2 \), we divide by \( c'^2 \) (where \( c' \) is the speed of light in fixed-point) to get beta in fixed-point.

5. **Lorentz Factor**: In full precision, it would be
6.  ![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/3ee2db9f-6ba0-43f8-976a-fa331dc5776a)
 \( \sqrt{1 - \beta} \). We approximate this by avoiding the square root (since we don't have a simple fixed-point square root function) and instead calculate:

![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/87e43cba-876c-4706-a678-8789c2c04db1)

\[ \gamma' = FIXED\_POINT - \text{beta'} \]

This is not the exact Lorentz factor but an approximation. The square root of values close to 1 can be approximated by \( 1 - \frac{1}{2}(1 - x) \) for small \( x \), which is somewhat similar to what's done here, but without the \( \frac{1}{2} \) scaling.

6. **Lorentz Contraction**: The final length \( L' \) is calculated in fixed-point arithmetic:

![image](https://github.com/SteveJustin1963/Lorentz-contraction/assets/58069246/bb19f1ab-1bb0-4e3f-adb9-1d2f0ef72108)

\[ L' = L \cdot \gamma' \]

Here \( L \) is multiplied by the approximate Lorentz factor \( \gamma' \), and since both are in fixed-point, the result is also in fixed-point and needs to be scaled down by dividing by the FIXED\_POINT constant.

7. **Output**: The result \( L' \) is then converted from fixed-point back to a normal integer representing meters by dividing by the FIXED\_POINT constant.

The program needs to ensure that all calculations are performed within the limits of 16-bit arithmetic, which might require additional handling for potential overflows, especially when squaring the velocity. The carry flag mentioned in the initial user input might be used for this purpose, depending on how the underlying system implements multiplication and other arithmetic operations.
code for 16-bit fixed-point arithmetic in Forth-83.

1. **Constants and Conversion**: Constants should reflect the fixed-point representation. The speed of light constant needs to be in a fixed-point representation that fits within a 16-bit integer while maintaining the necessary precision.

2. **Fixed-Point Multiplication and Division**: The operations on fixed-point numbers should account for the fixed-point format. For multiplication, you typically need to divide by the scale factor (here `FIXED-POINT`) to maintain the correct scale. For division, you multiply by the scale factor.

3. **Square Root**: Calculating the square root in fixed-point arithmetic is not straightforward and usually requires an iterative approach or a lookup table. However, for simplicity, we might use an approximation or a simplification.

4. **Handling Large Numbers**: The program must ensure that multiplication and other operations do not exceed the 16-bit limit. If they do, it must handle overflow appropriately, potentially using the carry flag for large numbers.

## code LC MINT
This MINT code is implementing the Lorentz contraction formula using fixed-point arithmetic. It defines constants and functions to perform the necessary calculations while staying within the constraints of a fixed-point, 16-bit environment. Here’s an explanation of each part of the code:

### Constants
```mint
16384 s !     // Scaling factor (2^14) for fixed-point representation
29979 c !     // Scaled speed of light value
```
- **`s`**: This is the scaling factor used for converting real numbers to fixed-point representation. \( 16384 \) (or \( 2^{14} \)) represents 1.0 in fixed-point format.
- **`c`**: This is the speed of light value, scaled to fit within the 16-bit range for fixed-point arithmetic.

### Function `:A` - Fixed-Point Multiplication
```mint
:A 
  * s / ;
```
- This function multiplies two numbers and then scales the result back down using the fixed-point scaling factor `s`. It ensures that the result remains within the proper scale of fixed-point arithmetic.

### Function `:B` - Fixed-Point Division
```mint
:B 
  s * / ;
```
- This function divides two numbers while scaling up the numerator by `s` to maintain precision. It ensures the division result stays consistent with the fixed-point format.

### Function `:C` - Fixed-Point Square Root Approximation
```mint
:C 
  " 0= ( ' 0 ) /E (                
    " s / 1+ " 2/                  
    /U (                           
      % % B % + 2/                
      2" =                         
    )
    ' s *                          
  )
;
```
- This function calculates the square root of a number in fixed-point format. The code uses an iterative binary search method to approximate the square root:
  - It checks if the input value is zero and, if so, returns zero.
  - It then sets an initial guess for the square root and refines it using the formula \( \text{new\_guess} = \frac{\text{guess} + \frac{n}{\text{guess}}}{2} \).
  - The loop continues until the guess stabilizes, indicating that the square root has been found.
  - The result is then scaled back to fit the fixed-point format.

### Function `:D` - Lorentz Contraction Calculation
```mint
:D 
  " s * c B
  " A
  s $ -
  C
  $ A
  s / ;
```
- This function calculates the Lorentz contraction using the fixed-point arithmetic approach:
  1. **Calculate the ratio** \( \frac{v}{c} \) in fixed-point:
     - Multiplies the velocity by the scaling factor and divides by the speed of light (`c`).
  2. **Square the ratio** \( \left(\frac{v}{c}\right)^2 \):
     - Uses function `A` to perform the fixed-point multiplication of this ratio with itself.
  3. **Compute \( 1 - \left(\frac{v}{c}\right)^2 \)**:
     - Subtracts the squared ratio from the scaling factor `s` (which represents 1.0 in fixed-point).
  4. **Take the square root**:
     - Uses function `C` to approximate the square root of the result.
  5. **Multiply by the original length**:
     - Multiplies the computed Lorentz factor by the input length using function `A`.
  6. **Scale down the result**:
     - Finally, divides by `s` to return the length contraction result in fixed-point format.

### Summary
The code implements the Lorentz contraction formula:

\[
L' = L \sqrt{1 - \left(\frac{v}{c}\right)^2}
\]

- **`s`** and **`c`** are constants adapted for fixed-point arithmetic.
- The functions (`A`, `B`, `C`, and `D`) perform the necessary arithmetic operations (multiplication, division, square root approximation) in fixed-point to ensure calculations stay within the 16-bit constraints of the environment.

The overall goal is to approximate the Lorentz contraction in a way that works within the limited precision of 16-bit fixed-point numbers while avoiding floating-point operations.
