# Contributing to this fork

This is a fork of [SoFriendly/2fhey](https://github.com/SoFriendly/2fhey). Keep upstream credits, funding and download links, the application identity, and the existing CC0 license intact. Describe whether a change applies to this fork or is intended for upstream.

## Build

Use full Xcode and the shared `TwoFHey` scheme. Xcode resolves the SQLite.swift and HotKey packages declared in the project. The main application target declares macOS 12.4; the helper target has its own deployment settings.

**Build side effect:** the existing project includes a “Reset Accessibility Permissions” script phase that runs `tccutil reset Accessibility` for the app's bundle identifier. Building can reset an installed copy's Accessibility permission. This documentation does not change that build phase.

```bash
xcodebuild -project TwoFHey.xcodeproj -scheme TwoFHey \
  -configuration Debug -derivedDataPath /tmp/2fhey-build \
  CODE_SIGNING_ALLOWED=NO build
```

This is an unsigned compilation check, not a release or distribution workflow. Do not infer a passing test suite from a successful build: the shared scheme references `TwoFHeyTests`, but this checkout does not include that test target.

## Parser and language changes

Follow the [custom-service](./README.md#custom-services) and [language-file](./README.md#adding-a-new-language) instructions. Keep JSON examples valid, use synthetic messages, and check both expected matches and ordinary messages that should not match.

The current parser has `skipGitHubUpdate = true`; bundled language files are used, and remote updates are disabled. Do not document remote delivery as active unless the configuration actually changes. The configured remote source belongs to upstream, so changing files in this fork alone does not publish them to that update source.

## Documentation and privacy

Preserve installation, supported-format examples, shortcuts, customization instructions, cache behavior, and upstream attribution. Verify README links, badges, and images in desktop and narrow layouts. The [banner provenance](./assets/README.md) identifies the reused project logo.

Never include real verification codes, Messages databases, message contents, or private account screenshots in pull requests. A documentation-only change does not require granting Messages or Accessibility access. Report build warnings and untested runtime behavior explicitly.
