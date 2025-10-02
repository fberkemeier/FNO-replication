# Fourier Neural Operators for studying DNA replication kinetics 

*Francisco Berkemeier, University of Cambridge*  

⚠️ This code is part of ongoing research and is not yet licensed for reuse. It is shared for transparency and collaboration purposes only. A publication and proper license will follow.

Requires the `neuraloperator` library: https://github.com/neuraloperator/neuraloperator

All rights reserved. For enquiries regarding use or collaboration, please contact Francisco Berkemeier ([fp409@cam.ac.uk](mailto:fp409@cam.ac.uk)).

## 1. Modelling DNA replication kinetics  

DNA replication in human cells is a tightly regulated process, essential for maintaining genomic integrity across cell divisions [1]. During S phase, approximately 50,000 origins fire in a probabilistic but coordinated spatio-temporal pattern [2,3]. Each firing generates two replication forks, which replicate over 3 billion bases at average speeds of 0.5–2 kb/min in approximately 8 hours (Fig. 1a). Forks proceed until they merge with others, reach chromosome ends, or stall at intrinsic or drug-induced obstacles [4].  

**Why kinetics matter.** Disruptions to fork speed or origin usage can induce replication stress—a key contributor to chromosomal instability, ageing, and cancer development. Conversely, chemotherapeutic agents such as ATR and PARP inhibitors exploit the vulnerability of stressed forks [5], underscoring the need for quantitative models that connect replication to cell fate.  

In our framework, kinetics are governed by two functions: the initiation rate $I(x,t)$, specifying the probability of origin firing at genomic position $x$ and time $t$, and the fork speed $v(x,t)$.  

**Replication–crystal analogy.** A powerful way to study these dynamics is through nucleation-and-growth models, which provide a rigorous framework for stochastic processes driven by random initiation and finite-speed propagation. In one spatial dimension, the Kolmogorov–Johnson–Mehl–Avrami (KJMA) theory [6,7] captures such dynamics: nucleation events occur randomly, generate expanding domains, and merge upon contact.  

This formalism maps naturally onto DNA replication, where origins fire probabilistically and forks advance at local speed $v(x,t)$. A key quantity is the replication fraction $f(x,t)$, representing the probability that locus $x$ has been replicated by time $t$. This requires at least one origin to have fired within the spacetime region from which a fork could have reached $x$ by that time—the “past light cone” $\Lambda_X[v]$ of the point $X = (x,t)$ (Fig. 1b). Assuming independent firing events, this probability is derived from the inverse condition (no firing within the past light cone), leading to a product integral over the initiation rate $I(x,t)$

$$
f(x,t)=1-\exp\left( -\iint_{\Lambda_{X[v]}} I(\xi,\tau)\,d\xi d\tau \right) \qquad \text{(1)}
$$

This expression lies at the heart of the crystal-growth analogy: it links local kinetic inputs to measurable replication features, and forms a basis for deriving a wide range of metrics: timing (Fig. 1c) [8], fork densities and velocities [9], inter-origin distances [10], and more.  

<p align="center">
  <img src="https://github.com/user-attachments/assets/010fdae8-9357-4c74-927b-a0c7281f9bf5" alt="Figure 1 placeholder" width="800"/><br>
  <em>Figure 1. DNA replication dynamics and the KJMA framework.  
  (a) Schematic model of DNA replication.  
  (b) Light-cone geometry in one-dimensional replication.  
  (c) Experimental and simulated timing in cancer cells, highlighting regions of potential replication stress.</em>
</p>

## 2. Drug-response modelling  

Therapeutic agents that interfere with replication aim to selectively induce replication stress in cancer cells while sparing healthy tissue. Understanding this selectivity requires models that connect biophysical perturbations to global outcomes, such as incomplete replication and cell death.  

**Example: Two-origin system.** Cancer cells often exhibit altered replication kinetics due to changes in origin usage, fork restart mechanisms, or checkpoint function. To capture these differences, we consider a region with two replication origins.  

We define $\rho_\pm(x,t)$ as the density of right (+) and left (–) moving replication forks, and $f(x,t)$ as the replication fraction. Drug dosage $D$ reduces fork speed in a spatiotemporal region, expressed as $v(x,t;D)$. The resulting dynamics are governed by the following "transport" equation [9]:  

$$
\frac{\partial f}{\partial t}+v(x,t;D)\frac{\partial f}{\partial x} = I(x,t)(1-f) \qquad \text{(2)}
$$  

Equation (2) reproduces both replication timing (through $f$ [8]) and fork polarity across drug dosages. We simulate two cell types—healthy and cancerous—with distinct origin usage. Healthy cells complete replication even under high drug pressure. In contrast, cancer cells fail to replicate before the S phase limit, mimicking fork collapse or checkpoint activation (Fig. 2). 

<p align="center">
  <img src="https://github.com/user-attachments/assets/82a7eaae-3106-4e72-8df5-ae07b5d985ad" alt="Figure 2 placeholder" width="800"/><br>
  <em>Figure 2. Drug response in cells with different kinetics.  
  Heat maps of replication fraction f(x,t) and fork speed v(x,t) under increasing drug doses.  
  Healthy cells complete replication, while cancer cells fail to do so, highlighting a therapeutic window.</em>
</p>

As additional data types are incorporated—such as fork directionality, stalling signatures, or timing heterogeneity—the system of equations grows in complexity. Solving them parametrically becomes computationally demanding, which is precisely where physics-informed machine learning offers a powerful alternative.  

## 3. Physics-informed machine learning bridge to cell fate  

**Learning the dose–kinetics operator.** To rapidly predict replication dynamics under therapeutic influence, we train a Fourier Neural Operator (FNO) [11] on synthetic input–output pairs $(D \mapsto f(x,t))$ generated from Eq. (2). The FNO learns the full solution operator using mean squared error loss, without requiring spatial meshing or manual feature design. Once trained, it enables inference at unseen doses in milliseconds, bypassing the need to numerically solve the PDEs for each new condition.  

**Coupling to stress–fate rules.** To connect replication dynamics to cell fate, we introduce a mechanistic rule for replication stress: when total fork density $\rho_+(x,t) + \rho_-(x,t)$ exceeds a critical threshold for longer than a persistence time, the locus is assumed to accumulate double-strand breaks (DSBs).  

The spatially integrated DSB burden reflects replication-induced damage and serves as a proxy for genomic instability. This burden is mapped to a probability of cell death,  

$$
p_\text{death}(D) = \left[1+ \exp(-\alpha \cdot (\text{DSB}(D) - \theta)) \right]^{-1},
$$  

where $\alpha$ controls sensitivity, and $\theta$ is the burden threshold for 50% lethality.  

This enables simulation of dose-response curves and extraction of features such as replication-completion time $T_{95}(x,D)$, DSB loads, and tumour–healthy selectivity (Fig. 3).  

<p align="center">
  <img src="https://github.com/user-attachments/assets/7bc1ed83-3449-43b0-8a50-c4d2899d7a45" width="600"/><br>
  <em>Figure 3. From kinetics to cell fate via physics-informed machine learning.  
  (a) Replication-completion time $T_{95}(x,D)$ increases with drug dose.  
  (b) DSB burden rises sigmoidally with dose due to fork collapse.  
  (c) Cell-death probability $p_\text{death}(D)$ shows greater tumour sensitivity than healthy tissue.</em>
</p>

---

## References  

1. Gefter M.L. (1975).  
2. Rhind N., Gilbert D.M. (2013).  
3. Leonard A.C., Méchali M. (2013).  
4. Mirkin E.V., Mirkin S.M. (2007).  
5. Lloyd R.L. et al. (2020).  
6. Kolmogorov A. (1937).  
7. Jun S., Zhang H., Bechhoefer J. (2005).  
8. Berkemeier F. et al. (2025).  
9. Gauthier M.G. et al. (2012).  
10. Jun S., Bechhoefer J. (2005).  
11. Li Z. et al. (2020).  
