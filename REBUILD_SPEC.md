# Glitch DAW Rebuild Spec (Scratch Plan)

## 0) Product Intent
A **video-first DAW for glitch manipulation**: fast timeline editing, real-time-ish preview, stacked effects, modulation/automation, and reliable export.

- Primary user: solo creator on **Mac M4** (dogfooding first).
- Scope: get users far enough in experimental manipulation before round-tripping to full NLEs.
- Philosophy: **CPU-first with abstraction for future GPU acceleration**, non-destructive editing, and explicit performance safety rails.

---

## 1) Current Agreed Feature State (Formalized)

## 1.1 Media Ingest + Validation
- Import paths:
  - Drag/drop onto preview.
  - Drag/drop onto timeline.
  - File → Import.
- Auto track behavior:
  - First clip goes to first track.
  - Additional dropped clips create new tracks if needed.
- Validation gates before import:
  - Corrupt container/stream rejection.
  - Unsupported codec rejection.
  - Oversized file warning with recommendation flow.
- Required file handling outcomes:
  - Clear reject reasons.
  - “Proceed anyway” path where technically possible.

## 1.2 Timeline + Playback Core
- Multi-track timeline (single-track was only early mental model).
- Layer order: top track = frontmost visual layer.
- Same-track overlap disallowed; cross-track overlap allowed.
- Playback:
  - Preview + timeline playhead fully synchronized.
  - Seamless play/pause transport at top-center.
  - JKL shuttle, frame-step support.
- Scrubbing:
  - Playhead scrub with synced preview (audio-aware).
- Timeline zoom:
  - Drag gesture on timeline ruler/strip for zoom in/out.
- Selection model:
  - click, shift-click chaining across app.
  - Region selection distinct from clip selection.

## 1.2A Workspace Layout + Panel UX
- Pane boundaries/dividers are resizable (Ableton/Photoshop/Premiere-inspired interaction model).
- Core layout supports drag-resize between timeline, preview, and rack/panel zones.
- Preview controls include:
  - fullscreen toggle.
  - pop-out toggle to detachable window.
- Pop-out preview window is movable and resizable.
- Transport includes top-center play/pause control (in addition to shortcuts/shuttle).
- Loop state/control remains visible from top transport surface (not timeline-only).

## 1.3 Clip Editing + Navigation
- Split clip: `Cmd+E` (selected clip at playhead/selection boundary).
- Duplicate: `Cmd+D`.
- Clip drag with magnetic snapping by default.
- Snap bypass modifier while dragging (e.g., Shift).
- Visual snap indicator line.
- Fade-in / fade-out on clip (`F` toggle for fade handles).
- Crossfades for adjacent/sliced clip transitions.
- Moving a split region to a new lane auto-creates destination track when absent.
- Markers:
  - Add markers on timeline.
  - Click marker to jump and play from marker.

## 1.4 Looping + In/Out + Render Region
- Loop toggle: `Cmd+L`.
- Highlight timeline region then loop between brackets.
- Loop state visible in top transport and timeline styling.
- In/Out points for focused playback/export.
- Render to In/Out.

## 1.5 Track + Group Model
- New track creation:
  - `Cmd+T`.
  - Track header context menu: create track.
  - Auto-create track when dropping new media where needed.
- Track controls:
  - mute/solo.
  - opacity.
  - blend mode.
- Grouping:
  - Nested groups supported.
  - Group-level mute/solo/opacity/blend.
  - Group-level effect chains.
- Track lock behavior:
  - Lock implies non-editable timeline state.
  - Default policy: lock freezes the track to reduce CPU (future preference toggle optional).

## 1.6 Effects Rack + Chain Semantics
- Effects live in bottom rack (Ableton-inspired).
- Effects browser/library at side panel.
- Effect module requirements:
  - enable/disable button.
  - drag handle for reorder.
  - parameters with labels + units.
  - dry/wet control.
- Chain behavior:
  - stacked and reorderable.
  - chain is non-destructive by default.
- Grouping effects:
  - select multiple effects + group (`Cmd+G` or context action).
- Freeze/Flatten:
  - **Track-level freeze/flatten required**.
  - Partial-chain freeze allowed only as contiguous prefix (if freezing middle implies earlier included).
  - Attempting invalid mid-chain freeze shows warning specifying affected effects.
  - Unfreeze restores prior chain state.
  - Flatten prints permanently.

## 1.7 Utility + Color + Keying + Masking
Utility effects category includes:
- Chroma key (range/selective keying).
- Luma key / brightness threshold transparency.
- Color correction stack:
  - hue, saturation, contrast, brightness.
  - curves and color controls expected for pro baseline.
- Blend and transparency controls.
- Mask tool (V1):
  - shape masks (rectangle/ellipse/polygon baseline).
  - feather.
  - invert.
  - transform/position controls.
  - clip vs track attachment modes.
  - overlay visualization states (outline, shaded mask, alpha matte preview).

## 1.8 Automation + Modulation + Performance
- Automation lanes toggle: `A`.
- Keyboard musical input toggle: `M`.
- Per-lane dropdown model:
  - select effect.
  - select parameter.
- Automation authoring:
  - double-click nodes.
  - pencil/draw mode.
  - shape tools (Ableton-like).
  - copy/paste/move automation segments.
- Write automation by performance pass:
  - record button captures live parameter moves.
  - overdub multiple passes.
- Conflict policy:
  - if direct automation is written to a parameter currently LFO-modulated, modulation link becomes invalid/disconnected per policy.
- Modulation operators:
  - LFO (multi-parameter and multipole style direction).
  - envelope/gate sidechain controls.
  - mapping to multiple parameters.
  - modulation-of-modulation allowed with conservative caps.
- Constraint:
  - one modulation operator per parameter, except grouped contexts where controlled aggregation is allowed.

## 1.8A Audio Handling Boundary
- Audio remains attached to source video clips for sync/reference editing.
- Video effects do not process/alter audio content in MVP/V1.
- Audio may be used as a control/modulation signal source where specified (sidechain/reactivity).
- No audio effects/post-processing roadmap in current execution scope.

## 1.9 Sidechain (Expanded)
- Sidechain sources:
  - video-derived metrics.
  - audio-derived metrics.
  - MIDI trigger/gate.
  - multimodal combinations.
- Sidechain controls:
  - threshold.
  - lookahead.
  - ADSR.
  - source-specific parameterization.
- Recommended initial video metrics:
  - luma average, luma percentile.
  - RGB channel averages.
  - saturation level.
  - motion magnitude (frame delta).
  - edge density.
- Audio metrics (if enabled for control only, not processing FX):
  - RMS/peak envelope.
  - band-limited energy (selectable range).

## 1.10 Performance Track (Separate Type)
- Dedicated track type for live triggering/performance workflows.
- Inputs:
  - computer keyboard.
  - external MIDI device.
- Features:
  - drum-rack-like pad mapping to layers/effects.
  - gate and one-shot modes.
  - ADSR envelope.
  - choke groups.
  - per-patch polyphony settings.
  - MIDI learn/mapping to knobs and pads.
- MIDI Capture:
  - buffer captures last pass.
  - Capture action retro-inserts missed performance if record wasn’t armed.

## 1.11 Preview System
- Real-time preview target for smooth editing.
- Dynamic resolution scaling allowed but never below **40%**.
- Split-view preview mode (before/after) with explicit indication of compared effect/stack.
- Fullscreen and pop-out preview windows:
  - pop-out movable/resizable.

## 1.12 Undo/Redo + History + Autosave + Project Files
- Undo: `Cmd+Z`.
- Redo: `Cmd+Shift+Z`.
- History panel (Photoshop-like):
  - latest action on top.
  - revert to any prior state.
  - stores parameter state changes.
- Autosave:
  - hybrid action + time model.
  - no-idle churn.
- Project persistence:
  - save/open project files.
  - double-click project file opens app into project state.
  - portable project format.
  - “Collect All and Save” mode for transferable asset bundles.

## 1.13 Export
- Export routes:
  - right-click region/selection export.
  - File → Export (full project).
- batch export queue is future-scope (separate job exports), not current execution scope.
- Export expectations:
  - reliable, professional-grade controls.
  - strong codec/container options (not necessarily full Premiere parity at v1).
  - in/out and loop-region aware.
- alpha/transparency preservation is a first-class requirement in preview/freeze/export paths.

---

## 2) Architecture Decisions (Current Direction)

1. **Engine**: CPU-first non-destructive graph-like pipeline with future GPU backend abstraction.
2. **Preview strategy**: full-res attempt + dynamic scaling floor at 40%; proxies held as fallback/hardening path.
3. **Cache strategy**: hybrid cache with strict RAM/disk budgets, prioritized around recently edited nodes.
4. **History strategy**: slim hybrid state + command deltas, preserving parameter timeline without exploding memory.
5. **Freeze strategy**: manual first; optional guided recommendations from warnings; selective prefix-chain freeze allowed.
6. **Modulation bus**: option B-style structured modulation routing with explicit invalidation rules to avoid ambiguity.
7. **Performance CPU protection**: early policy to mitigate thermal throttling via warnings + recommended freeze points.

---

## 3) Concerns / Flags + User-Facing Warnings (with Mitigations)

1. **High automation density warning**
   - Trigger: node count/time exceeds threshold.
   - Message: “Automation density is high on Track X (Param Y).”
   - Mitigation recommendations:
     - Smooth/reduce nodes.
     - Bake selected automation segment.
     - Freeze track prefix.
     - Lower preview resolution.

2. **Render graph pressure warning**
   - Trigger: sustained frame miss / queue growth.
   - Mitigations:
     - disable selected heavy effects.
     - freeze earliest heavy effects in chain.
     - switch preview split mode off.

3. **Memory budget warning**
   - Trigger: RAM cache cap near limit.
   - Mitigations:
     - flush stale cache.
     - freeze inactive tracks.
     - enable proxy fallback for long clips.

4. **Thermal/CPU throttling warning**
   - Trigger: sustained CPU load + detected slowdown.
   - Mitigations:
     - freeze recommendation list by track.
     - reduce preview FPS/res.
     - pause background cache precompute.

5. **Invalid partial freeze warning**
   - Trigger: attempt to freeze non-prefix middle-only subset.
   - Message lists exact effects that must be included.
   - Actions:
     - Apply expanded freeze.
     - Cancel.

6. **MIDI event saturation warning**
   - Trigger: event burst above handling threshold.
   - Mitigations:
     - thin MIDI stream.
     - cap capture density.
     - reduce mapped target count.

All warning dialogs should provide:
- **Recommendations list** (non-destructive guidance).
- **Continue without change** option.
- Optional one-click action shortcuts when target is unambiguous.

## 3.1 Warning Delivery UX Contract (Placement + Interaction)
- Primary warning surface appears in/near the effects area when issue source is effect-chain related (to keep context local).
- Warning card includes:
  - problem summary.
  - likely cause.
  - ranked mitigation recommendations.
  - explicit target references (track/effect names, not “upstream/downstream” jargon).
- Action model:
  - recommendation-first (no forced auto-action on ambiguous targets).
  - explicit **Continue without change** option always present.
  - one-click actions only when affected target is unambiguous.
- Warnings can be collapsed to a session health center to reduce interruption.

---

## 4) Resource Simulation Framework (for 2.5h session on 30-min MP4)

## 4.1 Scenario Variants
- Variant A (Light): 3 tracks, moderate effects, sparse automation.
- Variant B (Medium): 6 tracks, stacked effects, active masking + automation.
- Variant C (Heavy performance): 8+ tracks, live modulation, MIDI capture, sidechain sources.

## 4.2 Metrics to Track
- CPU avg/peak and throttling windows.
- GPU utilization (if/when backend enabled).
- RAM working set, cache occupancy.
- Disk cache throughput + eviction rate.
- Playback FPS stability and dropped frame ratio.
- Automation evaluation time.
- MIDI input event rate and drop count.

## 4.3 Policy Tweaks Agreed
- Reduce prior conservatism by ~25%.
- Dynamic scaling floor fixed to 40%.
- Automation considered in revised envelope.

## 4.4 Validation Output Required
For each variant:
- Expected steady-state envelope.
- Spike profile (seek, scrub, heavy effect burst).
- Warning trigger points.
- Recommended fallback order.
- Known failure modes and user-safe recovery paths.

## 4.5 Simulated Resource Envelopes (Revised with Automation + 25% Less Conservative)

Assumptions used for this simulation pass:
- Session length: 2.5 hours.
- Source: 30-minute MP4, mixed glitch stack complexity.
- Machine target: Mac M4-class local dogfood setup.
- Preview scaling floor enforced at 40%.
- Automation lanes active in medium/heavy variants.

### Variant A — Light Session (3 tracks, moderate FX, sparse automation)
- Estimated steady-state:
  - CPU: 28–42% avg, 55% peak during scrub bursts.
  - RAM working set: 5–8 GB.
  - Disk cache: 8–16 GB active.
  - Playback stability: ~0.2–0.8% dropped frames in dense seek windows.
- Trigger likely:
  - short spikes during repeated seek/scrub + split-view preview.
- Recommended fallback order:
  1) disable split preview,
  2) reduce preview resolution,
  3) freeze inactive tracks.

### Variant B — Medium Session (6 tracks, stacked FX, masks, active automation)
- Estimated steady-state:
  - CPU: 48–68% avg, 82% peak under concurrent automation redraw + scrubbing.
  - RAM working set: 10–16 GB.
  - Disk cache: 24–48 GB active with frequent churn.
  - Playback stability: ~1.2–3.5% dropped frames before mitigation.
- Trigger likely:
  - automation density + mask operations + split-view + live param tweaks.
- Recommended fallback order:
  1) smooth/reduce automation nodes on hottest lanes,
  2) freeze prefix of heavy chains,
  3) pause background cache precompute,
  4) lower preview FPS/res while keeping >=40% scale floor.

### Variant C — Heavy Performance Session (8+ tracks, live modulation, MIDI capture, multimodal sidechain)
- Estimated steady-state:
  - CPU: 65–88% avg, 95% peak during capture+overdub+sidechain bursts.
  - RAM working set: 16–24 GB.
  - Disk cache: 48–96 GB active with high eviction pressure.
  - Playback stability: ~3.5–8.0% dropped frames without mitigation; can return to ~1.5–3.0% with aggressive fallbacks.
- Trigger likely:
  - simultaneous MIDI bursts, modulation-of-modulation updates, sidechain fusion, and mask-heavy sections.
- Recommended fallback order:
  1) freeze inactive/locked tracks first,
  2) reduce active modulation target count,
  3) temporarily disable split preview and noncritical heavy effects,
  4) use warning-recommended stabilization actions before continuing.

### Failure Modes to Proactively Handle in this Envelope
- CPU thermal throttling causes delayed UI feedback and timing drift risk in live capture.
- Cache thrash introduces stutter even when average CPU appears acceptable.
- Dense automation can saturate evaluation scheduler before render threads saturate.

### Instrumentation Required to Validate These Estimates
- Per-track effect cost telemetry.
- Automation evaluation time budget counters.
- MIDI input rate/drop counters.
- Cache hit/miss and eviction-rate dashboard.
- “Preview degraded” state with explicit active degradations.

---

## 5) Gaps / Open Decisions / Holes

1. Exact import limits:
   - max file size behavior (hard cap vs warning-only).
   - codec whitelist and fallback transcode policy.
2. Precise project file schema/versioning.
3. Cache budget numbers for M4 baseline (RAM %, disk GB).
4. Autosave cadence constants (action count + minutes).
5. Final modulation conflict precedence table (automation vs operator vs macro).
6. MIDI maximum event rate target and graceful degradation thresholds.
7. Sidechain multimodal fusion formula defaults.
8. Final list of V1 utility effects vs V1.1.
9. Blend automation expansion criteria (mode switching remains excluded; continuous controls may expand).
10. Lock preference UX details (whether freeze-on-lock can be user-toggled in settings).

---

## 6) Quality-of-Life / Table-Stakes Checklist (to include in v1 planning)
- Zoom to selection.
- Snap indicators + bypass modifier.
- Marker management.
- In/out + render in/out.
- Clip fades + crossfades.
- Track mute/solo/lock.
- Project collect/save portability.
- Keyboard shortcuts discoverability panel.
- Error messaging with plain-language cause + mitigation.
- Crash recovery into latest autosave.

---

## 7) Recommended Build Sequence (Validation-First)

## Milestone 1 — Foundation Playback Editor
- Import/validation, timeline core, preview sync, transport, basic edit ops.
- Validation:
  - long clip ingest, corruption handling, playhead sync, basic edits.

## Milestone 2 — Effects Chain + Freeze/Flatten
- Effect rack, reorder, on/off, dry/wet, utility baseline, freeze/flatten semantics.
- Validation:
  - deterministic output by chain order, freeze correctness, unfreeze restoration.

## Milestone 3 — Automation + History
- Automation lanes/tools, undo/redo, history panel, autosave hybrid.
- Validation:
  - stress test node density + full state restore correctness.

## Milestone 4 — Grouping + Performance Track + MIDI
- Group tracks/effects, performance track, MIDI mapping/capture.
- Validation:
  - live pass capture integrity, overload handling, mapping persistence.

## Milestone 5 — Sidechain + Advanced Mask/Color + Export Hardening
- Multisource sidechain, mask UX, advanced color stack, export robustness.
- Validation:
  - known benchmark projects export reproducibly and re-open safely.

---

## 7A) Serial Component Delivery Mode (Preferred over MVP/V1 Buckets)

Delivery policy update: build in **serial component stages** with short validation loops after each stage, rather than batching many changes into long UAT cycles.

Stage order (UX-flywheel biased):
1. Project shell + logging foundation.
2. Media ingest basic path + immediate export baseline (full + selected region).
3. Ingest validation fast checks.
4. Deep probe async scanner + warning hooks.
5. Timeline base model + single clip playback sync.
6. Transport controls + scrubbing correctness.
7. Multi-track layering + z-order rules.
8. Core edit ops (split/duplicate/move).
9. Snap/markers/loop/in-out controls.
10. History/undo-redo event model.
11. Autosave + crash recovery determinism.
12. Effects rack skeleton (on/off, reorder, params).
13. Freeze/unfreeze non-destructive pipeline.
14. Flatten destructive print path + safety prompts.
15. Group tracks + nested group behavior.
16. Render-order truth table implementation + parity tests.
17. Mask v1 + key/color utility stack.
18. Automation lanes + write/overdub basics.
19. Performance track + MIDI input/capture basics.
20. Export hardening + portability (collect-and-save + relink + alpha guarantees).
21. Advanced mask path boolean editor.

Rules:
- Each stage must have a standalone regression pack before proceeding.
- No stage closes without deterministic reopen checks for impacted state.
- Performance telemetry deltas are reviewed stage-by-stage to normalize warning behavior.
- Structured testing sprint follows every stage close (no bundled multi-feature UAT).

---

## 8) Priority Tiers (MVP / V1 / V2) [Legacy Reference Only]

- Note: active planning mode is Section 7A serial stages; this section is retained only as historical grouping context.

## MVP (legacy grouping; replaced by serial stage delivery)
- Import + validation.
- Multi-track timeline + playback sync.
- Basic clip edit + loop + markers.
- Effect rack (core controls + reorder).
- Freeze/flatten track + prefix-chain rules.
- Export baseline.
- Undo/redo + autosave + project save/open.

## V1 (legacy grouping)
- Group tracks/effects + nested groups.
- Performance track + MIDI mapping + capture.
- Sidechain (foundational metrics + ADSR/lookahead).
- Mask tool v1 + utility color/key stack.
- Warning center with mitigation guidance.

## V2 (legacy grouping)
- GPU backend acceleration path.
- Proxy workflow automation/hybridization.
- Expanded modulation ecosystem.
- Deeper export preset/profile system.
- Extended solo performance tooling (no multi-user collaboration roadmap).

---

## 9) Validation Checkpoints per Milestone

- **Functional**: expected behaviors from acceptance criteria.
- **Performance**: sustained playback target under variant workloads.
- **Reliability**: crash recovery + autosave restoration.
- **State integrity**: project open/save determinism.
- **Render parity**: preview-vs-export visual consistency tolerance.
- **UX safety**: warnings appear before failure and provide clear mitigations.
- **Cadence**: structured testing sprint after each serial stage; avoid multi-feature UAT bundles.

---

## 10) Next Discussion Targets
1. Lock down numeric budgets (cache, autosave cadence, event rates).
2. Finalize file ingest matrix (supported codecs/containers).
3. Define exact sidechain metric formulas and defaults.
4. Define performance track mapping UX flow end-to-end.
5. Draft acceptance criteria tables per milestone feature.

---

## 11) Red-Team Review (Stress-Test All Major Decisions)

Purpose: aggressively challenge assumptions before implementation so we avoid expensive rewrites.

### 11.1 Product Positioning Risks

**Decision under test:** “Glitch DAW, not full NLE.”

- Failure mode: feature creep recreates Premiere + Ableton + Photoshop simultaneously.
- Risk signal: backlog dominated by parity language (“like X app”) without ruthless exclusions.
- Impact: endless v1, unstable core, diluted UX.
- Red-team recommendation:
  - Define explicit **non-goals** for MVP (e.g., no multicam, no full compositing graph editor, no advanced audio post, no multi-user collaboration).
  - Gate new features with: “Does this help glitch iteration speed in-session?”

### 11.2 CPU-First Engine Risks

**Decision under test:** CPU-first with future GPU abstraction.

- Failure mode: architecture ossifies around CPU assumptions and blocks later GPU migration.
- Risk signal: effects API exposes CPU-specific buffer semantics or synchronous execution assumptions.
- Impact: expensive engine rewrite when scaling to heavier stacks.
- Red-team recommendation:
  - Enforce an execution abstraction now (device-agnostic frame op contract).
  - Keep effects deterministic and side-effect free to enable backend switching.
  - Add “backend parity tests” to avoid hidden coupling.

### 11.3 Real-Time Preview + 40% Floor Risks

**Decision under test:** dynamic resolution scaling never below 40%.

- Failure mode: on heavy stacks, 40% floor still misses real-time and user blames instability.
- Risk signal: repeated warning spam + persistent frame drops despite scaling.
- Impact: poor trust; users disable advanced features.
- Red-team recommendation:
  - Add second fallback dimension: frame-rate scaling + selective effect bypass in preview-only path.
  - Show “Preview is degraded” badge with exact active degradations.
  - Keep one-click “restore quality” action.

### 11.4 Freeze/Flatten Semantics Risks

**Decision under test:** prefix-chain freeze allowed, flatten irreversible.

- Failure mode: users misunderstand what is frozen and accidentally commit wrong look.
- Risk signal: high undo churn immediately after freeze actions.
- Impact: confidence loss; history bloat.
- Red-team recommendation:
  - Pre-flight summary: “Will freeze effects 1..N on Track X.”
  - Offer temporary audition render before commit.
  - Snapshot metadata of frozen source for safety rollback (before flatten).

### 11.5 History + Autosave Hybrid Risks

**Decision under test:** slim history + action/time autosave.

- Failure mode: key state not reconstructable after crash due to over-thinning.
- Risk signal: reopened project differs from pre-crash view.
- Impact: catastrophic trust failure.
- Red-team recommendation:
  - Define “never-thin” critical actions (imports, deletes, freeze/flatten, routing changes).
  - Add periodic full-state checkpoints in addition to deltas.
  - Add deterministic replay validator in CI.

### 11.6 Group/Nested Group Complexity Risks

**Decision under test:** nested groups with effects and opacity/blend.

- Failure mode: ambiguous precedence (group blend vs track effect vs clip mask).
- Risk signal: users can’t predict render order; bug reports read as “random output.”
- Impact: support burden + architectural churn.
- Red-team recommendation:
  - Publish strict render order table early.
  - Build “debug inspector” showing resolved stack for selected pixel/track.
  - Cap nesting depth for MVP.

### 11.7 Automation/Modulation Conflict Policy Risks

**Decision under test:** writing automation can invalidate LFO link.

- Failure mode: user perceives “automation deleted my modulation.”
- Risk signal: frequent accidental disconnects.
- Impact: creative flow interruption.
- Red-team recommendation:
  - Replace silent invalidation with explicit conflict chooser:
    1) replace modulation,
    2) suspend modulation in region,
    3) blend with weighted sum.
  - Add conflict markers in lane UI.

### 11.8 Modulation-of-Modulation Risks

**Decision under test:** allowed with conservative caps.

- Failure mode: chaotic, hard-to-debug behavior and nonlinear instability.
- Risk signal: spikes in CPU and “why is this parameter jittering?” tickets.
- Impact: feature viewed as broken/unsafe.
- Red-team recommendation:
  - Hard cap graph depth and update rate.
  - Add cycle detection with clear error.
  - Provide “stability meter” per modulation patch.

### 11.9 Sidechain Multimodal Risks

**Decision under test:** video/audio/MIDI/multimodal sidechain.

- Failure mode: normalization mismatch makes modulation useless (too weak/too extreme).
- Risk signal: users constantly tweak thresholds but no musical/visual control.
- Impact: high complexity, low payoff.
- Red-team recommendation:
  - Start with one standardized 0..1 normalized control signal pipeline.
  - Ship preset source adapters (luma pulse, kick-envelope, MIDI gate) before free-form fusion.
  - Add scope visualization of sidechain signal before mapping.

### 11.10 Performance Track + MIDI Capture Risks

**Decision under test:** dedicated performance track with retro-capture.

- Failure mode: timing drift / quantization ambiguity during capture insert.
- Risk signal: captured pass feels “late” relative to preview.
- Impact: live performance feature becomes untrusted.
- Red-team recommendation:
  - Timestamp events against engine clock, not UI clock.
  - Let user choose capture alignment mode (raw, nearest frame, quantized).
  - Provide post-capture nudge tool.

### 11.11 Mask Tool in V1 Risks

**Decision under test:** Photoshop-like mask baseline in v1.

- Failure mode: mask UX consumes schedule and blocks core DAW reliability.
- Risk signal: disproportionate UI bug volume in masking interactions.
- Impact: delayed shipping of core editing loop.
- Red-team recommendation:
  - Constrain v1 mask scope to 2–3 primitives + feather/invert only.
  - Defer advanced boolean/path editing to V1.1+.
  - Ensure mask operations are fully keyframeable before adding advanced shapes.

### 11.12 Warning System Risks

**Decision under test:** rich warning + mitigation UX.

- Failure mode: alert fatigue; users auto-dismiss everything.
- Risk signal: high warning frequency with no behavior change.
- Impact: mitigations unused, crashes persist.
- Red-team recommendation:
  - Add severity levels + rate limiting.
  - Collapse repeated warnings into a single “health center.”
  - Include “don’t show again for this session” controls.

### 11.13 Export Competitiveness Risks

**Decision under test:** robust export, “competitive with Premiere” direction.

- Failure mode: too many codec controls too early create fragile matrix.
- Risk signal: export failures concentrated in edge codec combinations.
- Impact: support nightmare.
- Red-team recommendation:
  - Launch with curated, battle-tested presets first.
  - Expose advanced controls behind “expert mode.”
  - Add preflight export validator (dimensions, pixel format, alpha compatibility).

### 11.14 File Validation + Large Media Risks

**Decision under test:** corruption checks + large-file handling.

- Failure mode: false negatives on import (valid media rejected) or long blocking scans.
- Risk signal: import appears hung or inconsistent.
- Impact: first-run frustration.
- Red-team recommendation:
  - Two-stage validation: fast header sanity then background deep probe.
  - Allow provisional import with warning while deep probe completes.
  - Cache probe results per file hash.

### 11.15 Project Portability Risks

**Decision under test:** portable project + collect all and save.

- Failure mode: path remapping inconsistencies, missing assets on other machines.
- Risk signal: “project opens but media offline.”
- Impact: collaboration and archival failure.
- Red-team recommendation:
  - Relative-path-first strategy with explicit media manifest.
  - “Verify portability” command before sharing.
  - One-click relink assistant.

### 11.16 Top 10 Pre-Mortem Scenarios (What Could Kill V1)

1. Playback never feels stable on medium projects.
2. Freeze semantics confuse users and create mistrust.
3. History restore is nondeterministic after crash.
4. Modulation routing becomes unintelligible.
5. Sidechain is powerful but unusable due to poor defaults.
6. Mask UI overwhelms core timeline work.
7. MIDI capture timing drift ruins performance workflow.
8. Export matrix is too broad and too flaky.
9. Warning system becomes noisy and ignored.
10. Scope creep from “like Ableton/Photoshop/Premiere” blocks shipping.

### 11.17 Red-Team Countermeasures Checklist (Execution Policy)

- Ship hidden debug panels early (render order, signal scopes, performance counters).
- Add hard budget gates per milestone (max dropped frames, max memory, max crash rate).
- Require golden-project regression suite before adding net-new feature class.
- Keep strict “feature freeze” windows to stabilize architecture.
- Track “time-to-first-glitch” as north-star UX metric.

### 11.18 Recommended Spec Adjustments Now

1. Add explicit non-goals section under Product Intent.
2. Add render-order truth table appendix.
3. Add conflict-resolution UX for automation vs modulation.
4. Add severity/rate-limit policy for warnings.
5. Add deterministic replay/crash-recovery acceptance tests.
6. Narrow V1 mask scope with explicit deferrals.
7. Launch export with curated presets + expert mode toggle.


## 12) Decision Register (What We Still Need to Decide)

Use this as the single checklist to close before implementation.

### 12.1 Product Scope Decisions
1. **MVP non-goals**: explicitly exclude which categories (e.g., advanced compositing, deep audio post, batch export).
2. **Premiere-competitive definition**: curated reliable presets only vs full expert codec controls at launch.
3. **Mask scope in V1**: minimal primitives only vs broader Photoshop-like tooling.

### 12.2 Media Ingest + Project Decisions
4. **Max file size policy**: hard cap, soft warning, or adaptive by codec/resolution.
5. **Supported ingest matrix**: exact codec/container whitelist for MVP.
6. **Transcode policy**: auto background transcode, manual prompt, or no transcode.
7. **Validation mode** (**LOCKED**): fast pre-check + background deep probe with surfaced warnings and downstream hooks (ingest/transcode/warnings/export preflight).
8. **Project format spec**: schema/versioning strategy and forward/backward compatibility policy.
9. **Collect-and-save behavior**: always copy media vs reference-in-place with optional collect.

### 12.3 Playback/Preview/Performance Decisions
10. **Realtime target policy**: prioritize frame rate, quality, or deterministic consistency when constrained.
11. **Preview degradation order**: resolution scaling, FPS scaling, selective bypass ordering.
12. **Dynamic scaling floor lock**: keep hard floor at 40% or allow override in expert settings.
13. **Proxy posture**: disabled by default with fallback prompts vs optional proactive generation.
14. **Performance budgets (M4)**: concrete CPU/RAM/disk thresholds and warning trigger levels.

### 12.4 Timeline + Editing Decisions
15. **Snap policy defaults**: always-on magnet with modifier bypass vs configurable project setting.
16. **Overlap rules edge cases**: behavior on paste/duplicate when conflict occurs on same track.
17. **Track lock semantics** (**LOCKED**): default is lock=freeze; only optional preference toggle remains open.
18. **In/Out + loop precedence**: if both set, export/playback follows which range by default.

### 12.5 Effects/Render Graph Decisions
19. **Render order truth table**: final precedence across track/group/effect/mask/blend operations.
20. **Freeze granularity** (**LOCKED**): prefix-chain freeze is in scope (with explicit affected-effect messaging).
21. **Flatten safety policy**: confirmation UX and checkpoint retention before irreversible commit.
22. **Utility effects MVP list**: exact v1 set vs deferred v1.1 list.
23. **Blend automation policy** (**LOCKED**): blend-mode switching is excluded; continuous blend intensity/opacity controls may be automated.

### 12.6 Automation/Modulation/Sidechain Decisions
24. **Conflict policy**: base behavior is locked (automation write invalidates modulation link); only UX treatment (chooser/markers) remains open.
25. **Modulation depth caps**: max routing depth and update-rate ceilings.
26. **Per-parameter modulation rule**: one source only vs grouped aggregation exceptions.
27. **Sidechain normalization model**: standard 0..1 normalization pipeline definition.
28. **Sidechain source defaults**: which video/audio/MIDI metrics ship in MVP.
29. **Multimodal fusion**: formula/preset strategy for combining heterogeneous sources.

### 12.7 Performance Track + MIDI Decisions
30. **Performance track mapping UX**: routing model from performance controls to targets.
31. **MIDI event-rate limits**: max supported rate + degradation strategy.
32. **Capture alignment policy**: raw timing vs quantized/frame-aligned insert modes.
33. **Computer keyboard mode defaults**: global toggle behavior and focus/typing safety rules.

### 12.8 History/Autosave/Reliability Decisions
34. **History storage model**: delta-only vs hybrid with periodic full checkpoints.
35. **Never-thin actions**: canonical list of actions that must always persist.
36. **Autosave cadence constants**: exact action/time thresholds and idle behavior.
37. **Crash recovery policy**: auto-open latest recovery, version picker, and restore confidence checks.

### 12.9 Warning UX Decisions
38. **Warning severity taxonomy**: info/warn/critical definitions and UI treatment.
39. **Warning rate limiting**: suppression, aggregation, and session-level mute behaviors.
40. **Mitigation execution style**: recommendation-only vs one-click auto-actions when unambiguous.
41. **Proceed semantics label**: exact wording for “continue without change” behavior.

### 12.10 Export Decisions
42. **MVP export surface**: preset-first only vs mixed preset + advanced controls.
43. **Alpha/color management defaults**: output color space and alpha handling policy.
44. **Export validation strictness**: block invalid combos or warn and allow override.
45. **Render range default**: timeline full length vs in/out when set (open; current intent leans in/out when explicitly set).

### 12.11 Governance Decisions
46. **Acceptance gates**: milestone pass/fail metrics (drop frames, crash rate, reopen determinism).
47. **Regression suite scope**: minimum golden projects for release readiness.
48. **Feature-freeze cadence**: stabilization windows between milestone feature expansions.



---

## 13) Conversation-Fidelity Reconciliation (No Meaning Drift)

This section cross-checks the spec against the full conversation and marks what is **locked** vs still **open** so we do not accidentally add/remove intent.

### 13.1 Locked by Conversation (Do Not Reopen Unless Explicitly Changed)
- Multi-track model is required; “single track” was only a scaffolding mental model.
- CPU-first architecture is intentional for dogfooding; keep GPU-first as future optimization path.
- Dynamic preview scaling must not go below **40%**.
- Performance track is a separate track type (Ableton MIDI-track equivalent).
- Modulation bus direction is option-B style (structured routing).
- Hybrid history model should favor slimness while preserving parameter-state reversibility.
- Blend-mode switching automation is rolled back (not in immediate scope); blend intensity/opacity-style continuous controls may be automatable.
- Track/group overlap rule: no overlap on same track; overlap allowed across tracks with top track in front.
- Freeze/flatten must support safe unfreeze (freeze) and permanent print (flatten).
- Batch export queue is a future feature (separate multi-output jobs); current scope prioritizes robust region/full export reliability.

### 13.2 Clarifications Added to Prevent Misreads
- “Not a full NLE” means focus boundaries for v1, **not** removal of requested power features (automation, sidechain, mask, grouping, color stack).
- Audio remains in scope as a **control signal source** (sidechain/reactivity), while heavy audio-FX/post workflows remain out of MVP scope.
- Warning actions are recommendation-forward; user must retain a clear “continue without change” path.
- Freeze recommendations must identify exact affected tracks/effects before action.

### 13.3 Still Open (True Decisions Remaining)
- Numeric thresholds only: exact cache budgets, autosave cadence, MIDI event-rate caps, warning trigger constants.
- Exact ingest matrix: codecs/containers, size policy, transcode policy.
- Exact sidechain defaults: final metric formulas and multimodal fusion presets.
- Export surface details: expert controls depth, alpha/color defaults, strictness policy.
- Governance gates: benchmark project suite and release pass/fail thresholds.

### 13.4 Drift Guardrail
If a future update conflicts with any item in **13.1 Locked by Conversation**, it must be tagged as a deliberate change request (not a silent reinterpretation).


---

## 14) Second-Pass Diff Integrity Audit (No Collapsed Meanings)

This is an explicit re-check to ensure no key decisions from the conversation were accidentally collapsed, weakened, or inverted.

### 14.1 Confirmed Preserved Meanings
- **Multi-track intent preserved**: single-track language remains documented as temporary mental scaffolding only.
- **Performance track separation preserved**: kept as dedicated track type, not merged into regular video tracks.
- **Preview floor preserved**: dynamic scaling floor stays at 40% minimum.
- **Automation + modulation behavior preserved**: direct automation writing over an LFO-mapped parameter still invalidates that modulation link in base policy.
- **Freeze model preserved**: freeze is reversible, flatten is printed/permanent, and prefix-chain behavior is retained.
- **Warning philosophy preserved**: recommendation-first with explicit “continue without change” path.
- **Export scope preserved**: robust reliable export prioritized; batch export remains out of immediate scope.

### 14.2 Potential Drift Points Corrected in This Pass
- Decision Register items that were already settled were mislabeled as open; they are now explicitly marked **LOCKED** (track lock default, freeze granularity, blend mode automation scope).
- Conflict-policy item now states what is fixed vs what is still optional UX refinement.

### 14.3 Remaining Risk of Misinterpretation (Still Open by Design)
- Numerical thresholds (cache/autosave/MIDI/warnings) are intentionally open pending benchmarking.
- Codec/container and transcode policy remain open until ingest matrix is finalized.
- Sidechain defaults and multimodal fusion presets remain open pending practical tuning tests.

### 14.4 Audit Rule
If a future edit changes a **LOCKED** item, include an explicit “Change Request” note with rationale and migration impact.


---

## 15) Last-Third Fidelity Reconciliation (Late-Conversation Coverage)

Focused check: verify late-stage conversation directives are represented in the planning/decision sections and not lost in summarization.

### 15.1 Late-Stage Requirements Confirmed Present
- Build sequencing + milestone validation structure is explicitly documented (M1→M5, with validation notes).
- Priority tiering is present and separated into MVP / V1 / V2.
- Warning system includes mitigation-first guidance and explicit continue-without-change semantics.
- Performance concerns are modeled with simulation variants and fallback order.
- Freeze/flatten semantics include selective-prefix rules and explicit warning behavior for invalid partial freeze attempts.

### 15.2 Clarifications Added for Consistency with Existing Locked Items
- “Open decisions” language now avoids re-opening already-locked lock/freeze direction; only settings-level UX toggle remains open.
- Blend mode automation remains excluded by default and is only discussed as future reintroduction criteria.

### 15.3 Remaining Last-Third Discussion Threads (Intentionally Open)
- Final acceptance metric thresholds (numerical values, not policy shape).
- Export expert-mode depth and default color/alpha policy constants.
- Final performance-track routing UX wireframe details.

### 15.4 Integrity Rule for Future Edits
Any future summary section must reference locked items from Sections 13–14 to avoid accidental “open issue” regressions in later revisions.
