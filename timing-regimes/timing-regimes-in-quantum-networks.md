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
We label the input modes as $a$ and $b$.
The output modes of the BS are labelled with $a$ and $b$ as well, with the understanding that if output mode $a$ corresponds to input mode $a$ being transmitted.
Similarly for output mode $b$, as shown in the Figure below.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/HOM.png"/>
</p>

The input state can be expressed as
$$|\psi ^{\text{in}}\rangle_{ab} = \hat{a}^{\dagger}_j\hat{b}^{\dagger}_k |0\rangle _{ab},$$
where $\hat{a}^{\dagger}_j$ and $\hat{b}^{\dagger}_k$ are the bosonic creation operators corresponding to BS output modes $a$ and $b$, respectively.
The indices $j$ and $k$ represent other properties of the photons that determine how distinguishable the photons are.
For example, $j$ and $k$ could represent:
- polarizations,
- spectral modes,
- temporal modes,
- arrival time,
- transverse spatial mode.

We are interested in the observed bahavior at the output modes of the BS.
There are four possible cases that may occur:
- Case A: photon in mode _a_ is reflected, while photon in mode _b_ is transmitted.
- Case B: both photons are transmitted.
- Case C: both photons are reflected
- Case D: photon in mode _a_ is transmitted, while photon in mode _b_ is reflect.

The action of the BS is represented by a unitary operator $\hat{U}_{ab}$,
$$\hat{a}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{1-\eta}\hat{a} + \sqrt{\eta}\hat{b}, \qquad \hat{b}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{\eta}\hat{a} + \sqrt{1-\eta}\hat{b}.$$
The output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \hat{U} _{ab} |\psi ^{\text{in}}\rangle _{ab} = \left( \sqrt{\eta(1-\eta)}\hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \eta \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - (1-\eta) \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \sqrt{\eta(1-\eta)} \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
For a 50:50 BS with $\eta=1/2$, we obtain,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( \hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{k} - \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$


### Polarization
If the input photons are distinguishable (orthogonal), for example $j=H$ and $k=V$, the output state is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( |1;H\rangle_a |1;V\rangle_a + |1;V\rangle_a |1;H\rangle_b - |1;H\rangle_a |1;V\rangle_b - |1;H\rangle_b |1;V\rangle_b \right).$$
It is easy to see that the probability of coincidence is $1/2$.

On the other hand, if the input photons have indistinguishable polarizations, for example $j=k=H$, the output state is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{\sqrt{2}} \left( |2;H\rangle_a - |2;H\rangle_b \right).$$
Both photons exit the BS in the same output mode, therefore the probability of a coincide vanishes.

### Temporal distinguishability


## References

[1] Hong, C.K., Ou, Z.Y., and Mandel L., Measurement of subpicosecond time intervals between two photons by interference, _Phys. Rev. Lett._ __59__, 2044 (1987).
