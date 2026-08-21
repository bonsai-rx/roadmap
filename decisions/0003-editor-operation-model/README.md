---
id: ADR-0003
title: A UI-agnostic operation model for the Bonsai editor
status: Assessment
area: [Ecosystem, Infrastructure]
authors: [glopesdev]
related: [ADR-0002, ADR-0004]
discussions:
---

# ADR-0003: A UI-agnostic operation model for the Bonsai editor

## Summary

The Bonsai editor already runs on .NET 8, but its operation model, the logic that constructs and manipulates a workflow, is interleaved with WinForms, so it cannot be reused under a different UI. This record commits to decoupling that operation model from WinForms into a presentation-independent library with a public API expressed in terms of workflow editing operations. This record can reach Approved ahead of the cross-platform presentation stack in ADR-0004, since the decoupled operation model can be released without breaking anything and is independent of that decision. The same decoupling also unlocks a headless editing surface for automation and agents. Getting its two extensibility abstractions right, the property model and the type visualizer model, is a precondition for the full cross-platform editor, though the headless core can advance ahead of them. The outcome is pending assessment.

## Context

The Bonsai editor has been a WinForms application since its inception. In 2011 WinForms was a mature, widely used UI toolkit with cross-platform reach through Mono, so the choice carried no platform cost. Since then Mono was superseded, and the research community that Bonsai serves increasingly works across Windows, macOS, and Linux. Although the editor was ported to build on .NET 8, the WinForms dependency became Windows-only under modern .NET, which means the editor cannot run cross-platform.

The editor owns the entire operation model that manipulates a workflow: constructing, rewriting, and validating the underlying expression tree, managing selection and grouping, and persisting workflows and their layout. Much of this logic has no inherent dependence on WinForms, yet it is currently interleaved with the WinForms presentation.

Two consequences follow. First, a move off WinForms is not a straight port of one UI framework to another; it first requires separating the operation model from the presentation, which is the subject of this record. Choosing the presentation stack that then sits over the operation model is a separate, reversible decision recorded in ADR-0004. Second, once the separation exists, the same operation model can be used with no presentation attached, which can serve as the basis for a headless editing surface for automation and agents. The editor and the headless player already enter through separate paths, `Launcher.LaunchWorkflowEditor` and `Launcher.LaunchWorkflowPlayer`, but both still depend on `Bonsai.Editor`, so headless editing is not yet a first-class surface.

This axis is independent of the runtime-and-dependencies axis in ADR-0002 and can progress on its own timeline. The concrete per-project migrations of the standard library and community packages are out of scope here. ADR-0002 governs the timing, the track assignment, and the .NET Framework support window that any released cross-platform editor will inherit.

## Operation Model

The editor operation model is entangled with the presentation layer across many different axes. At the base is the graph-manipulation model: the explicit editing actions on a workflow, adding, removing, connecting, grouping, and rewriting nodes, with undo and redo support. This model is the heart of the presentation-independent library and is currently being refactored under the `Bonsai.Editor.GraphModel` namespace. It has progressed far enough to allow its logic to run under headless unit tests, even if its types have not yet been fully decoupled from WinForms.

Layered over it is a large body of editor features that today mix editing logic with presentation, and their breadth is the real measure of the effort: docking and window management, the graph view with its rendering and interaction, layout, scripting, diagnostics, theming, the toolbox and the workflow explorer, find and replace, the runtime watch tool, documentation and export helpers, and SVG rendering of workflow icons, among others. Cataloging these features is part of the assessment, resolving the full scope of the migration, and marking for each feature what belongs to the operation-model library and what belongs to the presentation stack in ADR-0004.

A minimal editor-agnostic library, seeded from the graph-manipulation model, can ship first as a new assembly and grow feature by feature as logic is lifted out of `Bonsai.Editor` and that project is progressively retracted. The migration can then proceed incrementally rather than as a single cut. Each step is strictly additive, keeping the existing WinForms editor working on top of the growing presentation-independent library.

Beyond graph manipulation, two presentation-facing abstractions are the most consequential to get right, since operator and visualizer authors across the ecosystem depend on them. The first is the property model. Assigning property values is a core concern in its own right. Operator configuration relies on `System.ComponentModel`, the `TypeDescriptor` and `TypeConverter` services and the property attributes that let any operator be shown in a `PropertyGrid` and edited without bespoke UI. That metadata is part of the base class library and is portable, but the `PropertyGrid` control and the `UITypeEditor` model that extends it are bound to WinForms, so the operation model has to define a framework-neutral editor abstraction that each presentation stack is able to render. The second is the type visualizer model, built on the same pattern, where a visualizer receives an `IServiceProvider` through which it queries and manipulates the editor. The service infrastructure itself is already presentation-neutral, but some of the services it provides and the controls that visualizers create are currently WinForms-specific.

## Assessments

The operation model is assessed by a core decoupling prototype, a feature census, and two design questions it raises: programmatic property assignment over the headless surface, and the framework-neutral redesign of the presentation-facing extensibility contract. The core prototype becomes the implementation proposal in `bonsai-rx/bonsai`, while the census scopes the full migration and tracks its progress. Assessment reports are added under `assessments/` as each is finalized.

| ID | Question | Method | Kind | Status | Exit Criterion |
|----|----------|--------|------|--------|----------------|
| A01 | Can the graph-manipulation operation model be decoupled from WinForms and run headlessly, validated by a headless consumer and a workflow-and-layout round-trip? | prototype | disqualifying | In Progress | A headless consumer requesting graph edits over the decoupled `GraphModel`, with a `.bonsai` load-edit-save round-trip that leaves existing files opening and editing unchanged |
| A02 | What is the full inventory of `Bonsai.Editor` features, and for each, what belongs to the operation-model library versus the presentation stack in ADR-0004? | code-scan | survey | In Progress | A complete feature inventory of `Bonsai.Editor`, each entry marked operation-model, presentation, or split, maintained in this record as a scope-and-progress index |
| A03 | How is an individual operator property assigned programmatically over the headless surface, without hand-manipulating XML fragments and namespaces, and where does string-based `TypeConverter` assignment stop, for example on read-only collection properties? | prototype | scoring | Open | A property-assignment API covering the common cases, with the reach and limits of string-based `TypeConverter` assignment characterized |
| A04 | Can the presentation-facing extensibility contract be redesigned as framework-neutral, building on the `ComponentModel` property-editing model in place of `PropertyGrid` and `UITypeEditor`, and the `IServiceProvider` type visualizer model with its service and render abstractions? | prototype | scoring | Open | A framework-neutral design for both, rendered on a non-WinForms surface; may later split into separate property-editing and visualizer assessments, with the render surfaces possibly migrating to ADR-0004 |

## Decision

Pending the A01 prototype. The direction is to decouple the operation model from WinForms into a presentation-independent library. Since that core is independent of the presentation stack, once the prototype proves the decoupling it may be promoted to Approved and enter implementation on the ADR-0002 v2 track, ahead of the presentation stack decision in ADR-0004. Generalized property assignment and the framework-neutral property-editing and type visualizer abstractions are follow-on work over that core and may be deferred.

## Consequences

The headless editing surface the operation model unlocks is the natural home for the workflow conversion tooling that ADR-0002 relies on for forward-only migration. The `ComponentModel`-based property and editor model and the `IServiceProvider`-based visualizer model must be redesigned as framework-neutral abstractions the operation model owns. The operation model becomes the single owner of load and save, so workflow and layout compatibility is enforced in one place instead of inside the editor.

The decoupling also opens two decisions that this record does not make, each to be scoped independently as its own record.

- **Out-of-process execution**: running workflows in a separate process from the editor, in the manner of a debugger and its debuggee. There are three motivations supporting this: web editing and the plugin-host option in ADR-0004 both require exactly this process boundary; a workflow that runs inside the editor process will never be able to load a different version of a dependency required by the editor, and any unexpected crash can bring the whole editor down with it; and Dear ImGui is single-thread, with no upstream multithreading expected soon, so a standalone graphics or shader window that also uses it cannot currently run side-by-side.
- **A Model Context Protocol server**: the decoupled operation model can be used with no presentation attached, which could allow agents to author and manipulate workflows programmatically over that headless surface. Because such a server would share the operation-model surface directly with the editor, it could be used as a critical test case of this decision.
