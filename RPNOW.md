# Lightning Studio fork of plus_plugins (RP Nowz)

This fork exists for one patch to **connectivity_plus**, used by the RP Nowz
app (`rpnow-app/pubspec.yaml`, a git dependency pinned to a commit of the
branch `rpnow/connectivity-windows-teardown`). Every other package here is
upstream's, unchanged, and the app takes them from pub.dev.

## What is patched

Branch `rpnow/connectivity-windows-teardown`, from the tag
`connectivity_plus-v7.3.1` (7.3.2 has the same Windows code). Only
`packages/connectivity_plus/connectivity_plus/windows/` changes:

- `connectivity_plus_plugin.cpp`: the plugin's destructor stops the network
  listener instead of calling `NetworkManager::Cleanup()`, leaving COM
  cleanup to the last owner of the shared manager (the event-stream handler
  can outlive the plugin). This is upstream PR
  [fluttercommunity/plus_plugins#4009](https://github.com/fluttercommunity/plus_plugins/pull/4009)
  by Zihao (Adherentman), cherry-picked with its standalone COM test in
  `windows/tests/`.
- `network_manager.cpp` / `.h`: `Cleanup()` stops the listener before it
  releases the Network List Manager and calls `CoUninitialize()`, calls it
  at most once, and only if `Init()`'s `CoInitialize()` succeeded.

## Why

On Windows the app crashed as it quit (Sentry FLUTTER-6T, 1.18.0+30,
`EXCEPTION_ACCESS_VIOLATION_READ` in `NetworkManager::StopListen` calling
`IConnectionPoint::Unadvise`): the plugin's destructor uninitialized COM while
the event stream still listened, then the manager's destructor stopped the
listener through a proxy COM had already unloaded and uninitialized COM a
second time, for one `CoInitialize()`.

## When to drop this fork

As soon as upstream publishes a connectivity_plus release with #4009 (or an
equivalent fix) merged: put `connectivity_plus: ^<that version>` back in
`rpnow-app/pubspec.yaml`, run `flutter pub get`, build the Windows app and
check that quitting leaves no "Application Error" for rpnowz.exe in Event
Viewer. Then this repository can be archived.

## Licence

Unchanged: the BSD licence in `LICENSE` and each package's `LICENSE` file
apply to this fork as to upstream.
