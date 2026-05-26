---
name: domain-modeling
description: Use when designing models, planning features, structuring controllers, or making architectural decisions in a Rails app. Triggers on questions like "how should I model X", "where does this logic go", "should I make a service object", adding new resources, or restructuring existing domain logic. Use this skill whenever the user is thinking about where behavior lives in their app.
---

# Domain Modeling

This skill guides domain modeling decisions. The approach favors rich models, vanilla Rails conventions, and pragmatic simplicity over architectural ceremony.

The principles here are strong defaults, not commandments. Sometimes the right call is something different. Trust your judgment and the user's context.

## The Core Idea

The domain model *is* the application. Controllers are thin dispatchers. Views render state. But the interesting stuff — the business rules, the state transitions, the "what happens when" — lives on the models.

This means when someone asks "where should this logic go?", the answer is almost always: on a model. Not in a service object. Not in a form object. Not in a "manager" or "interactor" or "use case." On the model that the behavior belongs to.

## Rich Models, Thin Controllers

Controllers should read like a table of contents. They find the record, call a method, and render a response. The method name on the model should reveal the intention — `@card.close`, `@article.publish`, `@message.deliver` — without the controller needing to orchestrate the steps.

```ruby
# Good — the controller states intent, the model knows how
class Cards::ClosuresController < ApplicationController
  def create
    @card.close
  end
end

# The model owns the behavior
module Card::Closeable
  def close(user: Current.user)
    unless closed?
      transaction do
        not_now&.destroy
        create_closure! user: user
        track_event :closed, creator: user
      end
    end
  end
end
```

If you find a controller method growing beyond a few lines of domain logic, that logic probably belongs on the model.

## Concerns as the Primary Decomposition Tool

When a model accumulates behavior (and it will — that's the point), decompose it using concerns namespaced under the model. Each concern groups a cohesive slice of behavior: the associations, scopes, validations, and methods that relate to one capability.

```
app/models/card.rb              # includes Card::Closeable, Card::Assignable, etc.
app/models/card/closeable.rb    # everything about closing/reopening
app/models/card/assignable.rb   # everything about assignment
app/models/card/watchable.rb    # everything about watches/notifications
```

The model file itself becomes a manifest — you read the `include` list and immediately understand what the model can do. Each concern is self-contained: it declares its own associations, scopes, callbacks, and methods.

This is the natural Rails mechanism for this. You don't need a new pattern.

### When to Extract a Concern

- When a cluster of methods, scopes, and associations all serve the same capability
- When you can name the concern with an adjective or "-able" that describes the capability (`Closeable`, `Searchable`, `Taggable`)
- When the concern can be understood in isolation, without reading the rest of the model

### When NOT to Extract a Concern

- For a single method — that's just a method, leave it on the model
- When the behavior is deeply entangled with other model behavior and the concern wouldn't make sense on its own
- To hide complexity — if the concern is just as confusing as the original code, extraction didn't help

## Controllers as Resources, Not RPC

Every meaningful action gets its own controller, modeled as a RESTful resource. "Closing a card" isn't a custom action on `CardsController` — it's `Cards::ClosuresController#create`. "Reopening" is `Cards::ClosuresController#destroy`.

This pattern scales well because:
- Each controller stays small (often just `create` and/or `destroy`)
- The URL structure reveals the domain model
- You avoid `CardsController` becoming a junk drawer of custom actions

```ruby
# Instead of CardsController#close, CardsController#assign, CardsController#pin...
# Each gets its own resource controller:
Cards::ClosuresController       # create/destroy
Cards::AssignmentsController    # create/destroy
Cards::PinsController           # create/destroy
Cards::DraftsController         # show/update
```

Nest controllers to express relationships. `Boards::ColumnsController` tells you columns belong to boards. `Cards::Comments::ReactionsController` tells you reactions belong to comments on cards. Let the URL structure mirror the domain.

## What About Service Objects?

You probably don't need them.

The common argument for service objects is "my model is too big." But a big model decomposed into well-named concerns is easy to navigate. A constellation of service objects is not — it scatters the domain across files that exist outside the natural Rails conventions.

Service objects also tend to multiply. Once you have `CloseCardService`, someone will create `ReopenCardService`, `AssignCardService`, `PinCardService`... and now the model is a data bag and all the behavior lives in a parallel hierarchy that Rails doesn't help you with.

That said, sometimes an operation genuinely doesn't belong to any single model — it coordinates across multiple aggregates, or it represents a process that's bigger than any one record. In those cases, a plain Ruby class is fine. Just don't reach for it as the default. The bar should be: "I tried putting this on a model and it genuinely doesn't fit."

## Naming and Language

Name things after what they *are* in the domain, not after patterns or technical roles.

- `Card::Closeable`, not `CardClosingService`
- `Account::Export`, not `AccountExportJob` (the job is just the async wrapper — the export logic lives on the model)
- `Board::Publication`, not `BoardPublisher`

Use `_later` for methods that enqueue background work, `_now` for the synchronous version:

```ruby
class Card
  def notify_watchers_later
    NotifyWatchersJob.perform_later(self)
  end

  def notify_watchers_now
    watchers.each { |w| w.notify(self) }
  end
end
```

## State Through Records, Not Flags

Prefer modeling state transitions as the creation or destruction of associated records rather than boolean flags or status columns.

A card isn't `closed: true` — it *has a closure*. A card isn't `pinned: true` — it *has a pin*. This gives you timestamps for free (`closure.created_at`), an actor (`closure.user`), and the ability to store additional context on the state transition.

```ruby
# The existence of a closure record means the card is closed
def closed?
  closure.present?
end

def close(user: Current.user)
  create_closure!(user: user)
end

def reopen
  closure&.destroy
end
```

This won't always be the right fit. A simple `published` boolean is fine when you don't need history or attribution. Use judgment.

## Keep the Gemfile Short

Fight hard before adding a dependency. Every gem is a future upgrade burden, a potential security surface, and a thing the next developer has to understand. If you can do it with Rails and a few lines of code, do that.

This applies to architectural gems especially — things that introduce their own conventions for organizing code (Trailblazer, Dry-rb, Interactor, etc.). They're solving a problem that vanilla Rails already handles if you use it well.

## Summary of Defaults

| Situation | Default approach |
|---|---|
| Where does behavior go? | On the model |
| Model getting big? | Extract concerns |
| Need a new action? | New resource controller |
| Multi-step operation? | Method on the model, in a transaction |
| Async work? | `_later` method wraps a job that calls `_now` |
| State transition? | Create/destroy an associated record |
| "Should I add a gem?" | Probably not |
| "Should I make a service object?" | Probably not — try a concern first |

These are starting points. If the situation calls for something else, do something else.
