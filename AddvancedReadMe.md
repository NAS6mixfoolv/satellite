# Advanced satellite Documentation  
  
This document provides detailed explanations of satellite.htm, advanced features, and detailed technical specifications.  
  
---
  
### Table of contents  
* [The Orbital Element Consistency Algorithm (Conceptual Overview)](#the-orbital-element-consistency-algorithm-conceptual-overview)  
* [satellite.htm](#satellitehtm)  
* [satellite.js](#satellitejs)  
  * [Global Variable Declarations](#global-variable-declarations)  
  * [Building and Initializing N6LPlanet](#building-and-initializing-n6lplanet)  
  
[Back to NAS6LIB Repository](https://github.com/NAS6mixfoolv/NAS6LIB/)  

---
  
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
  
---

# satellite.htm
[Back to Table of contents](#table-of-contents)  
<br>  
＜!-- load X3DOM !--＞<br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/x3dom/jquery-2.1.4.min.js"></script><br>  
＜link rel='stylesheet' type='text/css' href='./javascripts/x3dom/x3dom.css'＞<br>  
<script language="JavaScript" type='text/javascript' src='./javascripts/x3dom/x3dom-full.js'></script><br>  
＜link rel='stylesheet' type='text/css' href='./javascripts/x3dom/x3dom.css'＞<br>  
<br>  
＜!-- load NAS6LIB !--＞<br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6/common.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/timer.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/vector.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/matrix.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/quaternion.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/planet.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/masspoint.js"></script><br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6lib/rngkt.js"></script><br>  
＜!-- load own js !--＞<br>  
<script language="JavaScript" type="text/javascript" src="./javascripts/nas6/satellite.js"></script><br>  
<br>  
<style><br>  
article, aside, dialog, figure, footer, header,<br>  
hgroup, menu, nav, section { display: block; }<br>  
＜!-- X3DOM canvas style !--＞<br>  
#x3dabs{<br>  
    position: absolute;<br>  
    float: left;<br>  
    top: 420px;<br>  
    left: 20px;<br>  
    background-image:  url("./img/mimiback.png");<br>  
    border: 2px #000000 solid;<br>  
}<br>  
</style><br>  
<br>    
The above is exactly what it looks like.
  
# satellite.js  
[Back to Table of contents](#table-of-contents)  
This JavaScript file manages the orbital simulation of multiple celestial bodies (planets and satellites)  
and their 3D visualization on the web using X3DOM.  
  
# Global Variable Declarations 
  
This section defines the configuration values, timers, object IDs, simulation control flags,  
time management variables, physical constants, and custom class instances that are shared across the entire application.  
  
* **TMan (N6LTimerMan)**: Manages the simulation's timer events.  
* **IDTransA, IDTransZ**: Arrays of IDs used for setting the translation (position) information of 3D objects (celestial bodies) in X3DOM.  
  * **IDTransA**: An object ID list controlling movement along the X-axis in the ecliptic plane.  
  * **IDTransZ**: An object ID list controlling movement along the Z-axis in the ecliptic plane.  
* **IDT, IDL**: Arrays of IDs related to the display of celestial body orbital lines.  
  * **IDT**: An X3DOM object ID list for setting the orbital inclination and orbital plane rotation of celestial bodies.  
  * **IDL**: An X3DOM object ID list for setting the geometric data (lineSegments) of each celestial body's orbit.  
* **x3domRuntime**: The X3DOM runtime object, used for manipulating the 3D scene.  
* **planetnum**: The number of celestial bodies (planets) being simulated (here, 11).  
* **bBBB, bRunning, bWaiting, fFst, bRead, bLAM**: Various control flags related to the simulation's state and user input.  
  * **bRunning**: A flag indicating if the simulation is currently active.  
  * **bWaiting**: A flag indicating if the simulation is in a waiting state.  
* **intvl**: The timer interval in milliseconds.  
* **Speed**: The time progression speed of the simulation.  
* **Zoom**: The zoom scale in the 3D view.  
* **dat**: The base date object for the simulation.  
* **time, dt**: Variables related to time calculation within the simulation.  
* **CNST_AU**: A constant defining 1 Astronomical Unit (AU) in meters.  
* **planet (N6LPlanet Array)**: An array storing N6LPlanet objects, each containing  
the orbital elements and physical properties of a celestial body.  
* **mpadj, mp (N6LMassPoint Array)**: Arrays managing N6LMassPoint objects, which are  
abstracted mass points used for gravitational calculations.  
* **rk (N6LRngKt)**: An instance of the N6LRngKt class, which handles the numerical integration  
(Runge-Kutta method) for accurately updating celestial body positions and velocities.  
  
---  
  
# Building and Initializing N6LPlanet  
[Back to Table of contents](#table-of-contents)  
  
# PlanetInit(dat) Function  
This function calculates and sets the initial position and velocity for each celestial body (N6LPlanet instance)  
based on the specified dat date. It primarily uses Kepler's laws to determine the position and velocity in an elliptical orbit  
and converts them to the ecliptic coordinate system as needed. If NaN (Not a Number) values occur during calculation,  
it resets the position and velocity to zero to prevent errors.  
  
# setline() Function  
This function generates the orbital line data for each celestial body and applies it to the corresponding X3DOM elements.  
For celestial bodies where mp[i].mass <= 0.0 (likely dummy or central bodies), it draws a simple square orbit.  
For other celestial bodies, it constructs the orbit by dividing the circumference into 32 segments  
and calculating each segment's position using Kepler's equations. The computed line data is then set as  
the lineSegments attribute of the relevant X3DOM LineSet element. Additionally, it calculates and applies rotation information  
(reflecting orbital plane inclination and longitude of the ascending node) to the X3DOM element.  
  
# init(b) Function  
This function initializes the simulation environment by reading user input such as simulation speed, zoom scale,  
and selected celestial body ID from the user interface. If b is false, it resets all mass points (mp array).  
It then calls setmp() (assumed to be a function that sets mass points) and setline() to update the display.  
Finally, it invokes InitRelative() for a more detailed simulation initialization.  
  
# InitRelative() Function  
This function initializes the simulation time, initial positions/velocities of celestial bodies, and the numerical integrator (N6LRngKt).  
Based on the current simulation time, it calls PlanetInit() to establish the initial state of the celestial bodies.  
Subsequently, it configures the N6LRngKt instance with the current array of mass points and the time step (dt),  
making it ready for numerical integration.  
  
# onRunning() Function  
This function serves as the main loop for the simulation, delegating frame-by-frame updates to the UpdateFrameRelative() function.  
  
# UpdateFrameRelative() Function  
This function contains the primary logic for updating each simulation frame. Based on the specified time step (dt) and speed (Speed),  
it repeatedly calls the rk.UpdateFrame() method to update the position and velocity of each celestial body. Crucially, after updates,  
it includes a process to correct the positions of other celestial bodies relative to the Sun (assumed to be the body at index 0),  
effectively re-centering the simulation on the Sun. Finally, it calls setmp() and settime() (assumed to update the display of  
mass points and time) to reflect the updated positions.  
  
---  

[Back to Table of contents](#table-of-contents)  



  
