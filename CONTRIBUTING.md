# Contributing

## Branching

Work off a short-lived feature branch cut from `main`. Open a PR early and mark it as a Draft
if it's still in progress.

## Commit convention

This repo uses [Conventional Commits](https://www.conventionalcommits.org/): `type(scope): subject`,
imperative mood, <= 72 characters. Common types: `feat`, `fix`, `docs`, `style`, `refactor`,
`perf`, `test`, `chore`. Scope is optional but recommended (e.g. `listeners`, `maui`, `tests`).

## Pull requests

Fill out the PR template. Keep each PR scoped to one logical change. CI must pass before merge.

## Code style

Follow this repo's `.editorconfig`. Prefer clarity over cleverness; avoid unnecessary abstraction
— this is a firm preference, not a suggestion. Keep the core listener abstractions in
`Plugin.ExceptionListeners` platform-agnostic; anything that touches native platform exception
types (`NSException`, `Java.Lang.Throwable`, Win32 exceptions) belongs in
`Plugin.ExceptionListeners.Maui`. Every listener should implement `IDisposable` and raise the
shared `ExceptionEventArgs` model — see `.github/copilot-instructions.md` for the existing
conventions.

## Testing

All new functionality needs xUnit tests in `Plugin.ExceptionListeners.Tests`, using
FluentAssertions for assertions. Cover both the "exception raised" and "listener disposed
correctly" paths.

## Documentation

Public APIs need XML doc comments (`<summary>`, `<param>`, `<returns>`, `<exception>`). If you
add a new listener type, update the README's feature list too.
