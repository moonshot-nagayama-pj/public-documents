Summary of timing regimes:

A. interferometric stabilization: need for sub-wavelength stability is photonic qubit representation-dependent  
B. photon wave packet overlap: technology-dependent photon wave packet length, but roughly nanoseconds  
C. opening and closing of detector itming windows, detector recovery time: nnoseconds to microseconds  
D. measurement basis selection (if required in BSA): performance will contraint entanglement attempt rate  
E. optical switch ocntrol: switching of trains of wave packets  
F. pre-configured event-driven tasks such as timing-triggered or measurement-triggered execution of quantum circuits: microseconds  
G. urgent but not synchronous-critical tasks (e.g. execution of classical code that processes RuleSet messages and selects or creates new quantum circuits for execution): milliseconds  
H. host-side application-level tasks (e.g. post-measurement operations): milliseconds  
I. bckground tasks (link tomography calculations, routing table updates): seconds to minutes  

Some of these can only be achieved using high-quality hardware, while others are software tasks. Detailed analysis of these regimes will affect core software design in each network node type.
