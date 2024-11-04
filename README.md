Here is a simple MINT program for calculating the Lorentz contraction:

Given the Lorentz contraction formula:

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
 
