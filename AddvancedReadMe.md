#### The Orbital Element Consistency Algorithm (Conceptual Overview)  
  
The core logic for resolving orbital element inconsistencies can be found in the `myCalc1` function within `satellite.js` on the   `satellite.htm` page. This algorithm intelligently adjusts input parameters based on fundamental celestial mechanics,  
primarily Kepler's Third Law.
  
It operates in two main patterns, adapting to the provided input:  
  
**Pattern 1: Adjusting Radii based on Period and Masses**  
If the **period (P)** and **masses (M1 + M2)** are considered primary, the algorithm:  
1.  **Recalculates the semi-major axis (a):** It derives `my1A1` using Kepler's Third Law, where
2.  $a^3 = (G(M1+M2)P^2) / (4\pi^2)$. This ensures the semi-major axis is consistent with the given period and masses.  
3.  **Adjusts periapsis and apoapsis distances:** Using the newly calculated `my1A1` and the input eccentricity (`my1E`),
4.  it recalculates and overwrites the initial periapsis (`my1RA1`) and apoapsis (`my1RA2`) distances. This ensures
5.  the orbital radii match the derived semi-major axis and eccentricity:  
    * `my1RA2 = my1A1 * (1 + my1E)` (apoapsis distance)  
    * `my1RA1 = my1A1 * (1 - my1E)` (periapsis distance)  
    This pattern ensures that the orbital radii are consistent with the `period` and `masses`.  
  
**Pattern 2: Adjusting Mass or Period based on Radii**  
If the **periapsis (`my1RA1`) and apoapsis (`my1RA2`) distances** are primary, the algorithm:  
1.  **Recalculates the semi-major axis (a):** It first derives `my1A1` from
2.  the average of `my1RA1` and `my1RA2`: $a = (my1RA1 + my1RA2) / 2.0$.  
3.  **Adjusts either mass or period:** Based on user selection, it then either:  
    * **Recalculates `my1M1` (mass):** If the period is fixed, it adjusts `my1M1` using Kepler's Third Law:
    * $M1 = (4\pi^2 a^3) / (G P^2) - M2$.  
    * **Recalculates `my1P` (period):** If the masses are fixed, it adjusts `my1P` using Kepler's Third Law:
    * $P = 2\pi \sqrt{a^3 / (G (M1 + M2))}$.
    This pattern ensures that the `mass` or `period` (whichever is adjusted) is consistent with the `orbital radii`.  
  
Finally, the algorithm performs additional calculations for orbital velocities (`my1Vst`, `my1VA1`, `my1VA2`) and recalculates  
the eccentricity (`my1E = (my1RA2 - my1A1) / my1A1`) to ensure overall physical consistency across all derived orbital parameters.  
