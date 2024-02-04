Summary of timing regimes:

A. __interferometric stabilization:__ need for sub-wavelength stability is photonic qubit representation-dependent  
B. __photon wave packet overlap:__ technology-dependent photon wave packet length, but roughly nanoseconds  
C. __opening and closing of detector itming windows, detector recovery time:__ nanoseconds to microseconds  
D. __measurement basis selection (if required in BSA):__ performance will contraint entanglement attempt rate  
E. __optical switch ocntrol:__ switching of trains of wave packets  
F. __pre-configured event-driven tasks such as timing-triggered or measurement-triggered execution of quantum circuits:__ microseconds  
G. __urgent but not synchronous-critical tasks (e.g. execution of classical code that processes RuleSet messages and selects or creates new quantum circuits for execution):__ milliseconds  
H. __host-side application-level tasks (e.g. post-measurement operations):__ milliseconds  
I. __background tasks (link tomography calculations, routing table updates):__ seconds to minutes  

Some of these can only be achieved using high-quality hardware, while others are software tasks. Detailed analysis of these regimes will affect core software design in each network node type.

## Hong-Ou-Mandel Effect

Entnaglement distribution in quantum networks is performed by entanglement swapping (ES) on photonic qubits.
Central to photonic ES is the Hong-Ou-Mandel (HOM) effect [1], regardless of the photonic qubit encoding or of the particular protocol implementing photonic ES.
in this section, we give a brief overview of this effect.

Consider two photons incident on a beamsplitter (BS) with reflectivity $\eta$.
We label the input modes as _a_ and _b_.
The output modes of the BS are labelled with _a_ and _b_ as well, with the understanding that if output mode _a_ corresponds to input mode _a_ being transmitted.
Similarly for output mode _b_, as shown in the Figure below.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/HOM.png"/>
</p>

We are interested in the observed bahavior at the output modes _c_ and _d_.
There are four possible cases that may occur:
- Case A: photon in mode _a_ is reflected, while photon in mode _b_ is transmitted.
- Case B: both photons are transmitted.
- Case C: both photons are reflected
- Case D: photon in mode _a_ is transmitted, while photon in mode _b_ is reflect.

The input modes are described by bosonic creation/annihilation operators $\hat{a}/\hat{a}^{\dagger}$ and $\hat{b}/\hat{b}^{\dagger}$, while the output modes are described by $\hat{c}/\hat{c}^{\dagger}$ and $\hat{d}/\hat{d}^{\dagger}$.
The individual input modes are transformed at the BS according to the following rule,
$$\hat{a} \rightarrow \frac{\hat{c}+\hat{d}}{\sqrt{2}},\qquad\text{ and}\qquad\hat{b} \rightarrow \frac{\hat{c}-\hat{d}}{\sqrt{2}}.$$
The minus sign ensures that the transformation is unitary.
The initial state is transformed to the following output state,
$$|1,1\rangle_{\text{ab}} = \hat{a}^{\dagger}\hat{b}^{\dagger}|0,0\rangle_{\text{ab}} \rightarrow \frac{1}{2}\left(\hat{c}^{\dagger}+\hat{d}^{\dagger}\right)\left(\hat{c}^{\dagger}-\hat{d}^{\dagger}\right)|0,0\rangle_{\text{cd}}=\frac{1}{\sqrt{2}}\left(|2,0\rangle_{\text{cd}}-|0,2\rangle_{\text{cd}}\right).$$
This shows that both photons are found in the same output mode of the BS with equal probability of $1/2$.
Placing a single-photon detector (SPD) in each output mode results in only one of the detectors registering a click, with no coincidence counts occurring.

In order to observe the HOM effect, the two input photons must in __indistinguishable__, meaning they must
- have the same __polarization__,
- have the same __wavelength__,
- arrive at the same __time__.

Partial distinguishability of the photons will result in a finite probability of observing coincidence events.
In the context of photonic ES, the effect is to reduce the fidelity of the end-to-end qubit pair.


## References

[1] Hong, C.K., Ou, Z.Y., and Mandel L., Measurement of subpicosecond time intervals between two photons by interference, _Phys. Rev. Lett._ __59__, 2044 (1987).
