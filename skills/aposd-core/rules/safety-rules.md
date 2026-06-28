# Safety Rules: Preventing Broad Rewrites

This document enforces discipline in refactoring: we deepen existing modules, we don't proliferate abstraction layers.

## Rule 1: Don't Split a Module Just Because It's Long

**Principle**: Length alone is not a reason to split.

**Violations**:
- "This file is 500 lines, let's split it into 3 files"
- "This class has 20 methods, let's move some to a new class"
- "This function is 50 lines, let's extract a helper"

**Why**: Splitting to reduce line count often creates shallow modules, information leakage, and change amplification. The original module was long for a reason: it has a coherent responsibility.

**What to do instead**:
1. Understand what the module is doing (cohesion check)
2. Ask: if I removed method X, would the module still make sense?
3. If yes, move it. If no, it's part of the core responsibility; leave it.
4. Look for duplicate logic or special cases (actual reasons to split)

**Checklist**:
- [ ] The split is driven by information hiding or clear responsibility boundary, not line count
- [ ] The new module doesn't become shallow (would it exist without the parent?)
- [ ] Callers don't need to know about the split
- [ ] The change doesn't ripple to 5+ files

---

## Rule 2: Don't Introduce Layers (DDD/Repository/Service) As Default

**Principle**: Layering should solve a real problem, not be architectural dogma.

**Violations**:
- Adding a "service layer" because "that's how we do it"
- Creating a repository for every domain object
- Adding dependency injection and interfaces "for testability"
- Splitting API request handling into handler → service → repository → model

**Why**: Layers without purpose create pass-through abstractions, change amplification, and shallow modules. Callers must know about more layers; changes cascade through them.

**When layers ARE justified**:
- Information hiding requires it (API payload ≠ domain model ≠ database schema)
- Different teams own different layers
- Cross-cutting concerns (logging, transactions) need a consistent checkpoint

**What to do instead**:
1. Keep code in one module first; move later if information hiding demands it
2. If splitting is needed, define exactly what each layer decides/hides
3. Test: can the inner layer be used directly without the outer layer? If yes, the outer layer is pass-through

**Checklist**:
- [ ] The layer solves a specific problem (information hiding, not "organization")
- [ ] Removing the layer would break something real
- [ ] Callers don't touch multiple layers for one logical operation
- [ ] The layer isn't just a 1:1 forwarding of method calls

---

## Rule 3: Avoid Adding Wrapper Classes Just to "Group" Related Code

**Principle**: Composition and delegation are not automatic solutions.

**Violations**:
- "These 3 functions are related, let's create a Handler class"
- "This module does two things, let's split into UserManager and OrderManager classes"
- "Let's extract a utility class for these 5 helper functions"

**Why**: Wrapper classes often become shallow modules. They add vocabulary and indirection without reducing complexity. Callers must now know about the wrapper.

**What to do instead**:
1. Keep related functions in the same module; use clear naming to show relationship
2. If truly separate concerns, ask: what would change in one but not the other?
3. Make the module deeper (more responsibility, simpler interface) instead of wider (more classes)

**Checklist**:
- [ ] The new class has state or makes real decisions, not just forwarding calls
- [ ] Callers benefit from the wrapper's existence (can't call the underlying functions directly)
- [ ] Removing the wrapper would lose meaningful abstraction

---

## Rule 4: Don't Defer Hard Design Decisions via Configuration

**Principle**: Flags, toggles, and options should be rare.

**Violations**:
- Adding parameters like `use_cache=True`, `is_admin=False`, `retry_on_error=True`
- Config classes with 20+ optional fields
- "If feature_flag X, do Y; else do Z" scattered through code
- Constructor that accepts a dozen configuration objects

**Why**: Configuration defers decisions to callers. Callers must understand all configuration options and their interactions. This increases cognitive load and creates special-case mixture.

**What to do instead**:
1. Make the hard decision inside the module; expose a clean, simple interface
2. If configuration truly needed, keep it minimal (ideally 1-2 required fields)
3. Use builder pattern or factory for complex setup, but hide complexity

**Checklist**:
- [ ] Each configuration option is justified (not "just in case")
- [ ] Caller doesn't need to understand internal algorithm to use this module
- [ ] Invalid configuration combinations are impossible (not just "documented")
- [ ] Adding one more feature doesn't require adding a new parameter

---

## Rule 5: Don't Use Messaging, Events, or Queues to Hide Coupling

**Principle**: If you need messaging, it should be for a real reason, not to avoid design.

**Violations**:
- Adding event listeners because "components shouldn't depend on each other"
- Using queues to decouple service A from service B when a direct call is simpler
- Emitting events from every database operation
- Message format that mirrors internal domain objects exactly

**Why**: Messaging introduces asynchrony, eventual consistency, and distributed debugging without solving an actual problem. It hides coupling instead of eliminating it.

**When messaging IS justified**:
- True asynchrony needed (deferred processing, throttling)
- Real decoupling (different codebases, deployment boundaries)
- Cross-cutting concern (analytics, audit trails)

**What to do instead**:
1. Use direct calls first. Make modules deep enough that coupling is limited.
2. Design module boundaries so information hiding prevents tight coupling
3. Only move to messaging if direct calls cause actual problems (performance, availability)

**Checklist**:
- [ ] Messaging solves a concrete problem (not just "architectural cleanliness")
- [ ] Message contract is stable and doesn't mirror implementation details
- [ ] You've measured and understand the latency/consistency implications
- [ ] Removing messaging would break something real

---

## Rule 6: Don't Proliferate Abstraction Levels

**Principle**: Each abstraction should hide something. Otherwise, it's noise.

**Violations**:
- Interface for each implementation (even if there's only one)
- Abstract base classes that do nothing
- Protocols/traits used only by one class
- DTOs, ViewModels, and Domain Objects that are nearly identical

**Why**: Each layer of abstraction adds cognitive load. Callers must know about more types. Changes ripple through more files.

**What to do instead**:
1. Use concrete classes. Add interfaces only when you have 2+ implementations
2. When you DO need an interface, hide it inside the module; external callers use the concrete type
3. Use type aliases and generics to reduce boilerplate without adding layers

**Checklist**:
- [ ] This abstraction prevents something concrete (e.g., swapping implementations)
- [ ] Without it, the code would be duplicated, not just "organized"
- [ ] Removing it would make callers more complex, not simpler

---

## Rule 7: Before Refactoring, Measure What's Actually Broken

**Principle**: Refactoring is not cleaning up; it's fixing a bug in design.

**Violations**:
- "This code is hard to read, let's refactor it" (without knowing why)
- "This module violates [architecture rule], let's split it" (without knowing impact)
- "Let's add type hints to improve clarity" (without trying first to rename things)
- "This test is hard to write; the design is bad" (maybe the test is bad)

**Why**: Refactoring without a hypothesis wastes time and introduces risk. The change might make things worse.

**What to do instead**:
1. State the problem clearly: "Tests take 10 seconds to run", "PR reviewers don't understand the logic", "Changes to [X] ripple to [Y] unexpectedly"
2. Verify the problem: can you reproduce it? Is it happening repeatedly?
3. Propose a hypothesis: "If we split [X], we can test it independently" or "If we hide [implementation detail], changes won't ripple"
4. Measure the impact after the refactor

**Checklist**:
- [ ] You can describe the problem in one sentence
- [ ] You've seen the problem occur multiple times (not just once)
- [ ] Your proposed fix addresses that specific problem
- [ ] You have a way to verify the fix worked

---

## Rule 8: When in Doubt, Deepen the Existing Module

**Principle**: Prefer making existing modules more powerful over creating new ones.

**Violations**:
- Creating a new "manager" class when an existing class could do the work
- Moving a method to a "helper" when it belongs in the original module
- Extracting a utility function when it's specific to one caller

**Why**: New modules proliferate shallow abstractions. Deepening an existing module concentrates related logic and reduces caller knowledge.

**What to do instead**:
1. Ask: does this logic belong in the existing module? (Does it share responsibility?)
2. If yes, move it there and simplify the interface if needed
3. Only create a new module if the answer is genuinely "no" to multiple places

**Checklist**:
- [ ] This logic is related to the existing module's core responsibility
- [ ] Moving it inward simplifies callers
- [ ] The existing module doesn't become too big (still cohesive)
- [ ] The interface doesn't become more complex

---

## Rule 9: Avoid Splitting Error Handling from Logic

**Principle**: Errors should be defined out of existence, not caught everywhere.

**Violations**:
- Try-catch blocks in multiple places for the same operation
- Different error handling for the same logic in different callers
- Retries, timeouts, and circuit breakers in multiple layers
- Error type hierarchies that don't help

**Why**: Scattered error handling creates change amplification and cognitive load.

**What to do instead**:
1. Define errors out of existence (return Optional, use None, validate upfront)
2. If exceptions needed, handle them at the source (close to where they occur)
3. Keep error types minimal; rich error messages, not many types
4. Document which operations can fail and why

**Checklist**:
- [ ] Errors are handled in one place (where they occur)
- [ ] Callers don't need to know about exception types
- [ ] Error context (original exception, stack) is preserved
- [ ] You could remove a try-catch block without affecting behavior

---

## Summary: The Discipline

When you feel like refactoring, stop and ask:

1. **What's the actual problem?** (not "this code is long" but "this change affected 5 files")
2. **Is it a module boundary issue?** (information hiding, responsibility mismatch)
3. **Is deepening the existing module an option?** (make it more powerful, simpler interface)
4. **Would a new layer solve it?** (only if information hiding truly demands it)
5. **Will this refactor reduce change amplification?** (if not, why do it?)
6. **Can I do this in one small PR?** (if not, the scope is too large)

If you answer yes to 3-5 AND no to 6, you've found a real refactoring. Otherwise, wait or ask for guidance.

---

## Checklist Before Committing Refactoring Code

- [ ] Refactoring addresses a specific, measurable problem
- [ ] Changes deepen at least one existing module (not create new ones)
- [ ] No new layer or abstraction added without justification
- [ ] Configuration, flags, or special cases reduced (not increased)
- [ ] Change amplification reduced (changes localized, not rippled)
- [ ] All existing callers still work without modification
- [ ] PR reviewers can understand the before/after in one pass
- [ ] Cognitive load reduced (fewer things to understand)
