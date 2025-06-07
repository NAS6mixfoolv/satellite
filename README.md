# satellite
How to use the Runge-Kutta method for numerical analysis of gravity-based many-body problems in the theory of relativity  
using JavaScript in NAS6LIB and how to use x3dom for displaying the results  
LIB:  
https://github.com/NAS6mixfoolv/NAS6LIB  
WIKI:  
https://github.com/NAS6mixfoolv/NAS6LIB/wiki/Physics-Simulations-(Kepler-&-Runge%E2%80%90Kutta)  
DEMO:  
https://nas6mixfoolv.github.io/NAS6LIB/  
https://nas6mixfoolv.github.io/solarsystem/  
https://nas6mixfoolv.github.io/satellite/  
  
Jupiter swing-by:  
![Jupiter swing-by orbit simulation](satellite05.gif)  
  
# NAS6LIB: A Versatile Mathematical Library for 3D Graphics and Celestial Mechanics  
  
NAS6LIB is a robust and versatile JavaScript library engineered for **rigorous mathematical computations**  
in 3D graphics and celestial mechanics. It provides foundational tools for precise calculations related to vectors, matrices,  
and quaternions, essential for accurately modeling various physical phenomena.  
  
While NAS6LIB itself focuses on computational precision, its capabilities extend to complex applications. For instance,  
the **`satellite.htm`** demonstration, built upon NAS6LIB, offers a platform to **visually explore relativistic gravitational  
N-body problems**. This allows users to observe and understand how fundamental theories of relativity subtly yet profoundly influence  
orbital trajectories, such as Mercury's anomalous perihelion precession.  
  
### Automated Orbital Consistency  
  
NAS6LIB includes powerful tools for 3D graphics and mathematical computations. For celestial orbit simulations, a significant challenge is   maintaining the physical consistency of orbital elements. **When users arbitrarily set parameters such as orbital period, masses,  
or various orbital radii, these values frequently lead to mathematical contradictions within the system.  
Such inconsistencies inevitably result in unstable orbits or immediate simulation failures—for example, celestial bodies  
flying off into space or crashing.**  
  
Our **`satellite.htm`** demonstration showcases an **automatic contradiction resolution feature** that adjusts input orbital parameters to   ensure physically stable and accurate orbits.  
  
#### The Orbital Element Consistency Algorithm (Conceptual Overview)  
  
The core logic for resolving orbital element inconsistencies can be found in the `myCalc1` function within `satellite.js` on the   `satellite.htm` page. This algorithm intelligently adjusts input parameters based on fundamental celestial mechanics,  
primarily Kepler's Third Law.
  
### Deeper Dive & Advanced Details  
For more detailed information, including a detailed walkthrough of  
this example and specific implementation theory, including  
automation course corrections, please see the advanced documentation.  
  
Advanced satellite Documentation  
[AddvancedReadMe](AddvancedReadMe.md)  

