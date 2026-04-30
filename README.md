# Project-Codes-Final
The final collection of codes used in my 3rd year project
# Single-Diode Photovoltaic Model with NASA Climate Data and Floating PV Cooling Analysis
# Introduction
This repository contains the MATLAB and Simulink files developed for my Third-Year
Individual Project at the University of Manchester
The project uses a single-diode model of the Shell SP150 photovoltaic module in Simulink, 
validated against the manufacturer datasheet, and applies this to
NASA POWER climate data for three locations (Furnace Creek, Pontianak, Oslo) to
find temperature-induced energy losses. Two floating photovoltaic (FPV) cooling
configurations are then applied using the Faiman thermal model and evaluated.

The corresponding written report is submitted separately on Canvas.

# Project context
The code in this repository implements the methodology described in Section 2 of the
report. The main components are:

1. A Simulink model of the single-diode equation including parasitic resistances,
   temperature-dependent reverse saturation current (via the Varshni bandgap
   relation), and irradiance-dependent photocurrent.
2. Parameter-optimisation scripts that fit Rs, I0_ref and n against datasheet IV
   curves at varying temperatures and irradiances.
3. RMSE validation scripts that quantify model accuracy against the SP150 datasheet.
4. Time-series simulation scripts that apply NASA POWER data to the validated model
   to compute power, efficiency, and cumulative energy across a representative day.
5. FPV thermal model scripts implementing the Faiman heat-transfer equation for two
   cooling configurations.

   ## Software requirements
- MATLAB R2025b or later 
- Simulink
- Standard MATLAB toolboxes only — no specialist toolboxes required

The model uses the Simulink `Algebraic Constraint` block to resolve the implicit
single-diode current equation. No third-party Simulink libraries are used.

# How to run
The codes are designed to be run in any order, simply copy them into the MATLAB command window. If you wish to swap between june and december just replace all relevant words.
As long as the SP150 model is available in the MATLAB Drive, these codes will work. 
If this code were to be applied to a different PV module, just change the modelname = "X"; line

# 1. Parameter optimisation

Each CODE performs a two-stage grid search (coarse, then fine) to minimise RMSE
between the Simulink output and the corresponding datasheet IV curve. The objective
function for the temperature sweep includes a Voc penalty term weighted at 0.25
alongside RMSE, to enforce open-circuit-voltage agreement at high temperatures.

# 2. Validation

```matlab
% Once the lookup tables are populated:
run('RMSE_all_curves.m')           % produces Table 3.1.2 in the report
```

This computes RMSE across all five temperature curves and all five irradiance
curves and prints two tables to the command window.

# 3. Time-series simulation
Each time-series code loads the relevant `<location>_<date>_AVERAGE.xlsx` file,
which contains 24-hour averaged irradiance (W/m²) and ambient temperature (°C) for
that location and date. Cell temperature is derived from the Ross/NOCT relation
T_cell = T_amb + ((NOCT − 20)/800) · G with NOCT = 45 °C, then passed to the
Simulink model alongside G to compute the maximum power point at each hour.

# 4. Thermal analysis
# Model parameters
The validated model uses the following parameters (Section 2.4 of the report):

| Parameter            | Symbol  | Value                       | Source                  |
|----------------------|---------|-----------------------------|-------------------------|
| Short-circuit current| I_sc    | 4.8 A                       | SP150 datasheet         |
| Open-circuit voltage | V_oc    | 0.603 V (per cell)          | SP150 datasheet         |
| Series resistance    | R_s     | Variable, lookup table      | Fitted via optimisation |
| Shunt resistance     | R_sh    | 1×10⁷ Ω                     | Fitted via optimisation |
| Ideality factor      | n       | Variable, lookup table      | Fitted via optimisation |
| Bandgap energy       | E_g0    | 1.12 eV                     | Varshni 1967 [1.9]      |
| Reference saturation | I_0,ref | Variable, lookup table      | Fitted via optimisation |
| Cells in series      | N_s     | 72                          | SP150 datasheet         |


# Technical details/ Equations in the model

# Single-diode equation (implicit form)
I = I_ph(G,T) − I_0(T) · [exp((V + I·R_s)/(n·V_T))−1] − (V + I·R_s)/R_sh
resolved in Simulink via an algebraic constraint with initial guess I = 3.8 A
(I_sc at NOCT). Simulation stop time = 100 s to capture the full voltage sweep.

# Temperature-dependent reverse saturation current
I_0(T) = I_0,ref · (T/T_ref)³ · exp[(E_g/k)·(1/T_ref − 1/T)]

# Varshni bandgap relation
E_g(T) = E_g0 − α·T² / (β + T)
with α = 4.73×10⁻⁴, β = 636 (Varshni 1967, [1.9] in report).

# Cell temperature (land PV, default)
T_cell = T_amb + ((NOCT − 20)/800) · G        with NOCT = 45 °C [Ross/NOCT]

# Cell temperature (FPV, Faiman model)
T_m = T_amb + G / (U_0 + U_1 · v)
where U_0= and U_1=

with constant wind speed v = 1 m/s and the two coefficient sets defined in
Section 2.8 of the report.

 Data sources
- SP150 datasheet IV curves digitised using WebPlotDigitizer (Rohatgi 2023).
- Hourly irradiance and ambient temperature obtained from the NASA POWER
  database (https://power.larc.nasa.gov/data-access-viewer/), averaged across
  2004–2024 for each location on 21 June and 21 December.

# Known issues and limitations
- The Faiman heat-transfer coefficients used for the FPV cases are not validated
  against measured FPV data, they are adapted from typical land-based values
  reported by Faiman (2008). Quantitative FPV results should be interpreted as
  indicative of the cooling mechanism not absolute predictions.
- Constant wind speed of 1 m/s is assumed for the FPV analysis to isolate
  passive cooling effects.
- The single-diode model does not capture recombination effects relevant in
  partial-shading or very-low-irradiance conditions; a double-diode model would
  be more accurate in those regimes.
- Cell temperature is treated as spatially uniform across the module surface.
- The Trina Vertex N module was initially considered but its datasheet did not
  provide temperature and irradiance dependent IV curves; the SP150 was selected
  for validation purposes which is an older model (see report Section 2.3).

# Possible future improvements

- Replace constant wind speed with hourly NASA POWER wind data.
- Extend simulation to a full annual cycle rather than just two days.
- Implement a control algorithm to maintain NOCT-equivalent operation through active
  cooling.
- Validate the Faiman coefficients against an experimental FPV setup.
