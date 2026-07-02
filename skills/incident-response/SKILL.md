---
name: incident-response
description: Use when production is broken or degraded RIGHT NOW — outage triage, mitigation-first discipline, rollback decisions, incident communication, and blameless postmortems afterwards.
---

# Incident Response

During an incident you are not debugging — you are stopping harm. Root cause is tomorrow's job; mitigation is now's.

## Mitigate first, understand later

- **The order is: stop the bleeding → then diagnose.** Rollback, flag off, scale up, failover, rate-limit — any of these before root-causing. A perfect diagnosis of an ongoing outage is a failed response.
- **"What changed?" is the first question.** Recent deploy, flag flip, config change, dependency/upstream change, traffic shift, cert expiry, quota reset. Most incidents are self-inflicted and recent. Correlated with a change → revert it EVEN IF you can't prove causality yet; proof comes later.
- Rollback hesitancy kills: if rollback is safe (expand-contract migrations make it safe), the bar is "might help", not "proven cause".
- Can't revert? Reduce blast radius: feature flag off, degrade gracefully (serve cached/partial), shed load, isolate the failing dependency (open the circuit).

## During — discipline

- **One incident owner** driving; others investigate assigned threads and report back. Ten people all restarting things = a second incident.
- Communicate on a cadence (even "no update yet, next update in 20 min") — silence generates escalations that consume the responders.
- Capture evidence as you go (timestamps, dashboard screenshots, log queries used) in a running doc — but never block mitigation to collect it. Post-restart, logs and pod state evaporate; `kubectl logs --previous` and snapshots first if cheap.
- Every manual prod intervention (scaled a deployment, edited a configmap) gets written down the moment it's made and reconciled back into IaC/manifests after — snowflake state from incident hands is the seed of the next incident.
- Verify the mitigation actually worked in the metrics before declaring stable — error rate down, latency recovered, queue draining.

## After — blameless postmortem (for real incidents, not every blip)

- Timeline (detection → mitigation → resolution), user impact quantified, root cause**s** — there's rarely one; the trigger + the missing guardrails that let it propagate.
- Blameless = name the system gap, not the person: "deploy lacked a canary gate", not "X deployed a bug".
- Action items: specific, owned, deadlined, and tracked to done — a postmortem with orphaned actions is theater. **Fix the class, not the instance**: the alert that would have caught it earlier, the rollback that would have been faster, the validation that would have blocked it.
- Feed back into config: new alert thresholds (observability skill), new pipeline gate (cicd-releases), new runbook entry (docs).

## Checklist

Mitigated before diagnosed · recent changes checked + reverted on suspicion · one owner, cadence comms · manual changes recorded + reconciled · recovery verified in metrics · postmortem blameless with owned actions.
