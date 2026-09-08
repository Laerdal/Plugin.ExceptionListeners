---
name: code-review
description: Use for reviewing pull requests on Plugin.ExceptionListeners — checks IDisposable lifecycle correctness, the Core/Maui platform boundary, and shared-event-model consistency across listener implementations.
---

# Code Review — Plugin.ExceptionListeners

## Checklist for new or changed listeners

- [ ] **`IDisposable` lifecycle**: every listener subscribes in its constructor (or an explicit
      `Start`/init method) and unsubscribes cleanly in `Dispose()`. A listener that can leak its
      underlying event subscription — e.g. subscribing twice on re-init, or not unsubscribing at
      all — is a real finding, not a style nit.
- [ ] **Shared `ExceptionEventArgs` model**: every listener raises the existing
      `ExceptionEventArgs`, not a parallel/ad-hoc payload type. If new data is genuinely needed,
      it should extend `ExceptionEventArgs`, not fork the event model.
- [ ] **Core vs. `.Maui` boundary**: anything referencing a native platform exception type
      (`NSException`, `Java.Lang.Throwable`, Win32 exceptions) belongs in
      `Plugin.ExceptionListeners.Maui`, never in the platform-agnostic
      `Plugin.ExceptionListeners` core project. Flag any `#if` platform conditional or native
      type reference that's landed in core.
- [ ] **New listener naming/location**: follows the existing `Listeners/*Listener.cs` pattern
      (core) or the `.Maui` project's native-bridging pattern — not a new ad-hoc location.
- [ ] **XML docs**: new public members have accurate `<summary>`/`<param>`/`<returns>`/
      `<exception>` docs.
- [ ] **README feature list**: a new listener type should be reflected there.

## What to flag as a real risk, not a nit

- A listener that swallows or logs-and-drops the underlying exception without giving the
  subscriber a way to observe it — the entire point of this library is not silently losing
  exception information.
- Any dependency added on `Plugin.ByteArrays`/`Plugin.BaseTypeExtensions` — these are declared
  in `Directory.Packages.props` under "INTERNAL DEPENDENCIES" but not currently referenced by
  any `.csproj` here. If a PR starts actually using one, that's worth calling out explicitly
  rather than letting it slide in as an incidental dependency.
