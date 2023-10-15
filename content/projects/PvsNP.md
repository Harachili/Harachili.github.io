# Exploring the P vs. NP Problem

In the Honors Program for the academic year 2020/2021, under the guidance of Professor [Nicola Galesi](https://scholar.google.com/citations?user=0Ch05b4AAAAJ), I delved into the renowned `P = NP?` problem. This problem, formulated by Stephen Cook of the University of Toronto in the 1970s, seeks to answer a fundamental question: Are efficient deterministic computations sufficient to capture efficient non-deterministic computations? In this context, efficiency refers to algorithms with polynomial time complexity.

## The Challenge Illustrated

To illustrate this concept, let's consider the task of arranging accommodations for a group of four hundred university students when there is limited space available, and only a hundred students can be accommodated in the dormitory. To make things more challenging, the university's rector has provided a list of incompatible student pairs and requested that none of these pairs appear in the final selection. This is an example of what is called an `NP problem` because it is easy to verify if a colleague's proposed choice of a hundred students is acceptable (i.e., that none of the pairs from the rector's list appears in the colleague's list). However, generating such a list from scratch seems so challenging that it appears practically impossible. In fact, the total number of ways to choose a hundred students out of four hundred candidates, denoted as ![equation](https://latex.codecogs.com/svg.latex?%5Ccolor%7Bwhite%7D%5Cbinom%7B400%7D%7B100%7D), exceeds the number of atoms in the known universe!

## The P vs. NP Dilemma

The "P vs. NP" problem was independently formulated by Stephen Cook and Leonid Levin in 1971. It is one of the most significant questions in computer science. It revolves around the quest to determine whether there exist problems whose answers can be quickly verified but require impossibly long times to be solved directly. Problems like the one mentioned above seem to fit this description, but as of now, no one has definitively proven that any of them are as challenging as they appear, meaning that there might be a feasible way to generate answers with the help of a computer.

## Defining Complexity Classes

To formally define the complexity classes `P` and `NP`:

- `P` is the set of languages that can be decided in polynomial time on a single-tape deterministic Turing machine. In other words:  
  
![equation](https://latex.codecogs.com/svg.latex?%5Ccolor%7Bwhite%7DP%20=%20%5Cbigcup%5Climits_%7Bk%20%5Cin%20%5Cmathbb%7BN%7D%7DTIME(n%5Ek))

- "$NP$" is the set of languages that can be decided in polynomial time on a non-deterministic Turing machine. In other words:  
  
![equation](https://latex.codecogs.com/svg.latex?%5Ccolor%7Bwhite%7DNP%20=%20%5Cbigcup%5Climits_%7Bk%20%5Cin%20%5Cmathbb%7BN%7D%7DNTIME(n%5Ek))


Additionally, we introduced the complexity class `coNP` A decision problem `X` is a member of `coNP` if and only if its complement ![equation](https://latex.codecogs.com/svg.latex?%5Ccolor%7Bwhite%7D%5Cbar%7BX%7D) is in the `NP` complexity class. More formally, if `S` is a problem over an alphabet `A`, it belongs to the `coNP` class if and only if ![equation](https://latex.codecogs.com/svg.latex?%5Ccolor%7Bwhite%7DA%5E%7B*%7D%5Cbackslash%20S=S%5E%7Bc%7D) is a problem in the `NP` class.



# My work

In the next brief sections, I will describe the work I have done during the Honour's Program, in particular it will discuss what I researched, going deeper on the last part of the project.  

## Chapter 1: The SAT Problem, TAUT, Proof Systems

In this chapter, I introduced the fundamental concepts of the Boolean satisfiability problem (`SAT`) and tautologies (`TAUT`). I explored what these problems entail and their pivotal roles in computational complexity theory. Additionally, I've delved into the concept of a `proof system`, discussing how these systems work, how usually they are structured and their importance in both mathematical logic and computer science.  
In particular, on proof systems, I've discussed the following topics:  

- Propositional Proof Systems (PPS)
- Deductive Systems
- Refutational Systems


## Chapter 2: "Poly-Bounded" Proof Systems

Chapter 2 delves into the concept of "poly-bounded" proof systems. I've explained what it means for a proof system to be `poly-bounded` and explore its significance within the realm of complexity theory. I've briefly discussed the implications of these systems and their connection to efficient computation.

## Chapter 3: The Cook-Reckhow Theorem

This chapter is dedicated to the Cook-Reckhow theorem, a simple but foundational result in the study of proof complexity. I've written the proof of this fundamental pillar in the theory of computation.

## Chapter 4: Tree-Like RES and Tree-Like RESxor(⊕)

Chapter 4 is an exploration of tree-like resolution (`RES`) and its variant, tree-like resolution XOR (`RESxor`, represented as `⊕`). I started from defining these particular kind of refutation systems, and to give a clear explanation of how this works, I introduced the [Prover-Delayer game from Pudlák and Impagliazzo](https://dl.acm.org/doi/pdf/10.5555/338219.338244)

## Chapter 5: The Pigeon Hole Principle (`PHP`)

In this chapter, I've introduced the Pigeon Hole Principle (`PHP`) and its relevance to proof complexity. The concept is well known and trivial: it states that if `n` objects are placed in m containers, with `n > m`, then at least one container must contain more than one object.

## Chapter 6: The PHP in Tree-Like RES(⊕)

My final chapter ties together the concepts explored in previous sections. I've investigated how the `Pigeon Hole Principle` is employed to analyze and demonstrate the constraints of `tree-like resolution XOR` proof systems. I've, moreover, discussed noteworthy results and insights gleaned from this analysis.  
In particular, I've talked about the proof, which is the actual state of the art as a lower bound for ![equation](https://latex.codecogs.com/svg.latex?%5Ccolor%7Bwhite%7DPHP_%7Bn%7D%5E%7Bm%7D) by [Itsykson e Sokolov](https://logic.pdmi.ras.ru/~sokolov/files/papers/splitting.pdf).  


### Conclusion

In this journey through the fascinating world of computational complexity theory, I have had the privilege of being one of the six individuals elected for the Honour's Program. It's a distinction I hold with great pride, and I've approached the realm of theoretical computer science with great enthusiasm.



