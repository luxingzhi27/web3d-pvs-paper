# 05 — Shared Conservative Boundary Learning

## Section goal

Explain why the paper needs more than a high-AUC classifier.

If every new scene requires visibility labels to calibrate a decision threshold, the “compile geometry and deploy” claim is weakened. V5 therefore trains:

\[
z=0
\]

to have a shared conservative meaning.

## 5.1 Fixed zero boundary

Define:

\[
h_{keep}(z)=\frac{\operatorname{softplus}(z)}{\ln2},
\]

\[
h_{miss}(z)=\frac{\operatorname{softplus}(-z)}{\ln2}.
\]

At zero:

\[
h_{keep}(0)=h_{miss}(0)=1.
\]

Decision:

\[
z\ge0\Rightarrow\text{keep}.
\]

The goal is not probability calibration; it is a stable deployment boundary shared by heterogeneous scenes.

## 5.2 Extra-retention objective

For source scene \(s\):

\[
J_{extra}^{(s)}
=
\frac{
\sum_{i:y_i=0}q_i h_{keep}(z_i)
}{
G_s
}.
\]

where \(q_i\) corrects pose sampling and \(G_s\) is the source-train visible-occurrence total.

Interpretation:

> penalize unnecessarily retained invisible candidates.

This is the efficiency side.

## 5.3 Unified safety risk

Scene mean positive visual weight:

\[
\mu_s=W_s/G_s.
\]

Blended positive weight:

\[
\tilde w_i
=
\rho+(1-\rho)\frac{w_i}{\mu_s}.
\]

Current formal configuration:

\[
\rho=0.1.
\]

Safety risk:

\[
R_s
=
\frac{
\sum_{i:y_i=1}
q_i\tilde w_i h_{miss}(z_i)
}{
G_s
}.
\]

Interpretation:

- the count floor protects small visible units;
- visual weights prioritize visually important visible units;
- a single risk avoids maintaining separate count and visual decision boundaries.

## 5.4 Domain-robust aggregation

Risk domains:

- five real scenes;
- five synthetic structure families.

Total:

\[
K=10.
\]

Maintain EMA risk:

\[
m_s\leftarrow\beta m_s+(1-\beta)R_s,
\qquad \beta=0.99.
\]

SmoothMax weights:

\[
p_s=\operatorname{softmax}(\alpha m_s),
\qquad \alpha=16.
\]

Training problem:

\[
\min_\theta J_{extra}
\quad\text{s.t.}\quad
R_{robust}\le0.01.
\]

Lagrangian:

\[
\mathcal L
=
J_{extra}
+
\lambda(R_{robust}-0.01).
\]

There is one shared multiplier \(\lambda\).

## 5.5 Current formal objective versus retired designs

Current robust-boundary runs:

- do **not** use field NLL;
- do **not** read external-hit probes;
- keep the structured survival-field representation;
- use fixed-zero evaluation as the main deployment protocol.

Older design notes that describe field-supervised objectives are historical and should not define the final paper.

## 5.6 PBCE control

`PBCE_OBJECTIVE`:

- keeps the Full representation;
- changes only the task objective to pose-balanced BCE;
- does not add extra field supervision.

Scientific question:

> Is a conventional balanced classification objective sufficient to produce a scene-independent conservative zero boundary?

The paper should compare:

- fixed-zero WR / LCB;
- calibrated diagnostics;
- PR-AUC;
- score distributions;
- training dynamics where stable.

This isolates boundary learning from representation quality.

## Training protocol to report

Current formal shared training:

- 5 real source scenes;
- 96 synthetic train scenes;
- two real steps per one synthetic step;
- 36k updates per real scene;
- 90k synthetic updates;
- 270k total updates;
- 4 poses per step;
- global yaw augmentation;
- AdamW;
- model LR 2e-4;
- weight decay 1e-5;
- robust dual settings from the frozen run config.

Only frozen run-config values should enter the final manuscript.
