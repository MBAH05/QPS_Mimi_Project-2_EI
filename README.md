Write a Qiskit function that takes a

\begin{equation}
2^n
\end{equation}

dimensional vector,

\begin{equation}
\phi \in \mathbb{C}^{2^n},
\end{equation}

such that

\begin{equation}
\| \psi \|_2 = 1,
\end{equation}

and outputs a circuit, such that,

\begin{equation}
U|0\rangle_n= \sum_{x=0}^{2^n - 1} \psi_x |x\rangle_n,
\end{equation}

The construction may use any number of ancillas, arbitrary 1-qubit gates and multi-controlled RZ gates, (the number of controlled may be arbitrarily large). No classical bit and measurements allowed.

Expectations:

• Documentation.

• Working Qiskit code for all n and demo for n = 4.

Note: This is a more open-ended problem, there are fairly straightforward, but inefficient ways to do (which are neverthelesss have great education value), but you can also try to aim for something more SOTA. For the very modern approach, I recommend this paper. I’m also happy to recommend other example
