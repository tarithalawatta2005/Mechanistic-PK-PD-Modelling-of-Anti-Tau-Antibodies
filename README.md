# Mechanistic PK/PD Tau Aggregation Kinetics Model
**QSP Study of Anti-Tau Immunotherapies**

MATLAB-based mechanistic PK/PD model simulating endogenous tau dynamics and the pharmacodynamic effect of two anti-tau monoclonal antibodies (**Tilavonemab and Gosuranemab**) on tau aggregation in the Alzheimer's disease brain, closely following the MERCK paper's evaluation of its own PK/PD model.

---

## Phase 1: Physiological Infrastructure and Tau Homeostasis

Phase 1 establishes the physiological network of the Central Nervous System (CNS) and validates the steady-state kinetics of endogenous tau protein in a healthy patient. This baseline serves as the control prior to the introduction of pathology or therapeutic intervention.

---

### 1. Compartments

To simulate Absorption, Distribution, Metabolism, and Excretion (ADME) of therapeutic agents, the model uses a multi-compartment architecture representing distinct physiological spaces in the necessary  [4]:

- **Central (Plasma):** Represents systemic circulation (3500 mL) and the primary site for **intravenous antibody administration** [3]
- **Brain Interstitial Fluid (ISF):** The target site for tau pathology (280 mL) where **drug-target engagement** occurs [13]
- **Cerebrospinal Fluid (CSF):** The drainage pathway (150 mL) for CNS solutes, used for **clinical biomarker validation** [9]

**Flow of tau:**

```
Source (null) → Brain ISF → CSF → null (metabolisation + excretion))
```

---

### 2. Endogenous Tau Kinetics (Healthy State)

Tau homeostasis is modelled as a balanced system of constant production and concentration-dependent clearance [4].

#### A. Zero-Order Production (*k*_in)

Tau monomers are synthesised and released into the ISF at a **constant rate** [4]:

$$\frac{d[\text{Tau}_{ISF}]}{dt}\bigg|_{\text{production}} = k_{in}$$

- **Parameterisation:** $k_{in} = 0.00891 \ \text{pmol/min}$ [4]
- A zero-order reaction is used as ribosome activity is assumed to be at steady-state during early AD — independent of substrate concentration. Mass action kinetics ($v = k[\text{reactant}]$) are not applicable here as the source is a null node [4]

#### B. First-Order Glymphatic Clearance (*Q*_bulk)

Soluble tau is cleared from the brain parenchyma into the CSF via the glymphatic system's convective bulk flow [13]:

$$\frac{d[\text{Tau}_{ISF}]}{dt}\bigg|_{\text{clearance}} = -Q_{bulk} \cdot [\text{Tau}_{ISF}]$$

- **Parameterisation:** Volumetric flow rate $Q_{bulk} = 0.33 \ \text{mL/min}$ [4]
- **Kinetics:** The efflux rate is concentration-dependent — higher ISF tau density increases clearance velocity [13]

#### C. CSF Drainage to Null

Tau exits the CSF through the arachnoid granulations into venous blood for peripheral metabolism and excretion. This step is simplified to a null sink (not routed through the plasma compartment) as it does not affect tau aggregation kinetics or antibody PD [4]:

$$\frac{d[\text{Tau}_{CSF}]}{dt}\bigg|_{\text{drainage}} = Q_{bulk} \cdot [\text{Tau}_{ISF}] - Q_{bulk} \cdot [\text{Tau}_{CSF}]$$

The same $Q_{bulk}$ constant is applied, as intracranial pressure is assumed constant [9].

---

### 3. Mathematical Framework

The full ODE governing the temporal change in tau concentration within the brain ISF is [4]:

$$\boxed{\frac{d[\text{Tau}_{ISF}]}{dt} = \frac{1}{V_{ISF}} \cdot \left(k_{in} - Q_{bulk} \cdot [\text{Tau}_{ISF}]\right)}$$

Where:

| Parameter | Value | Units | Source |
|-----------|-------|-------|--------|
| $k_{in}$ | 0.00891 | pmol/min | [4] |
| $Q_{bulk}$ | 0.33 | mL/min | [4] |
| $V_{ISF}$ | 280 | mL | [13] |
| $V_{CSF}$ | 150 | mL | [9] |
| $V_{plasma}$ | 3500 | mL | [3] |

---

### 4. Validation and Steady-State Equilibrium

At steady state, $\frac{dC}{dt} = 0$, giving the analytical solution:

$$[\text{Tau}_{ISF}]_{ss} = \frac{k_{in}}{Q_{bulk}}$$

- **Simulation target:** Mathematical equilibrium reached at ~6,000 minutes
- **Result:** The calibrated model yields a soluble tau monomer concentration of **27.0 pM** (0.027 pmol/mL) within the CSF/ISF [4]
- **Clinical correlation:** This output serves as the "Healthy Baseline" validation, aligning with human preclinical benchmarks established in the Merck PBPK-PD framework [4]

---

## Phase 2: Pharmacokinetics(PK) of Anti-Tau Antibodies

Two monoclonal antibodies are modelled, both targeting the **N-terminus of tau** to block cell-to-cell propagation of tau pathology [4,11]:

| shortform | Drug | Clinical Trial | References |
|-----------|------|---------------|------------|
| **MonB**  | Gosuranemab | TANGO (Phase 2) | [14,15] |
| **MonT**  | Tilavonemab | Phase 2 (Florian et al.) | [6,10] |

Antibody concentrations are expressed in **nM**; tau in **pM** — units differ due to the large difference in scale between species and to prevent irrelevant scaling to infinity

---
