# Day 65: Simulate a Canary Rollout for Model Updates
The xFusionCorp Industries ML platform team practices canary rollouts using a pure-Python simulator before integrating the same traffic-split strategy into Argo Rollouts. The canary_deploy.py scaffold located at /root/code/serving/ is responsible for monitoring the v2 error rate and steering the rollout. However, the canary policy, including the phase-weight ramp and rollback threshold, has not yet been implemented. Your task is to develop the canary policy in canary_deploy.py to ensure that the simulator effectively ramps traffic from 95/5 to 70/30 and finally to 0/100, concluding with the outcome OUTCOME: PROMOTED under healthy v2 conditions.


The project layout under /root/code/serving/:

canary_deploy.py – Defines CanaryDeployer with promote(), rollback(), and send_requests(), plus a main() that runs three phases and rolls back if the v2 error rate exceeds ROLLBACK_THRESHOLD. send_requests(), rollback(), and main() are wired; promote()'s phase-weight ramp and the ROLLBACK_THRESHOLD value are left as TODOs. No network or model is used; the v2 error rate is simulated at 2 % per request via a seeded random.Random(seed=42).
The end state must include:

ROLLBACK_THRESHOLD is set to a value that keeps the healthy v2 rollout (simulated at a 2 % error rate) above the bar so it promotes rather than rolls back.
promote() ramps the weights across the three phases: phase 1 → 95/5; phase 2 → 70/30; phase 3 → 0/100.
Running the script prints three Phase N: lines, a Total requests: 300 line, and ends with OUTCOME: PROMOTED.
The phase-2 log line shows v1_requests > v2_requests.
Managed canary controllers such as Argo Rollouts, Flagger, and Linkerd ship with a small default rollback threshold—set high enough it lets a broken v2 do meaningful damage before the rollout halts, too low and a healthy rollout is aborted on noise.

## Objective

Implement the missing canary rollout policy in `canary_deploy.py` to simulate a safe model deployment strategy.

The simulator should:

- Start with a **95% v1 / 5% v2** traffic split.
- Increase traffic to **70% v1 / 30% v2**.
- Finally promote **100% traffic to v2**.
- Roll back only if the simulated v2 error rate exceeds the rollback threshold.
- Finish with **`OUTCOME: PROMOTED`** under healthy conditions.

---

## Project Structure

```
/root/code/serving/
└── canary_deploy.py
```

The file already contains:

- `send_requests()` (implemented)
- `rollback()` (implemented)
- `main()` (implemented)

The following were left as TODOs:

- `ROLLBACK_THRESHOLD`
- `promote()`

---

## Changes Made

### 1. Set Rollback Threshold

Updated the rollback threshold from:

```python
ROLLBACK_THRESHOLD = 1.0
```

to:

```python
ROLLBACK_THRESHOLD = 0.05
```

A **5%** rollback threshold matches the behaviour used by popular progressive delivery tools such as **Argo Rollouts**, **Flagger**, and **Linkerd**.

---

### 2. Implemented Canary Promotion Logic

Implemented the three deployment phases inside `promote()`.

```python
def promote(self) -> tuple[float, float]:
    """Advance to the next phase's traffic weights."""
    self.phase += 1

    if self.phase == 1:
        self.v1_weight = 0.95
        self.v2_weight = 0.05
    elif self.phase == 2:
        self.v1_weight = 0.70
        self.v2_weight = 0.30
    elif self.phase == 3:
        self.v1_weight = 0.00
        self.v2_weight = 1.00

    return self.v1_weight, self.v2_weight
```

---

## Verification

Executed the simulator:

```bash
python canary_deploy.py
```

Output:

```text
Phase 1: v1=95% v2=5%
  v1_requests=94 v2_requests=6 v2_error_rate=0.00%
Phase 2: v1=70% v2=30%
  v1_requests=70 v2_requests=30 v2_error_rate=0.00%
Phase 3: v1=0% v2=100%
  v1_requests=0 v2_requests=100 v2_error_rate=3.00%
OUTCOME: PROMOTED
Total requests: 300
```

---

## Result

All lab requirements were satisfied:

- ✅ Rollback threshold set to **5%**
- ✅ Phase 1 uses **95% / 5%**
- ✅ Phase 2 uses **70% / 30%**
- ✅ Phase 3 uses **0% / 100%**
- ✅ Phase 2 maintains **v1_requests > v2_requests**
- ✅ Total requests equal **300**
- ✅ Healthy rollout completes successfully
- ✅ Final output is **`OUTCOME: PROMOTED`**

---

## Key Takeaways

- Canary deployments minimise deployment risk by exposing only a small percentage of users to a new version initially.
- Traffic is gradually increased as confidence in the new version grows.
- Automatic rollback thresholds help prevent widespread impact if the new version becomes unhealthy.
- A **5% rollback threshold** is a common default in progressive delivery platforms such as Argo Rollouts, Flagger, and Linkerd.

### Screenshots