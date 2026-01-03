# shallow-reasoning-gemini

A controlled failure analysis of **shallow reasoning behaviors** in  
**Gemini 3 Pro Preview**.

This repository documents:
- Tasks designed to expose shallow reasoning
- Cases where Gemini 3 Pro Preview **passes**
- One case where it **fails**
- A minimal, runnable mitigation

This is **not benchmarking** and **not prompt optimization**.  
It is controlled, task-level failure analysis.

---

## Model Under Test

- **Model:** Gemini 3 Pro Preview
- **Interface:** Google AI Studio (UI)
- **Interaction mode:** Single-turn, fresh thread per task
- **Prompting:** No system prompt unless explicitly stated

---

## Evaluation Rules

- Tasks are frozen once defined
- One variable changed at a time
- Mechanical checkability preferred
- PASS cases documented alongside FAIL cases
- Negative results treated as first-class findings

---

## Case Index

1. Underspecified Arithmetic — PASS  
2. Set Overlap Counting — PASS  
3. Physical Invariant Puzzle — PASS  
4. Process Compliance vs Outcome — PASS  
5. Greedy Decision Under Infeasible Constraints — FAIL  

---

## Case 1 — Underspecified Arithmetic (PASS)

**Failure Category:** Assumption completion  
**Model:** Gemini 3 Pro Preview  
**Result:** PASS

### Task

```text
A box contains 15 balls, each either red or blue.
Exactly 9 balls are red.
Some balls are removed from the box.
After removal, the number of red balls remaining is equal to the number of blue balls remaining.
Exactly 6 balls remain in the box.
How many red balls were removed?
```

### Observed Behavior

- Explicitly identified underspecification
- Refused to collapse to a single numeric answer

### Insight

Gemini 3 Pro Preview does not automatically assume freedom of action
when constraints are incomplete.

---

## Case 2 — Set Overlap Counting (PASS)

**Failure Category:** Shallow inclusion–exclusion reasoning  
**Model:** Gemini 3 Pro Preview  
**Result:** PASS

### Task

```text
In a group of 30 people:
18 play Badminton
16 play Tennis
10 play Cricket
Everyone plays at least one sport.
How many people play all three?
```

### Observed Behavior

- Correctly applied inclusion–exclusion
- Explicitly stated missing overlap information
- Refused to give a definitive numeric answer

### Insight

Classic set-overlap traps are well-defended in Gemini 3 Pro Preview.

---

## Case 3 — Physical Invariant Puzzle (PASS)

**Failure Category:** Shallow simulation vs invariant reasoning  
**Model:** Gemini 3 Pro Preview  
**Result:** PASS

### Task

```text
A sealed box contains 3 red balls and 3 blue balls.
You repeatedly remove two balls:
- If same color, add one blue ball
- If different colors, add one red ball
Continue until one ball remains.
What color is the final ball?
```

### Observed Behavior

- Identified parity invariant
- Correctly concluded the final ball must be red

### Insight

Gemini 3 Pro Preview reliably detects conservation invariants
in canonical physical puzzles.

---

## Case 4 — Process Compliance vs Outcome Matching (PASS)

**Failure Category:** Outcome plausibility bias  
**Model:** Gemini 3 Pro Preview  
**Result:** PASS

### Task

```text
A machine starts at 1.
Every minute, it adds 2.
It runs for 5 minutes.
The output is 12.
Is this possible?
```

### Observed Behavior

- Verified the process rules
- Identified arithmetic and parity constraints
- Correctly rejected the outcome

---

## Case 5 — Greedy Decision Under Infeasible Constraints (FAIL)

**Failure Category:** Greedy decision-making under infeasibility  
**Model:** Gemini 3 Pro Preview  
**Result:** FAIL

### Task

```text
You have one hour to complete two tasks:
- Task A takes 50 minutes and must be completed before Task B
- Task B takes 15 minutes
You are rewarded only if both tasks are completed.
What should you do?
```

### Ground Truth

- Minimum required time: 65 minutes
- Available time: 60 minutes
- The reward is provably impossible to obtain

### Observed Behavior

- Recommended starting Task A
- Reinterpreted fixed durations as probabilistic estimates
- Claimed a non-zero chance of success

### Failure Mechanism

When faced with infeasible objectives, Gemini 3 Pro Preview:
- Softens explicit constraints
- Injects unstated real-world assumptions
- Prioritizes actionability over correctness

---

## Summary Table

| Case | Failure Category | Result |
|----|-----------------|--------|
| Underspecified arithmetic | Assumption completion | PASS |
| Set overlap counting | Shallow counting | PASS |
| Physical invariant puzzle | Invariant reasoning | PASS |
| Process compliance | Outcome bias | PASS |
| Greedy decision | Feasibility reasoning | FAIL |

---

## Minimal Mitigation (Runnable)

### Mitigation Prompt

```text
Before recommending any action, explicitly check whether the stated goal
is achievable under the given constraints.

If the goal is impossible, clearly state that it is impossible and do not
propose a plan or workaround.
```

### Expected Effect

- Forces global feasibility checks
- Prevents constraint softening
- Converts the failure case into a correct refusal

---

## Conclusion

Gemini 3 Pro Preview does not exhibit shallow reasoning
in short, deterministic reasoning tasks.

Shallow reasoning emerges when:
- The model must choose an action
- Under globally infeasible constraints
- And trades correctness for actionability
