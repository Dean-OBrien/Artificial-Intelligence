# NLL, Productive Uncertainty and Premature Collapse

NLL is a useful metric for measuring how surprised a model is by the expected answer. A low NLL means the model assigned a high probability to the target, while a high NLL means the target was relatively surprising. This makes sense when the goal is prediction against a known answer, and it is naturally correlated with accuracy in a way that makes lower NLL appear universally better.

The problem is that NLL only describes one part of model behaviour. It tells us how much probability the model assigned to the expected answer, but it does not tell us whether the model had a good representation of the wider solution space, whether it maintained other valid hypotheses, or whether it collapsed onto one answer too early.

A model can exist in four broad states:

- confidently correct
- confidently incorrect
- uncertainly incorrect
- uncertainly correct

Confidently correct is the ideal terminal state when a problem is genuinely resolved. Confidently incorrect is a dangerous failure because the model has collapsed onto a bad answer and suppressed alternatives. Uncertainly incorrect is mostly confusion. The interesting state is uncertainly correct, where the model has not yet collapsed but the uncertainty is distributed across valid or defensible possibilities.

This can be thought of as **productive uncertainty**.

Productive uncertainty is not just uncertainty. A weak model can be uncertain because it does not understand the problem. The important difference is whether the model's uncertainty exists inside a valid solution space.

Conceptually:

\[
\text{Productive Uncertainty}
\propto
\text{Validity}
\times
\text{Diversity}
\times
\text{Uncertainty}
\]

For example, consider two models evaluating three possible hypotheses.

Model A:

```text
A = 0.98
B = 0.01
C = 0.01
```

Model B:

```text
A = 0.40
B = 0.32
C = 0.28
```

If A is the labelled answer then Model A will achieve a much better NLL. But if A, B and C are all genuinely valid interpretations, Model B may contain a better representation of the problem. It has preserved multiple valid paths instead of collapsing nearly all probability mass onto the most expected answer.

This matters because novelty does not necessarily come from the highest probability path. If B or C contains the useful insight for a new problem, then training pressure that continuously rewards probability concentration may actively suppress the path required for discovery.

The important distinction is therefore not simply low NLL versus high NLL. High NLL itself is not desirable. What matters is the structure of the probability distribution.

A model might produce:

\[
P(A)=0.40,\quad
P(B)=0.30,\quad
P(C)=0.20,\quad
P(D)=0.10
\]

where A, B and C are valid and D is wrong.

The model is uncertain about the exact answer, but 90% of its probability mass still exists inside the valid solution space. It is locally uncertain while being globally confident that it is somewhere near the correct solution.

This suggests that a more useful metric for reasoning would consider the combined probability assigned to valid hypotheses:

\[
P(V)=\sum_{y\in V}P(y)
\]

where \(V\) is the set of valid semantic outcomes.

A corresponding loss could be:

\[
L_V=-\log\left(\sum_{y\in V}P(y)\right)
\]

This would allow a model to retain uncertainty between several valid possibilities without treating that uncertainty as equivalent to being wrong.

The problem becomes even clearer when considering that NLL is fundamentally measured over tokens, while correctness usually exists at a higher semantic level.

A model may produce several different continuations:

```text
It fell from the table.
It struck the floor.
The impact caused it to break.
It broke when it hit the ground.
```

At the token level these are different paths. Semantically they may all represent essentially the same hypothesis.

The relationship is closer to:

\[
\text{token sequence}
\rightarrow
\text{linguistic expression}
\rightarrow
\text{semantic hypothesis}
\rightarrow
\text{correctness}
\]

NLL operates near the beginning of that chain. Reasoning quality exists much further along it.

This means that evaluating reasoning through NLL can conflate probability concentration with cognitive quality. A model may have a lower NLL because it strongly reproduces the expected continuation, while another model may have a richer and more useful hypothesis space but score worse because its probability is distributed across multiple valid alternatives.

A better evaluation could separate two things.

The first is how much of the model's probability mass exists inside valid hypotheses.

A simple form would be **Valid Hypothesis Mass**:

\[
VHM=\sum_i P(h_i)v(h_i)
\]

where \(h_i\) is a hypothesis and \(v(h_i)\) is its validity.

The second is how much diversity remains inside the valid region. That can be represented as entropy over the valid hypotheses:

\[
H_V=
-\sum_{h_i\in V}
\tilde{P}(h_i)
\log
\tilde{P}(h_i)
\]

where \(	ilde{P}\) is the probability distribution renormalised over the valid hypotheses.

These two measurements separate the four states more clearly:

| Validity | Valid uncertainty | Interpretation |
|---|---|---|
| High | Low | Confidently correct |
| High | High | Productive uncertainty |
| Low | High | Confused |
| Low | Low | Confidently incorrect |

This changes the interpretation of uncertainty.

Uncertainty is not necessarily a failure state. It can be a required intermediate state in reasoning.

A strong reasoning process may need to behave more like:

\[
\text{generate}
\rightarrow
\text{maintain alternatives}
\rightarrow
\text{test}
\rightarrow
\text{eliminate}
\rightarrow
\text{collapse}
\]

The important capability is not simply producing uncertainty or eliminating it. It is controlling when each process happens.

Reasoning can therefore be described as a balance between two operations:

\[
\text{Expansion}
\leftrightarrow
\text{Collapse}
\]

Expansion preserves or generates possible hypotheses. Collapse removes them as evidence increases.

Too much expansion produces noise and indecision. Too much collapse produces brittle certainty and hallucination. A capable model needs both, but more importantly it needs to collapse at the correct time.

This creates a possible explanation for some apparent reasoning failures in LLMs. They may not always be failing because they cannot reason. In some cases they may be collapsing too early.

Current training strongly rewards increasing probability on the expected continuation. For conventional prediction tasks this is correct. But on open-ended reasoning tasks it may create pressure to suppress alternatives before enough evidence exists to justify doing so.

That is largely invisible on standard benchmarks because the benchmark already contains the answer. A model that quickly converges onto the expected answer looks highly capable. But scientific discovery, architecture design, debugging, causal inference and genuinely novel problem solving are different. The useful answer may not initially be the highest probability answer, and the system may need to preserve lower probability but still valid possibilities long enough to evaluate them.

This means intelligence is probably not well described by:

\[
\min \mathrm{NLL}
\]

A more complete description would include something closer to:

\[
\text{hypothesis quality}
+
\text{hypothesis diversity}
+
\text{evaluation quality}
+
\text{collapse accuracy}
+
\text{collapse timing}
\]

NLL is still useful. It measures exactly what it is designed to measure: how much probability the model assigns to the observed target. The mistake is treating that as a complete measure of reasoning quality.

A capable reasoning system should be able to preserve uncertainty across several valid hypotheses, evaluate those hypotheses against evidence, and only collapse into confidence when there is enough justification to do so.

The important state is therefore not permanent uncertainty. It is **productive uncertainty followed by justified collapse**.

If this is correct, then some of the optimisation pressure that makes language models better predictors may also make them worse explorers. Lower NLL can indicate better knowledge and better calibration, but it can also reward earlier probability concentration. For tasks where novelty matters, the ability to delay that collapse may be as important as the ability to eventually arrive at the correct answer.
