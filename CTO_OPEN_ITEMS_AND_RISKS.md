# CTO Decision Brief: Serial Build, Tight Validation, and Risk Controls

This version reflects your feedback:
- no MVP/V1 framing,
- no multi-user roadmap,
- no audio-effects roadmap,
- serial component delivery with small, regression-friendly steps.

## Legend
- **[DECISION REQUIRED]** = you need to pick/approve.
- **[LOCKED]** = already decided.
- **[RISK FLAG]** = monitor + mitigate.

---

## 0) Plain-Language Terms (clarified)

### Batch export queue (definition)
A feature where you line up multiple exports (e.g., many clips/regions/presets) and render all automatically.  
**Status:** future roadmap item (export multiple outputs separately), not current execution scope.

### “Premiere-competitive export” (definition)
Not effect presets. It means export reliability + control:
- predictable successful renders,
- alpha/transparency preservation,
- color-space correctness,
- useful professional format controls.

### “Tested ingest set” (definition)
A living list of media types we actively verify (container+codec+resolution+alpha).  
Yes—this should evolve based on real uploads and incident reports.

### “Transcode” (definition)
Create an editing-friendly copy when source media is hard to preview smoothly.  
Original remains untouched.

### “Fast header sanity” + “background deep probe”
- fast check: quick metadata/container sanity,
- deep probe: deeper corruption/edge-case scan in background.

### FPS reduction + selective preview bypass
- FPS reduction: lower preview frame rate under load,
- selective bypass: temporarily skip heaviest effects in preview only (final export still full chain).

---

## 1) Decisions and Commitments

### [LOCKED] Premiere-competitive export is required
Definition for execution: robust professional export controls + reliability, with alpha-safe paths and strong preflight coverage.

## 1.1 Scope + Product Boundaries
- **[LOCKED]** No multi-user editing roadmap.
- **[LOCKED]** No audio effects/post-processing; audio is ancillary to video and usable only as control signal where applicable.
- **[LOCKED]** Do not use MVP/V1 buckets for execution; use serial modular stages.

### [LOCKED] 1.1-A Blend behavior policy (updated)
You clarified:
- do **not** automate switching between blend modes,
- but do allow automation of blend-related continuous controls (e.g., opacity/dry-wet-like intensities).

**Recommendation:**
1. Lock blend-mode selection as discrete/manual (non-automatable for now).
2. Allow automation on blend intensity/opacity parameters.
3. Revisit mode-switch automation only after render-order parity is proven stable.

### [DECISION REQUIRED] 1.1-B Scope guardrails to prevent complexity compounding
Recommended non-goals for current execution window:
1. batch export queue,
2. collaborative editing,
3. deep audio FX/post chain,
4. blend-mode switching automation.

## 1.2 Ingest + Validation + Portability

### [LOCKED] 1.2-A Upload size policy
Soft warning + hard safety limit approach approved.

### [LOCKED] 1.2-B Tested ingest set
Approved direction.  
Additional requirement: maintain a **living ingest compatibility registry** that updates from production upload telemetry + failures.

### [LOCKED] 1.2-C Validation downstream contract (approved)
Validation is now approved to feed:
1. ingest gate (accept/reject/warn),
2. transcode recommendation engine,
3. warning system (corrupt-risk tags),
4. export preflight hints.

### [LOCKED with clarification] 1.2-D Freeze/flatten behavior
- Freeze must always support unfreeze (non-destructive).
- Flatten is destructive/printed.
- Keep frozen artifacts locally cached with lifecycle policy.
- Collect-and-save includes needed project media for transfer.

---

## 2) Untested Assumptions → Concrete Tests

## 2.1 Readable Assumption Test Plan

### A1 CPU-first remains viable for medium sessions
- Test: 45-min benchmark (6 tracks, masks, automation).
- Pass: dropped frames and frame-time stay inside budget.
- Recommendation: keep CPU-first with perf regressions required per stage.

### A2 40% preview floor is enough
- Test: A/B 40% floor vs lower floor on heavy scenes.
- Pass: users still trust preview decisions.
- Recommendation: keep 40% floor; add FPS/bypass fallback.

### A3 Freeze UX is understandable
- Test: uncoached task test (freeze/unfreeze/flatten).
- Pass: high completion without confusion.
- Recommendation: preflight summary + explicit affected-effect list.

### A4 Hybrid history is deterministic (expanded as requested)
- Test design:
  1. Generate 500 deterministic action traces (edits/effects/automation/freeze).
  2. Inject crashes at random checkpoints.
  3. Reopen from autosave/history replay.
  4. Compare restored project hash + key rendered frames vs expected.
- Pass criteria:
  - 99.5%+ trace replay parity,
  - zero “silent state drift” on never-thin actions,
  - no orphan references in recovered projects.
- Recommendation:
  - keep hybrid model,
  - enforce never-thin event classes,
  - add periodic full checkpoints every N actions or T minutes.

## 2.2 Render-Order Truth Table: simulation + mitigation

### Failure simulation examples
1. **Mask/Blend ordering mismatch** between preview and export → alpha edge artifacts.
2. **Group-vs-track effect order drift** → different keying/compositing result.
3. **Freeze render path order mismatch** → unfreeze/refreeze alters look unexpectedly.

### Mitigation stack
1. Publish canonical render-order truth table (single source of truth).
2. Use one shared evaluator path for preview/freeze/export where possible.
3. Maintain golden compositing fixtures (alpha/mask/group edge cases).
4. Block merges when parity diffs exceed tolerance.

## 2.3 Warning taxonomy + centralized threshold config (requested)

### Fixed taxonomy wording
- **INFO**: advisory only, no immediate risk.
- **WARN**: degraded performance/quality risk, user should mitigate soon.
- **CRITICAL**: high probability of failure/stall; strongly recommend action.

### Centralized threshold config contract
- Store thresholds in one versioned config file (e.g., `config/perf_thresholds.yaml`).
- Include:
  - warning trigger equations,
  - per-severity thresholds,
  - hardware profile overrides,
  - release version metadata.
- Process:
  1. benchmark each milestone,
  2. tune thresholds,
  3. publish changelog for threshold updates.

### Milestone benchmark + warning telemetry loop
For each milestone:
1. run canonical benchmark suite,
2. collect warning count/rate/mitigation-click telemetry,
3. compare against previous milestone deltas,
4. approve/reject milestone based on trend + gates.

---

## 3) Serial Roadmap in 21 Small Stages (inclusive, UX-flywheel biased)

This sequence is reordered to optimize user feedback loops, confidence, and testability—not just technical layering.


1. **Project shell + logging foundation**
2. **Media ingest basic path + immediate export baseline (full project + selected region)**
3. **Ingest validation layer (fast checks)**
4. **Deep probe async scanner + warning hooks**
5. **Timeline base model + single clip playback sync**
6. **Transport controls + scrubbing correctness**
7. **Multi-track layering + z-order rules**
8. **Core edit ops: split/duplicate/move**
9. **Snap/markers/loop/in-out controls**
10. **History/undo-redo event model**
11. **Autosave + crash recovery determinism**
12. **Effects rack skeleton (on/off, reorder, params)**
13. **Freeze/unfreeze non-destructive pipeline**
14. **Flatten destructive print path + safety prompts**
15. **Group tracks + nested group behavior**
16. **Render-order truth table implementation + parity tests**
17. **Mask v1 + key/color utility stack**
18. **Automation lanes + write/overdub basics**
19. **Performance track + MIDI input/capture basics**
20. **Export hardening + portability (collect-and-save + relink + alpha guarantees)**
21. **Advanced mask path boolean editor**

### What is NOT included in these 21 stages
- multi-user collaboration,
- batch export queue (future: multi-output separate renders),
- deep audio FX/post tooling,
- blend-mode switching automation,
- advanced mask path-boolean editor.

### Stage-level validation rule (same for all 20)
- unit/integration checks for stage behavior,
- regression fixture update,
- deterministic reopen check if state model changed,
- benchmark + warning telemetry delta check for performance-sensitive stages.

---

## 4) CTO Risk Register (3 mitigations each)

## R1 Scope creep
1. signed non-goal contract,
2. intake rubric with explicit “not now,”
3. structured testing sprints after each stage.

## R2 Performance instability
1. stage-level perf gates,
2. mitigation UX shipped early,
3. continuous benchmark automation.

## R3 Recovery drift
1. never-thin event class enforcement,
2. checkpoint cadence,
3. crash-replay CI gate.

## R4 Debuggability collapse
1. strict subsystem boundaries,
2. trace IDs for event/frame pipelines,
3. debug inspector for render order + degradations.

## R5 Render-order ambiguity
1. canonical truth table,
2. golden alpha/mask/group fixtures,
3. preview/freeze/export parity gate in CI.

## R6 Export reliability risk
1. preflight validation,
2. alpha-safe defaults,
3. release-candidate export matrix tests.

## R7 Throughput risk
1. one owner per stage,
2. weekly risk burndown,
3. cap concurrent deep-work streams.

---

## 5) Immediate decisions to confirm next
1. Confirm 20-stage sequence.
2. Confirm non-included list.
3. Confirm blend policy (automate intensity yes, mode switching no).
4. Confirm validation downstream contract.
5. Confirm warning taxonomy + centralized thresholds config process.


---

## 6) Holes, UX gaps, and next features (recommendations)

## 6.1 Likely holes to close now
1. **Command discoverability gap**
   - Add inline shortcut hints + searchable command palette.
2. **State visibility gap**
   - Always-visible “why playback degraded” indicator with active mitigation states.
3. **Freeze state ambiguity**
   - Track-level and chain-prefix freeze badges directly in rack/timeline.
4. **Render-order opacity**
   - Inspector panel showing exact resolved order for selected track/group.

## 6.2 Cohesion upgrades for UX flywheel
1. **First-run guided project** with one sample clip and a guided split->effect->freeze->export journey.
2. **One-click stress test button** that runs lightweight benchmark and gives optimization suggestions.
3. **Session health ribbon** summarizing CPU/cache/warnings with actionable recommendations.

## 6.3 Next features after Stage 21
1. **Batch export queue** (separate output jobs by region/preset/profile).
2. **Effect macro snapshots** (save/recall rack states quickly).
3. **A/B compare lanes** (rapid before/after plus variant audition).
4. **Template projects** (performance, masking, chroma workflows).
5. **Operator preset packs** for modulation/sidechain quick-starts.

## 6.4 Most innovative opportunities (practical)
1. **Video-reactive modulation templates** (luma/rgb/motion pre-mapped to key params).
2. **Auto-suggest freeze points** driven by per-effect cost profiling.
3. **Adaptive preview advisor** that predicts stutter and recommends minimal-impact mitigations before playback drops.
