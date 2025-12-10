# Swahiba_3Laminar — 2D Laminar DNS of Transverse Hydrogen Jet in Crossflow  
**OpenFOAM v13** | Finite-rate chemistry: `2H₂ + O₂ → 2H₂O`  

## Problem Overview
Direct Numerical Simulation (DNS) of a **2D laminar transverse hydrogen jet** injected into a subsonic air crossflow with **finite-rate Arrhenius chemistry**. This case serves as a precursor to full validation against the supersonic experiment of [Gamba & Mungal (2015)](https://doi.org/10.1017/jfm.2015.475).

> **Motivation**: Build confidence in solver setup (`multicomponentFluidFoam`), chemistry implementation, and mesh resolution before targeting the high-enthalpy supersonic regime.



## Physical Parameters (Laminar Case)

| Quantity | Value | Notes |
|---------|-------|-------|
| **Jet** | H₂, 400 m/s, 300 K, *d* = 2 mm | Subsonic: *Ma*ⱼ ≈ 0.31 |
| **Crossflow** | Air, 100 m/s, 300 K | |
| **Momentum ratio** | *J* = ρⱼ*u*ⱼ² / ρ∞*u*∞² ≈ **1.12** (Eq. 10) | ρⱼ ≈ 0.081 kg/m³, ρ∞ ≈ 1.161 kg/m³ |
| **Reynolds number** | *Re* = ρⱼ*u*ⱼ*d* / μⱼ ≈ **3500** (Eq. after 3.1.2) | μⱼ from Sutherland: *Aₛ* = 1.672×10⁻⁶, *Tₛ* = 170.672 K |
| **Initial field** | *T* = 2000 K, *Y*ₙ₂ = 1, **u** = 0, *p* = 10⁵ Pa | Ignition-assist (quiescent, pure N₂) |
| **Domain** | 153 mm × 155 mm | Jet centered at *x* = 63 mm |
| **Mesh** | ~124,000 structured 2D hex cells | Fig. 2b |



## Chemistry Model

### Reaction
\[
\ce{2H2 + O2 -> 2H2O} \quad \text{(irreversible)}
\]

### Arrhenius Rate Constant
\[
k_f = A \exp\left(-\frac{T_a}{T}\right), \quad
A = 1.0 \times 10^{14}  \text{m}^3\text{/mol/s}, \;
\beta = 0, \;
T_a = 8000  \text{K} \quad \text{(Eq. 6)}
\]

### Molar Reaction Rate
\[
\dot{r} = k_f \left(\frac{\rho Y_{\ce{H2}}}{W_{\ce{H2}}}\right)^2 \left(\frac{\rho Y_{\ce{O2}}}{W_{\ce{O2}}}\right)
= \frac{A \exp(-T_a/T)}{W_{\ce{H2}}^2 W_{\ce{O2}}} \rho^3 Y_{\ce{H2}}^2 Y_{\ce{O2}}
\]

### Mass Production Rates (kg·m⁻³·s⁻¹)  
From *Progress Report 2*, Eqs. (7)–(9):
\[
\begin{aligned}
\dot{\omega}_{\ce{H2}} &= -2 \rho^2 k_f \frac{Y_{\ce{H2}}^2 Y_{\ce{O2}}}{W_{\ce{H2}}^2 W_{\ce{O2}}} \cdot W_{\ce{H2}} \\
\dot{\omega}_{\ce{O2}} &= -   \rho^2 k_f \frac{Y_{\ce{H2}}^2 Y_{\ce{O2}}}{W_{\ce{H2}}^2 W_{\ce{O2}}} \cdot W_{\ce{O2}} \\
\dot{\omega}_{\ce{H2O}} &= +2 \rho^2 k_f \frac{Y_{\ce{H2}}^2 Y_{\ce{O2}}}{W_{\ce{H2}}^2 W_{\ce{O2}}} \cdot W_{\ce{H2O}} \\
\dot{\omega}_{\ce{N2}} &= 0
\end{aligned}
\]

Molecular weights (kg/kmol):  
\( W_{\ce{H2}} = 2.016 \), \( W_{\ce{O2}} = 32 \), \( W_{\ce{H2O}} = 18 \), \( W_{\ce{N2}} = 28 \)


## Numerical Setup (OpenFOAM v13)

- **Solver**: `multicomponentFluidFoam` (PIMPLE algorithm)  
- **Governing equations**: Compressible multicomponent Navier–Stokes + species + finite-rate chemistry (Eqs. 1–5)  
- **Time discretization**: **Implicit Euler** (1st-order, Eq. 12)  
- **Spatial discretization**:
  - Gradients: `Gauss linear` (2nd-order, Sec. 4.2)  
  - Convection: `limitedLinear` (2nd-order smooth, 1st-order near extrema, TVD)  
  - Diffusion: `Gauss linear orthogonal` (2nd-order, Sec. 4.2)  
- **Thermodynamics**: Sensible enthalpy *hₖ(T)* from JANAF polynomials (Eq. 3)  
- **Transport**: Sutherland viscosity for H₂:  
  \[
  \mu = A_s \sqrt{T} / \left(1 + T_s/T\right),\; A_s = 1.672 \times 10^{-6},\; T_s = 170.672  \text{K}
  \]

## Case Structure
├── 0/ # Initial fields (T=2000 K, YN2=1, u=0, p=1e5)
├── constant/
│ ├── thermophysicalProperties
│ ├── chemistryProperties
│ ├── transportProperties
│ └── polyMesh/ # Mesh files
├── system/
│ ├── controlDict
│ ├── fvSchemes
│ └── fvSolution
├── Allrun # Run script
├── Allclean # Cleanup script
├── 2DJISCFMeshv212file.msh Fluent mesh
└── README.md


## How to Run

# Run
./Allrun

## How to clean
./Allclean

# or manually:
rm -rf [0-9]* log.*
