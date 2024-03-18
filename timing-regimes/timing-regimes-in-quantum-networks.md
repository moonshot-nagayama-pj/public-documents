# Timing Regimes in Quantum Networks

Quantum networks that distribute end-to-end entanglement involve a number of tasks with varying demands on timing precision and jitter. The design of a quantum network will involve a layered protocol architecture where different layers take responsibility for meeting these differing constraints. This document describes the various timing regimes, from most to least stringent, in order to assist the process of making key design decisions.

The range of time scales of interest extends from ensuring the sub-wavelength stability of optical paths up to batch monitoring of the operation of the network itself. Light with a wavelength of $1.5\mu\text{m}$ (common in communications, including quantum communications) has a frequency of approximately $200$ THz ($2\times 10^{14}$ Hertz), for a cycle time of $5\times 10^{-15}$ seconds.  Ranging from sub-wavelength stabilization through background operations such as routing, therefore, covers some 16 or more decimal orders of magnitude.  Add in a 24-hour thermal drift that must be compensated for in many cases, and we reach twenty decimal orders of magnitude from the bottom to the top. Naturally, meeting this range of demands requires the use of a variety of mechanisms.  This document avoids specifying solutions to the problems, and instead presents the functions and how their requirements are calculated (or measured).  Thus, each individual network design should apply the methods introduced here and present a numerical summary of the resulting values, after which corresponding solutions can be proposed and implemented.

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
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/A1-HOM.png"/>
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
- Case C: both photons are reflected.
- Case D: photon in mode _a_ is transmitted, while photon in mode _b_ is reflected.

The action of the BS is represented by a unitary operator $\hat{U}_{ab}$,
$$\hat{a}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{1-r}\hat{a}^{\dagger} + \sqrt{r}\hat{b}^{\dagger}, \qquad \hat{b}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{r}\hat{a}^{\dagger} - \sqrt{1-r}\hat{b}^{\dagger}.$$
The output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \hat{U} _{ab} |\psi ^{\text{in}}\rangle _{ab} = \left( \sqrt{r(1-r)}\hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + r \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - (1-r) \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \sqrt{r(1-r)} \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
For a 50:50 BS with $r=1/2$, we obtain,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( \hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
From this expression, we can see that when $j=k$, in other words when the input photons are indistinguishable, the output state has the following form,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{\sqrt{2}} \left( |2\rangle_a - |2\rangle_b \right).$$
The probability amplitudes for the cases where both input photons are transmitted or both reflected (Cases B and C in the figure above) interfere destructively.
Perfectly indistinguishable input photons always exit the BS in the same ouput mode.
It is this interference effect that is at the heart of quantum networking.

In order to quantify the effect that distinguishability has on the HOM interference, we consider the __probability of a coincidence detection__, $p_{\text{coin}}$, where one photon is detected in the BS output mode $a$, and the other photon in output mode $b$.
This probability is defined as
$$p_{\text{coin}} = \langle\psi^{\text{out}}|_{ab} \hat{P}_a \otimes \hat{P}_b |\psi^{\text{out}}\rangle _{ab},$$
where $\hat{P}_i$, for $i=a,b$, are the projection operators representing a detection of a single photon in output mode $i$ of the BS.
For completely indistinguishable input photons that undergo the full HOM interference, we have $p _{\text{coin}}=0$.
On the other hand, for fully distinguishable photons, the probability of a coincidence detection attains its maximum value $p _{\text{coin}}=1/2$.

An often-used measure that quantifies the degree of HOM interference is the __visibility__ $V$, defined via the probability of a coincidence detection,
$$V = \frac{p_{\text{coin}}^{\text{max}} - p_{\text{coin}}^{\text{min}}}{p_{\text{coin}}^{\text{max}}} = 1 - 2 p_{\text{coin}}^{\text{min}},$$
where we used the fact that the maximum probability of a coincidence detection is $1/2$.
We observe that the visibility varies from $V=0$ for fully distinguishable input photons to $V=1$ for perfectly indistinguishable ones.

Visibility $V$ plays a useful role when modelling the effects of imperfect HOM interference in the context of entanglement swapping.
Consider the case when the input photons $a$, $b$ are entangled with auxiliary systems $s_1$ and $s_2$, respectively.
The BSA performs ES by measuring the input photons, entangling systems $s_1$ and $s_2$ in the process.
Fidelity of the new entangled pair is directly proportional to the visibility $V$ of the HOM interference.
Denote by $\rho_{s_1s_2}^{\text{no-deph}}$ the density matrix resulting from an ideal ES at the BSA with unit visibility of the HOM interference.
Non-ideal HOM interference can be modelled as a two-qubit dephasing [3],
$$\rho_{s_1s_2} = V \times \rho_{s_1s_2}^{\text{no-deph}} + (1 - V) \times \rho_{s_1s_2}^{\text{deph}},$$
where $\rho_{s_1s_2}^{\text{deph}}$ is a fully dephased state obtained by setting all off-diagonal elements of $\rho_{s_1s_2}^{\text{no-deph}}$ to zero.

In the following subsections, we address and quantify how distinguishable photons affect the visibility of the HOM interference.

## A.2. Polarization

We now consider the case when the input photons differ in their polarization degree of photons.
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
$$p _{\text{coin}} = \langle\psi ^{\text{out}}| _{ab} \hat{P}_a \otimes \hat{P}_b |\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( 1 - \left| \langle\vec{\epsilon}'|\vec{\epsilon}\rangle \right|^2 \right) = \frac{1}{2} \sin^2\theta,$$
where the overlap between the polarization unit vectors is parametrized by $\theta$, and can be written as $\langle\vec{\epsilon}'|\vec{\epsilon}\rangle = \cos\theta$.

Ensuring that the two input photons are indistinguishable in their polarization degree of freedom is critical for proper operation of the BSA.
Care must be therefore taken to characterize the photons just before they are incident onto the BS, as it is possible for the polarization of a photon to __drift__ during its transmission and change its state from the one that the photon possessed immediately after emission. 
This issue is mainly relevant in non-polarization-maintaining single-mode fibers.

We can define the corresponding visibility as a function of the nagle between the two polarization vectors,
$$V(\theta) = 1 - 2 p_{\text{coin}} = \cos^2\theta.$$
When the photons have identical polarization, $\theta=0$, the visibility reaches its maximum of $V=1$.
On the other hand, when the photons are fully distinguishable and their polarization vectors are orthogonal, $\theta=\pm\pi/2$, visibility is $V=0$.
The visibility $V(\theta)$ and the probability of a coincidence detection $p_{\text{coin}}$ are both displayed in the Figure below.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/A2-visibility_polarization.png", width="400"/>
</p>

__Interference of photons from two independent EPPS__  
The preceding discussion was concerned with two independent pure photons of different polarization.
In the context of quantum networking, a much more common scenario is that of two entangled pairs of photons originating from two independent EPPS nodes, where two qubits, one from each pair, are incident onto a BS and undergo HOM interference.
The two pairs are in the following initial state,
$$|\psi^{\text{in}}\rangle_{ab} = \frac{1}{\sqrt{2}}\left( |HV\rangle_{ab} + e^{i\theta_1} |VH\rangle_{ab} \right), \qquad |\psi^{\text{in}}\rangle_{cd} = \frac{1}{\sqrt{2}}\left( |HV\rangle_{cd} + e^{i\theta_2} |VH\rangle_{cd} \right),$$
where $\theta_1$ and $\theta_2$ represent the polarization drift induced in the single-mode fiber.
Photons $b$ and $c$ are incident onto a BS, where they undergo HOM interference.
Following the same calculation as above, it can be shown that the probability of a coincidence event is
$$p_{\text{coin}} = \frac{1}{4}, \quad\text{regardless of the polarization drift }\theta_{1/2}.$$
This suggests that the visibility is insensitive to the polarization drift.
However, the polarization drift must be tracked regardless because it affects the fidelity of the post-ES state of photons $a$ and $d$,
$$|\Psi^{\pm}\rangle_{ad} = \frac{1}{\sqrt{2}} \left( |HV\rangle_{ad} \pm e^{i(\theta_1-\theta_2)} |VH\rangle_{ad} \right).$$
It is therefore important to characterize the polarization drift at the BSA at regular intervals and compensate for it.
This can be done at the nodes generating the photon pairs at the cost of the BSA having to communicate polarization drift results to the ends nodes.
Or it can be compensated for directly at the BSA using waveplates at the cost of increased complexity of the BSA.
In [3], the process of polarization drift characterization and compensation at the BSA takes a few minutes and is performed every 20 minutes.

## A.3. Spectral distinguishability

Another important source of distinguishability in HOM interference is the spectral property of the input photons.
The photon wave packet of a photon is represented by its __spectral amplitude function__ $\phi(\omega)$, where $\int d\omega |\phi(\omega)|^2=1$.
Two input photons become distinguishable if their respective spectral amplitude functions are not equal.
The two photons may have different central frequencies $\bar{\omega}_a$ and $\bar{\omega}_b$.
The input photons may be distinguishable even if $\bar{\omega}_a = \bar{\omega}_b$, provided that the shape of the wave packet is different, as shown in the Figure below.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/A3-distinguish_spectral.png", width="800"/>
</p>
In this subsection, we analyze the requirements in terms of the photonic spectral amplitude function that lead to high visibility of the HOM interference.

__Pure states.__  
We begin the discussion by focusing on pure states of the input photons first.
Single-photon state with a spectral amplitude function $\phi(\omega)$ is a superposition written as
$$|1;\phi\rangle_a = \int d\omega \phi(\omega) \hat{a}^{\dagger}(\omega) |0\rangle_a,$$
where $\hat{a}^{\dagger}(\omega)$ creates a photon in the BS input mode $a$ with frequency $\omega$.
Two input photons with arbitrary spectral arbitrary functions $\phi$ and $\varphi$, are described by
$$|\psi ^{\text{in}}\rangle _{ab} = |1;\phi\rangle_a |1;\varphi\rangle_b = \int d\omega_1 \phi(\omega_1)\hat{a}^{\dagger}(\omega_1) \int d\omega_2 \varphi(\omega_2)\hat{b}^{\dagger}(\omega_2) |0\rangle _{ab}.$$
We assume that the BS acts on the different frequency modes independently, and that the reflectivity is frequency-independent.
Applying the same transformation rules for the creation operators, the output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \int d\omega_1 \phi(\omega_1) \int d\omega_2 \varphi(\omega_2) \left[ \hat{a}^{\dagger}(\omega_1)\hat{a}^{\dagger}(\omega_2) + \hat{a}^{\dagger}(\omega_2)\hat{b}^{\dagger}(\omega_1) - \hat{a}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) - \hat{b}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) \right] |0\rangle _{ab}.$$
The projection operators corresponding to a detection event in output mode $a$ and output mode $b$ are given by
$$\hat{P}_a = \int d\omega\hat{a}^{\dagger}(\omega) |0\rangle_a\langle0|_a\hat{a}(\omega),\quad \hat{P}_b = \int d\omega\hat{b}^{\dagger}(\omega) |0\rangle_b\langle0|_b\hat{b}(\omega)$$
and similarly for mode $b$.
The probability of a coincidence detection is then
$$p _{\text{coin}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1\phi^\ast(\omega_1)\varphi(\omega_1) \int d\omega_2 \varphi^\ast(\omega_2) \phi(\omega_2).$$
The form of this expression is the same as the one in Subsection A.2, where the probability of a coincidence detection depended on the overlap between the polarization vectors $\vec{\epsilon}$ and $\vec{\epsilon}'$.
Now, $p _{\text{coin}}$ depends on the overlap between the spectral amplitude functions.
If the input photons are fully distinguishable, their respective spectral amplitude functions $\phi(\omega)$ and $\varphi(\omega)$ are orthogonal and the integrals vanish, meaning $p _{\text{coin}}=1/2$.
On the other hand, for completely indistinguishable input photons we have $\phi(\omega)=\varphi(\omega)$, and due to the normalization condition we obtain $p _{\text{coin}}=0$.

__Mixed states.__  
Previous discussion of pure states can be extended to include mixed states of the input photons.
Such states will inevitably arise due to imperfections in the preparation procedure and due to the input photons being entangled with other degrees of freedom.
These can include other photons or quantum memories.

The mixed state of an input photon is described by the following density matrix,
$$\rho_a = \sum_k u_k |1;\phi_k\rangle_a\langle1;\phi_k|_a,$$
where the state of the photon is a mixture of pure single-photon states with spectral amplitude function $\phi_k(\omega)$, weighted by probability $u_k$.
The normalization condition requires that $\sum_k u_k=1$.
The two-photon input state can be written as
$$\rho ^{\text{in}} _{ab} = \sum _{kk'} u_k v _{k'} |1;\phi_k\rangle_a |1;\varphi _{k'}\rangle_b \langle 1;\phi_k|_a \langle 1;\varphi _{k'}|_b.$$
It is not necessary to repeat the entire calculation we did for pure states.
Due to the linearity of quantum mechanics, we can immediately write the expression for the probability of coincidence as a sum of pure-state coincidence probabilities weighted by $u_k$ and $v'_k$,
$$p _{\text{coin}} = \frac{1}{2} - \frac{1}{2} \sum _{kk'} u _k v _{k'} \int d\omega_1\phi^\ast_k(\omega_1)\varphi _{k'}(\omega_1) \int d\omega_2 \varphi _{k'}^\ast(\omega_2) \phi_k(\omega_2).$$

__Example 1: Gaussian wave packets__  
In this example, we consider input photons with Gaussian spectral amplitude functions.
The spectral amplitude functions are given by
$$\phi_i(\omega) = \frac{1}{\pi^{1/4}\sqrt{\sigma_i}} e ^{-\frac{(\omega-\bar{\omega}_i)^2}{2\sigma^2_i}},\quad\text{for } i=a,b.$$
The probability of a coincidence detection is then
$$p _{\text{coin}} = \frac{1}{2} - \frac{1}{2\pi\sigma^2} \left( \int d\omega_1 e ^{-\frac{(\omega_1-\bar{\omega}_a)^2}{2\sigma_a^2}} e ^{-\frac{(\omega_1-\bar{\omega}_b)^2}{2\sigma_b^2}} \right) \left( \int d\omega_2 e ^{-\frac{(\omega_2-\bar{\omega}_a)^2}{2\sigma_a^2}} e ^{-\frac{(\omega_2-\bar{\omega}_b)^2}{2\sigma_b^2}} \right) = \frac{1}{2} -\frac{\sigma_a\sigma_b}{\sigma_a^2 + \sigma_b^2} e^{-\frac{(\bar{\omega}_a-\bar{\omega}_b)^2}{\sigma_a^2+\sigma_b^2}}.$$

__Case A (different central frequencies):__
We assume that the two spectral amplitude functions have the same standard deviation $\sigma_a=\sigma_b=\sigma$, which simplifies the expression for the probability of a coincidence detection to
$$p _{\text{coin}} = \frac{1}{2} \left( 1 - e^{-\frac{(\bar{\omega}_a-\bar{\omega}_b)^2}{2\sigma^2}} \right).$$
We observe that for identical photons, when $\bar{\omega}_a=\bar{\omega}_b$, the probability of a coincidence detection vanishes.
For fully distinguishable wave packets, when $\bar{\omega}_a-\bar{\omega}_b\rightarrow\infty$, the probability approaches 1/2, as expected.
We can now define visibility as a function of the difference between the central frequencies $\bar{\omega}_a-\bar{\omega}_b$,
$$V(\bar{\omega}_a-\bar{\omega}_b) = e^{-\frac{(\bar{\omega}_a-\bar{\omega}_b)^2}{2\sigma^2}}.$$

__Case B (different standard deviations):__
The spectral amplitude functions have the same central frequencies, $\omega_a=\omega_b$, which gives the following expression for the probability of coincidence,
$$p_{\text{coin}} = \frac{1}{2} - \frac{\sigma_a\sigma_b}{\sigma_a^2 + \sigma_b^2}= \frac{1}{2} - \frac{\sigma_b/\sigma_a}{1 + (\sigma_b/\sigma_a)^2},$$
allowing us to define the visibility of the HOM interference as a function of the ratio between the standard deviations of the spectral amplitude functions,
$$V(\sigma_b/\sigma_a) = \frac{2\sigma_b/\sigma_a}{1 + (\sigma_b/\sigma_a)^2}.$$

The probability of coincidence and corresponding visibility for both Cases are shown in the Figure below.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/A3-visibility_spectral.png", width="800"/>
</p>


# B. Wave Packet Overlap

So far we have assumed that the two input photons arrive at the BS at exactly the same time.
In this subsection, we address this unrealistic assumption and quantify how temporal distinguishability affects the visibility of HOM interference.
Even for photons with identical spectral amplitude functions, different arrival times result in decreased overlap between the photons' wave packets, diminishing the visibility of the HOM interference.
If the times of arrival are too different as shown in the Figure below, the probability of a coincidence detection will reach its maximum value of 1/2, and the visibility will vanish.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/B-distinguish_temporal.png", width="800"/>
</p>

Without loss of generality we assume that photon $b$ is delayed by a time $\tau$, which transforms its creation operator,
$\hat{b}^{\dagger}(\omega) \rightarrow \hat{b}^{\dagger}(\omega) e^{-i\omega\tau}.$
Two input photons with arbitrary spectral arbitrary functions $\phi$ and $\varphi$, with photon $b$ arriving late, are described by
$$|\psi ^{\text{in}}\rangle _{ab} = |1;\phi\rangle_a |1;\varphi\rangle_b = \int d\omega_1 \phi(\omega_1)\hat{a}^{\dagger}(\omega_1) \int d\omega_2 \varphi(\omega_2)\hat{b}^{\dagger}(\omega_2) e^{-i\omega_2\tau} |0\rangle _{ab}.$$
We assume that the BS acts on the different frequency modes independently, and that the reflectivity is also frequency-independent.
Applying the same transformation rules as in the previous section, the output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \int d\omega_1 \phi(\omega_1) \int d\omega_2 \varphi(\omega_2) e^{-i\omega_2\tau} \left[ \hat{a}^{\dagger}(\omega_1)\hat{a}^{\dagger}(\omega_2) + \hat{a}^{\dagger}(\omega_2)\hat{b}^{\dagger}(\omega_1) - \hat{a}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) - \hat{b}^{\dagger}(\omega_1)\hat{b}^{\dagger}(\omega_2) \right] |0\rangle _{ab}.$$
For pure input states, the probability of a coincidence detection is
$$p ^{\text{pure}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1\phi^\ast(\omega_1)\varphi(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi^\ast(\omega_2) \phi(\omega_2) e^{i\omega_2\tau},$$
while for mixed states is can be generalized to the following form,
$$p ^{\text{mix}} _{\text{coincidence}} = \frac{1}{2} - \frac{1}{2} \sum _{kk'} u_k v _{k'} \int d\omega_1\phi_k^\ast(\omega_1)\varphi _{k'}(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi _{k'}^\ast(\omega_2) \phi_k(\omega_2) e^{i\omega_2\tau}.$$

__Example: Gaussian wave packets__  
Consider two identical wavepackets, which arrive at the BS with a time difference given by $\tau$.
The probability of coincidence is given by
$$p_{\text{coin}} = \frac{1}{2} \left( 1 - e^{-\frac{1}{2}\sigma^2\tau^2} \right) = \frac{1}{2} \left( 1 - e^{-\frac{1}{2}\tau^{\prime 2}} \right),$$
where we have defined a dimensionless time rescaled by the standard deviation of the spectral amplitude function of the photon, $\tau^{\prime} = \sigma\tau$.
The corresponding visibility is given by
$$V(\tau^{\prime}) = e^{-\frac{1}{2}\tau^{\prime 2}}.$$
Figure below displays the visibility and probability of coincidence for this case.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/B-visibility_temporal.png", width="400"/>
</p>

__Photon emission time jitter__  
Photon emission is fundamentally a __non-deterministic__ process, owing to the Heisenberg uncertainty principle $\Delta E \Delta t \geq \hbar/2$.
Following our discussion in Section A.3 (Spectral distinguishability), it is desirable for the emitted photons to be spectrally pure, which corresponds to small $\Delta E$.
This on the other hand results in a large uncertainty of the emission time, given by $\Delta t$.
Therefore, even if the two remote nodes commence their atom excitation procedure in a perfectly synchronized fashion, they will likely emit their photons at slightly different times, leading to photon emission time jitter, $J_{\text{emission}}$, and finite difference of arrival $\tau$ at the BSA.

The amount of emission time jitter depends on the physical system used to implement the emissive memory.
For example, recent trapped ions experiment [3] characterized the emission probability as a function of time, leading to a Poissonian distribution with standard deviation of approximately 10 $\mu\text{s}$.

# C. Detector Timing Windows

In this section, we discuss how properties of single-photon detectors (SPDs) affect the timing regimes in quantum networks.
An ideal SPD generates an electrical signal after absorbing a photon, and generates no signal in the absence of a photon.
This is not always true for real-world SPDs [4].

## C.1. Detector basics

Performance of SPDs can be quantified by the following characteristics,
- __Spectral range:__ SPDs are sensitive over a limited range of wavelengths. This range depends on the materials used in the fabrication of the detector. Typical spectral ranges are in the near-infrared, around 1550nm, where commercial optical fibers perform best in terms of photon loss rates.
- __Detection efficiency:__ The overall probability that an incoming photon registers a count, denoted by $\eta$.
This efficiency can be further broken down.
Probability of losing the photon before it reaches the detector is described by the *coupling efficiency*, $\eta_{\text{coupling}}$.
The type of material and geometry of the detector determine the photon *absorption efficiency*, $\eta_{\text{absorption}}$.
Finally, the probability that an electric signal is generated upon successful absorption of a photon is described by the *registering efficiency*, $\eta_{\text{registering}}$.
The overall *system detection efficiency* is given by the product of these three,
$$\eta_{\text{sde}} = \eta_{\text{coupling}} \times \eta_{\text{absorption}} \times \eta_{\text{registering}}.$$
The *device detection efficiency* is given by
$$\eta_{\text{dde}} = \eta_{\text{absorption}} \times \eta_{\text{registering}}.$$
Detection efficiency affects the rate at which entanglement can be distributed.
- __Recovery time:__ Denoted by $\tau_{\text{recovery}}$ and also known as 'dead' time.
It is the time duration following an absorption of a photon during which the detector is unable to reliably detect another photon.
Recovery time affects the maximum detection rate.
If the source of photons has low efficiency, the clock rate does not need to be limited by the recovery time, as majority of the trials will not produce a photon.
This could also be the case if the probability of losing the photon is high (either due to loss in fiber or due to low system detection efficiency $\eta_{\text{sde}}$).
On the other hand, if the photon source is highly efficient, it is important to ensure that the separation between the wavepackets is longer than $\tau_{\text{recovery}}$ to ensure effcient use of the generated photons.
- __Dark count rate:__ SPDs have a finite chance to produce an output electric signal even in the absence of a photon. This may be caused by materials properties of the detector, biasing conditions, or external noise. It is usually given in Hz [counts per second]. Dark counts decrease the fidelity of the distributed entangled states.
- __Timing jitter:__ Denoted by $J_{\text{timing}}$.
Describes the variation in time between the photon being absorbed and the output electric signal being generated. A few example profiles are shown in the Figure below taken from [4].
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/C-timing_jitter.png", width="500"/>
</p>

The table below shows the above characteristics for a SNSPD [5].

| Wavelength                  | 800 nm  | 1550 nm |
|-----------------------------|---------|---------|
| System detection efficiency | > 90%   | > 90%   |
| Recovery time               | 10 ns   | 20 ns   |
| Dark count rate             | < 1 Hz  | < 1 Hz  |
| Timing jitter               | < 15 ps | < 15 ps |

## C.2. Acceptance window

In the previous Section, we used $\tau$ to denote the difference between the arrival time of the photons at the BSA.
However, due to emission jitter it is impossible to know the precise time of arrival of a photon.
The only information that is available comes from the electric output signal of a detector.
Time of detection is in general different from the time of photon arrival due to finite time needed to generate the output signal described by the timing jitter $J_{\text{timing}}$.
Therefore, we will use $\tau$ to denote the difference in detection time of the two photons.

Measurement at the BSA is successful when the correct pattern of detector clicks is observed, and the difference in detection times $\tau$ is smaller than a given detection __acceptance window__, $T_{\text{window}}$.
The size of this window affects both the fidelity and the generation rate of the entangled pairs that the link produces.
Large acceptance windows produce high rates but low fidelity, while small acceptance windows result in low rates and high fidelity.
The appropriate size of the acceptance window must be chosen in order to satisfy the demands of the application requesting the entangled states.
Reaching the requested fidelity should take priority over high generation rate.

## C.3. Separation in a train of wavepackets

Current experiments on quantum repeaters use single quantum memory per QNIC [3].
As quantum technologies improve, it is likely that QNICs will be equipped with multiple quantum memories.
This will allow for generation of link-level entanglement in a multiplexed manner, where trains of photons, each originating from a different memory inside the same QNIC, are sent to the BSA.
The photons making up a train must be well separated such that upon a successful BSM, the BSA can uniquely identify which two photons were measured.
We refer to the minimum separation between the photons as the __separation time__ $T_{\text{separation}}$.

The size of the separation time depends on the following:
- __Wave packet shape:__ Individual photons cannot have overlapping spatial wavepackets, which may lead to incorrect assignement of entangled qubits following a successful meadurement at the BSA. We will use $T_{\text{photon}}$ to denote the length of a wavepacket in seconds.
- __Detector recovery time:__ Spacing the wavepackets too close to each other may result in some of the photons being lost due to the detector recovering following a detection event, leading to inefficient use of initially generated entangled pairs (either memory-photon or photon-photon).
- __Memory emission jitter:__ The separation between the wavepackets must take into account the probabilistic nature of photon emission from a quantum memory in order to prevent wavepacket overlap.
- __Detector timing jitter:__ Generation of the electric signal following absorption of a photon varies in duration, leading to a variance in timing of the detection event. This may lead to the BSA mislabelling which photons were part of a successful measurement if their wavepackets are spaced too closely.

General (conservative) separation time should therefore be set to
$$T_{\text{separation}} \ge T_{\text{photon}} + J_{\text{emission}} + J_{\text{timing}}.$$

The above discussion assumes that the photons can be generated nearly on-demand.
This is a fair assumption in the case of quantum memories based on trapped ions [3].
Here, the memory must be first initialized by cooling it to its ground state, a process which takes $<1\text{ms}$.
The memory is then excited by a laser pulse of ~ $50\mu \text{s}$ that generates a photon.

In the case of memory-less link architectures, the picture is slightly different.
Here, EPPS nodes utilizing the principle of spontaneous parametric down-conversion (SPDC) generate entangled photon pairs.
Each photon is sent to a different BSA, where they are measured with a photon originating from a different EPPS node.
SPDC is an inefficient process with success probability of around $10^{-6}$ per pump photon. In system design, the intensity of the pump laser is adjusted so that the average number of photons is appropriate; generally this must be set below one photon per time window in order to avoid polluting the signal with two-photon states.
This means that most of the time windows given by the separation time will not contain a photon.
However, the separation time should be maintained in order to correcly identify the photons that were part of a successful measurement at the BSA.
The separation time governs the maximum rate at which EPPS attempts to generate the entangled photon pairs, which is given by $1/T_{\text{separation}}$.

# D. Measurement basis selection

We have encountered Bell-state measurements performed by the BSA on photonic qubits that are needed for entanglement swapping.
These measurements were static in the sense that we did not need to change the measurement basis.
Observed detection pattern determined whether the post-measurement state of the remote quantum systems (either quantum memories or other photons) was $|\Psi^+\rangle$ or $|\Psi^-\rangle$.
Projection onto two of the four Bell-basis states was achieved probabilistically without actively applying any transformations on the photonic qubits.
We will see in this Section that the photonic BSA is a very special case in this regard, and that changing the basis of the measurement is an indispensable part of quantum networking.
Entanglement swapping on stationary qubits stored in quantum memories is not possible without applying appropriate unitaries first that change the basis of the measurement.
There are also cases, where change of basis is required even when dealing with only photonic qubits.
An example of this are the so-called all-photonic quantum repeaters, where measurement basis is conditioned on the outcomes of previous measurements, leading to the requirement of very fast basis switching.

## D.1. Measurement basics

We will first discuss quantum measurements in general before discussing concrete implementations and their timing requirements based on their physical implementations.

__Single-qubit measurements:__  
For simplicity, we begin with measurements on a single qubit before generalizing to two qubit measurements.
Consider a general state of the qubit,
$$|\psi\rangle = \alpha |0\rangle + \beta |1\rangle,$$
where $|\alpha|^2+|\beta|^2=1$.
Measurement in an arbitrary basis $\hat{M}$ projects the initial state $|\psi\rangle$ onto one of the eigenvectors of $\hat{M}$, given by $\\{|\phi\rangle,|\phi^{\perp}\rangle\\}$.
Probabilities of the two possible measurement outcomes are given by the overlaps between the initial state $|\psi\rangle$ and the eigenvectors of the observable $\hat{M}$,
$$\text{Pr}(|\phi\rangle;|\psi\rangle)=|\langle\phi|\psi\rangle|^2, \quad\text{and}\quad \text{Pr}(|\phi^{\perp}\rangle;|\psi\rangle)=|\langle\phi^{\perp}|\psi\rangle|^2.$$
We read this notation as ``probability of the measurement outcome being the state $|\phi\rangle$, given that the initial state was $|\psi\rangle$''.
For example, measurement in the Pauli $Z$ basis projects onto the states $\\{|0\rangle,|1\rangle\\}$, while measurement in the Pauli $X$ basis projects onto the states $\\{|+\rangle,|-\rangle\\}$.

It is often difficult to directly measure the qubit in an arbitrary basis when it comes to real-world implementation.
In such a case, the qubit needs to be pre-rotated by an appropriate unitary operation, and then measured in the $Z$ basis, which can usually be implemented in a straightforward way.
This approach greatly simplifies the implementation of arbitrary measurements.

Consider that the observable $\hat{M}$ is related to the Pauli $\hat{Z}$ by unitary $\hat{U}$,
$$\hat{M} = \hat{U} \hat{Z} \hat{U}^{\dagger}.$$
This means the unitary $\hat{U}$ also relates the eigenvectors of the two observables,
$$|\phi\rangle = \hat{U} |0\rangle, \quad\text{and}\quad |\phi^{\perp}\rangle = \hat{U}|1\rangle.$$
We can perform measurement in the $\hat{M}$ basis by applying $\hat{U}^{\dagger}$ to the initial state $|\psi\rangle$, and them measuring it in the Pauli $Z$ basis.
This can be easily verified by rewriting the above probabilities corresponding to the two measurement outcomes,
$$\text{Pr}(|\phi\rangle;|\psi\rangle) = |\langle\phi|\psi\rangle|^2 = |\langle0|\hat{U}^{\dagger}|\psi\rangle|^2 = \text{Pr}(|0\rangle;\hat{U}^{\dagger}|\psi\rangle),$$
$$\text{Pr}(|\phi^{\perp}\rangle;|\psi\rangle) = |\langle\phi^{\perp}|\psi\rangle|^2 = |\langle1|\hat{U}^{\dagger}|\psi\rangle|^2 = \text{Pr}(|1\rangle;\hat{U}^{\dagger}|\psi\rangle).$$
This is pictured in the Figure below.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D1-meas_1qubit.png", width="500"/>
</p>

__Two-qubit measurements:__  
The same principle of changing the measurement basis can be generalized to two qubits.
This time state $|\psi\rangle$ represents a general two-qubit state, unitary $\hat{U}^{\dagger}$ acts on both qubits, which are both finally measured in Pauli $Z$ basis.
In the majority of cases, we are interested in performing measurements in the Bell basis.
Required unitary $\hat{U}^{\dagger}$ is the Hermitian conjugate of the unitary that creates a Bell pair when the qubits are both initialized in $|0\rangle$, as shown in the Figure below, where we have dropped the $\hat{Z}$ label indicating the measurement basis.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D1-meas_2qubit.png", width="250"/>
</p>

## D.2. Measurements on quantum memories

In this Section, we discuss various methods of implementing measurements of quantum memories.
These methods vary based on the quantum technology used as the quantum memory, and even within the same technology there are usually variations.
We are mainly concerned with giving an overview of the different measurement methods, and their respective timing regimes.

__Trapped ions:__  
Trapped ions possess two degrees of freedom [6].
The first one is the motional degree of freedom, resulting from the ion oscillating around its equilibrium position in the trap.
The second one is the internal degree of freedom, represented by the ground state $|g\rangle$ and the excited state $|e\rangle$.
It is the latter degree of freedom which is used to encode a qubit and hence acts as a quantum memory.

Measurement in the __Pauli $\hat{Z}$__ basis is performed by __electron shelving__ via the use of a third atomic level $|r\rangle$, with much shorter life time than the excited state $|e\rangle$, $\tau_e \gg \tau_r$ [7].
Figure below demonstrates this method works.
<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D2-shelving.png", width="250"/>
</p>

The ion is illuminated by light tuned to resonate with the transition $|g\rangle\leftrightarrow|r\rangle$, represented by the red straight arrow in the Figure above.
If fluorescence is immediately observed, this corresponds to measuring the ion in the ground state $|g\rangle$.
If no fluorescence is observed, the ion is measured in the excited state $|e\rangle$.
Hypothetically a single fluorescent photon would be sufficient, however, the fluorescent photons are only rarely captured into the measurement apparatus (typically involving lenses and a camera) and observed, and stray photons are also often captured, so a relatively long __integration time__ is used to confirm the fluorescence with high probability.  (Solid-state systems such as quantum dots and superconducting qubits also need relatively long integration times in their measurement processes.)
Combined with laser pulses that apply a single-qubit rotation, measurement of a __single ion in an arbitrary basis__ can be performed in $1-2 \text{ ms}$ [3].

The __CNOT gate__ can be applied in two different ways.
The original proposal is due to Cirac and Zoller [8], where the ions needed to be cooled to their collective motional ground state first.
This approach was demonstrated experimentally using $^{40}\text{Ca}^+$ ions [9].
Application of the take took around $600\mu\text{s}$, with the achieved fidelity being $<0.8$.
The second approach is due to Molmer and Sorensen [10], and is more robust against motional excitation.
This led to high-fidelity demonstrations of $>0.99$, and gate times of around $50\mu\text{s}$ .

__NV centers in diamond:__  
Text

## D.3. Measurements on photonic qubits

Measurement of polarization-encoded photonic qubits can be performed with the aid of a __polarizing beam splitter__ (PBS), a __half waveplate__ (HWP), a __quarter waveplate__ (QWP), and two detectors (one detector is enough in fact but less efficient) [12].
The idea is the same as in the case of measurements performed on stationary qubits discussed above.
Setting the HWP and QWP at particular angles applies the unitary $\hat{U}^{\dagger}$ that picks the basis of the measurement, while the PBS filters out vertical and horizontal polarizations that then get detected by the detectors placed in the output paths of the PBS.
Horizontal polarization gets transmitted through the PBS, while vertical polarization gets reflected.
This setup is shown in the figure below.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D3-2detector_setup.png", width="300"/>
</p>

General pure state of a polarization-encoded qubit can be written as
$$|\psi\rangle = \cos\left(\frac{\theta}{2}\right)|H\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)|V\rangle.$$
This is directly equivalent to expressing the qubit state in the computational basis, and can be visualized with the help of the __Poincaré sphere__.
The table below summarizes this equivalence.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D3-polarization_table.png", width="600"/>
</p>

The Figure below shows the Poincaré sphere along with the position of the polarization states.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D3-poincare.png", width="450"/>
</p>

Polarization of light is manipulated by waveplates.
Waveplate rotated by an angle $\alpha$ (zero is aligned with the fast axis) rotates the polarization state around an axis, located at an angle of $2\alpha$ with the horizontal state $|H\rangle$ in the horizontal plane, as shown in Figure above.
Half waveplate rotates the polarization state by an angle $\pi$, while a quarter waveplate rotates by an angle $\pi/2$ in the Poincaré sphere.
The action of the waveplates is captured by the corresponding unitary operations,

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D3-waveplates_matrix.png", width="650"/>
</p>

The idea behind measurements in arbitrary basis $\\{|\psi\rangle, |\psi^{\dagger}\rangle\\}$ is to choose the angles for the waveplates such that the following transformation is achieved,
$$U_{HWP}(\alpha_{HWP})U_{QWP}(\alpha_{QWP}) |\psi\rangle \rightarrow |H\rangle, \quad U_{HWP}(\alpha_{HWP})U_{QWP}(\alpha_{QWP}) |\psi^{\perp}\rangle \rightarrow |V\rangle.$$
Settings for the three Pauli bases are summarized in the table below.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/D3-waveplate_angles_table.png", width="325"/>
</p>

Changing the basis of measurement requires physical rotation of the waveplates and coordination with the detectors.


# E. Optical Switch Control

# F. Pre-configured Event-driven Tasks
In this section, we discuss synchronization-critical tasks that must be conducted when an event occurs. Most stationary qubits are under the control of a classical analog circuit that includes a local oscillator (LO) coupled to the corresponding frequency of the qubit itself. Avoiding drift between the _understood_ phase of the qubit and the _actual_ phase of the LO is a key part of hardware design for a qubit, but is beyond the scope of this document.

Events can be _local_ to a quantum computer or repeater node, or _remote_, generally implying reception and processing of a classical message.

In some cases, when a memory is used to emit a photon, the ultimate disposition of the qubit in memory might be measurement immediately after the emission of the photon (as in QKD). Alternatively, in systems involving QEC, immediately after emission of the photon, the memory qubit may be encoded into a logical qubit. In general, such events can trigger execution of a local quantum circuit.

For links using HOM-based entanglement generation, inevitably there is a delay between the BSA operation completing successfully and the generation, transmission and reception of the confirmation message. Over distances of a few kilometers, this can require a few microseconds.

# G. Urgent but Not Synchronization-critical Tasks

# H. Host-side Application-level Tasks
The service provided by a quantum network is entangled states, which may be either delivered to applications on quantum computers, held in limited-capability quantum memories for later release and use, or directly measured as creation is completed, corresponding to the capabilities of COMP, STOR and MEAS end node types, respectively.

An application that uses the services of a quantum network passes through several phases:

* __Planning__: selecting application-level tasks and communication partners, including defining application quantum circuits (for distributed computation) or measurement bases and characteristics (for QKD or sensing applications), required distributed quantum states (at the moment, presumed to be Bell pairs), distributed entanglement fidelity, and number or rate of entangled states needed.
* __Computational resource preparation__: for distributed quantum computation, allocation of quantum computing resources attached to the quantum network, compilation of application circuits for processing and consumption of the entangled states delivered by the network service.
* __Connection setup__: the classical process of establishing communication between nodes.  Depending on the network architecture, this may include allocation of resources at repeater nodes.
* __Real-time receipt and disposition of entangled states__: as the network delivers entangled states to the end nodes, they will be consumed by applications, ultimately resulting in classical information which may determine further quantum actions at the level of immediate, result-dependent actions.
* __Near-real time computation__: many quantum algorithms are hybrid classical/quantum computation and may require larger-scale adaptation or recompilation of application quantum circuits.
* __Post-quantum processing__: the classical data generated by the quantum circuits or measurements can be delivered to larger classical computation and communication systems, e.g. for use as cryptographic keys or as part of a much larger computation. At this point, processing is completely decoupled from the quantum network.
* __Connection teardown__: After completion of the quantum network service requested by the application or larger IT service (e.g., encrypted classical connection), resources along the communication path can be recovered; partially complete entangled states are discarded or repurposed, and physical resources reallocated to other connections.
* __Computational resource release__: reserved quantum computational resources are released.
* __Completion__: the end-to-end hybrid quantum+classical service is terminated.

All of the above except those marked real-time and near-real time are almost entirely insensitive to timing issues, except as necessary for the end-to-end service to meet the users' needs. If allocated resources sit unused for extensive periods of time, the service delivery of the network as a whole may be negatively impacted; introduction of proper pricing or admission control may be needed to resolve such issues.

# I. Background Tasks
Network operations include a number of tasks that monitor and maintain the integrity and performance of the network. In the case of a quantum network, uses of the quantum portion of the network can often be deferred until the network is idle or pre-scheduled time slots arrive, in order to minimize the impact on application requests.  Once the quantum operations are begun, of course, they are subject to all of the constraints listed above, but the accompanying classical calculation and inter-node reconciliation can proceed in the background.

Such tasks include:

* __Link monitoring__: Each link must be monitored continuously in order to inform routing (below) and RuleSet creation during connection setup.  Reconstruction of the link density matrix and entanglement success rates involve classical information sharing between the two nodes at opposite ends of the link. This information must be shared reliably but does not have hard real-time constraints, as so is well suited to transmission over a reliable protocol such as TCP without concern for delays. The required classical information is the outcomes of measurements of the quantum portion of the link. That data can be collected from entangled states specifically assigned to the link monitoring task.  It can also be collected from application-targeted uses of the link, provided that appropriate coordination can be achieved and connection privacy maintained.
* __Routing__: Creation and update of routing tables at each node is an ordinary, distributed classical task that shares the information collected about links as above. The expected completion time of this tasks should be quick enough that the network converges to provide seamless service upon topology changes.  Unless nodes are mobile, propagation and recalculation of such changes at the level of seconds should be acceptable.
* __Malicious use monitoring__: It is known that a hijacked or malfunctioning repeater can be used to impede the overall service of the network or even to partition the network. It is also known that QKD-derived monitoring of the network using randomly selected measurement bases on a portion of the network capacity can serve as a detection mechanism for this malicious behavior.

## References

[1] C.K. Hong, Z.Y. Ou, and L. Mandel, Measurement of subpicosecond time intervals between two photons by interference, [*Phys. Rev. Lett.* __59__, 2044 (1987)](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.59.2044).  
[2] A. M. Branczyk, Hong-Ou-Mandel Interference, [*arXiv:1711.00080* (2017)](https://arxiv.org/abs/1711.00080).  
[3] V. Krutyanskiy _et al._, Entanglement of Trapped-Ion Qubits Separated by 230 Meters, [*Phys. Rev. Lett.* __130__, 050803](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.130.050803) (2023).  
[4] R. H. Hadfield, Single-photon detectors for optical quantum information applications, [*Nature Photonics* __3__, 696](https://www.nature.com/articles/nphoton.2009.230) (2009).  
[5] Single Quantum, [Link](https://singlequantum.com/wp-content/uploads/2022/12/SQ-General-Brochure.pdf).  
[6] D. Leibfried, R. Blatt, C. Monroe, and D. Wineland, Quantum dynamics of single trapped ions, [*Rev. Mod. Phys.* __75__, 281 (2003)](https://journals.aps.org/rmp/abstract/10.1103/RevModPhys.75.281).  
[7] C. Gardiner, and P. Zoller, The Quantum World of Ultra-Cold Atoms and Light: The Physics of Quantum-Optical Devices, [Imperial College Press, (2015)](https://www.amazon.co.jp/Quantum-World-Ultra-Cold-Atoms-Light/dp/1783266163).  
[8] J.I. Cirac, and P. Zoller, Quantum Computations with Cold Trapped Ions, [*Phys. Rev. Lett.* __74__, 4091 (1995)](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.74.4091).  
[9] F. Schmidt-Kaler *et. al.*, Realization of the Cirac–Zoller controlled-NOT quantum gate, [*Nature* __422__, 408 (2003)](https://www.nature.com/articles/nature01494).  
[10] K. Molmer, and A. Sorensen, Multiparticle Entanglement of Hot Trapped Ions, [*Phys. Rev. Lett.* __82__, 1835 (1999)](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.82.1835).  
[11] J. Benhelm *et al.*, Towards fault-tolerant quantum computing with trapped ions, [*Nature Physics* __4__ 463 (2008)](https://www.nature.com/articles/nphys961).  
[12] J. Altepeter, D.F.V. James, and P.G. Kwiat, Qubit quantum state tomography, [*Lecture Notes in Physics* __649__, 113 (2004)](https://link.springer.com/chapter/10.1007/978-3-540-44481-7_4).
