---
name: rspec-test-prof
description: Use when writing, reviewing, or improving RSpec tests in a Rails app. Triggers on writing new specs, adding test coverage, reviewing test code for quality, dealing with slow tests, setting up test infrastructure, or any question about RSpec best practices. Use this skill whenever the user is working with RSpec tests, even if they don't mention TestProf or performance — the patterns here produce better tests regardless.
---

# Writing Quality RSpec Tests with TestProf

This skill guides writing RSpec tests that are fast, clear, and maintainable. It uses [TestProf](https://test-prof.evilmartians.io) recipes as the default toolkit — not because speed is the only thing that matters, but because the patterns TestProf encourages (`let_it_be`, `before_all`, `factory_default`) also lead to better-structured tests.

These are strong defaults, not rules. Sometimes vanilla `let!` or `create` is the right call. Trust judgment and context.

## The Core Idea

Most slow test suites aren't slow because the tests do too much — they're slow because the *setup* does too much. A 50-example spec file that uses `let!(:user) { create(:user) }` creates (and destroys) that user 50 times. The actual assertions take milliseconds; the database churn takes minutes.

TestProf's recipes fix this by sharing setup across examples safely, using database transactions for isolation. The result: tests that run faster *and* read better, because the setup reflects what's actually shared vs. what varies per test.

## Setup: Share What's Shared

### `let_it_be` over `let!`

`let_it_be` creates a record once for the entire example group, wrapped in a transaction that rolls back after the group finishes. Each example gets the same record, and database isolation is handled automatically.

```ruby
describe Card do
  let_it_be(:user) { create(:user) }
  let_it_be(:board) { create(:board, creator: user) }
  let_it_be(:card) { create(:card, board: board, creator: user) }

  it "belongs to a board" do
    expect(card.board).to eq(board)
  end

  it "has a creator" do
    expect(card.creator).to eq(user)
  end
end
```

### `let_it_be` handles mutations safely

`let_it_be` creates a savepoint before each example and rolls back to it after, so database changes made during an example are automatically undone. The Ruby object in memory is the same instance across examples though, so in-memory mutations persist unless you use a modifier.

Use modifiers to control how the object refreshes between examples:

- **`reload: true`** — calls `.reload` before each example. Resets attributes from the (rolled-back) database. Use when tests modify the record via DB operations.
- **`refind: true`** — does `Model.find(id)` before each example. Returns a completely fresh ActiveRecord object, clearing association caches too. Use when tests mutate the in-memory object.
- **`freeze: true`** — freezes the object so mutations raise immediately. Great as a safety net to catch unintended modification.

```ruby
# Tests that modify the card's DB state — reload gets the rolled-back version
let_it_be(:card, reload: true) { create(:card, creator: user) }

# Tests that mutate associations in memory — refind gets a clean object
let_it_be(:card, refind: true) { create(:card, creator: user) }
```

A good global default is to enable `freeze` so accidental mutations are caught early:

```ruby
# spec/support/test_prof.rb
TestProf::LetItBe.configure do |config|
  config.default_modifiers[:freeze] = true
end
```

Fall back to `let!` only when each example genuinely needs different attributes — not because of mutation concerns, which `let_it_be` handles via savepoints and modifiers.

### `before_all` for complex setup

When setup involves multiple related records or procedural steps, use `before_all` instead of `before(:each)`. Same transaction-wrapping as `let_it_be`, but for imperative setup code.

```ruby
describe "board with populated columns" do
  before_all do
    @board = create(:board)
    @columns = create_list(:column, 3, board: @board)
    @cards = @columns.flat_map { |col| create_list(:card, 5, column: col) }
  end

  it "has 15 cards total" do
    expect(@board.cards.count).to eq(15)
  end

  it "distributes cards across columns" do
    expect(@columns.map { |c| c.cards.count }).to all(eq(5))
  end
end
```

Without `before_all`, this setup runs before every single example — creating 18 records each time. With `before_all`, it runs once.

## Factories: Kill the Cascade

### The problem

A factory cascade happens when creating one record triggers creation of many associated records behind the scenes. `create(:comment)` might create a user, a post, another user for the post, a board, a category — 6 records when you only asked for 1.

### `create_default` to share associations

`create_default` registers a record as the default for its factory. Any subsequent factory call that would normally create that association reuses the default instead.

```ruby
describe "commenting on posts" do
  let_it_be(:user) { create_default(:user) }
  let_it_be(:board) { create_default(:board) }
  let_it_be(:post) { create(:post) } # reuses default user and board

  it "creates a comment without duplicating users" do
    comment = create(:comment, post: post)
    # comment.user is the default user — no new user created
    expect(comment.user).to eq(user)
  end
end
```

This is especially powerful combined with `let_it_be` — the defaults are created once and reused across every example and every factory call in the group.

### `build_stubbed` when you don't need the database

If the test only exercises in-memory behavior — formatting, calculations, validations — don't hit the database at all.

```ruby
# Good — no database needed to test a display method
it "formats the full name" do
  user = build_stubbed(:user, first_name: "Ada", last_name: "Lovelace")
  expect(user.full_name).to eq("Ada Lovelace")
end
```

Use `build_stubbed` over `build` when possible — it assigns a fake ID and stubs persistence checks, so association methods work without touching the DB.

## Structure: Organize for Clarity

### Describe behavior, not implementation

```ruby
# Good — describes what the user-facing behavior is
describe Card::Closeable do
  describe "#close" do
    it "marks the card as closed" do ...
    it "records who closed it" do ...
    it "is idempotent if already closed" do ...
  end
end

# Avoid — testing implementation details
describe Card do
  it "calls create_closure! with the current user" do ...
  it "sets the closed_at timestamp" do ...
end
```

### Multiple assertions per example are fine

Don't split related expectations into separate examples just for the sake of "one assertion per test." Verifying multiple facets of the same behavior in one `it` block gives you the full picture in one place and avoids redundant setup.

```ruby
it "closes the card" do
  card.close(user: closer)
  expect(card).to be_closed
  expect(card.closure.user).to eq(closer)
  expect(card.closed_at).to be_present
end
```

This works especially well when the project has `aggregate_failures` enabled globally — when one expectation fails, the rest still run so you see everything that's wrong at once. That's a project-level configuration choice, not something to add per-test.

### Use contexts to vary conditions, not to nest deeply

```ruby
describe "#publish" do
  let_it_be(:author) { create(:user) }
  let_it_be(:article) { create(:article, author: author) }

  context "when the article is a draft" do
    it "publishes successfully" do ...
  end

  context "when the article is already published" do
    it "raises an error" do ...
  end
end
```

Avoid nesting contexts more than 2 levels deep. If you're nesting deeper, the test is probably trying to cover too many scenarios in one file — split it up.

## Profiling: Diagnose Before You Optimize

When tests are slow, measure before guessing. TestProf provides profilers you run via environment variables — no code changes needed.

### Step 1: Where is time going?

```bash
# Time breakdown by test type (model, controller, feature, etc.)
TAG_PROF=type bundle exec rspec
```

### Step 2: Is it factory usage?

```bash
# Top files by factory creation time
EVENT_PROF='factory.create' bundle exec rspec

# Visualize factory cascades as a flamegraph
FPROF=flamegraph bundle exec rspec
```

### Step 3: Is it setup overhead?

```bash
# Time in before hooks vs. actual test body
RD_PROF=1 bundle exec rspec
```

If RSpecDissect shows 80%+ time in `before` hooks, that's a clear signal to convert to `before_all` / `let_it_be`.

### Step 4: Apply the right recipe

| Diagnosis | Recipe |
|---|---|
| Same records recreated per example | `let_it_be` |
| Heavy `before(:each)` setup | `before_all` |
| Factory cascades (create 1, get 6) | `create_default` |
| Records needed suite-wide | `AnyFixture` |
| No database needed for this test | `build_stubbed` |

## Setup Checklist

When adding TestProf to a project, configure it in `spec/support/test_prof.rb`:

```ruby
require "test_prof/recipes/rspec/let_it_be"
require "test_prof/recipes/rspec/before_all"
require "test_prof/recipes/rspec/factory_default"

TestProf::LetItBe.configure do |config|
  config.default_modifiers[:freeze] = true
end

TestProf::FactoryDefault.configure do |config|
  config.preserve_traits = true
end
```

Consider enabling `aggregate_failures` globally so every example reports all failures at once instead of stopping at the first:

```ruby
# spec/spec_helper.rb or spec/rails_helper.rb
RSpec.configure do |config|
  config.define_derived_metadata do |metadata|
    metadata[:aggregate_failures] = true
  end
end
```

Ensure the test suite uses transactional tests (the Rails default). TestProf's transaction wrapping doesn't work with truncation-based database cleaning strategies.

## Summary of Defaults

| Situation | Default approach |
|---|---|
| Record shared across examples? | `let_it_be` |
| Record modified by examples? | `let_it_be` with `reload: true` or `refind: true` |
| Complex multi-record setup? | `before_all` |
| Factory creates too many records? | `create_default` for shared associations |
| Testing in-memory behavior? | `build_stubbed` |
| Tests are slow? | Profile first (TagProf → EventProf → RSpecDissect → FactoryProf) |
| Deeply nested contexts? | Flatten or split the file |
