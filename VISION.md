# SPF, DKIM, and DMARC Notes

## Vision

Explore whether a structured AI-driven writing process can produce a concise, technically accurate, high-quality learner journey for a bounded technical subject.

Use SPF, DKIM, and DMARC as the initial subject.

The experiment is not primarily about creating another explanation of email authentication. There is already extensive material available on these technologies.

The interesting problem is whether AI can take a body of technical information and, through a deliberate sequence of research, outlining, writing, review, and compression, produce educational material that is substantially better than a conventional one-shot generated tutorial.

The final material should be:

- concise;
- professional;
- technically accurate;
- information-dense;
- logically structured;
- easy to move through;
- supported by useful references;
- and largely free from recognizable AI writing habits.

This project should explore the workflow required to reliably produce that result.

## Problem

AI can readily generate technical explanations, but first-pass output often has predictable weaknesses.

It may:

- repeat the same idea several times;
- over-explain simple concepts;
- introduce unnecessary framing;
- use generic introductions and conclusions;
- create artificial transitions between sections;
- favour prose where a compact explanation would be clearer;
- omit small but important technical details;
- drift away from the intended learning objective;
- or gradually lose information when repeatedly asked to shorten a document.

Simply asking an AI to "write a good tutorial" does not provide enough structure to control these behaviours.

The hypothesis behind this project is that quality improves when content generation is decomposed into distinct stages with explicit artefacts and review responsibilities.

## Core Workflow

The general process should be:

> research → information outline → learner journey → initial writing → review → targeted revision → review → final material

Do not collapse these stages into a single generation prompt.

The intermediate artefacts are part of the experiment.

In particular, the information outline should survive throughout the entire process and act as a reference against which later versions of the material can be evaluated.

## Research and Information Collection

Begin by collecting the information required to explain SPF, DKIM, and DMARC properly.

At this stage, do not attempt to write polished educational material.

The objective is to establish what needs to be known.

Useful areas will likely include:

- the problem email authentication is attempting to solve;
- SMTP identities relevant to authentication;
- DNS as part of the authentication model;
- SPF;
- SPF record construction and evaluation;
- SPF alignment;
- DKIM;
- DKIM signatures and selectors;
- public and private key relationships;
- DKIM verification;
- DKIM alignment;
- DMARC;
- DMARC policy;
- identifier alignment;
- aggregate and forensic/reporting concepts where relevant;
- message flow;
- forwarding and intermediary behaviour;
- common failure modes;
- deployment considerations;
- limitations of each mechanism;
- and how SPF, DKIM, and DMARC work together.

The project should discover the appropriate scope rather than treating this list as a fixed curriculum.

Prefer authoritative or primary references where practical.

Capture references alongside the information they support rather than attempting to reconstruct sourcing after the material has already been written.

## Information-Dense Outline

Convert the collected knowledge into a compact outline.

This is one of the most important artefacts in the project.

The outline should primarily consist of:

- bullet points;
- short factual statements;
- relationships;
- definitions;
- important caveats;
- examples where necessary;
- dependencies between concepts;
- and links to supporting references.

It should optimize for information density rather than readability as a finished tutorial.

For example, the outline does not need several paragraphs explaining SPF if a small collection of precise bullets can capture everything the eventual tutorial needs to communicate.

The purpose is to produce a compact representation of the intended knowledge.

The outline becomes the reference model for the rest of the content pipeline.

## Learner Journey

Once the information space is understood, determine how someone should learn it.

This is different from the information outline.

The outline answers:

> What information needs to be communicated?

The learner journey answers:

> In what order should someone encounter this information so that each new concept makes sense?

The learner journey should identify:

- prerequisite concepts;
- the sequence of topics;
- where concepts should first be introduced;
- where they should later be expanded;
- useful examples;
- points where multiple technologies should be connected;
- and the expected understanding after each section.

Avoid organizing material simply because that is how source documentation happens to be structured.

The learning sequence should be designed around comprehension.

## Initial Writing

Once the outline and learner journey exist, produce the initial set of learning materials.

The first writing pass should be complete enough to contain the intended knowledge but does not need to be editorially perfect.

The objective is to get the entire learner journey into written form.

Do not repeatedly optimize individual paragraphs while large portions of the tutorial remain unwritten.

Complete the first version and then allow the review process to improve it systematically.

Each section should remain traceable to the information outline and learner journey.

## Persistent Outline

Do not discard the information outline after the initial writing pass.

It should remain available to every significant review stage.

This is important because repeated editorial compression creates a risk of information loss.

A document can become shorter, cleaner, and easier to read while quietly losing important technical details.

The review process should therefore continually compare the current material with the original compact knowledge representation.

Conceptually:

> shorter is useful only if the important information remains.

The outline provides the mechanism for checking that.

## Review Pipeline

Use several distinct review or judge passes rather than asking one agent to generically improve the material.

Approximately three major review perspectives should be explored.

The exact implementation can evolve, but the responsibilities should remain meaningfully separate.

### Alignment and Coverage Review

Evaluate the material against the original vision, information outline, and learner journey.

Look for:

- missing information;
- incorrect information;
- sections that no longer serve their intended purpose;
- concepts introduced in the wrong order;
- unnecessary tangents;
- duplicate material;
- gaps between modules;
- and divergence from the intended learning experience.

The central question is:

> Did we actually produce the material we intended to produce?

### Compression and Information-Density Review

Evaluate whether the same useful content can be communicated more efficiently.

Look for:

- repeated explanations;
- unnecessary setup;
- excessive transition sentences;
- redundant examples;
- paragraphs that contain only one useful fact;
- explanations that are substantially longer than the concept requires;
- generic introductions;
- generic conclusions;
- and recognizable AI-generated filler.

The objective is not arbitrary brevity.

Important technical detail should remain.

The goal is:

> preserve the knowledge while reducing the amount of writing required to communicate it.

### Final Editorial and Learning Review

Evaluate the material as an actual learner would encounter it.

Consider:

- clarity;
- consistency;
- terminology;
- progression;
- technical precision;
- readability;
- professional tone;
- examples;
- reference quality;
- and whether the material feels intentionally authored rather than generated.

This stage should also catch issues introduced by earlier compression.

## Ticket-Based Revision

Review findings should not simply trigger complete regeneration of the documents.

Turn meaningful findings into discrete pieces of work.

For example:

- clarify the relationship between SPF authentication and DMARC alignment;
- remove repeated explanation of DKIM selectors;
- add a missing forwarding scenario;
- shorten the DMARC introduction;
- verify an example against an authoritative reference;
- or restructure two sections whose learning order is incorrect.

Treat these as tickets or equivalent work items.

An agent can then implement specific changes without rewriting unrelated material.

This provides several benefits:

- changes remain explainable;
- already-good content is less likely to be damaged;
- review findings can be tracked;
- work can be parallelized;
- and subsequent judges can verify whether a particular problem was actually resolved.

The project therefore includes a lightweight project-management workflow around the content itself.

## Style

The resulting material should favour straightforward technical writing.

Prefer:

- short sections;
- direct statements;
- precise terminology;
- compact examples;
- useful diagrams where appropriate;
- bullet points when the information naturally forms a list;
- and links to deeper references rather than unnecessarily reproducing everything.

Avoid stylistic patterns that add length without improving understanding.

In particular, watch for generated prose that repeatedly announces what it is about to explain, summarizes what it just explained, or adds artificial emphasis to otherwise straightforward concepts.

The goal is not minimalism for its own sake.

The goal is high information density with enough explanation for the material to remain understandable.

## Automation

The implementation should explore how much of this pipeline can be performed by agents.

Potentially separate responsibilities include:

- research;
- source collection;
- outline construction;
- learner-journey design;
- drafting;
- alignment review;
- compression review;
- editorial review;
- ticket creation;
- ticket implementation;
- and final verification.

These roles do not necessarily require separate models or sophisticated orchestration.

Use whatever implementation is sufficient to test the workflow.

The important property is that the responsibilities remain explicit enough to evaluate whether the staged process produces better results.

## Questions

The project should help answer questions such as:

- Does building the information outline before writing materially improve the final result?
- Does retaining the outline prevent information loss during compression?
- Can AI design a sensible learning sequence from a technical knowledge outline?
- How much useful compression can be achieved without reducing technical completeness?
- Do specialized judge passes produce better results than generic editing prompts?
- Are three review stages useful, excessive, or insufficient?
- Can judge findings reliably be converted into actionable tickets?
- Does targeted ticket-based revision outperform complete document regeneration?
- Which prompts consistently produce the desired writing style?
- Which common AI writing patterns remain difficult to remove?
- How well can the workflow preserve authoritative references?
- Can the resulting process be generalized to technical subjects other than email authentication?

## Boundaries

Do not prematurely build a general-purpose documentation platform.

The initial goal is to prove or disprove the content-production process.

SPF, DKIM, and DMARC provide a bounded technical domain against which the workflow can be evaluated.

Likewise, avoid spending substantial effort on presentation, publishing infrastructure, websites, or learning-management features before the quality of the underlying content pipeline has been demonstrated.

The important artefacts are initially the knowledge, outline, learner journey, documents, reviews, and resulting revision work.

## Expected Output

The project should produce both the educational material and evidence about the process that produced it.

Useful outputs include:

- a sourced technical knowledge corpus;
- an information-dense master outline;
- a learner-journey structure;
- initial tutorial material;
- review findings;
- generated revision tickets;
- revised material;
- and the final concise learner journey.

Retain enough of the intermediate material to compare stages and understand what each review process contributed.

## Success

The project is successful if the final SPF, DKIM, and DMARC material is meaningfully better than what could be obtained through a single well-written generation prompt.

In particular, the workflow should demonstrate that AI can produce material that is:

- complete without being bloated;
- concise without becoming shallow;
- technically grounded;
- consistently structured;
- aligned with an intentional learning sequence;
- and written in a restrained professional style.

The broader result should be an understanding of whether this workflow can serve as a repeatable method for producing other technical learner journeys.
