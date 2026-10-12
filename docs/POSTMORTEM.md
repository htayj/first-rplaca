# first-rplaca postmortem

**2026-10-12 · Draft for publication**

first-rplaca is our first attempt at a Lisp-native coding-agent environment. Our central mistake was optimizing for a distinctive editor and interface before proving that the whole system was a useful end-to-end coding agent. We put too much weight on what the environment should feel like, and too little on whether we could reliably use it to complete ordinary coding tasks.

This is a retrospective judgment about our priorities, not a claim that no agent functionality existed or that McCLIM cannot support such a tool. The [README](../README.org) describes real agent capabilities and labels the project **pre-dogfood**. The [architecture](ARCHITECTURE.md#design-rules) explicitly intended McCLIM to be an adapter over core state, not the owner of core behavior. That was a sound boundary to aim for. In practice, making the editor and its interaction lifecycle work became a much larger part of the project than that thin-adapter aspiration suggests.

## We chose form before function

We wanted the programmability and tight feedback of Emacs, the presentation-oriented possibilities of McCLIM, and the feel of a Lisp machine. Those are worthwhile ambitions. They were not evidence that we had found the right coding-agent workflow.

The [architecture's opening premise](ARCHITECTURE.md) says the application should feel like “a Lisp machine editor with an agent attached,” rather than a chat client with editor features. In hindsight, that framing gave the editor a head start over the agent. We spent attention making the environment distinctive before establishing the simpler value proposition: can it understand a task, inspect the relevant files, make a change, run the appropriate checks, and let us review the result?

The mistake was sequencing, not having an aesthetic or choosing Lisp. A compelling interface should support a useful workflow. It should not become the prerequisite for discovering whether that workflow is useful.

## We underestimated render-cycle footguns for LLM-driven development

McCLIM's lifecycle requires reasoning about more than the current data model. A request to redisplay, an event queued for a grafted sheet, a handler applying state, and the pixels actually visible on screen are different moments. Coalescing repeated requests is useful, but the bookkeeping has to preserve work across those moments.

The [stability findings](STABILITY.md#findings-and-implemented-fixes) document concrete examples: a pre-graft request could set a coalescing latch without queuing an event and suppress later updates; a missing compose pane could nevertheless be marked synchronized; and a GUI scenario could reach its final semantic state before the corresponding redisplay completed, leaving a screenshot with stale pixels. These were not interchangeable failures. One involved lost scheduling, another initialization ordering, and another observation timing.

Our conclusion is that these contracts were difficult for an LLM to diagnose and repair when the available observations did not expose the relevant lifecycle state. Humans familiar with the framework can bring implicit assumptions about ownership, event ordering, pane readiness, and repainting to the investigation. An LLM cannot safely substitute a plausible local code change for those assumptions. It needs explicit evidence of which state changed, which event ran, and what had actually been rendered. Humans are not immune to these mistakes either; familiarity makes the implicit model easier to supply, not infallible.

This is not a blanket verdict against McCLIM. The stability record distinguishes application, harness, and upstream ownership, and [its later closure notes](STABILITY.md#post-adversarial-closure--2026-07-15) explicitly attribute most of the investigated failures and costs to RPLACA, its tests, or its harness. Our lesson is narrower: we chose a development surface whose temporal behavior demanded more observability and framework knowledge than we had made available to the agent.

## We optimized and committed too early

We pursued optimization and made foundational decisions before enough actual dogfooding could tell us which costs mattered. Our retrospective conclusion is that those commitments constrained iteration: changing direction meant revisiting assumptions already spread through the environment instead of adjusting a small, proven loop.

The documents do not establish a complete causal history for every decision, and we should not invent one. Nor does a measured performance improvement prove that the work was the right priority for an unproven product. The distinction is between improving something real and knowing that it is the thing we most needed to improve.

The next attempt should earn its complexity through use. Correctness still matters from the beginning. Optimization should follow measured pain, and foundational choices should leave room to revise the workflow while we are still learning what it needs to be.

## We tried to do too much, and differentiated too soon

We combined an editor, an extensible Lisp environment, agent interaction, and a distinctive presentation model before understanding which parts made daily coding better. We treated differentiation as an early design requirement rather than a conclusion that could emerge from useful work.

The [feature-gap analysis](FEATURE-GAP-ANALYSIS.md#scope-and-method) usefully distinguishes an end-to-end workflow from partial substrate or a named helper. It also [marks substantial areas as deferred research notes](FEATURE-GAP-ANALYSIS.md#former-p2--deferred--not-on-the-current-roadmap), not authorized implementation. Those notes are evidence of the breadth under consideration, not evidence that all those features were built. The lesson is not to count a roadmap as progress, or to answer uncertainty with a larger roadmap.

We needed a narrower question: what is the smallest coherent environment that helps us finish a real task? One decision and one task at a time would have made it easier to see what worked, what failed, and what was merely interesting.

## What is worth keeping

We should keep the desire for a programmable, inspectable agent environment. The architecture's separation of durable sessions, mutable working buffers, commands, tools, and a thin UI adapter remains useful. So does its preference for ordinary functions and explicit state beneath extension registries.

We should also keep the discipline visible in the stability work: preserve evidence, distinguish ownership, expose lifecycle state, and avoid turning a bounded result into a universal guarantee. Those practices help regardless of the next interface or framework.

What we should not preserve automatically is every interface choice or abstraction simply because we invested in it.

## Principles for the next attempt

- **Prove the minimal agent loop:** read, edit, run, check, and review the diff. A completed, inspectable task matters more than an impressive surface.
- **Dogfood actual tasks early:** use ordinary maintenance and development work to discover what the environment needs before expanding it.
- **Keep feedback short and state observable:** distinguish requested work, running work, applied results, and rendered output instead of inferring success from one layer.
- **Defer optimization until measured pain:** improve the bottleneck demonstrated by use, not the one most interesting to design around.
- **Make one decision and tackle one task at a time:** let each result inform the next choice, rather than committing to the whole environment in advance.

These are principles, not an implementation promise or a declaration that a particular framework must be abandoned. first-rplaca is worth retaining as a record of the attempt and the lessons. The next attempt should start by being useful, and earn its distinctiveness afterward.
