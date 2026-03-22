### Mathematical Design Tools
The coil design will fundamentally be a:
	 'thin walled finite length solenoid'

The primary mathematical tools used will be:

1. Biot-Savart

$$
B(0) = \mu_0 \, \frac{N I}{\ell} \,
       \frac{\tfrac{\ell}{2}}
            {\sqrt{r^2 + \left(\tfrac{\ell}{2}\right)^2}}
$$
2. Wheeler Inductance approx.

$$
L_{\mu H} = \frac{r^2 N^2}{9r + 10\ell}
$$
3. Inductive and capacitive Energy storage

$$
E_L = \tfrac{1}{2}\,L\,I^2
$$
$$
E_C = \tfrac{1}{2}\,C\,V^2
$$
4. Derived Optimal aspect ratio (Coil Length to Coil Radius) using Wheeler and Bio-Savart 

	 Solve Bio-Savart for required peak current (I) to achieve a target magnetic Field (B)
$$
I_{\text{req}}(r,\ell) \;=\;
\frac{2B_0}{\mu_0 N}\,\sqrt{\,r^2 + \Big(\tfrac{\ell}{2}\Big)^2\,}
$$
	Wheeler in Henry's

$$
L_{\mathrm{H}}(r,\ell) \;=\; 10^{-6}\,\frac{r^2 N^2}{9r + 10\ell}
$$
	Stored energy in an inductor

$$
E_L = \tfrac{1}{2}\,L\,I^2
$$
	Substitute current, and Inductance with Bio-Savart and Wheeler

$$
E(r,\ell) \;=\; \tfrac{1}{2}\,L_{\mathrm{H}}(r,\ell)\,I_{\text{req}}^{\,2}(r,\ell)
$$
	Drop Constants - retain only geometrically relevant terms. We are only interested in understanding how the change in geometry of the coil design will affect the performance of the coil. Differentiate with respect to length, and set to zero. 
	
	We will have an expression for the rate of change of energy, in terms of the geometric properties of a coil as defined by Biot-Savart and Wheeler - at the point where length and width will disturb the energy the least amount.
$$
f(\ell) = \frac{r^2 + \ell^2/4}{9r + 10\ell},
\qquad
\frac{df}{d\ell} =
\frac{(\ell/2)(9r + 10\ell) - (r^2 + \ell^2/4)\cdot 10}{(9r + 10\ell)^2} = 0
$$
	Take numerator equal to zero and simplify
$$
\frac{\ell}{2}(9r + 10\ell) - 10r^2 - \frac{10\ell^2}{4} = 0
$$
$$
\frac{9}{2} r\ell + 5\ell^2 - 10r^2 - 2.5\ell^2 = 0
$$
$$
9r\ell + 5\ell^2 - 20r^2 = 0.
$$
	 Divide by radius squared and let x = l/r to build a polynomial
$$
5x^2 + 9x - 20 = 0.
$$
	 Solve quadratic and take positive root value
$$
x = \frac{-9 + \sqrt{9^2 - 4\cdot 5 \cdot(-20)}}{2\cdot 5}
  = \frac{-9 + \sqrt{481}}{10}
  \approx 1.293.
$$
	 taking x = l/r - The result gives:
$$
\boxed{\;\ell \;\approx\; 1.293\,r\;}
$$
This represents an optimal relationship between coil geometry and energy 

This relationship will be used to define the coil geometry for this design.

5. Adiabatic temp rise for short duration pulse
$$
\Delta T
  = k\,J^2\,t_p
  = k \left(\frac{I}{A}\right)^{\!2} t_p
$$

	where 
$$
k \approx 4.99\times10^{-15}\;
\frac{\text{s·m}^3}{\text{A}^2·°\text{C}}.
$$
$$
t_p = \frac{\pi}{2\omega} = \frac{\pi}{2}\sqrt{L\,C}.
$$


### Octave Code
```
% Pulsed Power Coil Geometry Optimization
%
%#######################################################################################################
%
% Solenoidal Coil Paramters B = (u * N * I) / (2 *   sqrt(r^2 + (L/2)^2))  {Biot Sivart}

% Constants
B = 0.1; % Field strength
u = (4*pi)*1e-7; % Permeability in air
N = 100; % Turn count


coil_length_array_in = linspace(0.5,2); % inches
coil_radius_array_in = linspace(0.5,1.5); % inches
coil_length_array_m = linspace(0.5,2)*0.0254; % inches
coil_radius_array_m = linspace(0.5,1.5)*0.0254; % inches

[coil_length_m, coil_radius_m] = meshgrid(coil_length_array_m, coil_radius_array_m);
[coil_length_in, coil_radius_in] = meshgrid(coil_length_array_in, coil_radius_array_in);


% peak current
Peak_current_func = @(x,y) (2*B*sqrt((y^2)+((x/2)^2)))/(u*N);
Peak_current = arrayfun(Peak_current_func, coil_length_m, coil_radius_m);

% Inductance
L_coil_func = @ (x,y) (1e-6)*((y^2)*(N^2))/(9*y + 10*x);
L_coil = arrayfun(L_coil_func, coil_length_in, coil_radius_in);

% Energy
E_func = @(I,L) 0.5*L*(I^2);
Energy = arrayfun(E_func, Peak_current, L_coil);

% Wire Radius
delta_Temp = 15;
V = 500;
k = 4.99*1e-15;
Wire_radius_func = @(a,b,c) (1/0.0254)*sqrt(a/(pi*sqrt(delta_Temp/(k*0.5*pi*sqrt(b*(2*c/(V^2)))))));
Wire_radius = arrayfun(Wire_radius_func, Peak_current, L_coil, Energy);
MaxRadius = max(Wire_radius);

% Max wire radius = 0.0029118 inches or 0.006 in diameter.
% Using MWS wire 'Heavy Build' - 35AWG - 0.007 in diameter (max)
% MWS AWG35 Bare copper Diameter: 0.0056in
AWG35_BareCuRadius_in = 0.0056/2
AWG35_BareCuRadius = AWG35_BareCuRadius_in*0.0254;
Wire_Copper_Area = pi*AWG35_BareCuRadius^2;


%Capacitor bank capacitance
C_bank_func = @(a) (2*a)/(V^2);
C_bank = arrayfun(C_bank_func, Energy);

% Thermal Pulse length
t_thermal_func = @(a,b) (pi/2)*sqrt(a*b);
t_thermal = arrayfun(t_thermal_func, C_bank, L_coil);

% Delta Temp
Delta_temp_max = 15;
Delta_temp_func = @(a,b) k*((a/Wire_Copper_Area)^2)*b;
Delta_temp = arrayfun(Delta_temp_func, Peak_current, t_thermal);

% Temperature Range min and max 
dTmin = min(Delta_temp(:));
dTmax_data = max(Delta_temp(:));
printf("DeltaT range: %.3g .. %.3g °C\n", dTmin, dTmax_data)

Optimal_Length = 1.293.*coil_radius_array_in;

figure(1);
clf;
contourf(coil_radius_in, coil_length_in, Energy, 20, 'LineColor', 'none');
colorbar;
hold on;
contour(coil_radius_in, coil_length_in, Delta_temp, [Delta_temp_max Delta_temp_max], 'k:');
hold on;
plot(coil_radius_in, Optimal_Length, 'r:');
hold off;


xlabel('Radius (in)');
ylabel('Length (in)');
title('Energy Contour');
colormap('turbo');
legend({'Energy (J)','\DeltaT = 15^\circC','L \approx 1.293 R'}, 'Location','northwest');



```


### Design Results

The coil will be wound on a PLA bobbin (glass transition temperature ≈ 55 °C). Ambient laboratory temperature is ≈ 25 °C, providing an approximate allowable temperature rise of 30 °C. To retain margin for future pulsed-duty operation, the design will limit temperature rise to
$$\Delta T_{\text{max}} = {15}{\celsius}$$
For short-duration pulses, the adiabatic temperature rise is estimated as

$$ \Delta T
  = k\,J^2\,t_p
  = k \left(\frac{I}{A}\right)^{\!2} t_p$$

and will be used to determine the minimum conductor cross-sectional area required to maintain the temperature rise constraint.

$$ A = \frac{I}{\sqrt{\frac{\Delta T}{kt_p}}}
$$
Using the minimum cross sectional area, the calculation of the minimum wire radius helps select an appropriate wire gauge for a prototype build. 

$$ A = \pi r^2 
$$
$$ r = \sqrt{\frac{A}{\pi}}
$$
For this build, the octave script above yields a minimum wire radius of  0.0029118 inches or ~0.006 inch diameter. I will select a 35AWG wire gauge - this wire has diameter 0.007inch and will provide slight margin to keep the coil cooler during pulsed operation.

PLA is used here due to availability; material choice may be revisited in later iterations if thermal performance becomes a limiting factor.


![[Energy_vs_CoilDimensions.png]]
Figure 1. - 3D plot of coil geometry vs. Energy. 

The plot in Figure 1 suggests a compact design requires minimal energy to produce a magnetic field equal to the targeted 100mT.

![[Thermal_vs_EnergyContourMap 2.png]]
Figure 2. Contour plot of energy vs coil geometry with overlay of optimal aspect ratio (Red) and thermal boundary where coil energy will produce 15 °C temp rise.


Flattening Figure 1, and overlaying the result of 
$$
\boxed{\;\ell \;\approx\; 1.293\,r\;}
$$
and the boundary where the energy in the system will generate a temperature rise of 15 °C leads to Figure 2. 

Figure 2 is used to define the coil geometry of the first iteration design. The geometry will be selected such that the radius and length pair lie somewhere on the red line, and to the left of the green contour. The range of options available are not limited by thermal rise, and are therefore a geometric / mechanical constraint. Due to this being the first iteration, I will select a radius large enough to include a concentric secondary winding inside the primary coil. This secondary winding will serve as a means to 1. measure the magnetic field produced by the primary coil, and 2. simulate a fusion reaction induced magnetic field pushing energy 'into' the primary coil to be recovered by additional energy recovery circuitry. 

The selected coil geometry will be Radius: 1in, Length: 1.3in, Wire: 35AWG, n: 100 turns

This selection gives the following calculated system characteristics:
Energy: 0.53 J
Coil inductance: 454.55uH 
Capacitor bank capacitance: 4.23uF
Peak pulse current: 48.22 A
Pulse Length: 137.7uS
10Hz RMS current: 1.79 A
Approx wire length required: 54.36 ft.