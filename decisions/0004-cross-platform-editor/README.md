---
id: ADR-0004
title: A cross-platform presentation stack for the Bonsai editor
status: Assessment
area: [Ecosystem, Infrastructure]
authors: [glopesdev]
related: [ADR-0002, ADR-0003]
discussions:
---

# ADR-0004: A cross-platform presentation stack for the Bonsai editor

## Summary

Given the UI-agnostic operation model that ADR-0003 decouples from WinForms, this record decides which presentation stack the editor is rebuilt on to become cross-platform. Although ADR-0003 keeps the operation model largely independent of the presentation, the choice of stack is still critical, as rebuilding it is expensive, and operator and visualizer authors across the ecosystem depend on whatever the stack exposes, which is what has made the original WinForms choice so durable. The candidates are Avalonia, Uno, Dear ImGui, an existing code editor as a plugin host, and the WinForms-under-Wine baseline. The cutover to a non-WinForms stack is a breaking change held for the v3 batch. The outcome is pending assessment.

## Context

The editor is bound to Windows through WinForms. ADR-0003 decouples its operation model from WinForms so that the presentation can be replaced without changing it, and this record decides the replacement. The research community that Bonsai serves increasingly works across Windows, macOS, and Linux, while WinForms is Windows-only under modern .NET, so cross-platform reach is blocked by the UI framework rather than the runtime.

Two elements apply regardless of the stack choice. The visualizer surface is the same surface used to render live data, and Dear ImGui is the near-certain choice for it regardless of which shell hosts the editor, since its immediate-mode model increasingly backs the visualizer work. Visualizer previews embedded in the editor are rendered through the same GPU pipeline as the 3D graphics stack, and both need a GL context and a function loader. Choosing them separately would put two of each in one process, so the presentation stack and the graphics backend have to be decided together.

## Criteria

The criteria in [criteria.md](../../criteria.md) that materially bear on this decision, with Cross-Platform Reach added for this decision.

- Cross-Platform Reach, decision-specific: the extent to which the editor runs natively on Windows, macOS, and Linux without a compatibility layer. This is the primary motivation and is weighted highest.
- Ecosystem Continuity Risk: existing workflows and their layout files must keep opening and editing as before.
- Community Trust: a conservative user base judges the transition by whether existing work keeps running and by expanded reach for new users.
- Maintenance Burden: the ongoing cost of tracking a UI stack and its platform backends.
- Effort: reworking the editor is a large one-time cost against a bounded budget.
- Reversibility: a later change of stack should not reach into the operation model or stored workflows. It does not make the presentation investment itself cheap to repeat.

A trade-off matrix is framed below but left mostly to be resolved by assessment. At this stage the record draws the option space.

## Options

The operation model of ADR-0003 is common to every option and is not itself a choice here; the options are the presentation stack chosen over it. Options A, B, and C are native cross-platform toolkits; Option D hosts the editor inside an existing cross-platform code editor; Option E is the baseline. The choice between the two XAML options, A and B, turns on an open question, whether a browser or mobile editor is a committed goal within the v3 horizon. The presentation layer is costly enough to rebuild that the number of target stacks is itself a dominant factor, so a single cross-target codebase and a best-in-class desktop stack are competing goals. Option C answers the same multi-target cost a third way, one immediate-mode layer that runs wherever Dear ImGui runs.

### Option A: Avalonia

Build the presentation over Avalonia, a mature cross-platform XAML UI framework running natively on Windows, macOS, and Linux. Avalonia is proven at the scale of a full IDE through JetBrains Rider, and it has in-ecosystem precedent in the Harp device-configuration applications built on Avalonia 11 and .NET 8, so some contributor familiarity and a shared pattern already exist, even though those applications do not reuse editor infrastructure. On Windows, Avalonia can host a WinForms control through native HWND interop, which offers a transition bridge, so existing WinForms visualizers and property editors could keep running inside the Avalonia editor on Windows while their authors migrate. That bridge is Windows-only and subject to the usual native-embedding limits, so it eases the transition rather than extending those controls cross-platform. The open costs are the from-scratch rebuild of the workflow canvas and the `PropertyGrid` over the editor abstraction, and the XAML data-binding model, which is unfamiliar to the current WinForms-oriented contributors.

### Option B: Uno Platform

Build the presentation over Uno, a cross-platform framework that targets desktop, mobile, and WebAssembly from a single UI codebase, which would additionally open browser-hosted and mobile editors. Because the presentation layer is the expensive part of the rebuild for Bonsai, one codebase across every target is a real advantage over rebuilding the presentation once for desktop and again later for the web, and there is standing community interest in browser and mobile editors. Weighed against that, Uno has a less established desktop-native track record than Avalonia, its WebAssembly target cannot run hardware-bound workflows, so a browser editor is an authoring and monitoring surface rather than a full runtime, and it cannot host legacy WinForms controls, so it offers no Windows transition bridge. Uno uses the WinUI XAML dialect, so the unfamiliar data-binding model is shared with Avalonia.

### Option C: Dear ImGui

Build the shell itself in Dear ImGui, the immediate-mode toolkit already near-certain for the visualizer surface and already integrated in the ecosystem through `Bonsai.ImGui` over `Hexa.NET.ImGui`, so the editor, its visualizers, and the graphics backend share one rendering stack and one GL context. ImHex is an existence proof that an IDE-class interface, with docking, detachable windows, menus, inspector and property panels, theming, and custom fonts, is achievable in pure Dear ImGui, and it ships a WebAssembly build, so the paradigm reaches desktop and the web alike. The distinctive advantage is that the presentation layer is written once and runs on every target Dear ImGui reaches rather than once per stack, and the immediate-mode property and visualizer renderers sit directly on the framework-neutral abstractions the operation model owns. A standalone immediate-mode application has to manage retained editing state by hand, which here is already held in the operation model. The costs are a complete rewrite with no WinForms transition bridge, which is consistent with treating the .NET Framework build as the bridge, weaker accessibility and complex-text polish than a native or XAML stack, and a web target that runs .NET on WebAssembly hosting native Dear ImGui rather than the C++ and Emscripten path ImHex uses, which is less proven and must be verified.

### Option D: Editor as a Plugin

Rather than ship a standalone application shell, host the editing experience inside an existing cross-platform code editor such as VS Code or Zed, as an extension built on the operation model of ADR-0003. The appeal is real: VS Code is already the standard editor for Bonsai C# extensions, Bonsai projects are increasingly multi-language with configuration in Python and analysis in Python or MATLAB, and hosting the workflow editor there would place the whole project in one environment and inherit the cross-platform reach, window management, and extension ecosystem of the host. In practice the primary-editor story is weak. Language-server and other host protocols are text-oriented, so nothing standard assembles a visual editing layer, and the node canvas would be a webview built entirely from scratch, reusing less than any other option since it gains no rendering substrate, not even the structural guidance a Dear ImGui shell provides. It makes editor longevity depend on a third party and a fast-moving web substrate, adding the ongoing cost of tracking an extension API the project does not control, and it is not one target but a separate integration per host, since VS Code and Zed do not share an extension model. As a primary presentation stack it is therefore the weakest option, kept in the matrix for the history of why it was considered and for the downstream opportunities it seeds rather than as a likely winner.

### Option E: Status Quo, WinForms Under Wine

Keep the WinForms editor and reach non-Windows platforms through Wine, a cross-platform compatibility layer that reimplements the Windows API. Functional today and requiring no rework, but the editor is still not a native application on those platforms, as its behavior depends on Wine coverage of the specific Windows APIs the editor uses, and it does not resolve the underlying WinForms coupling. Scored as the baseline.

## Trade-off Matrix

Options as rows, criteria as columns. Cells that need measurement are framed as TBD against the assessment that will resolve them, and the rest are settled by reasoning here. No cell is filled by guesswork. Two must-pass checks sit outside the criteria columns: Ecosystem Continuity Risk, resolved by the decoupling prototype in ADR-0003, and complex-text support, resolved by A03.

| Criterion | A: Avalonia | B: Uno | C: Dear ImGui | D: Plugin Host | E: Wine Baseline |
|-----------|-------------|--------|---------------|----------------|------------------|
| Cross-Platform Reach | TBD -> A01 | TBD -> A01 | TBD -> A01 | TBD -> A01 | not native, via a compatibility layer |
| Ecosystem Continuity Risk | must-pass -> ADR-0003-A01 | must-pass -> ADR-0003-A01 | must-pass -> ADR-0003-A01 | must-pass -> ADR-0003-A01 | low, nothing changes |
| Community Trust | TBD -> A02 | TBD -> A02 | TBD -> A02 | TBD -> A02 | erodes over time, reach stays non-native |
| Maintenance Burden | TBD -> A02 | TBD -> A02 | TBD -> A02 | TBD -> A02 | medium, Wine upkeep |
| Effort | TBD -> A01 | TBD -> A01 | TBD -> A01 | TBD -> A01 | none |
| Reversibility | operation model insulated, WinForms bridge forfeited | operation model insulated | operation model insulated, shared GL stack also unpicked | not self-determined, the host can force a change | not applicable |

## Assessments

The decoupling prototype in ADR-0003-A01 is the shared foundation and is worked first, since it is what every option is built over. A03 is worked next, since a stack that cannot handle complex text is eliminated before any effort is spent prototyping it. A01 then compares the surviving stacks once the operation-model API exists, and A02 covers the two criteria a prototype cannot settle. Assessment reports are added under `assessments/` as each is finalized.

| ID | Question | Method | Kind | Status | Exit Criterion |
|----|----------|--------|------|--------|----------------|
| A01 | How do the candidate presentation stacks compare on the hardest widgets, a `ComponentModel`-based `PropertyGrid` and a slice of the workflow canvas over the editor abstraction, plus cross-platform reach and the decisive per-stack risk, hosting a legacy WinForms control on Windows for Avalonia, and the .NET-on-WebAssembly path and the accessibility of an immediate-mode surface for Dear ImGui? | prototype | scoring | Open | A scoped prototype per surviving stack rebuilding the two hardest widgets over the editor abstraction, an effort estimate to parity, a note on platform coverage, and the per-stack risk check: WinForms hosting for Avalonia, and the .NET-on-WebAssembly Dear ImGui path plus a comparative account of accessibility support for Dear ImGui |
| A02 | For each candidate stack, what is the ongoing cost of tracking it and its platform backends, and how is a move to it likely to be received by the existing user base and by prospective users on non-Windows platforms? | desk-research | scoring | Open | A per-stack account of release frequency, breaking-change history, platform backend count, and project governance, together with a read on community reception gathered from existing users and from prospective users currently blocked by the Windows-only editor |
| A03 | Can each candidate stack display and accept complex text, including input through a platform input method editor and right-to-left scripts, in the workflow canvas and in property values? | prototype | disqualifying | Open | Text entry and display verified on each target platform for input method editor composition and for right-to-left scripts, in a node name and in a property value. A stack that cannot do this is eliminated before the A01 comparison |

## Decision

Pending assessment. The record stays in Assessment while the presentation stacks are prototyped over the ADR-0003 operation model.

## Consequences

Pending the full decision, a few commitments already bear on dependent work. The presentation stack and the graphics backend have to be decided together, since embedded visualizer previews and the 3D graphics stack share one GL context and loader in the editor process. If the presentation stack is Avalonia, existing WinForms controls can be hosted in the editor on Windows as a transition bridge, which is Windows-only and not a substitute for cross-platform reimplementation.

Choosing between Options A and B needs a scope decision this record does not make: whether a browser or mobile editor is a committed goal within the v3 horizon. A single cross-target codebase is worth its cost only if that goal is committed, so the answer belongs to the v3 scope rather than here.

Option D carries downstream opportunities that are a separate story from the primary stack choice, contingent on the outcome of this record and on the scope decision above.

- **Remote and browser editing through a host**: hosting inside a code editor would inherit these, along with distribution, from the host largely for free.
- **A shared web build**: a webview could host the web build of whichever stack is chosen for B or C, rather than a rendering layer of its own.
- **Thin additive integrations over the operation model and the execution boundary**: a workflow preview renderer and a launch-and-debug integration over the Debug Adapter Protocol, alongside the future MCP surface.

## References

- Avalonia, at https://github.com/AvaloniaUI/Avalonia
- Uno Platform, at https://github.com/unoplatform/uno
- Dear ImGui, at https://github.com/ocornut/imgui
- ImHex, an IDE-class application built entirely in Dear ImGui with a WebAssembly build, at https://github.com/WerWolv/ImHex
- `Hexa.NET.ImGui`, the .NET bindings for Dear ImGui, at https://github.com/HexaEngine/Hexa.NET.ImGui
- `Bonsai.ImGui`, the existing ecosystem Dear ImGui integration built over those bindings, at https://github.com/bonsai-rx/imgui
