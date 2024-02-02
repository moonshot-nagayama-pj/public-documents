Summary of timing regimes:

A. __interferometric stabilization:__ need for sub-wavelength stability is photonic qubit representation-dependent  
B. __photon wave packet overlap:__ technology-dependent photon wave packet length, but roughly nanoseconds  
C. __opening and closing of detector itming windows, detector recovery time:__ nnoseconds to microseconds  
D. __measurement basis selection (if required in BSA):__ performance will contraint entanglement attempt rate  
E. __optical switch ocntrol:__ switching of trains of wave packets  
F. __pre-configured event-driven tasks such as timing-triggered or measurement-triggered execution of quantum circuits:__ microseconds  
G. __urgent but not synchronous-critical tasks (e.g. execution of classical code that processes RuleSet messages and selects or creates new quantum circuits for execution):__ milliseconds  
H. __host-side application-level tasks (e.g. post-measurement operations):__ milliseconds  
I. __background tasks (link tomography calculations, routing table updates):__ seconds to minutes  

Some of these can only be achieved using high-quality hardware, while others are software tasks. Detailed analysis of these regimes will affect core software design in each network node type.

## Hong-Ou-Mandel Effect

Entnaglement distribution in quantum networks is performed by entanglement swapping (ES) on photonic qubits.
Central to photonic ES is the Hong-Ou-Mandel (HOM) effect, regardless of the photonic qubit encoding or of the particular protocol implementing photonic ES.
in this section, we give a brief overview of this effect.

Consider two indistinguishable photons incident on a 50:50 beamsplitter (BS).
One photon is in the input mode _a_, while the pther photon is in the other input mode _b_, as shown in the Figure below.

<p align="center">
  <img src="https://github.com/moonshot-nagayama-pj/playground/blob/main/michal/HOM.png"/>
</p>

There are four possible cases that may occur:
- Case A: photon in mode _a_ is reflected, while photon in mode _b_ is transmitted.
- Case B: both photons are transmitted.
- Case C: both photons are reflected
- Case D: photon in mode _a_ is transmitted, while photon in mode _b_ is reflect.

However, if the photons are completely indistinguishable (same frequency, polarization and arive at the BS at the same time), Cases B and C are never observed.
