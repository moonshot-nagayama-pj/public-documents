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

# Hong-Ou-Mandel Effect

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
$$\hat{a}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{1-\eta}\hat{a} + \sqrt{\eta}\hat{b}, \qquad \hat{b}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{\eta}\hat{a} - \sqrt{1-\eta}\hat{b}.$$
The output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \hat{U} _{ab} |\psi ^{\text{in}}\rangle _{ab} = \left( \sqrt{\eta(1-\eta)}\hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \eta \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - (1-\eta) \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \sqrt{\eta(1-\eta)} \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
For a 50:50 BS with $\eta=1/2$, we obtain,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( \hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{k} - \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
The main quantity of interest is the probability of registering a coincidence count, $p _{\text{coincidence}}$, between the two output modes of the BS.
If the input photons are indistinguishable, that is if $j=k$, the output state is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{\sqrt{2}} \left( |2;H\rangle_a - |2;H\rangle_b \right).$$
Both photons exit the BS in the same output mode, therefore the probability of a coincidence vanishes.

## Polarization/temporal/spectral distibguishability

We are interested in the effect that distinguishability of the input photons, as shown in the Figure below, has on the probability of a coincidence event $p _{\text{coincidence}}$.
We begin with a treatment of the polarization degree of freedom before analyzing temporal and spectral distinguishability.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/spectral_temporal_dist.png"/>
</p>

### Polarization
If the input photons are distinguishable (orthogonal), for example $j=H$ and $k=V$, the output state is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( |1;H\rangle_a |1;V\rangle_a + |1;V\rangle_a |1;H\rangle_b - |1;H\rangle_a |1;V\rangle_b - |1;H\rangle_b |1;V\rangle_b \right).$$
It is easy to see that the probability of coincidence is $1/2$.

### Temporal and spectral distinguishability

We will now include the photon's _spectral profile_ and _time of arrival_ at the BS, and theie effects on the probability of detecting a coincidence event.
The 'shape' of the photon's wavepacket is characterized by the _spectral amplitude function_ $\phi(\omega)$, while the time delay between the photons' arrival at the BS is given by $\tau$.
We denote a photon with spectral amplitude $\phi(\omega)$ by
$$|1;\phi\rangle_a = \int d\omega \phi(\omega) \hat{a}^{\dagger}(\omega) |0\rangle_a,$$
where $\hat{a}^{\dagger}(\omega)$ creates a photon in the BS input mode $a$ with frequency $\omega$, and we have the normalization condition $\int d\omega |\phi(\omega)|^2=1$.

Without loss of generality we assume that photon $b$ is delayed by a time $\tau$, which transforms its creation operator,
$\hat{b}^{\dagger}(\omega) \rightarrow \hat{b}^{\dagger}(\omega) e^{-i\omega\tau}.$
Two input photons with arbitrary spectral arbitrary functions $\phi$ and $\varphi$, with photon $b$ arriving late, are described by
$$|\psi ^{\text{td}}\rangle _{ab} = |1;\phi\rangle_a |1;\varphi\rangle_b = \int d\omega_1 \phi(\omega_1)\hat{a}^{\dagger}(\omega_1) \int d\omega_2 \varphi(\omega_2)\hat{b}^{\dagger}(\omega_2) e^{-i\omega_2\tau} |0\rangle _{ab}.$$
We assume that the BS acts on the different frequency modes independently, and that the reflectivity is also frequency-independent.
Applying the same transformation rules as in the previous section, the output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \int d\omega_1 \phi(\omega_1) \int d\omega_2 \varphi(\omega_2) e^{-i\omega_2\tau} \left[ \hat{a}^{\dagger}(\omega_1)\hat{a}^{\dagger}(\omega_2) + \hat{a}^{\dagger}(\omega_2)\hat{b}^{\dagger}(\omega_1) - \hat{a}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) - \hat{b}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) \right] |0\rangle _{ab}.$$
The projector corresponding a detection event in mode $a$ is given by
$$\hat{P}_a = \int d\omega\hat{a}^{\dagger}(\omega) |0\rangle_a\langle0|_a\hat{a}(\omega),$$
and similarly for mode $b$.
The probability of detecting a coincidence of detecting one photon in each output mode is then
$$p _{\text{coincidence}} = \langle\psi ^{\text{out}}| _{ab} \hat{P}_a \otimes \hat{P}_b |\psi ^{\text{out}}\rangle _{ab}.$$
For two pure photons with arbitrary spectral amplitude functions, this expression reduces to
$$p ^{\text{arb}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1\phi^\ast(\omega_1)\varphi(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi^\ast(\omega_2) \phi(\omega_2) e^{i\omega_2\tau}.$$
When the two photons have the same specrtal amplitude functions, $\phi(\omega)=\varphi(\omega)$,
$$p ^{\text{arb}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1|\phi(\omega_1)|^2 e^{-i\omega_1\tau} \int d\omega_2 |\phi(\omega_2)|^2 e^{i\omega_2\tau}.$$

__Example: Gaussain photons__

Consider the photons to be Gaussian spectral amplitude function
$$\phi_i(\omega) = \frac{1}{(\pi)^{1/4}\sqrt{\sigma_i}} e^{-\frac{(\omega-\bar{\omega}_i)^2}{2\sigma_i^2}}, \qquad i=a,b,$$
where $\hat{\omega}_i$ is th central frequency of photon $i$, and $\sigma_i$ defines the spectral width of the photon.
The probability of a coincidence event is then
$$p ^{\text{Gauss}} _{\text{coincidence}} = \frac{1}{2} - \frac{\sigma_a\sigma_b}{\sigma^2_a+\sigma^2_b} e^{-\frac{\sigma^2_a\sigma^2_b\tau^2+(\bar{\omega}_a-\bar{\omega}_b)^2}{\sigma^2_a+\sigma^2_b}}.$$

## Temporal and spectral distinguishability of mixed photons

In this section, we consider input photons that are entangled with other degrees of freedom, such as other photons or other atoms.


## References

[1] Hong, C.K., Ou, Z.Y., and Mandel L., Measurement of subpicosecond time intervals between two photons by interference, _Phys. Rev. Lett._ __59__, 2044 (1987).
