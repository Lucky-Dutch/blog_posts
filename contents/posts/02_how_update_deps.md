---
title: "How to update deps in Elixir"
date: "2026-03-07"
description: "How (not) to break anything"
---
## Updating dependencies
### Verify it
Before trying to change anything, you need to know what is outdated. Try this command:
```
mix hex.outdated
```
This command shows you a table with the actual versions, the possibility of updates, and the newest version in hex.

### Strategy — Let's update!
Now that you know what's outdated, here's how to update without breaking your project.

### First Step
#### Read the changelog before updating anything.
Why?
Every library worth its salt has a changelog or a list of releases on GitHub. Before you blindly bump a version, check what actually changed. Breaking changes, deprecations, and migration guides all live there. Skipping this step means you might spend hours debugging something that was clearly documented in a two-line changelog entry. Five minutes of reading can save you an afternoon of pain.

### Second Step
#### Update one dependency at a time.
Why?
It's tempting to run `mix deps.update --all` and call it a day. Don't. If something breaks, you'll have no idea which package caused it. Instead, update them one by one:
```
mix deps.update package_name
```
This way, if your tests fail or your app crashes, you know exactly which update is responsible. It's slower, but it's the only sane approach for production projects.

### Third Step
#### Understand the difference between patch, minor, and major updates.
Why?
Most Elixir packages follow semantic versioning (semver): `MAJOR.MINOR.PATCH`. A patch update (e.g., 1.2.3 → 1.2.4) is a bug fix — almost always safe. A minor update (e.g., 1.2.4 → 1.3.0) adds features but should be backwards-compatible. A major update (e.g., 1.3.0 → 2.0.0) can break anything. Treat patch updates casually, minor updates with a quick test, and major updates with full attention and a fresh cup of coffee.

### Fourth Step
#### Compile with warnings as errors after each update.
Why?
After updating a dependency, run:
```
mix deps.get && mix compile --warnings-as-errors
```
This catches deprecation warnings immediately. Libraries often deprecate functions in minor versions and remove them in major ones. If you catch the warning now, you can fix your code before the next update forces you to. Ignoring warnings is just borrowing time from your future self.

### Fifth Step
#### Run the test suite after every single update.
Why?
This one is non-negotiable. After each dependency update:
```
mix test
```
If the tests pass, commit the change right away with a clear message like `Update phoenix from 1.7.10 to 1.7.12`. If they fail, you know exactly which update broke things. This also keeps your git history clean — each commit represents one working change, making rollbacks trivial.

### Sixth Step
#### Relax version constraints in `mix.exs` when needed.
Why?
Sometimes `mix hex.outdated` shows a newer version available, but your constraint blocks it. For example, if your `mix.exs` has `{:phoenix, "~> 1.6.0"}`, it won't let you upgrade to 1.7.x. The `~>` operator only allows the last digit to increase. To allow minor updates, change it to `{:phoenix, "~> 1.6"}`. But be intentional about it — don't just remove all constraints. They exist to protect you from accidental breaking changes.

### Seventh Step
#### Use `mix hex.audit` to check for security vulnerabilities.
Why?
Outdated dependencies aren't just a maintenance issue — they can be a security risk. Run:
```
mix hex.audit
```
This command checks if any of your dependencies have been marked as retired or have known vulnerabilities. If something shows up, prioritize updating that package immediately. Security updates should always jump the queue, no matter how busy you are.

### Eighth Step
#### Tackle big version jumps carefully — use official upgrade guides.
Why?
When a framework like Phoenix releases a new major version (e.g., 1.6 → 1.7), there's usually an official upgrade guide. Follow it step by step. Don't try to skip major versions (e.g., jumping from 1.5 straight to 1.7) — each guide assumes you're coming from the previous version. Take it one major version at a time, run your tests after each jump, and commit between steps. It's a marathon, not a sprint.
