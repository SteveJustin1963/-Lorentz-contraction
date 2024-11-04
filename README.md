[https://github.com/SteveJustin1963/tec-MINT

Here is a simple (far from it for MINT) program for calculating the Lorentz contraction:

Given the Lorentz contraction formula:

![image](https://github.com/user-attachments/assets/efa7aa21-4706-4f19-8288-8525385f61d1)

`L = L0((1 - v2/c2))1/2`

  example in MINT:

```mint
:L
10 l!        // Original length
2 v!         // Velocity (as percentage of c)
v v * t!     // Calculate v²
100 t - p!   // Calculate 1 - (v²/c²)
1 n!         // Start sqrt calculation
/U(
n n * p <= /W 
n 1 + n!
)
n 1 - s!     // Store sqrt result
l s * 100 / . `/100`  // Final calculation and display as ratio
;
```
 
but this doesn not work as we loose precision with 16bit integers.
as we're still losing our ratio to integer division.

Let's restructure to avoid dividing too early. For 2% speed:
1. v/c = 2/100 = 0.02
2. (v/c)² = 0.0004
3. Need to scale BEFORE dividing

```text
:L
0 n! 0 t! 0 v! 0 p! 0 s! 0 r! 0 q! 0 k! 0 /c!
100 l!
2 v!
`Length=` l .
`Velocity=` v .
v 100 * q!    
q q * t!
100 100 * c!
`V squared=` t .
`C squared=` c .
10000 t - p!
`1-V2/C2=` p .
1 n!
/U(
n n * p < /W
n 1 + n!
)
n k!
`Root=` k .
l k * 10000 /
. `/100`
;

```

Key changes:
1. Scale v up by 100 first
2. Square both v and c
3. Subtract from 10000
4. Avoid division until final step

result 
```
> L
Length=100 Velocity=2 V squared=-25536 C squared=10000 1-V2/C2=-30000 Root=189 1 /100
>
```

nope i give up i cant do any better

if we modify mint for 32bit signed...would help significantly because:

1. Range would be -2,147,483,648 to +2,147,483,647 instead of just -32,768 to +32,767

This means:
- Could scale numbers much larger before division
- Keep more precision in fractions
- Handle intermediate calculations without overflow
- Do proper scaling for v²/c² calculation (which needs 4 decimal places)

For example, with 32-bit could do:
1. v = 2 * 10000 = 20000 (scale velocity)
2. v² = 400,000,000 (still within 32-bit range)
3. c² = 100,000,000,000
4. v²/c² = 0.0004 (proper ratio)

But with 16-bit we're forced to:
- Limit scaling 
- Do early divisions that lose precision
- Break calculations into smaller steps
- Lose accuracy in intermediate results

i dont see JH adding 32-bit for more accurately? i know what he will say

what of we add the 9511 mpu!!

The AMD Am9511 Arithmetic Processing Unit would be perfect for this because it:

1. Has hardware floating point operations:
   - 16 or 32-bit fixed point
   - 32-bit floating point
   - Square root built in
   - Division built in

2. Perfect for Lorentz calculation because:
   - Could handle v²/c² directly (0.0004)
   - Built-in SQRT function
   - Maintains precision through calculations
   - No need for complex scaling

3. Would simplify the MINT code to:
   - Just feed numbers to 9511
   - Get results in floating point
   - Convert back for display

4. Operations it provides:
   - ADD, SUB, MUL, DIV
   - SQRT
   - SIN, COS, TAN
   - Convert between formats

ok asume we did , lets assum the 9511 in on port 0xc2

sudo program is
```text
:L
0 n! 0 t! 0 v! 0 p! 0 s! 0 r! 0 q! 0 k! 0 /c!
100 l!
2 v!
v #C2 /O    
v #C2 /O    
#C2 /I      
100 #C2 /O  
#C2 /I      
100 #C2 /O  
#C2 /I
l #C2 /O
#C2 /I
. `/100`    
;

```

Each /O sends to APU port 0xC2:
1. Push v twice for v×v
2. Read result (v²)
3. Push 100 twice for c×c
4. Read result (c²)
5. Divide v²/c²
6. Subtract from 1
7. Take square root
8. Multiply by l
9. Read final result

using proper APU command bytes between operations 

```
:L
2 #C2 /O         // Push v (2% of c)
#B8 #C2 /O      // PTOF - convert to float
2 #C2 /O        // Push v again
#B8 #C2 /O      // PTOF - convert to float
#A8 #C2 /O      // FMUL - multiply to get v²
100 #C2 /O      // Push c 
#B8 #C2 /O      // PTOF
100 #C2 /O      // Push c again
#B8 #C2 /O      // PTOF
#A8 #C2 /O      // FMUL - get c²
#A9 #C2 /O      // FDIV - v²/c²
1 #C2 /O        // Push 1
#B8 #C2 /O      // PTOF
#A5 #C2 /O      // FSUB - get 1-v²/c²
#AD #C2 /O      // FSQRT - square root
100 #C2 /O      // Push length
#B8 #C2 /O      // PTOF
#A8 #C2 /O      // FMUL - multiply by length
#B9 #C2 /O      // FFIX - convert back to integer
#C2 /I          // Read result
. `/100`        // Display as ratio
;
```

The 9511 has an internal 8-level stack - we can use that more efficiently:

```text
:L
2 #C2 /O       
#B8 #C2 /O     // v to float on stack
" #C2 /O       // duplicate on 9511 stack
#A8 #C2 /O     // v*v now on stack
100 #C2 /O
#B8 #C2 /O     // convert 100 to float
" #C2 /O       // duplicate on 9511 stack
#A8 #C2 /O     // c*c now on stack
#A9 #C2 /O     // v²/c² on stack
1 #C2 /O
#B8 #C2 /O     // 1.0 on stack
#A5 #C2 /O     // 1-v²/c² on stack
#AD #C2 /O     // sqrt on stack
100 #C2 /O     // original length
#B8 #C2 /O     // to float
#A8 #C2 /O     // L * sqrt(1-v²/c²)
#B9 #C2 /O     // back to fixed point
#C2 /I         // get result
. `/100`
;

```

Better because:
1. Uses 9511's stack operations
2. Needs fewer MINT variables
3. More efficient - all computation in 9511
4. Better precision using float throughout

The " command duplicates top of 9511 stack instead of reloading values.

now puch yourself in the balls!

](https://github.com/SteveJustin1963/tec-MINT

Here is a simple (far from it for MINT) program for calculating the Lorentz contraction:

Given the Lorentz contraction formula:

![image](https://github.com/user-attachments/assets/efa7aa21-4706-4f19-8288-8525385f61d1)

`L = L0((1 - v2/c2))1/2`

  example in MINT:

```mint
:L
10 l!        // Original length
2 v!         // Velocity (as percentage of c)
v v * t!     // Calculate v²
100 t - p!   // Calculate 1 - (v²/c²)
1 n!         // Start sqrt calculation
/U(
n n * p <= /W 
n 1 + n!
)
n 1 - s!     // Store sqrt result
l s * 100 / . `/100`  // Final calculation and display as ratio
;
```
 
but this doesn't work as we loose precision with 16bit integers.
as we're still losing our ratio to integer division.

Let's restructure to avoid dividing too early. For 2% speed:
1. v/c = 2/100 = 0.02
2. (v/c)² = 0.0004
3. Need to scale BEFORE dividing

```text
:L
0 n! 0 t! 0 v! 0 p! 0 s! 0 r! 0 q! 0 k! 0 /c!
100 l!
2 v!
`Length=` l .
`Velocity=` v .
v 100 * q!    
q q * t!
100 100 * c!
`V squared=` t .
`C squared=` c .
10000 t - p!
`1-V2/C2=` p .
1 n!
/U(
n n * p < /W
n 1 + n!
)
n k!
`Root=` k .
l k * 10000 /
. `/100`
;

```

Key changes:
1. Scale v up by 100 first
2. Square both v and c
3. Subtract from 10000
4. Avoid division until final step

result 
```
> L
Length=100 Velocity=2 V squared=-25536 C squared=10000 1-V2/C2=-30000 Root=189 1 /100
>
```

nope i give up i cant do any better

if we modify mint for 32bit signed...would help significantly because:

1. Range would be -2,147,483,648 to +2,147,483,647 instead of just -32,768 to +32,767

This means:
- Could scale numbers much larger before division
- Keep more precision in fractions
- Handle intermediate calculations without overflow
- Do proper scaling for v²/c² calculation (which needs 4 decimal places)

For example, with 32-bit could do:
1. v = 2 * 10000 = 20000 (scale velocity)
2. v² = 400,000,000 (still within 32-bit range)
3. c² = 100,000,000,000
4. v²/c² = 0.0004 (proper ratio)

But with 16-bit we're forced to:
- Limit scaling 
- Do early divisions that lose precision
- Break calculations into smaller steps
- Lose accuracy in intermediate results

i don't see JH adding 32-bit for more accurately? i know what he will say

what of we add the 9511 mpu!!

The AMD Am9511 Arithmetic Processing Unit would be perfect for this because it:

1. Has hardware floating point operations:
   - 16 or 32-bit fixed point
   - 32-bit floating point
   - Square root built in
   - Division built in

2. Perfect for Lorentz calculation because:
   - Could handle v²/c² directly (0.0004)
   - Built-in SQRT function
   - Maintains precision through calculations
   - No need for complex scaling

3. Would simplify the MINT code to:
   - Just feed numbers to 9511
   - Get results in floating point
   - Convert back for display

4. Operations it provides:
   - ADD, SUB, MUL, DIV
   - SQRT
   - SIN, COS, TAN
   - Convert between formats

lets assume the 9511 in on port 0xc2

```text
:L
0 n! 0 t! 0 v! 0 p! 0 s! 0 r! 0 q! 0 k! 0 /c!
100 l!
2 v!
v #C2 /O    
v #C2 /O    
#C2 /I      
100 #C2 /O  
#C2 /I      
100 #C2 /O  
#C2 /I
l #C2 /O
#C2 /I
. `/100`    
;

```

Each /O sends to APU port 0xC2:
1. Push v twice for v×v
2. Read result (v²)
3. Push 100 twice for c×c
4. Read result (c²)
5. Divide v²/c²
6. Subtract from 1
7. Take square root
8. Multiply by l
9. Read final result

using proper APU command bytes between operations 

```
:L
2 #C2 /O         // Push v (2% of c)
#B8 #C2 /O      // PTOF - convert to float
2 #C2 /O        // Push v again
#B8 #C2 /O      // PTOF - convert to float
#A8 #C2 /O      // FMUL - multiply to get v²
100 #C2 /O      // Push c 
#B8 #C2 /O      // PTOF
100 #C2 /O      // Push c again
#B8 #C2 /O      // PTOF
#A8 #C2 /O      // FMUL - get c²
#A9 #C2 /O      // FDIV - v²/c²
1 #C2 /O        // Push 1
#B8 #C2 /O      // PTOF
#A5 #C2 /O      // FSUB - get 1-v²/c²
#AD #C2 /O      // FSQRT - square root
100 #C2 /O      // Push length
#B8 #C2 /O      // PTOF
#A8 #C2 /O      // FMUL - multiply by length
#B9 #C2 /O      // FFIX - convert back to integer
#C2 /I          // Read result
. `/100`        // Display as ratio
;
```
The 9511 has an internal 8-level stack - we can use that more efficiently:

```text
:L
2 #C2 /O       
#B8 #C2 /O     // v to float on stack
" #C2 /O       // duplicate on 9511 stack
#A8 #C2 /O     // v*v now on stack
100 #C2 /O
#B8 #C2 /O     // convert 100 to float
" #C2 /O       // duplicate on 9511 stack
#A8 #C2 /O     // c*c now on stack
#A9 #C2 /O     // v²/c² on stack
1 #C2 /O
#B8 #C2 /O     // 1.0 on stack
#A5 #C2 /O     // 1-v²/c² on stack
#AD #C2 /O     // sqrt on stack
100 #C2 /O     // original length
#B8 #C2 /O     // to float
#A8 #C2 /O     // L * sqrt(1-v²/c²)
#B9 #C2 /O     // back to fixed point
#C2 /I         // get result
. `/100`
;

```

Better because:
1. Uses 9511's stack operations
2. Needs fewer MINT variables
3. More efficient - all computation in 9511
4. Better precision using float throughout

The " command duplicates top of 9511 stack instead of reloading values.

now punch yourself in the balls!

 
