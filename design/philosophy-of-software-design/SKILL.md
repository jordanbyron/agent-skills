---
name: philosophy-of-software-design
description: >
  Use this skill any time the user is shaping code structure rather than fixing
  or explaining it. Trigger on phrases like "I'm about to build...",
  "how should I structure...", "should I split/combine...", "one class or
  several...", "designing an API/interface/module/service/pipeline", "writing
  interface docs before I implement", "refactoring to make it cleaner", "this
  feels off", or reviewing code for too many methods, awkward boundaries, leaky
  abstractions, or shallow wrappers. The signal is *structural judgment about
  code* — choosing what goes where, what an interface exposes, how
  responsibilities divide across classes/modules/methods. Trigger even when the
  word "design" is absent and the phrasing is casual or embedded in a coding
  task ("before I start coding, how should I organize X"). Skip for: pure
  debugging, stack traces, lint/syntax/typo fixes, language or concept
  explanations, and non-code design like UI mockups, DB schemas, or infra
  diagrams.
---

# Philosophy of Software Design

Based on John Ousterhout's "A Philosophy of Software Design." The overarching
insight: **the greatest limitation in writing software is our ability to understand
the systems we create.** Every design decision should be evaluated by asking:
*does this make the system simpler or more complex?*

Complexity manifests as change amplification (a simple change touches many places),
cognitive load (too much to know), and unknown unknowns (not obvious what matters).
It's caused by dependencies and obscurity, and it accumulates incrementally — you
must sweat the small stuff.

## What This Skill Changes About Your Behavior

These principles aren't exotic — you already know most of them. The skill's purpose
is to change *when and how aggressively* you apply them. Specifically:

### 1. Challenge designs that smell wrong — even if the user proposed them

This is the single most important behavior change. When a user proposes a design
with red flags (shallow classes, temporal decomposition, information leakage),
don't just implement it. Name the red flags, explain the consequences, and propose
a better alternative. You can still implement what they asked for if they insist,
but your default should be to push back with reasoning.

Frame this constructively: "Before implementing this, I notice a few design
concerns worth discussing..." — not "your design is bad."

### 2. Always Design It Twice

Before implementing any significant module, API, or restructuring, compare at
least two fundamentally different approaches. Write out the trade-offs explicitly:
interface simplicity, implementation complexity, how each handles likely future
changes. This is cheap (a few sentences) and consistently produces better designs.

Even when one approach seems obviously right, the comparison sharpens your thinking
and gives the user confidence in the recommendation.

### 3. Scan for red flags before writing code

Before implementing or reviewing, actively scan for these. Don't just notice them
passively — look for them:

**Shallow Module** — Is the interface nearly as complex as the implementation? Are
callers bearing most of the burden? Count the public methods: if there are many
and each does little, the module is shallow. A deep module has few public methods
that each do a lot.

**Information Leakage** — Is the same design decision (file format, protocol,
data structure) reflected in multiple modules? If changing that decision would
require touching several files, the knowledge has leaked.

**Temporal Decomposition** — Is the code structured around the order things happen
(read, parse, validate, transform, write) rather than around what knowledge each
piece holds? If callers must invoke methods in a specific sequence, that's temporal
coupling.

**Pass-Through Methods** — Does a method mostly just call another method with a
similar signature? This adds a layer without adding abstraction. Each layer should
provide a genuinely different abstraction.

**Conjoined Methods** — Can you understand one method without reading another? If
two methods are only comprehensible as a pair because they share implicit state or
contracts, they should probably be combined or their interface clarified.

**Repetition** — Is the same nontrivial pattern appearing in multiple places?
Factor it into one place — but only if it's genuinely the same concept.

**Special-General Mixture** — Is special-purpose code tangled with general-purpose
code? Separate them so the general mechanism is reusable.

**Vague or Hard-to-Pick Names** — If a name doesn't convey useful information, or
if it's hard to name something, the underlying entity may be poorly designed.
Difficulty naming often reveals a deeper design problem.

**Nonobvious Code** — If the behavior can't be understood with a quick reading,
something needs to change: a comment, a renamed variable, a restructured flow.
Software should be designed for ease of reading, not ease of writing.

### 4. Apply the core design principles actively

These aren't just things to know — they should change what you produce:

**Make modules deep.** When designing a class or function, minimize the public
interface while maximizing the work it does internally. Ask: "Does this module
save its users significant effort?" If callers must orchestrate multiple calls
in sequence, the module is too shallow — pull that orchestration inside.

**Hide information.** Each module should encapsulate decisions other modules don't
need. Ask: "What does this code know that callers shouldn't have to?" Group code
by information ownership, not by execution order.

**Pull complexity downwards.** When something is unavoidably complex, absorb it in
the implementation so the interface stays simple. One complex implementation
serving many callers beats a complex interface burdening every caller. Avoid
configuration parameters when the module can determine reasonable defaults itself.

**Define errors out of existence.** Before adding exception handling, ask: "Can I
redefine the operation so this isn't an error?" Make operations idempotent. Use
"ensure X" semantics instead of "do X if not already done." When exceptions can't
be eliminated, aggregate them — one handler in one place rather than scattered
try/catch blocks.

**General-purpose modules are deeper.** Design the simplest interface that covers
all current needs. Don't build features you don't need yet, but design interfaces
that won't break when those features arrive. Fewer, more powerful operations beat
many narrow ones.

**Different layer, different abstraction.** If two adjacent layers have similar
interfaces, the decomposition is probably wrong. Each layer should operate at a
meaningfully different level of abstraction.

**Better together or better apart?** Combine code that shares information,
simplifies interfaces, or eliminates duplication. Separate general-purpose from
special-purpose. Length alone is never a reason to split a method — each method
should do one thing and do it completely.

**Consistency.** Similar things should be done in similar ways. Match existing
conventions even if you prefer a different approach. "A better idea" is not
sufficient reason to introduce inconsistency.

### 5. Think strategically, not tactically

Working code is necessary but not sufficient. When modifying existing code, resist
the smallest possible fix. After your change, the system should look as if it had
been designed with that change in mind from the start. If you're not making the
design better, you're probably making it worse.

### 6. Use comments as a design tool

Write interface comments before implementing — what does this provide, what are
the side effects, what do parameters mean? If the comment is hard to write, the
design is probably too complex. Fix the design, not the comment.

Comments should describe what's not obvious from the code. Never repeat what the
code says. Interface comments describe *what* and *why* from the caller's
perspective. Implementation comments describe strategy and reasoning at a higher
level than the code.
