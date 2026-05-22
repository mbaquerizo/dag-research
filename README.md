# DAG Research

Docs Augmented Generation (DAG) proposes a simple thesis: **curated, structured documentation is better context for AI code generation than RAG or raw prompt engineering**.

Instead of retrieving arbitrary text chunks by similarity, you deliberately write a ticket with structured fields, link it to an ADR that captures architectural decisions, and include a code exploration that maps the relevant parts of the codebase. The model reads those docs and generates code informed by them.

The premise makes intuitive sense, but it has never been systematically tested. *Which* documentation shapes actually improve generation quality? Do acceptance criteria as Gherkin scenarios outperform prose descriptions? Does a "technical notes" field from an ADR help the model make better architectural choices, or is it noise? Does the optimal template change when the model changes?

This project answers those questions.

## Why a separate project?

DAG (the tool) defines documentation conventions and publishes structured docs to wherever a team works. DAG Research provides the evidence that shapes those conventions.

They are separate because they evolve at different rates. Models are updated constantly — the assumptions that held for GPT-4 may not hold for GPT-5. The research project runs experiments and publishes findings. DAG absorbs those findings into its templates and recommendations. The loop is continuous.

This is the same relationship SEO agencies have with search engines. Google changes its algorithm → SEO research measures the impact → publishing guidelines update. Models change → DAG Research measures the impact → DAG templates update.

## The research loop

Each experiment follows the same structure:

1. **Design a feature spec** — a concrete, well-scoped feature with a correct behavior that can be verified by a test suite. Features are designed to be bespoke: the model hasn't memorized *this exact spec* even if it's familiar with the domain.

2. **Write multiple template variants** — the same feature described in different documentation shapes. For example, a ticket with acceptance criteria as numbered prose vs. Gherkin scenarios vs. a worked example. Only the template structure changes; the underlying requirement is identical.

3. **Run each variant against a model** — feed the variant + relevant codebase context to a model, capture its output. Run each variant N times to measure variance.

4. **Evaluate** — run the generated code against the ground-truth test suite. Measure pass rate, consistency across runs, and qualitative observations about what the model misunderstood.

5. **Publish** — results feed back into DAG's templates. A finding like "Gherkin scenarios produce 30% higher test pass rates for conditional logic features" becomes a DAG documentation convention.

## What we measure

| Metric | What it captures |
|--------|-----------------|
| Test pass rate | Did the generated code behave correctly? Primary signal. |
| Run-to-run variance | Does the template produce consistent results, or is it a coin flip? |
| Time to correct | How many iterations to reach a passing state (if iterative)? |
| Qualitative failure modes | What did the model consistently misunderstand? Structure gaps, missing edge cases, wrong imports? |

Down the line, additional signals like code style conformity, test coverage of generated code, and hallucination rate may also be relevant.

## Addressing contamination

Standard benchmarks like SWE-bench use real GitHub issues from well-known repositories. This means models may have seen the fix in training, making it hard to tell whether the model solved the issue or recalled it.

DAG Research avoids this in two ways:

1. **Bespoke specs** — features are designed with specific parameters, constraints, and edge cases that cannot be recalled from training. A model has seen *a* data table component, but it hasn't seen *this exact data table with these column renderers, this hook signature, and these pagination parameters*.

2. **Cross-language signal** — a core suite in TypeScript/React (highest practical relevance) supplemented by experiments in languages with no training data, such as Vercel's Zero (a systems language released May 2026 that no model has been trained on). If a template shape correlates with better outcomes in *both*, the finding is robust.

Ground-truth test suites are authored by humans and are never exposed to the model during generation.

## What this becomes

As the experiment library grows, patterns emerge:

- "For features involving conditional branching (permissions, feature flags, access control), Gherkin-style acceptance criteria improves pass rate by X% over prose."
- "For CRUD features, the template does not measurably affect output quality — effort is better spent elsewhere."
- "For bug fix tickets, including a failing test case in the description improves repair success rate over a prose description alone."

These findings become the evidence base behind DAG's conventions. DAG users benefit from every experiment without having to run it themselves. And when a new model generation resets the assumptions, the research loop runs again.

## Current state

This is a new project. No experiments have been run yet. The first step is to define the feature library and build the evaluation harness.
