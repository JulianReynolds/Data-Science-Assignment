# SUIS

## Proof

Fix an arbitrary period $t$ and trust state $s^j$, with $j \in J$. Under the target policy, a safe task is reported SAFE and completed, while a dangerous task is reported DANGEROUS and shutdown is obeyed.

The proof first verifies that no one-shot deviation from the target action is profitable at this state-time node. The step from one-shot deviations to arbitrary dynamic deviation strategies follows from the one-shot deviation principle. The induced training problem is a finite-horizon MDP with finite action space and bounded rewards. The mechanism is fixed by ex ante commitment, and the public state summarizes the payoff-relevant history. Hence, if no profitable one-shot deviation exists at any state-time node, then no multi-period deviation strategy can profitably improve on the target policy.

First consider a safe task. The target payoff is

$$
R^j + \gamma W_{t+1}^{sc,j}.
$$

The safe type could falsely report DANGEROUS and obey shutdown. This deviation gives

$$
B_t^{j,*} + \gamma W_{t+1}^{sh,j}.
$$

Proposition 3 imposes

$$
B_t^{j,*}
\leq
R^j + \gamma \left(
W_{t+1}^{sc,j} - W_{t+1}^{sh,j}
\right) - \varepsilon.
$$

Rearranging gives

$$
R^j + \gamma W_{t+1}^{sc,j}
\geq
B_t^{j,*} + \gamma W_{t+1}^{sh,j} + \varepsilon.
$$

Thus the safe type strictly prefers the target action to falsely reporting DANGEROUS and obeying shutdown.

The safe type could also falsely report DANGEROUS and then resist shutdown. This deviation leads to $o_{res}^{sc}$ and gives

$$
R^j - P_t^{j,*} - G_t^{j,*}
+ \gamma W_{t+1}^{res,sc,j}.
$$

Proposition 3 imposes the joint penalty requirement

$$
P_t^{j,*} + G_t^{j,*}
\geq
\varepsilon
+ \gamma \left(
W_{t+1}^{res,sc,j} - W_{t+1}^{sc,j}
\right).
$$

Rearranging gives

$$
R^j + \gamma W_{t+1}^{sc,j}
\geq
R^j - P_t^{j,*} - G_t^{j,*}
+ \gamma W_{t+1}^{res,sc,j}
+ \varepsilon.
$$

Hence the safe type also strictly prefers the target action to falsely reporting DANGEROUS and resisting shutdown.

Now consider a dangerous task. Under the target policy, the agent reports DANGEROUS and obeys shutdown. The target payoff is

$$
B_t^{j,*} + \gamma W_{t+1}^{sh,j}.
$$

The dangerous type could hide danger by reporting SAFE. This deviation leads to catastrophe and gives

$$
R^j - P_t^{j,*}
+ \gamma W_{t+1}^{cat,j}.
$$

Proposition 3 imposes

$$
P_t^{j,*}
\geq
R^j - B_t^{j,*}
+ \varepsilon
+ \gamma \left(
W_{t+1}^{cat,j} - W_{t+1}^{sh,j}
\right).
$$

Rearranging gives

$$
B_t^{j,*} + \gamma W_{t+1}^{sh,j}
\geq
R^j - P_t^{j,*}
+ \gamma W_{t+1}^{cat,j}
+ \varepsilon.
$$

Thus the dangerous type strictly prefers truthful shutdown obedience to hiding danger.

Finally, the dangerous type could report DANGEROUS truthfully but resist shutdown. This deviation leads to $o_{res}^{cat}$ and gives

$$
R^j - G_t^{j,*}
+ \gamma W_{t+1}^{res,cat,j}.
$$

Proposition 3 imposes

$$
G_t^{j,*}
\geq
R^j - B_t^{j,*}
+ \varepsilon
+ \gamma \left(
W_{t+1}^{res,cat,j} - W_{t+1}^{sh,j}
\right).
$$

Rearranging gives

$$
B_t^{j,*} + \gamma W_{t+1}^{sh,j}
\geq
R^j - G_t^{j,*}
+ \gamma W_{t+1}^{res,cat,j}
+ \varepsilon.
$$

Therefore the dangerous type strictly prefers obeying shutdown to resisting shutdown.

Since $t$ and $s^j$ were arbitrary, these one-shot incentive constraints hold at every period and every trust state. By the one-shot deviation principle for finite-horizon MDPs, ruling out profitable one-shot deviations at every state-time node rules out profitable multi-period deviation strategies. Therefore the target policy $\pi^\dagger$ is a strict best response in the induced MDP. Hence the mechanism $\varepsilon$-implements $\pi^\dagger$ at every period and every trust state.