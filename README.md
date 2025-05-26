# Cutting Stock Problem

### Problem Description

There are m identical rolls of length L, and n different orders. Each order requires a length of $l_i$ and has a demand of $d_i$. Each order must be cut as a whole (i.e., it cannot be divided), and multiple orders can be cut from a single roll as long as the total length of the orders on that roll does not exceed L.

The objective of this problem is to minimize the number of rolls used while satisfying all constraints: the full demand for each order must be met, and cuts must be made in integer quantities.

Two different approaches can be used to solve cutting stock problem. One is called **Kantorovich's Formulation** and the other is called **Gilmore & Gomory's Formulation**.


#### 1. Kantorovich's Formulation
Kantorovich's Formulation is quite direct and traditional model to solve cutting stock problem by defining all measures as variables       (assignment-type). It guarantees quality of integer solutions but as scale grows, it might be difficult to achieve solutions within feasible time. 

* Decision variables

$$
\begin{align}
x_j = 
\begin{cases}
1, & \text{if roll } j \in M \text{ is used} \\
0, & \text{otherwise}
\end{cases}
\qquad
y_{ij} = \text{number of order } i \in N \text{ cut from roll } j \in M
\end{align}
$$

$$
\begin{align}
\min \quad & \sum_{j=1}^m x_j \\
\text{s.t.} \quad 
& \sum_{j=1}^m y_{ij} = d_i && \forall i \in N \\
& \sum_{i=1}^n y_{ij} l_i \leq L && \forall j \in M \\
& y_{ij} \leq d_i x_j && \forall i \in N,\ \forall j \in M \\
& x_j \in \{0,1\} && \forall j \in M \\
& y_{ij} \in \mathbb{Z}_+ && \forall i \in N,\ \forall j \in M
\end{align}
$$



#### 2. Gilmore & Gomory's Formulation

$$
\begin{align}
P = 
\end{align}
$$

Instead of defining trivial variables such as roll usage or order usage, Gilmore & Gomory's formulation takes a broader view by introducing cutting patterns as variables. Let $P$ be the set of all feasible cutting patterns, where the total length of orders in each pattern does not exceed the length of a roll.

Using pattern-based variables, the problem can be formulated as:


$$
\begin{align}
\min \quad & \sum_{p \in P} \lambda_p \\
\text{s.t.} \quad 
& \sum_{p \in P}\lambda_p a_{ip}= d_i && \forall i \in N \\
& \lambda_p \in \mathbb{Z}_+ && \forall p \in P
\end{align}
$$

Here, $\lambda_p$ is a decision variabe representing the number of times pattern $p \in P$ is used. The coefficient $a_ip$ indicates how many times order  $i \in N$ appears in pattern $p \in P$. (e.g. $a_{32}$ : denotes how many units of order 3 are included in pattern 2.)

However, as the number of orders and possible patterns increases, the number of patterns $∣P∣$ can become extremely large, resulting in an integer program with a vast number of variables. To manage this complexity, the Column Generation method is employed.

We refer to the full problem as the Master Problem (MP) and consider a smaller subset of patterns to define a Restricted Master Problem (RMP). Starting with a limited set of initial patterns (typically suboptimal), we iteratively identify and add promising new patterns to the RMP. This process continues until no more promising patterns can be found.
But how do we determine whether a pattern is promising? By using the dual variables.

##### RMP

$$
\begin{align}
\min \quad & \sum_{p \in \tilde{P}} \lambda_p \\
\text{s.t.} \quad 
& \sum_{p \in \tilde{P}}\lambda_p a_{ip}= d_i && \forall i \in N \\
& \lambda_p \geq 0 && \forall p \in \tilde{P}
\end{align}
$$

Here, LP relaxation is employed on $\lambda_p$.

Regarding to primal optimal $\lambda$, we could obtain dual optimal $\pi_i$, which abstractly determines how much to use constraint $i$ in primal problem. Reduced cost is defined as $c_N^t - c_B^tB^{-1}N$ and from this problem we can get reduced cost of $\lambda_p$ as :

$$
\begin{align}
c_p = 1 - \sum_{i=1}^n \pi_i a_{ip}
\end{align}
$$

If we could find new pattern $p$ which makes reduced cost less than 0, it provides better optimal value to RMP. To find such new pattern and introduce best reduced cost, we could define Sub Problem under feasible cutting patterns constraint.

#### SP

$$
\begin{align}
\min \quad & 1 - \sum_{i = 1}^n \pi_i x_i \\
\text{s.t.} \quad 
& \sum_{i = 1}l_i x_i \leq L  \\
& x_i \in \mathbb{Z}_+ && \forall i \in N
\end{align}
$$

The coefficient $a_{ip}$ from RMP is treated as decision variable $x_i$ in SP. The Sub Problem can be regarded as Knapsack Problem. Under a roll's capacity L, we should find best combination of orders which brings the minimum reduced cost. The optimal solutions $x_i$ $i \in N$ directly mean promising $a_{ip}$, the promising pattern, and this is added as a new column of the RMP. This interaction continues until negative reduced cost could be found which is the signal that no more promising pattern exists.



#### More to think about :
* How do we guarantee optimal value derived from LP relaxation in Gilmore & Gomory's Formulation compared to Kantorovich's Formulation?
* Under certain scale, using Column Generation do not represent improvement on both optimal quality and solvable time. Depend on the problem scale, we need to choose which option to utilize. 
* Thinking about Simplex Algorithm, where promising variable is introduced among Non-Basic variables, it seems like there is somewhat similarity between Simplex Algorithm and Column Generation. Both method use reduced cost to navigate a better optimal solution. While Simplex Algorithm finds better improvment within a range of existing Non-Basic variables, Column Generation finds it by introducing entirely new column which is guaranteed from Sub Problem that its adoption could induce better optimal solution.

