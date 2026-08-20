# Aayush — Mathematical Notebook

## From Complexity Theory to Symmetric TSP, AVL Representation, and the Emerging Search Mathematics

> **Purpose of this document:** Aayush is the mathematical record of the problem-solving path. It is intentionally different from the higher-level APPLE research framework. This file emphasizes definitions, derivations, lemmas, algorithmic objects, proof ideas, experimental quantities, and falsification results.
>
>
> **Epistemic rule:** A statement is called a theorem/lemma only where its mathematical validity follows from the definitions. Experimental observations and conjectures are labeled separately. Where an earlier derivation is only partially preserved in the current record, the reconstruction is explicitly identified rather than presented as a proof that is no longer available verbatim.

---

# 1. Starting Point: Complexity, P, NP, NP-hard, NP-complete

## 1.1 Deterministic polynomial time: $P$

A decision problem belongs to $P$ if there exists a deterministic algorithm whose running time is polynomial in the input size.

Informally,

$$
P = \{L : L \text{ can be decided in time } n^{O(1)}\}.
$$

The precise meaning of "$n$" depends on the encoding length of the input, not merely a natural parameter used informally to describe the instance.

---

## 1.2 Polynomial verification: $NP$

A decision problem is in $NP$ if every yes-instance has a polynomial-size certificate that can be verified in polynomial time.

For an instance $x$ with certificate $y$,

$$
V(x,y)=1
$$

can be checked in polynomial time, and

$$
x \in L \iff \exists y\; V(x,y)=1.
$$

The central point is the distinction between **finding** a certificate and **verifying** one.

---

## 1.3 NP-hard

A problem $H$ is NP-hard if every problem in $NP$ can be reduced to $H$ by an appropriate polynomial-time reduction.

An NP-hard problem need not itself be a decision problem and need not lie in $NP$.

---

## 1.4 NP-complete

A problem is NP-complete if it is both:

1. in $NP$; and
2. NP-hard.

Thus, for a decision problem $D$,

$$
D\in NPC \iff D\in NP \text{ and } D \text{ is NP-hard}.
$$

The classical satisfiability problem $SAT$ is NP-complete.

---

## 1.5 Where TSP sits

The optimization form of the Travelling Salesperson Problem asks for a minimum-cost Hamiltonian cycle and is an NP-hard optimization problem.

The corresponding decision problem is:

> Given a complete weighted graph and a bound $B$, does there exist a Hamiltonian cycle of total cost at most $B$?

This decision version is NP-complete.

That distinction matters throughout the Aayush work:

- **optimization TSP:** find the minimum-cost Hamiltonian cycle;
- **decision TSP:** determine whether a Hamiltonian cycle of cost at most a supplied bound exists.

---

# 2. Travelling Salesperson Problem

## 2.1 Graph definition

Let

$$
G=(V,E,w)
$$

be a weighted undirected graph with

$$
|V|=n.
$$

In the complete symmetric case,

$$
G=K_n,
$$

and

$$
E=\{\{i,j\}:1\le i<j\le n\}.
$$

The symmetry assumption is

$$
w_{ij}=w_{ji}.
$$

---

## 2.2 Number of edges

For $K_n$,

$$
|E|=\binom{n}{2}=\frac{n(n-1)}2.
$$

This quantity is denoted by $m$ in the Aayush construction:

$$
\boxed{m=\frac{n(n-1)}2}.
$$

This is the number of edge records that are later inserted into the AVL representation.

---

## 2.3 Hamiltonian cycle

A Hamiltonian cycle visits every vertex exactly once and returns to the start.

Fix an anchor vertex $A$. A cycle can be represented as

$$
A\to v_1\to v_2\to\cdots\to v_{n-1}\to A.
$$

There are $n$ cycle edges.

A useful search representation is therefore:

- choose the next $n-1$ vertices/edges after the anchor;
- then validate the closing transition back to $A$.

At a partial state

$$
P_k=(A,v_1,\ldots,v_k),
$$

there remain $n-1-k$ new vertex selections before the closing edge.

Define the remaining discrete traversal horizon as

$$
\boxed{\tau_k=n-1-k.}
$$

---

# 3. Brute-Force Ground Truth for Small $K_n$

For small $n$, exact enumeration is feasible.

Fix the anchor $A$ and enumerate every permutation of the remaining $n-1$ vertices.

This gives at most

$$
(n-1)!
$$

candidate orientations for a fixed anchor.

For a candidate cycle

$$
C=(A,v_1,\ldots,v_{n-1}),
$$

its cost is

$$
\operatorname{cost}(C)
=
\sum_{k=0}^{n-2}w_{v_kv_{k+1}}
+w_{v_{n-1}A},
$$

where $v_0=A$.

Then

$$
C^*=\arg\min_C \operatorname{cost}(C).
$$

This exact solution is used as an **external validation oracle** in the later experiments. It is not supposed to enter a candidate-choice rule when that rule is being tested.

---

# 4. Mapping the TSP Edge Universe into an AVL Tree

## 4.1 Rank the edges

Sort the $m$ edge weights:

$$
w_{(1)}\le w_{(2)}\le\cdots\le w_{(m)}.
$$

The rank $r$ refers to an edge's position in this sorted list.

The Aayush representation inserts these ranks/edge records into an AVL tree.

The intended interpretation is:

- low rank = lower edge cost;
- high rank = higher edge cost;
- AVL position = structural coordinate for navigating the ordered cost universe.

The exact numerical cost can be retained as metadata, while the AVL structure is determined by the insertion/rank ordering.

---

## 4.2 AVL invariant

For each node $v$ in an AVL tree,

$$
|h_L(v)-h_R(v)|\le 1,
$$

where $h_L(v)$ and $h_R(v)$ are the heights of the left and right subtrees.

This balance constraint keeps the tree height logarithmic in the number of nodes.

---

## 4.3 AVL height recurrence

The minimum number of nodes needed for an AVL tree of height $h$, under the convention

$$
N(0)=1,
$$

satisfies

$$
N(h)=1+N(h-1)+N(h-2).
$$

This is Fibonacci-like.

With a different height convention (for example, empty tree height $-1$), the base cases shift but the same recurrence structure remains.

The important consequence is the logarithmic height bound:

$$
 h=O(\log n).
$$

---

# 5. Ascending Insertion and the Structural Role of Rank

The edge records are inserted into the AVL in ascending rank order.

A naive ordinary BST receiving ascending input would degenerate to a chain. The AVL rotations prevent that degeneration.

Thus a crucial representational idea is:

$$
\text{sorted edge ranks}
\longrightarrow
\text{balanced structural hierarchy}.
$$

For a fixed number of edge records $m$, the AVL balancing process determines a constrained tree shape. Therefore $n$ determines

$$
 m=\frac{n(n-1)}2,
$$

which strongly constrains the possible AVL height and structural depth pattern.

However, $n$ does **not** determine the optimal TSP traversal sequence, because different weight assignments to the graph edges produce different optimal Hamiltonian cycles even when $m$ and the AVL structure are identical.

---

# 6. Rotation Arithmetic and the Rank/Tree-Shape Connection

One mathematical line of investigation concerned why AVL balancing behaves predictably under monotonically increasing insertion.

The central observation is that sorted insertion repeatedly grows the rightmost path until an AVL imbalance appears, after which rotations restructure the path.

The number of inserted nodes therefore interacts with the balancing process in a way that is strongly related to binary/integer carry patterns.

The useful conceptual summary from the earlier derivation was:

> **AVL rotations under monotone insertion can be viewed through a binary-carry-like arithmetic of the insertion count.**

This explains why changes in $m$ produce structured changes in the final AVL shape instead of arbitrary ones.

### Proof status

The carry/rotation relationship was derived and checked during the earlier Aayush work, but the complete line-by-line scratch derivation is not preserved in the current conversation context. The claim should therefore be treated here as a **reconstructed established result** that should be compared against the original notebook if that source is later recovered.

---

# 7. Depth, Generation, and Structural Coordinates

For a node $v$ in the rooted AVL tree, define its depth by

$$
 d(v)=\text{number of edges from the root to }v.
$$

The root therefore satisfies

$$
 d(\text{root})=0.
$$

Children have depth $1$, grandchildren depth $2$, and so forth.

One early attempt used two coordinates:

$$
 x(v)=\text{root-to-node path length},
$$

and

$$
 y(v)=\text{generation/depth}.
$$

For an ordinary rooted tree these coincide:

$$
\boxed{x(v)=y(v).}
$$

This is important because it means they cannot serve as independent spatial dimensions in a literal 3D geometric interpretation.

That observation remains a mathematical constraint on the later spacetime analogy.

---

# 8. Left/Right Structural Polarity

A provisional orientation coordinate was introduced:

$$
 z(v)=
\begin{cases}
+1,&v\text{ is on the left side},\\
0,&v=\text{root},\\
-1,&v\text{ is on the right side}.
\end{cases}
$$

This was originally motivated by the practical traversal preference for moving into and out of the right-side region.

Later it was given a symbolic temporal interpretation, but mathematically it remains an AVL orientation variable.

---

# 9. Initial Traversal Rules

The initial search heuristic was built around the following preferences.

### 9.1 Start near the low-rank region

Since the left/low-cost region contains the smallest edge ranks, it was considered the preferred starting region.

### 9.2 Prefer downward motion

A candidate that moves downward in the AVL is generally preferred when it preserves feasibility.

### 9.3 Prefer a two-generation hop when useful

When two useful downward candidates exist, the two-generation jump was initially considered before a one-generation jump because it gives greater structural control.

### 9.4 Avoid unnecessary right-spine excursions

Right-side movement was treated as structurally expensive in the heuristic sense.

### 9.5 Return left quickly after entering the right side

If the search is forced to the right side, the next priority is to recover toward the left-side/central region as soon as the feasibility rules permit.

### 9.6 Root as a recovery boundary

A move to the root is permitted when doing so prevents an otherwise unnecessary or dangerous right-side excursion.

### 9.7 L/R flip is soft

An L/R flip may be preferred in a tie, but it is not a primary rule.

### 9.8 Preserve possible optimality

No structural rule is allowed to make a potentially optimal candidate impossible without evidence that the exclusion is mathematically justified.

The central principle is:

$$
\boxed{
\text{Structural rules guide candidate selection; they do not define optimality.}
}
$$

---

# 10. Starting Depth Hypothesis

A preliminary parity hypothesis was:

$$
\text{even AVL height}\Rightarrow d_{start}=2,
$$

$$
\text{odd AVL height}\Rightarrow d_{start}=1.
$$

This is an experimental hypothesis, not a theorem.

It should be tested against larger $K_n$ and multiple independently weighted instances.

---

# 11. $D_{\max}$ as an Emergent Structural Parameter

The early formulation used a maximum hop distance of $2$ generations.

Experiments then produced evidence that:

- a $3$-generation hop can be useful;
- an L/R flip need not accompany every hop;
- the same $n$ can have different useful hop patterns across different realities.

Therefore the correct mathematical role is:

$$
\boxed{D_{\max}=\text{empirical structural parameter}.}
$$

It is not equal to the normalization $c=1$ used in the later spacetime analogy.

The research question is whether

$$
D_{\max}=f(n,\text{AVL structure},\text{instance geometry},\text{reunion constraints})
$$

or perhaps whether a simpler universal law exists.

---

# 12. Online Feasibility State

Let

$$
P_k=(A,v_1,\ldots,v_k)
$$

be the current partial Hamiltonian path.

Let

$$
U_k=V\setminus\{A,v_1,\ldots,v_k\}
$$

be the unvisited vertices.

A candidate $u\in U_k$ is not merely judged by immediate structural cost. After selecting $u$, the search must update whether a completion still exists.

For small instances define

$$
\mathcal F_k(u)=
\{\text{all Hamiltonian completions extending }P_k\to u\}.
$$

Then

$$
\mathcal F_k(u)=\varnothing
$$

means the candidate is immediately impossible under the exact constraints being tested.

This is the mathematical basis for **online Hamiltonian validation**.

---

# 13. Future Solution Space

For each candidate $u$,

$$
\mathcal F_u
$$

represents the remaining feasible solution space.

This became more important than the candidate node considered in isolation.

The crucial question became:

> What structure does choosing $u$ induce in the future solution space?

This led to the shift from node-local heuristics toward candidate-conditioned future manifolds.

---

# 14. Future-State Profiles

Instead of simply counting complete cycles, retain a profile across future time slices:

$$
\boxed{
\mathbf S_u=
[S_u(0),S_u(1),\ldots,S_u(\tau)]
}
$$

where $S_u(r)$ is a measure of the future-state volume at $r$ transitions into the future.

Raw counting was found to be too symmetric in complete $K_n$ graphs, because many candidates have identical combinatorial numbers of completions.

Therefore the later definition used **weighted structural volume**:

$$
S_u(r)=
\sum_{s\in\mathcal F_u(r)}W_{\mathrm{struct}}(s,A)
$$

or its normalized/averaged variants.

The crucial methodological decision was:

$$
\boxed{\text{retain the entire profile, do not collapse it immediately.}}
$$

---

# 15. Contraction Rate

Given a positive future-volume profile $S(r)$, define the discrete contraction rate

$$
\boxed{
C(r)=-\Delta\log S(r)
}
$$

or explicitly,

$$
C(r)=-\log\left(\frac{S(r+1)+\epsilon_s}{S(r)+\epsilon_s}\right).
$$

Interpretation:

- $C(r)>0$: volume contracts;
- $C(r)<0$: volume expands;
- $C(r)\approx0$: local stability.

A second difference gives a curvature-like quantity:

$$
\boxed{\kappa(r)=\Delta C(r).}
$$

This is an analogy to curvature of the future-space profile, not physical spacetime curvature.

---

# 16. Failure of a Universal Contraction Profile

A $K_6$ experiment extracted contraction curves from optimal trajectories and attempted to estimate a universal profile

$$
C^*(u),\qquad u=r/\tau.
$$

The optimal trajectories were highly dispersed.

The learned characteristic profile did not predict unseen optimal transitions well.

Therefore:

$$
\boxed{
\text{No universal fixed contraction trajectory has been established.}
}
$$

This shifted the hypothesis toward **boundary-conditioned trajectory behavior** rather than a predetermined contraction law.

---

# 17. Reunion-Compatible Future Manifold

Define:

$$
\mathcal F_j(r)=\text{all feasible future states after candidate }j,
$$

and a subset

$$
\boxed{
\mathcal R_j(r)\subseteq\mathcal F_j(r)
}
$$

containing states that remain compatible with eventual reunion.

Then define

$$
S_j(r)=|\mathcal F_j(r)|
$$

and

$$
R_j(r)=|\mathcal R_j(r)|.
$$

The reunion-preservation fraction is

$$
\boxed{
\rho_j(r)=\frac{R_j(r)}{S_j(r)}.
}
$$

This is more informative than raw future count because it distinguishes future states that retain the reunion possibility from future states that do not.

---

# 18. Experimental Result for $\rho$

A $K_6$ experiment tested whether the optimal next node simply maximizes instantaneous $\rho$.

At a moderate structural reunion radius, the optimal node was the maximum-$\rho$ candidate in about $65.7\%$ of decisions, substantially above the approximate random baseline of $45.7\%$.

However, the optimal candidate did not always maximize instantaneous $
\rho$.

Therefore:

$$
\boxed{
\text{instantaneous reunion compatibility is informative but insufficient.}
}
$$

The next object became the trajectory

$$
\boxed{
\boldsymbol\rho_j=
[\rho_j(0),\rho_j(1),\ldots,\rho_j(\tau)]
}
$$

rather than a single $
\rho_j(r)$.

---

# 19. Universal Reunion Potential

The universal layer was then represented by a potential

$$
\boxed{H_U(s)\in[0,1]}
$$

for a state $s$.

Conceptually:

- high $H_U(s)$ = more compatible with the universal reunion structure;
- low $H_U(s)$ = less compatible.

The oracle is not an external agent. It is a compact mathematical representation of the universal structure:

$$
\boxed{
\text{Union}\rightarrow\text{Separation}\rightarrow\text{Reunion}.
}
$$

The instance-specific graph/AVL representation is the manifestation of the separation layer.

---

# 20. Universal Separation as Instance Geometry

A central conceptual mathematical hypothesis became:

$$
\boxed{
\text{Universal Separation}
\rightarrow
\text{instance-specific spacetime geometry}
\rightarrow
\text{trajectory/geodesic-like structure}.
}
$$

This says that the geometry should be inferred from the instance representation, rather than the algorithm being given an arbitrary geometric metric unrelated to the TSP.

The spacetime language remains a mathematical analogy.

---

# 21. Candidate Future Oracle Trajectory

For candidate $j$, the universal potential is evaluated over the candidate's future manifold:

$$
\boxed{
\mathbf H_j=
[H_j(0),H_j(1),\ldots,H_j(\tau)]
}
$$

with

$$
H_j(r)=
\frac{\sum_{s\in\mathcal F_j(r)}w(s)H_U(s)}
{\sum_{s\in\mathcal F_j(r)}w(s)}.
$$

This is the potential trajectory induced by the candidate.

---

# 22. Potential → Trajectory → Action → Probability

The current mathematical decomposition is:

$$
\boxed{
\text{oracle potential}
\rightarrow
\text{candidate trajectory}
\rightarrow
\text{action}
\rightarrow
\text{probability}
}
$$

For a temporal kernel $K(r,\tau)$,

$$
A_j=
\sum_{r=0}^{\tau}
K(r,\tau)\Phi(H_j(r)).
$$

A probabilistic choice can then be defined by

$$
P(j\mid s)=
\frac{e^{\beta A_j}}
{\sum_k e^{\beta A_k}}.
$$

Here $\beta$ controls the sharpness of the probabilistic choice. It is not the same $\beta$ used earlier for spatial/reunion focusing; notation should be disambiguated in future formal work.

---

# 23. Experimental Result: Universal Potential Has Signal

A first unseen-$K_6$ oracle experiment used 500 independent weighted realities and 2,500 optimal-path transition decisions.

A fixed universal structural oracle, without edge costs or historical optimal-path frequencies, achieved approximately:

$$
\boxed{\text{Top-1}\approx53.9\%}
$$

and

$$
\boxed{\text{MRR}\approx0.722}
$$

against an approximate random Top-1 baseline near

$$
\boxed{45.7\%}.
$$

A cost-informed immediate-edge baseline was substantially stronger, around

$$
\boxed{73.6\%\text{ Top-1}},
$$

which demonstrates that the universal structural field is not sufficient by itself to reconstruct the cost-dependent optimum.

---

# 24. Horizon Dependence

For the canonical universal-oracle experiment, predictive performance increased as the remaining horizon shrank.

Illustrative Top-1 values by stage were approximately:

$$
20.4\%,\quad21.4\%,\quad37.2\%,\quad48.2\%,\quad100\%.
$$

The final $100\%$ stage is structurally forced when only one candidate transition remains and therefore should not be interpreted as an impressive predictive result.

The useful observation is the general trend:

$$
\boxed{
\text{the universal boundary information becomes more discriminative as the horizon collapses.}
}
$$

---

# 25. Two Frozen Experimental Statements

The current checkpoint contains two important empirical statements:

$$
\boxed{
\text{Universal reunion potential has measurable predictive signal on unseen }K_6.
}
$$

and

$$
\boxed{
\text{Potential alone does not determine optimality.}
}
$$

These statements motivate the main current question:

$$
\boxed{
\textbf{What property of the trajectory through the universal potential converts potential into action?}
}
$$

---

# 26. Forward and Backward Information

Because a Hamiltonian cycle is reversible as a cycle, the solved trajectory can be examined in the opposite direction.

Forward:

$$
U\rightarrow s_1\rightarrow\cdots\rightarrow R.
$$

Backward:

$$
R\rightarrow s_{n-1}\rightarrow\cdots\rightarrow U.
$$

A proposed backward oracle should propagate information from the reunion boundary toward the separated states.

The hypothesis is:

$$
\boxed{
\text{forward oracle}
+
\text{backward oracle}
\rightarrow
\text{agreement}
\rightarrow
\text{action}.
}
$$

---

# 27. Candidate Agreement Functions

Several possible mathematical combinations were identified for experimentation:

### Product

$$
A_{FB}=F\,B.
$$

### Geometric mean

$$
A_{FB}=\sqrt{FB}.
$$

### Harmonic mean

$$
A_{FB}=\frac{2FB}{F+B}.
$$

### Minimum agreement

$$
A_{FB}=\min(F,B).
$$

### Arithmetic mean

$$
A_{FB}=\frac{F+B}{2}.
$$

### Closeness

$$
A_{FB}=1-|F-B|.
$$

### Phase-style agreement

If the oracle outputs are converted into phase-like variables,

$$
\theta_F=\arccos(F),
\qquad
\theta_B=\arccos(B),
$$

then a candidate agreement measure can be

$$
A_{FB}=\frac12\left[1+\cos(\theta_F-\theta_B)\right].
$$

A full bidirectional experiment comparing these formulations was proposed but was interrupted by tool rate limiting before completion. Therefore none of the formulations above is yet empirically validated.

---

# 28. Spacetime Candidate Mathematics

A symbolic Lorentzian architecture was explored with fixed normalization

$$
\boxed{c=1}.
$$

The candidate four-vector is

$$
X^\mu=(t,x,y,z).
$$

A Minkowski-like interval was proposed:

$$
\boxed{
d s^2=d t^2-d x^2-d y^2-d z^2.
}
$$

This is a **candidate abstraction**, not an established invariant of the TSP search.

---

# 29. Provisional Temporal Polarity

A symbolic AVL temporal coordinate was proposed:

$$
 t(v)=
\begin{cases}
+1,&v\text{ lies on the left side},\\
0,&v=\text{root},\\
-1,&v\text{ lies on the right side}.
\end{cases}
$$

Interpretation:

- left = suspended past;
- root = timeless boundary;
- right = present.

This is an analogy for the AVL traversal geometry, not a claim that graph nodes literally inhabit physical past/present time.

The discrete search horizon remains separate:

$$
\tau=n-1-k.
$$

---

# 30. $U(1)$, $SO(2)$, and $SO(1,3)$ as Candidate Mathematics

The following candidate abstractions were explored:

$$
U(1)\sim\text{unity/preserved phase structure},
$$

$$
SO(2)\sim\text{two-dimensional state evolution},
$$

$$
SO(1,3)\sim\text{effective spacetime transformations}.
$$

For $SO(1,3)$, one may write

$$
X'=\Lambda X,
\qquad
\Lambda\in SO(1,3).
$$

The intended long-term question is whether the actual traversal data admit a meaningful Lorentz-like invariant or geodesic structure.

No such structure has yet been proved.

---

# 31. Why $c=1$ and $D_{\max}$ are Different

The fixed normalization

$$
 c=1
$$

is merely the unit scale of the candidate spacetime analogy.

The AVL hop quantity

$$
D_{\max}
$$

is a separate empirical parameter to be discovered.

Thus

$$
\boxed{D_{\max}\ne c.}
$$

The experiments should determine whether $D_{\max}$ is universal, $n$-dependent, instance-dependent, or emergent from the reunion constraints.

---

# 32. Temporal and Spatial Focusing

A combined action-weighting family was proposed:

$$
\boxed{
 w(\tau)=
\frac{1}
{(\tau+\epsilon)^\alpha
[\mathcal S(\tau)+\epsilon_s]^\beta}
}
$$

where:

- $\epsilon$ is the minimum resolvable temporal scale;
- $\epsilon_s$ is the minimum resolvable reunion-space scale;
- $\alpha$ is the temporal focusing exponent;
- $\beta$ is the reunion/future-space focusing exponent.

A natural first discrete choice is

$$
\epsilon=1.
$$

This represents one node transition as the smallest meaningful traversal-time unit.

$\epsilon_s$ remains a modeling scale that must be chosen consistently with the structural normalization.

---

# 33. What the Experiments Have Taught Us About $\alpha$ and $\beta$

Experiments showed that naive candidate definitions of future volume can be symmetric across candidates, making the weighting exponents irrelevant.

This is why the model evolved from:

$$
|\mathcal F_j|
$$

to weighted structural profiles and then to reunion-compatible future manifolds.

The important methodological conclusion is:

$$
\boxed{
\text{An exponent cannot rescue an uninformative state variable.}
}
$$

The underlying geometric quantity must discriminate candidates first.

---

# 34. Greedy in the Universal Sense

A key conceptual refinement is:

> The optimal trajectory may be **greedy with respect to the boundary condition**, not greedy with respect to any individual local feature.

This means we should not assume:

$$
\max S,
$$

or

$$
\min C,
$$

or

$$
\min\kappa,
$$

or

$$
\max\rho.
$$

Instead we seek a function of the **entire future trajectory** that rewards preservation of reunion compatibility.

---

# 35. Closed-Loop Traversal

The search is now conceptualized as a closed-loop dynamical process:

$$
\boxed{
\text{measure}
\rightarrow
\text{construct future field}
\rightarrow
\text{evaluate action}
\rightarrow
\text{choose probabilistically}
\rightarrow
\text{move}
\rightarrow
\text{measure again}
}
$$

The process terminates at

$$
B_n=A.
$$

This is the mathematical/computational version of the Alice–Bob analogy.

---

# 36. What Counts as a Valid Scientific Test

For any new candidate heuristic:

1. Generate independent weighted $K_n$ realities.
2. Choose an anchor explicitly.
3. Compute the exact optimal cycle for training labels/validation.
4. Build the proposed heuristic without using the optimal cycle as an input.
5. Evaluate each candidate at every decision state.
6. Test on held-out realities.
7. Compare against random and classical baselines.
8. Report Top-1, MRR, optimality gap, and horizon dependence.
9. Ablate individual components.
10. Reject any component that fails to add predictive information.

The rule is:

$$
\boxed{\text{Do not protect the hypothesis; try to break it.}}
$$

---

# 37. Important Distinction: Proven, Observed, Conjectured

## Proven from definitions / standard mathematics

- $|E(K_n)|=n(n-1)/2$.
- A Hamiltonian cycle uses $n$ cycle edges.
- Fixing an anchor leaves an $(n-1)!$ brute-force orientation space before accounting for cycle reversal/symmetry reductions.
- AVL balance imposes a height recurrence of Fibonacci type.
- A rooted tree has depth equal to its root-to-node path length in edges.
- The basic feasibility definitions for partial Hamiltonian paths and completion spaces.

## Empirically observed in this project

- Same $n$ does not determine the optimal traversal sequence.
- Three-generation hops can arise.
- L/R flipping is not mandatory.
- Hard traversal restrictions can exclude optimal candidates.
- Raw future-completion counts can be too symmetric.
- Weighted structural future profiles contain more information than raw counts.
- A fixed universal structural reunion potential has measurable predictive signal on unseen $K_6$.
- Potential alone does not determine optimality.

## Current conjectures

- The trajectory through the universal potential is more informative than the instantaneous potential.
- Forward and backward boundary information may be complementary.
- Agreement between them may convert potential into action.
- The universal separation layer may be represented by instance-specific search geometry.
- A geodesic-like mathematical description may eventually emerge.
- Relative phase/interference may improve a probabilistic future-choice model.

None of these conjectures is established.

---

# 38. The Mathematical Direction Going Forward

The next central mathematical object should be a **boundary-conditioned trajectory functional**.

Candidate $j$ generates a future-manifold trajectory

$$
\mathcal M_j(r).
$$

The universal structure assigns potential

$$
H_U(s).
$$

The candidate then inherits a potential trajectory

$$
H_j(r).
$$

The outstanding task is to discover the functional

$$
\boxed{
A_j=\mathcal F\left[H_j(0),H_j(1),\ldots,H_j(\tau)\right]
}
$$

that most reliably identifies the next optimal transition.

The next major candidate is bidirectional:

$$
A_j
=
\mathcal F\left[
H^{\rightarrow}_j(r),
H^{\leftarrow}_j(r)
\right].
$$

The agreement law should be empirically chosen rather than assumed.

---

# 39. Final Mathematical Picture

The current mathematical chain is:

$$
K_n
\rightarrow
E=\frac{n(n-1)}2
\rightarrow
\text{sorted edge universe}
\rightarrow
\text{AVL structure}
\rightarrow
\text{partial Hamiltonian state}
$$

$$
\rightarrow
\text{candidate future manifold}
\rightarrow
\text{reunion-compatible future manifold}
\rightarrow
\text{universal reunion potential}
$$

$$
\rightarrow
\text{future potential trajectory}
\rightarrow
\text{trajectory action}
\rightarrow
\text{probabilistic node selection}
\rightarrow
\text{online feasibility update}
\rightarrow
\text{reunion at the anchor}.
$$

The underlying computational objective remains unchanged:

$$
\boxed{
\text{find the minimum-cost Hamiltonian cycle.}
}
$$

The emerging mathematical question is whether the route to that optimum can be understood as a structured trajectory through a universal reunion-conditioned geometry.

---

# 40. Current Open Problems

1. Define a nontrivial universal reunion potential $H_U$ that does not encode the optimal solution.
2. Derive a mathematically principled backward-propagated reunion potential.
3. Test whether forward/backward agreement improves unseen-instance prediction.
4. Determine the mathematically best agreement law among product, geometric, harmonic, minimum, arithmetic, closeness, phase-style, or alternatives.
5. Determine whether a meaningful relative phase emerges from candidate future-manifold relations.
6. Determine whether QFT-like transformations have any empirical value once an encoding is established.
7. Determine whether $D_{\max}$ has a universal law.
8. Determine whether the same preference hierarchy survives across $K_5,K_6,K_7,\ldots$.
9. Determine whether an effective metric or geodesic structure can actually be inferred from the traversal data.
10. Determine whether the observed universal-layer signal can eventually recover the edge-cost-dependent optimum without directly using edge costs in the choice function.

---

# 41. Closing Principle

Aayush is not a finished algorithm.

It is the mathematical notebook tracking the attempt to discover one.

The governing discipline is:

$$
\boxed{
\text{Let the mathematics decide what survives.}
}
$$

and, more strongly:

$$
\boxed{
\textbf{Do not protect the hypothesis; try to break it.}
}
$$

