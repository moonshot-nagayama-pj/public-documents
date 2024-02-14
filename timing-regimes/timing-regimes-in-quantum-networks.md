# Timing Regimes in Quantum Networks

Quantum networks that distribute end-to-end entanglement involve a number of tasks with varying demands on timing precision and jitter. The design of a quantum network will involve a layered protocol architecture where different layers take responsibility for meeting these differing constraints. This document describes the various timing regimes, from most to least stringent, in order to assist the process of making key design decisions.

Summary of timing regimes:

A. __interferometric stabilization:__ need for sub-wavelength stability is photonic qubit representation-dependent  
B. __photon wave packet overlap:__ technology-dependent photon wave packet length, but roughly nanoseconds  
C. __opening and closing of detector timing windows, detector recovery time:__ nanoseconds to microseconds  
D. __measurement basis selection (if required in BSA):__ performance will constrain entanglement attempt rate  
E. __optical switch control:__ switching of trains of wave packets  
F. __pre-configured event-driven tasks such as timing-triggered or measurement-triggered execution of quantum circuits:__ microseconds  
G. __urgent but not synchronization-critical tasks (e.g. execution of classical code that processes RuleSet messages and selects or creates new quantum circuits for execution):__ milliseconds  
H. __host-side application-level tasks (e.g. post-measurement operations):__ milliseconds  
I. __background tasks (link tomography calculations, routing table updates):__ seconds to minutes  

Some of these can only be achieved using high-quality hardware, while others are software tasks. Detailed analysis of these regimes will affect core software design in each network node type.

# A. Interferometric Stabilization

Entanglement distribution in quantum networks is performed by entanglement swapping (ES) on photonic qubits.
Central to photonic ES is the Hong-Ou-Mandel (HOM) interference [1,2], regardless of the photonic qubit encoding or of the particular protocol implementing photonic ES.
We begin by introducing the notation used, giving a brief overview of the effect, as well as discussing how to quantify the effect.
We then continue with a discussion of the requirements that must be satisfied in order to observe the effect.

## A.1. Hong-Ou-Mandel interference

Consider two photons incident on a beamsplitter (BS) with reflectivity $r$.
We label the input modes $a$ and $b$.
The output modes of the BS are labelled with $a$ and $b$ as well, with the understanding that output mode $a$ corresponds to the input mode $a$ being transmitted.
Similarly for output mode $b$, as shown in the Figure below.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/HOM.png"/>
</p>

The input state can be expressed as
$$|\psi ^{\text{in}}\rangle_{ab} = \hat{a}^{\dagger}_j\hat{b}^{\dagger}_k |0\rangle _{ab},$$
where $\hat{a}^{\dagger}_j$ and $\hat{b}^{\dagger}_k$ are the bosonic creation operators corresponding to BS input modes $a$ and $b$, respectively.
The indices $j$ and $k$ represent other properties of the photons that determine how distinguishable the photons are.
For example, $j$ and $k$ could represent
- polarizations (for polarization-encoded qubits),
- spectral modes,
- temporal modes (for time-bin qubits),
- arrival time (discussed in Section B),
- transverse spatial mode.

We are interested in the observed behavior at the output modes of the BS.
There are four possible cases that may occur:
- Case A: photon in mode _a_ is reflected, while photon in mode _b_ is transmitted.
- Case B: both photons are transmitted.
- Case C: both photons are reflected
- Case D: photon in mode _a_ is transmitted, while photon in mode _b_ is reflect.

The action of the BS is represented by a unitary operator $\hat{U}_{ab}$,
$$\hat{a}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{1-r}\hat{a}^{\dagger} + \sqrt{r}\hat{b}^{\dagger}, \qquad \hat{b}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{r}\hat{a}^{\dagger} - \sqrt{1-r}\hat{b}^{\dagger}.$$
The output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \hat{U} _{ab} |\psi ^{\text{in}}\rangle _{ab} = \left( \sqrt{r(1-r)}\hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + r \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - (1-r) \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \sqrt{\eta(1-r)} \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
For a 50:50 BS with $r=1/2$, we obtain,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( \hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
From this expression, we can see that when $j=k$, in other words when the input photons indistinguishable, the output state has the following form,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{\sqrt{2}} \left( |2\rangle_a - |2\rangle_b \right).$$
The probability amplitudes for the cases where both input photons are transmitted or both reflected (Cases B and C in the figure above) interfere destructively.
Perfectly indistinguishable input photons always exit the BS in the same ouput mode.
It is this interference effect that is at the heart of quantum networking.

In order to quantify the effect that distibguishability has on the HOM interference, we consider the __probability of a coincidence detection__, $p_{\text{coin}}$, where one photon is detected in the BS output mode $a$, and the other photon in output mode $b$.
This probability is defined as
$$p_{\text{coin}} = \langle\psi^{\text{out}}|_{ab} \hat{P}_a \otimes \hat{P}_b |\psi^{\text{out}}\rangle _{ab},$$
where $\hat{P}_i$, for $i=a,b$, are the projection operators representing a detection of a single photon in output mode $i$ of the BS.
For completely indistinguishable input photons that undergo the full HOM interference, we have $p _{\text{coin}}=0$.
On the other hand, for fully distinguishable photons, the probability of a coincidence detection attains its maximum value $p _{\text{coin}}=1/2$.

Often used measure that quantifies the degree of HOM interference is the __visibility__ $V$, defined via the probability of a coincidence detection,
$$V = \frac{p_{\text{coin}}^{\text{max}} - p_{\text{coin}}^{\text{min}}}{p_{\text{coin}}^{\text{max}}} = 1 - 2 p_{\text{coin}}^{\text{min}},$$
where we used the fact that the maximum probability of a coincidence detection is $1/2$.
We observe that the visibility varies from $V=0$ for fully distinguishable input photons to $V=1$ for perfectly indistinguishable ones.

In the following subsections, we address and quantify how distinguishable photons affect the visibility of the HOM interference.

## A.2. Polarization

We now consider the case when he input photons differ in their polrization degree of photons.
The maximum probability of a coincidence detection is obtained for orthogonally polarized photons, for example when $j=H$ and $k=V$.
The output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( |1;H\rangle_a |1;V\rangle_a + |1;V\rangle_a |1;H\rangle_b - |1;H\rangle_a |1;V\rangle_b - |1;H\rangle_b |1;V\rangle_b \right).$$
We can immediately see that $p _{\text{coin}} ^{\text{max}}=1/2$.

In general, the two input photons will have polarizations given by two unit vectors, $j=\vec{\epsilon}$ and $k=\vec{\epsilon}'$.
The output state can be written as
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( |1;\vec{\epsilon}\rangle_a |1;\vec{\epsilon}'\rangle_a + |1;\vec{\epsilon}'\rangle_a |1;\vec{\epsilon}\rangle_b - |1;\vec{\epsilon}\rangle_a |1;\vec{\epsilon}'\rangle_b - |1;\vec{\epsilon}\rangle_b |1;\vec{\epsilon}'\rangle_b \right).$$
The projection operators corresponding to a detection even at detector $i$ ($i=a,b$), is given by
$$\hat{P}_i = |1;\vec{\epsilon}\rangle_i \langle 1;\vec{\epsilon}|_i + |1;\vec{\epsilon}'\rangle_i \langle 1;\vec{\epsilon}'|_i.$$
Either a $\vec{\epsilon}$-polarized or a $\vec{\epsilon}'$-polarized photon is detected in the output mode $i$.
The probability of coincidence is then
$$p _{\text{coincidence}} = \langle\psi ^{\text{out}}| _{ab} \hat{P}_a \otimes \hat{P}_b |\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( 1 - \left| \langle\vec{\epsilon}'|\vec{\epsilon}\rangle \right|^2 \right) = \frac{1}{2} \sin^2\theta,$$
where the overlap between the polarization unit vectors is parametrized by $\theta$, and can be written as $\langle\vec{\epsilon}'|\vec{\epsilon}\rangle = \cos\theta$.

Ensuring that the two input photons are indistinguishable in their polarization degree of freedom is critical for proper operation of the BSA.
Care must be therefore taken to characterize the photons just before they are incident onto the BS, as it is possible for the polarization of a photon to __drift__ during its transmission and change its state from the one that the photon possessed immediately after emission. 
This issue is mainly relevant in non-polarization-maintaining single-mode fibers.
 
## Temporal/spectral distinguishability

We are interested in the effect that distinguishability of the input photons, as shown in the Figure below, has on the probability of a coincidence event $p _{\text{coincidence}}$.
We begin with a treatment of the polarization degree of freedom before analyzing temporal and spectral distinguishability.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/spectral_temporal_dist.png"/>
</p>

### Temporal and spectral distinguishability

We will now include the photon's __spectral profile__ and __time of arrival__ at the BS, and their effects on the probability of detecting a coincidence event.
The 'shape' of the photon's wavepacket is characterized by the __spectral amplitude function__ $\phi(\omega)$, while the time delay between the photons' arrival at the BS is given by $\tau$.
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
The projector corresponding to a detection event in output mode $a$ is given by
$$\hat{P}_a = \int d\omega\hat{a}^{\dagger}(\omega) |0\rangle_a\langle0|_a\hat{a}(\omega),$$
and similarly for mode $b$.
The probability of detecting a coincidence of detecting one photon in each output mode is then
$$p ^{\text{pure}} _{\text{coincidence}} = \langle\psi ^{\text{out}}| _{ab} \hat{P}_a \otimes \hat{P}_b |\psi ^{\text{out}}\rangle _{ab}.$$
For two pure photons with arbitrary spectral amplitude functions, this expression reduces to
$$p ^{\text{pure}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1\phi^\ast(\omega_1)\varphi(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi^\ast(\omega_2) \phi(\omega_2) e^{i\omega_2\tau}.$$
When the two photons have the same spectral amplitude functions, $\phi(\omega)=\varphi(\omega)$,
$$p ^{\text{pure}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1|\phi(\omega_1)|^2 e^{-i\omega_1\tau} \int d\omega_2 |\phi(\omega_2)|^2 e^{i\omega_2\tau}.$$

We are now in a position to treat the case when the input photons are in a __mixed state__.
Such states may arise as the result of a probabilistic preparation of a pure photon, or the photon being entangled with a different degree of freedom such as another photon or a quantum memory.
The density matrix representing a mixture of spectral amplitude functions is
$$\rho_{\phi} = \sum_k q_k |1;\phi_k\rangle_a\langle1;\phi_k|_a.$$
The two photon input state can be written as
$$\rho ^{\text{in}} _{ab} = \rho _{\phi} \otimes \rho _{\varphi} = \sum _{kk'} q_k q _{k'} |1;\phi_k\rangle_a |1;\varphi _{k'}\rangle_b \langle 1;\phi_k|_a \langle 1;\varphi _{k'}|_b.$$
Considering that input photon $b$ is delayed by $\tau$, the output state of the two photons is
$$\rho ^{\text{out}} _{ab} = \sum _{kk'} q_k q _{k'} |\psi ^{\text{out}} _{kk'}\rangle _{ab} \langle\psi ^{\text{out}} _{kk'}| _{ab},$$
where
$$|\psi ^{\text{out}} _{kk'}\rangle _{ab} = \frac{1}{2} \int d\omega_1 \phi_k(\omega_1) \int d\omega_2 \varphi _{k'}(\omega_2) e^{-i\omega_2\tau} \left[ \hat{a}^{\dagger}(\omega_1)\hat{a}^{\dagger}(\omega_2) + \hat{a}^{\dagger}(\omega_2)\hat{b}^{\dagger}(\omega_1) - \hat{a}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) - \hat{b}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) \right] |0\rangle _{ab}.$$
Using linearity of quantum mechanics, computing the probability of coincidence, $p ^{\text{mix}} _{\text{coincidence}}$, can be done by simply summing the probabilities $p ^{\text{pure}} _{\text{coincidence}}$ over the spectral amplitude function indices $k$ and $k'$ with the appropriate weighing probabilities $q_k$ and $q _{k'}$,
$$p ^{\text{mix}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2} \sum _{kk'} q_k q _{k'} \int d\omega_1\phi_k^\ast(\omega_1)\varphi _{k'}(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi _{k'}^\ast(\omega_2) \phi_k(\omega_2) e^{i\omega_2\tau}.$$

__Example: Independent SPDC sources__  
Two independent SPDC sources generate two pure entangled pairs of photons,
$$|\psi_1\rangle _{ab} = \int d\omega_1 \int d\omega_2 f(\omega_1, \omega_2) \hat{a}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) |0\rangle _{ab}, \quad |\psi_2\rangle _{cd} = \int d\omega_1 \int d\omega_2 h(\omega_1, \omega_2) \hat{c}^{\dagger}(\omega_1)\hat{d}^{\dagger}(\omega_2) |0\rangle _{cd}.$$
Since photons $a/c$ and $b/d$ are entangled, it is necessary to use a joint spectral amplitude function $f/h$ for the entire pair.
Photons $b$ and $c$ are the input photons for the BS, with their reduced states being
$$\rho _{\phi} = \sum_k u^2_k|1;\phi_k\rangle_b\langle1;\phi_k|_b, \quad \rho _{\varphi} = \sum_k v^2_k|1;\varphi_k\rangle_c\langle1;\varphi_k|_c,$$
where the functions $\phi_k$ and $\varphi_k$ are obtained from the Schmidt decomposition of the joint spectral amplitude function,
$$f(\omega_1,\omega_2)=\sum_k u_k \phi'_k(\omega_1)\phi_k(\omega_2), \quad h(\omega_1,\omega_2)=\sum_k v_k \varphi_k(\omega_1)\varphi'_k(\omega_2).$$
The coincidence probability then becomes
$$p ^{\text{SPDC}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2} \sum _{kk'} u_k v _{k'} \int d\omega_1\phi_k^\ast(\omega_1)\varphi _{k'}(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi _{k'}^\ast(\omega_2) \phi_k(\omega_2) e^{i\omega_2\tau}.$$

Often, one is also interested in how the visibility changes as a function of the difference in time in the detection events, which is given by the time delay $\tau$.
The expression for the visibility can be modified in the following way,
$$V(\tau) = \frac{p _{\text{max}} - p _{\text{coincidence}}(\tau)}{p _{\text{max}}}.$$

# Detector recovery timing (C)


## References

[1] Hong, C.K., Ou, Z.Y., and Mandel L., Measurement of subpicosecond time intervals between two photons by interference, _Phys. Rev. Lett._ __59__, 2044 (1987).  
[2] Branczyk, A. M., Hong-Ou-Mandel Interference, arXiv:1711.00080 (2017).
