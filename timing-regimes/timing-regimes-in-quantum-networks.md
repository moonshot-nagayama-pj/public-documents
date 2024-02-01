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
