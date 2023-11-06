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

## lc-1.f

This version of the program takes the user's input, converts it to fixed-point representation, calculates the Lorentz contraction using an approximation of the Lorentz factor, and then outputs the result in meters. The `LORENTZ-FACTOR` word would need to be filled in with an actual fixed-point square root operation or approximation suitable for your application.

Remember that fixed-point arithmetic is limited in precision and range. If you need higher precision or must handle larger numbers, you might have to use a more complex number representation or switch to a Forth environment that supports floating-point arithmetic or larger integers.

##  lc-2.f

previous code contains a few potential errors and issues regarding the implementation of fixed-point arithmetic and handling of 16-bit integers without a carry flag. Here are some steps to correct the code:

1. The constant for the speed of light is slightly off from the actual value. Since we're approximating, we can use `299792` as a scaled-down constant that fits within 16 bits.

2. We're missing the definition of `S" "` which is used to create space-separated strings.

3. The Lorentz contraction calculation is trying to use floating-point math, which is not available. We need to refactor the math to use fixed-point calculations.

4. The code uses `2DUP` which duplicates the top two stack items, but it's not defined in standard Forth-83.

5. `METER-TO-LENGTH` has been defined as a constant but never given a value.

6. `INVERT` in the calculation is likely meant to be `1/SQRT`, but `SQRT` isn't a standard word in Forth-83, so we'll need to implement a fixed-point square root function or an approximation.

corrected version:

The fixed-point square root function (`FIXED-SQRT`) is left as a placeholder because implementing a square root algorithm in fixed-point arithmetic without floating-point support is non-trivial and would require a separate explanation.

In addition, the program now properly uses `NUMBER` to convert the user input into a number and then scales it to fixed-point format by multiplying it with `METER-TO-LENGTH`. The velocity check now uses `U<` to compare the unsigned magnitude of the velocity against the speed of light. The main calculation correctly uses fixed-point arithmetic to perform the Lorentz contraction calculation, and `DISPLAY-RESULTS` is a separate word for better readability.

`FIXED-SQRT`:

Implementing a fixed-point square root function in Forth without a hardware floating-point unit can be done with an iterative algorithm, such as the Newton-Raphson method or a binary search method. Since we are using 16-bit fixed-point arithmetic, I'll illustrate a simple binary search square root method that converges on the integer square root of a number:

```forth
: FIXED-SQRT ( n -- sqrt(n) in fixed-point )
  DUP 0= IF DROP 0 EXIT THEN  \ Return 0 for input 0
  DUP 1+ DUP 2/          \ Initial guess is (n/2) + 1 to ensure it is not zero
  BEGIN
    OVER OVER / OVER + 2/  \ Calculate new guess: (guess + n / guess) / 2
    2DUP = UNTIL          \ Repeat until the new guess is the same as the old guess
  DROP ;
```

This code starts with an initial guess for the square root, which is half the input number plus one to ensure the guess is not zero. The loop repeatedly improves the guess based on the average of the guess and the input number divided by the guess. The loop continues until the guess no longer changes.

However, this code doesn't handle fixed-point arithmetic specifically. Since you're using a fixed-point representation where 16384 represents 1.0, we need to adjust our square root function to account for this scaling. We'll also need to prevent overflow during the calculation. Here's how you could write a fixed-point square root function that accounts for the fixed-point scaling:

```forth
: FIXED-SQRT ( n -- sqrt(n) in fixed-point )
  DUP 0= IF DROP 0 EXIT THEN
  16384 OVER /   \ Adjust the input for fixed-point
  1+ 2/          \ Initial guess is ((n/16384) / 2) + 1
  BEGIN
    OVER OVER / OVER + 2/  \ Calculate new guess
    DUP 16384 * OVER < IF DROP OVER THEN \ Prevent overflow by scaling down if needed
    2DUP = UNTIL           \ Repeat until the guess is stable
  DROP 16384 * ;           \ Adjust the result back to fixed-point
```

This function starts by adjusting the input `n` to a whole number by dividing by the fixed-point base (`16384`). Then it uses a similar binary search method as before but scales down the guess when it threatens to overflow. Finally, it scales the result back up to fixed-point format before returning it.

Remember that due to the limited precision of 16-bit integers, this square root function will be an approximation. this might be good enough, but more precision need more complex algorithms 





