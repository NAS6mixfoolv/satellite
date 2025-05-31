# Advanced satellite Documentation  
  
This document provides detailed explanations of satellite.htm, advanced features, and detailed technical specifications.  
  
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

# stellite.htm
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
  
# stellite.js  
  
//global position  
  
//timer manager  
var TMan = new N6LTimerMan()  
//object translation infomation name  
var IDTransA = new Array('sph00a', 'sph01a', 'sph02a', 'sph03a', 'sph04a', 'sph05a', 'sph06a', 'sph07a', 'sph08a', 'sph09a', 'sph10a');  
var IDTransZ = new Array('sph00z', 'sph01z', 'sph02z', 'sph03z', 'sph04z', 'sph05z', 'sph06z', 'sph07z', 'sph08z', 'sph09z', 'sph10z');  
//orbit line name  
var IDT = new Array('ln00t', 'ln01t', 'ln02t', 'ln03t', 'ln04t', 'ln05t', 'ln06t', 'ln07t', 'ln08t', 'ln09t', 'ln10t');  
var IDL = new Array('ln00l', 'ln01l', 'ln02l', 'ln03l', 'ln04l', 'ln05l', 'ln06l', 'ln07l', 'ln08l', 'ln09l', 'ln10l');  
  
//x3domRuntime  
var x3domRuntime;  
  
//Number of palanets  
var planetnum = 11;  
  
//control flags  
var bBBB;  
var bRunning = false;  
var bWaiting = false;  
var fFst = 1;  
var bRead = false;  
var bLAM = false;  
var intvl = 50;  
  
//time speed  
var Speed = 1.0;  
//zoom scale  
var Zoom = 1.0;  
//based time  
var dat = new Date(0);  
var time;  
var dt;  
  
//const  
var CNST_AU = 1.49597870700e+11;  
  
//N6LPlanet Array  
var planet = new Array();  
for(dt = 0; dt < planetnum; dt++) planet[dt] = new N6LPlanet();  
dt = 0;  
//N6LMassPoint Array  
var mpadj = new Array();  
var mp = new Array();  
//N6LRngKt  
var rk = new N6LRngKt();  
  
The declaration in the global position is roughly as shown above.  
  
### Building and initializing N6LPlanet  
  
# stellite.js  
  
...  
  
//Planetary initialization  
function PlanetInit(dat) {  
  var msecPerMinute = 1000 * 60;  
  var msecPerHour = msecPerMinute * 60;  
  var msecPerDay = msecPerHour * 24;  
  var i;  
  var j;  
    for(i = 0; i < planetnum; i++) {  
      if(mp[i].mass < 0.0) continue;  
      var dat0 = planet[i].m_dat0;  
      var datt = dat.getTime();  
      var dat0t = dat0.getTime();  
      var ddat = (datt - dat0t) / msecPerDay;  
      var nday = ddat;  
  
      var xx = new Array(new N6LVector(3));  
      var f = planet[i].kepler(nday, xx);  
      planet[i].x0 = new N6LVector(3);  
      planet[i].x0.x[0] = xx[0].x[0];  
      planet[i].x0.x[1] = xx[0].x[1];  
      planet[i].x0.x[2] = 0.0;  
  
      var xyz = new Array(new N6LVector(3));  
      planet[i].ecliptic(planet[i].x0.x[0], planet[i].x0.x[1], planet[i].x0.x[2], xyz);  
      if(isNaN(xyz[0].x[0]) || isNaN(xyz[0].x[1]) || isNaN(xyz[0].x[2])) {  
        planet[i].x0.x[0] = 0.0;  
        planet[i].x0.x[1] = 0.0;  
        planet[i].x0.x[2] = 0.0;  
      }  
      else {  
        planet[i].x0.x[0] = xyz[0].x[0];  
        planet[i].x0.x[1] = xyz[0].x[1];  
        planet[i].x0.x[2] = xyz[0].x[2];  
      }  
  
      planet[i].v0 = new N6LVector(3);  
  
      //Calculating orbital velocity from Kepler's equations  
      var xyz2 = new Array(new N6LVector(3));  
  
      var xxx = new Array(new N6LVector(3));  
      planet[i].kepler(nday + (1.0 / (24.0 * 4.0) * planet[i].m_t), xxx);  
      var vv = xxx[0].Sub(xx[0]);  
      //Fine speed adjustment  
      planet[i].v0.x[0] = (vv.x[0] / (60.0 * 60.0 * 24.0 / (24.0 * 4.0) * planet[i].m_t) / planet[i].CNST_C) * planet[i].m_mv;  
      planet[i].v0.x[1] = (vv.x[1] / (60.0 * 60.0 * 24.0 / (24.0 * 4.0) * planet[i].m_t) / planet[i].CNST_C) * planet[i].m_mv;  
      planet[i].v0.x[2] = 0.0;  
  
      planet[i].ecliptic(planet[i].v0.x[0], planet[i].v0.x[1], planet[i].v0.x[2], xyz2);  
      if(isNaN(xyz2[0].x[0]) || isNaN(xyz2[0].x[1]) || isNaN(xyz2[0].x[2])) {  
        planet[i].v0.x[0] = 0.0;  
        planet[i].v0.x[1] = 0.0;  
        planet[i].v0.x[2] = 0.0;  
      }  
      else {  
        planet[i].v0.x[0] = xyz2[0].x[0];  
        planet[i].v0.x[1] = xyz2[0].x[1];  
        planet[i].v0.x[2] = xyz2[0].x[2];  
      }  
      mp[i] = new N6LMassPoint(planet[i].x0, planet[i].v0, planet[i].m_m, planet[i].m_r, planet[i].m_e);  
    }  
}  

//Planetary orbital segment settings  
function setline() {  
  var msecPerMinute = 1000 * 60;  
  var msecPerHour = msecPerMinute * 60;  
  var msecPerDay = msecPerHour * 24;  
  var a = new Date(1996,6,1,0,0,0);  
  var ndayR = 0;  
  var i;  
  var j;  
  var k;  
  var n = 32;  
  var str;  
  for(i = 0; i < planetnum; i++) {  
    str = "";  
    if(mp[i].mass <= 0.0) {  
      var rt = 0.001;  
      var base = 4;  
      var ofs = i * 0.1;  
      var x;  
      var y;  
      for(x = base, y = base; -base < x ; x--)  
        str += Number((x + ofs) * rt).toString() + " " + Number((y + ofs) * rt).toString() + ", ";  
      for(; -base < y ; y--)   
        str += Number((x + ofs) * rt).toString() + " " + Number((y + ofs) * rt).toString() + ", ";  
      for(; x < base ; x++)   
        str += Number((x + ofs) * rt).toString() + " " + Number((y + ofs) * rt).toString() + ", ";  
      for(; y < base ; y++)   
        str += Number((x + ofs) * rt).toString() + " " + Number((y + ofs) * rt).toString() + ", ";  
      str += Number((x + ofs) * rt).toString() + " " + Number((y + ofs) * rt).toString();  
      var elm;  
      var sp;  
  
      elm = document.getElementById(IDL[i]);  
      elm.setAttribute('lineSegments', new String(str));  
  
      elm = document.getElementById(IDT[i]);  
      sp = new x3dom.fields.SFVec4f(0, 1, 0, 0);  
      elm.setAttribute('rotation', sp.toString());  
      continue;  
    }  
    var x0;  
    //Divide the circumference of the planet into 32 lines  
    for(j = 0; j < n; j++) {  
      var ad = (360.0 * 360.0 / 365.2425 / planet[i].m_nperday) * (j / n);  
      var days = (dat.getTime() - planet[i].m_dat0.getTime()) / msecPerDay;  
      var nday = days + ad;  
      var xx = new Array(new N6LVector(3));  
      var f = planet[i].kepler(nday, xx);  
      var eps = Math.PI / 32;  
      if(Math.PI - eps < f && f < Math.PI + eps) ndayR = nday;  
      var x1 = new N6LVector(3);  
      x1.x[0] = xx[0].x[0];  
      x1.x[1] = xx[0].x[1];  
      x1.x[2] = 0.0;  
      if(j == 0) x0 = new N6LVector(x1);  
      str += (x1.x[1] / CNST_AU / Zoom).toString() + " " + (-x1.x[0] / CNST_AU / Zoom).toString() + ", ";  
    }  
    str += (x0.x[1] / CNST_AU / Zoom).toString() + " " + (-x0.x[0] / CNST_AU / Zoom).toString();  
  
    var ss = planet[i].m_s * planet[i].CNST_DR;  
    var ii = planet[i].m_i * planet[i].CNST_DR;  
    var ww = planet[i].m_w * planet[i].CNST_DR;  
  
    var vec = new N6LVector(3);  
    var mat = new N6LMatrix(3);  
    mat = mat.UnitMat().RotAxis(vec.UnitVec(2), ss).RotAxis(vec.UnitVec(1).Mul(-1.0), ii).RotAxis(vec.UnitVec(2), ww);  
    var VecWK = new N6LVector(4);  
    var MatWK = new N6LMatrix(4);  
    MatWK.x[0] = VecWK.UnitVec(0);  
    MatWK.x[0].bHomo = false;  
    for(k = 1; k < 4; k++) {  
      MatWK.x[k] = mat.x[k - 1].NormalVec().ToHomo();  
      MatWK.x[k].x[0] = 0.0;  
      MatWK.x[k].bHomo = false;  
    }  
    VecWK = MatWK.NormalMat().Vector();  
  
/*Comment out starting from aphelion  
    var a;  
    var b = 0;  
    var c;  
    for(a = 0; a < planetnum; a++)  
      if(0.0 < mp[a].mass){b++;c=a;}  
    if(b == 2 && i) {  
      var msecPerMinute = 1000 * 60;  
      var msecPerHour = msecPerMinute * 60;  
      var msecPerDay = msecPerHour * 24;  
      document.F1.myFormTIME.value = (ndayR + planet[c].m_dat0.getTime() / msecPerDay) / 365.2425;  
      dat = new Date(ndayR * msecPerDay + planet[c].m_dat0.getTime());  
    }  
*/  
    var elm;  
    var sp;  
  
    elm = document.getElementById(IDL[i]);  
    elm.setAttribute('lineSegments', new String(str));  
  
    elm = document.getElementById(IDT[i]);  
    sp = VecWK.ToX3DOM();  
    elm.setAttribute('rotation', sp.toString());  
  }  
}  
  
...  
  
Initialize N6LPlanet as above and reinitialize it as necessary.  
  
# stellite.js  
  
function init(b) {  
  Speed = Number(document.F1.SPD.value);  
  if(b < 0) Speed *= -1;  
  Zoom = Number(document.F1.ZOM.value);  
  if(Zoom < 0.0) Zoom *= -1.0;  
  
  var radioList = document.getElementsByName("PUTSEL");  
  var i;  
  for(i = 0; i<radioList.length; i++){  
      if(radioList[i].checked){  
          id = Number(radioList[i].value) + 1;  
          break;  
      }  
  }  
  var v = new N6LVector(3);  
  if(b) { ; }  
  else {  
    for(i = 0; i < planetnum; i++) {  
      mp[i] = new N6LMassPoint(v.ZeroVec(), v.ZeroVec(), -1, -1, -1);  
    }  
  }  
  
  var msecPerMinute = 1000 * 60;  
  var msecPerHour = msecPerMinute * 60;  
  var msecPerDay = msecPerHour * 24;  
  var days = eval(document.F1.myFormTIME.value) * 365.2425;  
  dat = new Date(days * msecPerDay);  
  setmp();  
  setline();  
  InitRelative();  
  setmp();  
  setline();  
}  
  
function InitRelative() {  
  var msecPerMinute = 1000 * 60;  
  var msecPerHour = msecPerMinute * 60;  
  var msecPerDay = msecPerHour * 24;  
  var days = eval(document.F1.myFormTIME.value) * 365.2425;  
  dat = new Date(days * msecPerDay);  
  time = dat.getTime();  
  PlanetInit(dat);  
  setline();  
  dt = Speed * 60 * 60;  
  //Mass Point Array construction  
  var pmp = new Array();  
  var i;  
  for(i = 0; i < planetnum; i++) pmp[i] = new N6LMassPoint(mp[i]);  
  //Initialized N6LRngKt
  rk.Init(pmp, dt);  
  settime(dat);  
}  
  
function onRunning() {  
  //Main Loop  
  UpdateFrameRelative();  
}  
  
function UpdateFrameRelative() {  
  var msecPerMinute = 1000 * 60;  
  var msecPerHour = msecPerMinute * 60;  
  var msecPerDay = msecPerHour * 24;  
  
  var dat1;  
  var tm = Math.abs(Speed) * msecPerDay / 1000;  
  var adt = Math.abs(dt);  
  var t;  
  var i;  
  if(dt != 0.0) {  
    for(t = adt; t <= tm; t += adt) {  
      time = time + dt * 1000;  
      //Update Mass Points  
      rk.UpdateFrame()  
  
      //Sun origin correction  
      for(i = 1; i < planetnum; i++) {  
        rk.mp[i].x = rk.mp[i].x.Sub(rk.mp[0].x);  
        mp[i].x = new N6LVector(rk.mp[i].x);  
      }  
      rk.mp[0].x = rk.mp[0].x.ZeroVec();  
    }  
    var datt = dat.getTime();  
    var dat1t = datt + time;  
    var dat1 = new Date(dat1t);  
    setmp();  
    settime();  
  }   
}  
  
The initialization and update of the N6LRngKt class is shown above.  


