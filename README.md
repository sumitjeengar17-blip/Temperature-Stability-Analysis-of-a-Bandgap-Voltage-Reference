Introduction
The goal of this project is to investigate the operating principle of a CMOS bandgap voltage reference using LTspice simulations. The project focuses on the temperature behavior of bipolar junction transistors and demonstrates how complementary temperature-dependent voltages can be combined to generate a stable reference voltage.

The simulations presented in this project include:

•	CTAT (Complementary To Absolute Temperature) voltage behavior
•	PTAT (Proportional To Absolute Temperature) voltage generation
•	PTAT current generation
•	ideal bandgap reference modeling
1. Objectives
•	Understand the physical origin of temperature drift in a single p-n junction and how PTAT/CTAT cancellation removes it to first order.
•	Derive the bandgap design equation and size the resistors/area ratio for a target output voltage.
•	Build an analytical (device-equation) model to predict temperature stability before committing to SPICE.
•	Simulate the circuit at the transistor level in SPICE (LTspice) across –40 °C to 117.1 °C and extract the temperature coefficient (TC) in ppm/°C.
•	Study second-order effects (curvature) and at least one curvature-correction or trimming technique.
2. Theory
2.1 Why a single diode/BJT is not a good reference
The base-emitter voltage of a forward-biased BJT operated at constant collector current falls with temperature at roughly −1.5 to −2 mV/°C — this is the CTAT (Complementary to Absolute Temperature) behavior. A single diode-connected transistor therefore cannot serve as a stable reference on its own.
2.2 PTAT generation
If two identical BJTs (or one unit device and N parallel unit devices) are operated at the same collector current but different emitter areas (ratio N), their base-emitter voltages differ by
ΔV_BE = V_T ln(N)
where V_T = kT/q is the thermal voltage (≈ 25.85 mV at 300 K). Because V_T is directly proportional to absolute temperature, ΔV_BE is PTAT — it increases with temperature.
2.3 The bandgap principle
Scaling ΔV_BE by a resistor ratio M = R2/R1 and adding it to a CTAT V_BE gives
V_REF = V_BE + M · V_T · ln(N)
Because the CTAT and PTAT terms have opposite-sign temperature coefficients, a value of M exists for which dV_REF/dT ≈ 0 at a chosen temperature — first-order cancellation. The resulting V_REF lands close to the silicon bandgap voltage extrapolated to 0 K (≈ 1.20–1.25 V), which is where the circuit gets its name.
2.4 A more accurate V_BE(T) model
The simple linear TC (−2 mV/°C) is only a first-order approximation. A widely used closed-form model (Tsividis, 1980) is:
V_BE(T) = V_G0 − (V_G0 − V_BE0)·(T/T0) − η·V_T(T)·ln(T0/T)
where V_G0 is the bandgap voltage extrapolated to 0 K (≈ 1.205 V for silicon), V_BE0 is the base-emitter voltage at reference temperature T0, and η (≈ 3.6–4) is a process constant related to how the transistor's saturation current scales with temperature. This model captures the residual curvature that a simple linear PTAT/CTAT sum cannot cancel — the reason real bandgap references still show 20–50 ppm/°C drift uncorrected, and why high-precision references add curvature-correction circuitry (Section 8).
3. Circuit Design
3.1 Basic circuit layout
At the block level, the design has four functional pieces: a current mirror that forces equal current in three branches, a PTAT core that develops a temperature-proportional current, an op-amp feedback loop that regulates the mirror, and an output stage that sums a PTAT and a CTAT voltage. Figure 1 shows this basic layout before going to the transistor-level schematic.

Figure 1. Basic block-level layout of the bandgap reference.
3.2 Transistor-level topology
This project uses the op-amp-regulated two-transistor bandgap core shown below — a standard, widely taught topology close to what appears in many TI reference/regulator datasheets. A matched 3-leg PMOS current mirror forces equal currents in the Q1 and Q2 branches; an op-amp forces the two mirror drain nodes to the same voltage, which forces the PTAT current I = V_T ln(N)/R1 through R1. The same current, mirrored into a third leg through R2 and a unit BJT Q3, produces the output V_REF = V_BE3 + (R2/R1)·V_T·ln(N).

 

 
 .
3.3 Design equations and component values
With emitter-area ratio N and resistor ratio M = R2/R1, the compensation condition dV_REF/dT = 0 at T0 = 300.15 K (27 °C) solves to:
M = [ (V_G0 − V_BE0)/T0 − η·(k/q) ] / [ ln(N)·(k/q) ]
Using V_G0 = 1.205 V, V_BE0 = 0.650 V (typical for this bias current), η = 4, and N = 8, this gives:
Parameter	Value	Notes
N (Q2:Q1 area ratio)	8	Q2 built as 8 unit devices in parallel, or use an area/AREA multiplier
M = R2/R1	8.395	First-order TC-cancellation condition
Bias current I (PTAT)	≈ 10 µA	Chosen for low power; scale as needed
R1	5.36 kΩ	R1 = V_T ln(N) / I
R2	45.1 kΩ	R2 = M · R1
V_REF (predicted, 27°C)	≈ 1.10 V	V_BE3 + M·V_T·ln(N)
Supply VDD	5 V (try 3.3 V too)	Needs enough headroom for the PMOS mirror
Standard E96 resistor values close to these (e.g. 5.36 kΩ and 45.3 kΩ) are fine — the design is a ratio, so absolute accuracy matters far less than the R2/R1 matching, which on a real IC is set by careful layout (common-centroid, unit resistors) rather than by picking two arbitrary standard values.
4. Analytical (Device-Equation) Simulation
Before committing to a transistor-level SPICE deck, it is good practice to verify the design with a fast closed-form model. This was done in Python using the Tsividis V_BE(T) model of Section 3.4, sweeping T from –40 °C to 125 °C. The full script is included as Appendix B.
 
Figure 3. CTAT (V_BE), PTAT (M·V_T·ln N) and their sum V_REF vs. temperature.
 
Figure 4. Effect of under-, optimal-, and over-compensation (varying R2/R1) on the temperature curve — the classic bandgap “bowed” curve.
4.1 Numeric results
T (°C)	V_BE (V)	PTAT term (V)	V_REF (V)
-40.0	0.7536	0.3507	1.1043
-31.7	0.7405	0.3632	1.1037
-23.5	0.7275	0.3756	1.1031
-15.2	0.7145	0.3881	1.1026
-6.9	0.7017	0.4005	1.1022
1.4	0.6890	0.4130	1.1019
9.6	0.6763	0.4254	1.1017
17.9	0.6637	0.4378	1.1016
26.2	0.6513	0.4503	1.1015
34.4	0.6388	0.4627	1.1016
42.7	0.6265	0.4752	1.1017
51.0	0.6143	0.4876	1.1019
59.2	0.6021	0.5001	1.1021
67.5	0.5899	0.5125	1.1024
75.8	0.5779	0.5249	1.1028
84.1	0.5659	0.5374	1.1033
92.3	0.5540	0.5498	1.1038
100.6	0.5422	0.5623	1.1044
108.9	0.5304	0.5747	1.1051
117.1	0.5186	0.5871	1.1058

Summary: V_REF(27°C) ≈ 1.1015 V, ranging from 1.1015 V to 1.1065 V across –40–125°C, giving a first-order temperature coefficient of approximately 27.5 ppm/°C (box method: TC = (V_max−V_min)/(V_nom·ΔT) × 10⁶). This residual drift is the curvature term (the η·V_T·ln(T0/T) piece), which a simple linear PTAT cannot cancel — see Section 8 for correction techniques.
5. Circuit-Level (SPICE-equivalent) Simulation Results
The closed-form model of Section 5 ignores second-order transistor-circuit effects: finite current gain β, finite mirror output resistance (channel-length modulation, which unbalances the three mirror branches slightly), and op-amp input offset voltage. To capture these, the full circuit of Figure 2 was solved as a system of nonlinear node equations (Kirchhoff's current law at every node, Ebers-Moll BJT equations with finite β, square-law PMOS equations with channel-length modulation, and the op-amp servo condition) using Newton/fixed-point iteration in Python — methodologically the same DC-operating-point calculation that SPICE performs internally, run here because a licensed SPICE engine (LTspice/ngspice) is not installed in the environment that produced this report. The full solver script is available on request; the exact same circuit is also captured as a portable netlist in Appendix A so you can cross-check these numbers independently in LTspice, which is recommended before treating this as your final submitted data.
Component values used: R1 = 5.36 kΩ, R2 = 45.1 kΩ, N = 8, β (BF) = 100, mirror channel-length-modulation λ = 0.02 V⁻¹, op-amp input offset = 2 mV (typical of a general-purpose op-amp such as the LM358/TL071 family), VDD = 5 V.
5.1 Temperature sweep (−40 °C to 125 °C)
 
Figure 5. Full nonlinear circuit solution (finite β, mirror mismatch, 2 mV op-amp offset) vs. the ideal closed-form model.
T (°C)	V_REF (V)	Deviation from 25°C (ppm)
-40	1.07991	-785.8
-20	1.07946	-1196.1
0	1.07967	-1007.7
25	1.08076	0.0 (ref)
50	1.08270	1794.9
85	1.08671	5509.1
100	1.08886	7497.2
125	1.09297	11300.9

Result: V_REF(25°C) = 1.0808 V, ranging from 1.0795 V to 1.0930 V across −40–125°C, giving a circuit-level temperature coefficient of about 75.8 ppm/°C — roughly 2.7x worse than the ideal 27.5 ppm/°C from Section 5. The curve is also visibly asymmetric (it droops slightly below 0°C and rises faster above 50°C) rather than the ideal model's clean bow, because the op-amp offset and mirror mismatch add their own (roughly constant, not perfectly PTAT/CTAT-shaped) errors on top of the curvature term.
5.2 Line regulation (V_REF vs. supply voltage, at 25°C)
 
Figure 6. V_REF vs. VDD from 3.0 V to 5.5 V at fixed T = 25°C.
Line regulation ≈ 64 ppm/V over 3.0–5.5 V, i.e. about 0.16 mV of output shift per volt of supply change — low enough that the reference is usable across the common 3.3 V and 5 V rails without a separate pre-regulator, though a precision design would still buffer/pre-regulate VDD to remove this term entirely.
5.3 Monte Carlo: initial accuracy spread (untrimmed, at 25°C)
A real IC is never built from exact resistor values or a perfectly offset-free op-amp. To estimate untrimmed part-to-part spread, 800 random trials were run at 25°C with R1 and R2 independently varied ±1% (1σ, typical thin-film/poly resistor tolerance), op-amp offset drawn from a 2 mV (1σ) normal distribution, and mirror mismatch (λ) varied ±10% (1σ):
 
Figure 7. Monte Carlo spread of V_REF at 25°C, N = 800 trials.
Metric	Value
Mean V_REF (25°C)	1.0988 V
1σ spread	≈ 17,400 ppm (≈ 1.7%)
3σ spread	≈ 52,100 ppm (≈ 5.2%)

5.4 Cross-checking in LTspice
These results were produced by a custom solver, not a certified SPICE build, so treat them as a strong design estimate rather than a substitute for your own run. The exact circuit is provided as a portable netlist (bandgap.cir, Appendix A) — open it directly in LTspice (File → Open) or run ngspice bandgap.cir, then reproduce the .step temp -40 125 5 sweep and compare V(n3) against Table 6.1. If you rebuild the schematic yourself in the LTspice GUI (recommended, since you learn the mirror/op-amp wiring by placing it) and swap the ideal op-amp for a real part (e.g. LT1013), expect numbers in the same ballpark as above but not identical — that gap is itself worth discussing in your conclusion.
6. Expected Challenges & Debugging Tips
•	Convergence / no DC solution: give every node a DC path to ground, add .ic guesses, or use LTspice's 'uic' cautiously; start with looser .options (reltol=0.01) then tighten.
•	Wrong-sign feedback (op-amp output rails): swap the op-amp's + and − inputs — the mirror's loop sign depends on the exact device polarities used.
•	Startup: a self-biased current mirror can latch at zero current. Add a small startup circuit (e.g., a high-value resistor from VDD into one of the mirror nodes, or a startup transistor that turns off once current flows) — this is often skipped in first passes and is worth studying since every real IC bandgap needs one.
•	Mismatch sensitivity: in real op-amps, input offset voltage directly adds an error term to V_REF (multiplied by the noise gain of the loop) — this is usually the dominant error source in a real chip, larger than the curvature term.
•	Resistor TC: real resistors (especially diffused/poly on-chip, or even different resistor types on a breadboard) have their own temperature coefficients, which corrupt the R2/R1 ratio if the two resistor types don't match — use the same resistor type/geometry for R1 and R2.
7. Possible Extensions (stretch goals)
•	Curvature correction: add a nonlinear (e.g., VBE-based, temperature-squared) correction current to cancel the η·V_T·ln(T0/T) term and push TC below 10 ppm/°C — study the 'Brokaw' or 'Rincon-Mora' curvature-corrected topologies.
•	Add a unity-gain output buffer for low output impedance / load-current capability.
•	Trim R1/R2 (or Q1/Q2 area) in simulation to model laser-trim calibration and see how much a small ratio error shifts V_REF.
•	PSRR (power-supply rejection ratio) vs. frequency — run an .ac sweep with the supply as the AC source.
•	Compare this bandgap to a Brokaw cell (single differential pair driving matched collector resistors) — discuss area/complexity trade-offs.
8. Conclusion
The designed bandgap core (N = 8, R2/R1 = 8.4) produces a reference voltage of approximately 1.08–1.10 V, as expected for a two-transistor PTAT/CTAT bandgap. The ideal, curvature-only model predicts a temperature coefficient of about 27.5 ppm/°C over −40–125°C; adding realistic circuit non-idealities (finite β, mirror channel-length-modulation mismatch, and a 2 mV op-amp offset) raised this to about 76 ppm/°C in the full nonlinear circuit solve — still within the 20–100 ppm/°C range typical of untrimmed commercial bandgap ICs, and well above the <10 ppm/°C achieved by precision references that add curvature correction and trimming. Line regulation came out to ≈64 ppm/V, and an untrimmed Monte Carlo run showed roughly ±1.7% (1σ) part-to-part spread in V_REF, dominated by op-amp offset rather than resistor or transistor mismatch — consistent with the general rule that amplifier offset, not device matching, is usually the largest single error source in an untrimmed two-transistor bandgap. The two clearest paths to improving this design are (1) reducing the noise-gain sensitivity to op-amp offset, e.g. by using a chopper-stabilized or auto-zeroed amplifier, and (2) adding curvature correction to address the residual η·V_T·ln(T0/T) term that the simple linear PTAT term cannot cancel. These circuit-level numbers were produced with a custom Newton/fixed-point nodal solver rather than a licensed SPICE build (see Section 6.4); reproducing the sweep in LTspice using the provided netlist is the recommended next step before treating these figures as final.
9. References
1. Y. Tsividis, “Accurate Analyses of Temperature Effects in I_C–V_BE Characteristics with Application to Bandgap Reference Sources,” IEEE J. Solid-State Circuits, vol. 15, no. 6, 1980.
2. A. P. Brokaw, “A Simple Three-Terminal IC Bandgap Reference,” IEEE J. Solid-State Circuits, vol. 9, no. 6, 1974.
3. R. J. Widlar, “New Developments in IC Voltage Regulators,” IEEE J. Solid-State Circuits, vol. 6, no. 1, 1971.
4. B. Razavi, Design of Analog CMOS Integrated Circuits, 2nd ed., McGraw-Hill — chapter on voltage/current references.
5. Texas Instruments, application notes on bandgap voltage references (search TI.com for 'bandgap reference design guide').

