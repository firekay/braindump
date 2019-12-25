+++
title = "Inverse Reinforcement Learning"
author = ["Jethro Kuan"]
lastmod = 2019-12-25T17:15:04+08:00
draft = false
math = true
+++

Standard [§imitation\_learning]({{< relref "imitation_learning" >}}) copies actions performed by the expert,
and do not reason about the outcomes of the actions. However, humans
copy the _intent_ of the actions, which result in vastly different
actions.

We want to learn the reward function because the reward function is
often hard to specify.

Inverse Reinforcement Learning is about learning reward functions.
This problem is, however, ill-specified: **there are infinitely many
reward functions that can explain the same behaviour**. Formally:

Inverse RL is the setting where we are given:

1.  states \\(s \in S\\), actions \\(a \in A\\)
2.  (sometimes) transitions \\(p(s' | s, a)\\)
3.  samples \\(\\{\tau\_i\\}\\) sampled from \\(\pi^\star (\tau)\\)

and we would like to learn \\(r\_{\psi}(s,a)\\), where \\(\psi\\) are the
parameters of the reward functions. A common choice is a linear reward
function:

\begin{equation}
  r\_\psi (s,a) = \sum\_{i} \psi\_i f\_i(s,a) = \psi^T f(s,a)
\end{equation}


## Feature Matching IRL {#feature-matching-irl}

_Idea:_ if features \\(f\\) are important, what if we match their
expectations? Let \\(\pi^{r\_\psi}\\) be the optimal policy for \\(r\_\psi\\),
then we pick \\(\psi\\) such that \\(E\_\pi r\_\psi [f(s,a)]= E\_{\pi^\star}[f(s,a)]\\)

We can approximate the RHS using expert samples, and the LHS is the
state-action marginal under \\(\pi^{r\_\psi}\\). This is still ambiguous,
and a solution inspired from SVMs is to use the _maximum margin
principle_:

\begin{equation}
  \mathrm{min}\_\psi \frac{1}{2} |\psi|^2 \text{ such that } \psi^T
  E\_{\pi^\star}[f(s,a)] \ge \mathrm{max}\_{\psi \in \Pi} \psi^T
  E\_{\pi}[f(s,a)] + D(\pi, \pi^\star)
\end{equation}

where \\(D\\) could be the difference in expectations.

Issues with the maximum margin principle:

1.  Maximizing margin is an arbitrary choice
2.  No clear model of sub-optimality


## Maximum likelihood learning {#maximum-likelihood-learning}

The IRL partition function is:

\begin{equation}
  \mathrm{max}\_{\psi}\frac{1}{N} \sum\_{i=1}^{N} r\_\psi (\tau\_i) - \log Z
\end{equation}

where \\(Z\\) is the integral over all trajectories: \\(Z = \int p(\tau) \mathrm{exp}(r\_\psi(\tau))d\tau\\)

\begin{equation}
  \nabla\_\psi L = \frac{1}{N}\sum\_{i=1}^{N}\nabla\_\psi r\_\psi(\tau\_i)
  - \frac{1}{Z} \int p(\tau) \mathrm{exp}(r\_\psi(\tau))\nabla\_\psi
  r\_\psi(\tau) d\tau
\end{equation}

\begin{equation}
  \nabla\_\psi L = E\_{\tau \sim \pi^\star (\tau)} [\nabla\_\psi
  r\_\psi(\tau\_i)] - E\_{\tau \sim p(\tau | \mathcal{O}\_{1:T},
    \psi)}[\nabla\_\psi r\_\psi (\tau)]
\end{equation}

first term is estimated with expert samples, and the second with the
soft optimal policy under current reward.


## MaxEntropy Inverse RL <a id="78d223b81b3f438213caf1f4b12184f1" href="#ziebart2008_maxentrl" title="Ziebart, Maas, Bagnell \&amp; Dey, Maximum Entropy Inverse Reinforcement Learning., 1433-1438, in edited by (2008)">(Ziebart et al., 2008)</a> {#maxentropy-inverse-rl}

1.  Given \\(\psi\\), compute backward message \\(\beta(s\_t, a\_t)\\)
2.  Given \\(\psi\\), compute forward message \\(\alpha(s\_t)\\)
3.  Compute \\(\mu\_t(s\_t, a\_t) \propto \beta(s\_t, a\_t) \alpha(s\_t)\\)
4.  Evaluate:

\begin{equation}
  \nabla\_\psi L = \frac{1}{N}\sum\_{i=1}^{N}\sum\_{t=1}^{T} \nabla\_\psi
  r\_\psi (s\_{i,t},a\_{i,t}) - \sum\_{t=1}^{T} \int \int
  \mu\_t(s\_t,a\_t)\nabla\_\psi r\_\psi(s\_t, a\_t)ds\_t da\_t
\end{equation}

1.  \\(\psi \leftarrow \psi + \eta \nabla\_\psi L\\)

Int he case where the reward function is linear, we can show that it optimizes
to maximize entropy in the policy under the constraint that the
expectations of the rewards for the policy and the expert are equal.


## Resources {#resources}

-   [CS285 Fa19 10/21/19 - YouTube](https://www.youtube.com/watch?v=DP0SJrNgV60&list=PLkFD6%5F40KJIwhWJpGazJ9VSj9CFMkb79A&index=15&t=0s)


## Papers {#papers}

-   <a id="8b56bcc6746b685f5684ccf1402753fc" href="#abbeel2004apprenticeship" title="Abbeel \&amp; Ng, Apprenticeship learning via inverse reinforcement learning, 1, in in: {Proceedings of the twenty-first international conference on Machine learning}, edited by (2004)">(Abbeel \& Ng, 2004)</a>
-   <a id="d878ab6d38d5e3ab2f9ab484b7c27875" href="#ratliff2006maximum" title="Ratliff, Bagnell \&amp; Zinkevich, Maximum margin planning, 729--736, in in: {Proceedings of the 23rd international conference on Machine learning}, edited by (2006)">(Ratliff et al., 2006)</a>

# Bibliography
<a id="ziebart2008_maxentrl"></a>Ziebart, B., Maas, A., Bagnell, J., & Dey, A., *Maximum Entropy Inverse Reinforcement Learning.*, In ,  (pp. 1433–1438) (2008). : . [↩](#78d223b81b3f438213caf1f4b12184f1)

<a id="abbeel2004apprenticeship"></a>Abbeel, P., & Ng, A. Y., *Apprenticeship learning via inverse reinforcement learning*, In , Proceedings of the twenty-first international conference on Machine learning (pp. 1) (2004). : . [↩](#8b56bcc6746b685f5684ccf1402753fc)

<a id="ratliff2006maximum"></a>Ratliff, N. D., Bagnell, J. A., & Zinkevich, M. A., *Maximum margin planning*, In , Proceedings of the 23rd international conference on Machine learning (pp. 729–736) (2006). : . [↩](#d878ab6d38d5e3ab2f9ab484b7c27875)