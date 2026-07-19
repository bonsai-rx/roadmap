---
id: ADR-0002
title: Modernizing the Bonsai .NET stack
status: Assessment
area: [Ecosystem, Infrastructure, Packaging]
authors: [glopesdev]
related: []
discussions:
---

# ADR-0002: Modernizing the Bonsai .NET stack

## Summary

Bonsai has historically targeted .NET Framework. Although the core and editor already run on .NET 8, modernizing the full stack means more than the runtime: it includes moving the ecosystem dependencies off .NET Framework, moving the editor off WinForms toward cross-platform, and modernizing the infrastructure around them. Both managed and native dependencies carry their own compatibility breaks, so the packaging guidelines that govern the feed of community packages have to be coordinated to preserve forward compatibility, keeping existing workflows and packages running as those dependencies move. This is a multi-year, cross-project effort that will shape the ecosystem for the next decade. This record is the umbrella that decides how and when the modernization happens, coordinates the sub-decisions it spawns, and batches the unavoidable breaking changes into the next major version, so the ecosystem migrates once along a single clear path.

## Context

Bonsai has run on .NET Framework since its inception. Back in 2011 WinForms was a mature, widely used UI toolkit with solid cross-platform reach through the Mono Framework. The Reactive Extensions, Rx, were being considered for inclusion in the .NET Framework itself, on par with LINQ. OpenCV still exposed a strong C ABI, a robust interop foundation for high-performance computer vision in C#, which we leveraged to build the `OpenCV.Net` wrapper. `OpenTK`, meanwhile, struck a rare balance between a raw graphics API and a full engine, unifying windowing, math, input, graphics, and audio into one interoperable toolkit, which we committed to as the foundation for graphics and audio.

Since then, all of these foundational assumptions have shifted. OpenCV 3 adopted C++11 and signaled a sharp turn away from the original C API, which OpenCV 5 has since completed by removing all traces of legacy C implementations from its modern C++17 interface. Rx has moved into the oversight of the .NET Foundation, with binary compatibility compromised by a change in assembly strong-name keys. `OpenTK` went through repeated breaking rebuilds under a new maintainer and is only now settling onto modern .NET. Finally, .NET itself moved into a new era following a complete rewrite of the runtime and BCL, which brings better performance and true cross-platform reach. New language features, tooling, and a growing share of the package ecosystem increasingly target modern .NET only. Moving to the modern .NET stack is not without tradeoffs, however, as binary backward compatibility is not guaranteed and, most relevant for Bonsai, the desktop UI frameworks WinForms and WPF became Windows-only, with Mono long since superseded. All this created migration pressure.

Modernization runs along two distinct axes that are easy to conflate. The first is the runtime and its dependencies. The core already runs on .NET 8, so the runtime itself is not the obstacle; the dependencies are. The standard library, a large body of community packages, and the workflows built on them still target .NET Framework, and many cannot be recompiled or republished, while a conservative user base drawn from research laboratories values a workflow recorded years ago still opening today over access to the newest runtime.

The central tension on this axis is how long .NET Framework must remain supported, weighed against the accumulating cost of frozen dependencies and a frozen runtime. Migrating the whole ecosystem realistically takes two to five years, so .NET Framework support is a hard requirement through that window rather than a matter of preference, and every dependent decision inherits this timing. `OpenCV.Net` is the first such case, and the release of OpenCV 5 is what makes its timing pressing.

The second axis is the presentation layer. The editor already runs on .NET 8 as well, but it remains bound to Windows through WinForms, so cross-platform reach is blocked by the UI framework rather than the runtime. This axis is independent of the first and can progress on its own timeline, by moving off WinForms toward a cross-platform editor.

The concrete per-dependency and per-project migrations are outside the scope of this record. They will be worked as their own records that reference this one for timing and policy. This record decides the overall strategy; it does not decide how any single dependency or project is carried across.

## Migration Model

Every change falls into one of two tracks. The additive track carries improvements that keep the current stack working without breaking anything, and these are released incrementally into the v2 line as they become ready, the way the v2.10 work already folds in the create-object proposal for addressable modular components and the Roslyn support for compiling local extensions against the latest language features. The breaking track holds the changes that cannot be made compatibly, and these are batched into a single next major version, v3, so the ecosystem migrates only once and does not split into incompatible camps. The v3 batch is held until all its members are ready together. The tension to manage is that batching maximizes the value of migrating once while coupling every breaking change to the readiness of the slowest one.

The v3 major version has a concrete marker: the first officially released .NET 8 installer. Today the core and editor build for multiple target frameworks, but only .NET Framework installers are released, so for the effective user base a modern .NET version of Bonsai does not yet exist as a deployable product. That is deliberate. The cross-platform editor, the modern reactive stack, and the modern graphics and audio backends all converge on that one artifact.

Through the transition, continuity is provided by the persistent .NET Framework build, where existing workflows and published packages, including those that cannot be recompiled, keep running untouched. The modern .NET build is the new world that a package developer reaches by multi-targeting, retaining .NET Framework while adding modern .NET, ideally with little or no breaking source-level changes. Researchers stay on .NET Framework for as long as they need and still receive new packages as the ecosystem moves. Over several years, .NET Framework is retired with little disruption. Migration of stored workflows across this boundary is forward-only by default, so a conversion from .NET Framework to modern .NET need not be reversible and a modern .NET workflow need not open on .NET Framework, with external conversion tooling provided to bridge specific cases. A dependent decision may offer more than this where it is cheap to do so.

## Criteria

All criteria in [criteria.md](../../criteria.md) bear on the modernization, but two dominate. Ecosystem Continuity Risk is paramount, since the failure mode is a recorded workflow or a published package that no longer opens, and Community Trust is close behind, since a conservative user base judges the transition by whether existing work keeps running and by how the change is communicated. Maintenance Burden and Effort bound the achievable pace, given that supporting two target frameworks at once is itself an ongoing cost. A trade-off matrix is deferred until the options for the support window and the migration order are drawn; at this stage the record frames the questions rather than scoring answers.

## Open Questions and Sub-Decisions

The modernization resolves into a set of decisions, several of which are being scoped as their own records that reference this umbrella:

- The cross-platform editor architecture: decoupling the editor operation model from WinForms into a UI-agnostic editing core, then choosing a presentation stack over it. To be created as its own record, which will unlock a headless editing surface for automation and agents.
- Adopting modern Rx.NET across target frameworks: containing the Rx strong-name break at the target-framework boundary, and recommending a version for package developers who multi-target. To be created as its own record.
- The future of `OpenCV.Net`, the primary image-processing dependency, to be created as its own record.
- A stable abstraction for Bonsai 3D graphics and audio: the future of the `OpenTK`-dependent graphics and audio stack, and how to introduce an owned abstraction where none exists today. To be created as its own record.

Beyond the sub-decisions, one cross-cutting assessment remains at the umbrella level: a per-project survey of what concretely blocks a full move to modern .NET today.

The umbrella is expected to spawn further records as scoping continues, including a general-principle record on retaining release control over critical dependencies, since that control is what lets a dependency keep a stable public surface while its backend is replaced, as with `OpenCV.Net`.

## Decision

Pending. The record stays in Assessment while the sub-decisions are worked. What this record itself decides is the assignment of each change to the additive v2 track or the v3 breaking batch, the v3 major version release timeline, the .NET Framework support window, the order of migration across projects, and the packaging and distribution guidance for multi-targeting across both frameworks.

## Consequences

Pending the full decision, a few commitments are already firm and shape the dependent records. Every dependent decision must assume a live .NET Framework target for the whole transition window, which is what tilts the `OpenCV.Net` decision toward a backend that can serve both frameworks. Package developers bear the standing cost of multi-targeting across both frameworks through the transition. The remaining consequences, including the communication plan for the eventual retirement of .NET Framework, are recorded here once the support window is decided.
