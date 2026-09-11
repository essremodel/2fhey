<div align="center">

<img src="./assets/hero.png" alt="2FHey — the existing blue and teal 2FA speech-bubble logo beside the project name." width="100%">

# 2FHey

Automatically detect and copy verification codes from iMessage and SMS.

[![Swift](https://img.shields.io/badge/language-Swift-39c8d8?style=flat&labelColor=202020)](./TwoFHey.xcodeproj/project.pbxproj) [![CC0 license](https://img.shields.io/github/license/essremodel/2fhey?style=flat&labelColor=202020)](./LICENSE) [![Upstream fork](https://img.shields.io/badge/fork-SoFriendly%2F2fhey-39c8d8?style=flat&labelColor=202020)](https://github.com/SoFriendly/2fhey)

[Start here](#start-here) · [Installation](#installation) · [Shortcuts](#keyboard-shortcuts) · [Languages](#multi-language-support) · [Development](#development-notes)

</div>

[![Fund contributors](https://img.shields.io/badge/%F0%9F%91%91_Fund_contributors-royalty.dev-BB953A?style=for-the-badge&labelColor=1a1a1a)](https://app.royalty.dev/SoFriendly/2fhey)

This repository is a fork of [SoFriendly/2fhey](https://github.com/SoFriendly/2fhey). The application identity, upstream funding link, and upstream download destination are retained. Documentation here describes this checkout; the upstream downloadable app may differ.

## Start here

- **Install the upstream app:** follow [Installation](#installation), then review [Permissions](#permissions).
- **Use the shortcut:** jump to [Keyboard Shortcuts](#keyboard-shortcuts).
- **Customize detection:** see [Custom Services](#custom-services) and [Multi-Language Support](#multi-language-support).
- **Work on this fork:** see [Development Notes](#development-notes) and [CONTRIBUTING.md](./CONTRIBUTING.md).

## Features

- 🔐 Automatic OTP detection from messages
- 📋 Auto-copy to clipboard
- ⌨️ Optional auto-paste
- 🔔 Desktop notifications
- 🎯 Global keyboard shortcut (⇧⌘E) to resync messages
- 🌐 Keyword-based service identification plus an explicit [known-services list](./TwoFHey/OTPParser/OTPParserContants.swift)

## How It Works

2FHey uses smart keyword-based detection to identify verification code messages. When a message contains words like "verification", "code", "OTP", "PIN", etc., it automatically:

1. Extracts 4-8 digit codes (or alphanumeric codes)
2. Identifies the service (Google, Apple, Bank, etc.)
3. Copies the code to your clipboard
4. Shows a notification overlay
5. Optionally auto-pastes the code

## Supported Formats

The app automatically detects codes in various formats:
- Standard digits: `123456`
- Spaced/dashed: `123-456` or `123 456`
- Alphanumeric: `ABC123`, `X7Y9Z2`
- Google format: `G-12345`
- Chinese brackets: `【验证码123456】`

## Installation

1. Download from [2fhey.com](https://2fhey.com) (via Gumroad)
2. Move to Applications folder
3. Launch and follow the permission setup below.

### Permissions

| Permission | Purpose |
| --- | --- |
| **Full Disk Access** | Read the Messages database for iMessage/SMS detection |
| **Accessibility** | Support auto-paste and keyboard shortcuts |

Verification codes and message contents are sensitive. Do not include real messages, codes, database exports, or account details in issue reports. The examples below are illustrative code formats.

## Keyboard Shortcuts

- **⇧⌘E** (Shift + Command + E) - Resync messages and copy the latest OTP code to clipboard
  - Useful if 2FHey missed a message or you need to retrieve a recent code again
  - Can be disabled in Settings if it conflicts with other apps

## Custom Services

If you use a service that isn't automatically recognized, you can add it to the known services list by creating a PR to update `OTPParserConstants.knownServices` in `TwoFHey/OTPParser/OTPParserContants.swift`.

## Multi-Language Support

2FHey supports OTP detection in multiple languages including English, French, Spanish, Portuguese, German, Chinese, and Hebrew.

**Current checkout:** `skipGitHubUpdate = true` in [SimpleOTPParser.swift](./TwoFHey/OTPParser/SimpleOTPParser.swift), so the parser uses bundled files and skips remote updates. The remote-update flow below is implemented but only applies when that flag is disabled.

### How It Works

With remote updates enabled, the app uses a three-tier loading strategy:

1. **First launch:** Loads from bundled language files and custom patterns (immediate availability)
2. **Subsequent launches:** Loads from cached files (fast)
3. **Background update:** Fetches latest from GitHub on each app launch

When enabled, updates use these upstream URLs:
```text
https://raw.githubusercontent.com/SoFriendly/2fhey/main/TwoFHey/OTPKeywords/{language}.json
https://raw.githubusercontent.com/SoFriendly/2fhey/main/TwoFHey/OTPKeywords/custom-patterns.json
```

### Adding a New Language

To add support for a new language:

1. Create a new JSON file in `TwoFHey/OTPKeywords/` with the following structure:
   ```json
   {
     "keywords": [
       "code",
       "verification",
       "verify"
     ],
     "patterns": [
       "code[\\s:]+([\\d\\s-]{4,8})",
       "verification[\\s:]+([\\d\\s-]{4,8})"
     ]
   }
   ```

2. Add the filename to the `languageFiles` array in [SimpleOTPParser.swift](./TwoFHey/OTPParser/SimpleOTPParser.swift)

3. Submit a pull request

4. With the current bundled-only setting, rebuild to include the new file. Existing remotely loaded language files can update on a later launch when remote updates are enabled and the changes are published to the configured upstream repository; adding a language also requires updating the app's filename list.

### Adding Custom Service Patterns

For services with unique OTP formats (like Chase, Geico, etc.), you can add patterns to `TwoFHey/OTPKeywords/custom-patterns.json`:

```json
{
  "customPatterns": [
    {
      "service": "YourService",
      "pattern": "YourService code: (\\d{6})"
    }
  ]
}
```

**Pattern tips:**
- Use capture groups `()` to extract the code
- First capture group will be used as the OTP code
- Service name is automatically associated with the pattern
- Patterns are checked first (highest priority)

Custom patterns can also update from GitHub on launch when remote updates are enabled. The current bundled-only setting requires rebuilding to include changes.

### Language File Format

Each language file contains:

- **keywords**: An array of words that indicate an OTP message (e.g., "code", "verification", "vérification")
  - All keywords from all languages are merged and checked together
  - This allows detection of multilingual messages

- **patterns**: An array of regex patterns for language-specific code extraction
  - These are high-priority patterns like `"验证码：123456"` or `"code: 123456"`
  - Pattern captures should extract just the numeric code in capture group 1
  - Patterns are checked before generic digit extraction

### Cache Location

When remote updates are enabled, downloaded language files and custom patterns are cached at: `~/Library/Caches/OTPKeywords/`

### Offline Support

With remote updates enabled, the implementation can fall back to the following when GitHub is unreachable:
1. Cached files from previous downloads
2. Bundled files included in the app

## Development Notes

The existing documentation describes **Version 2.0+** as using a simplified keyword-based OTP parser (`SimpleOTPParser.swift`). The current implementation combines keyword heuristics with language and custom regex patterns.

The older `AppConfig.json` and service-specific regex approach remains in the repository for reference. The active simplified parser uses keyword detection and configured patterns; recognition is not guaranteed for every service.

The shared Xcode scheme is `TwoFHey`; the main app target declares macOS 12.4. See [CONTRIBUTING.md](./CONTRIBUTING.md) for an unsigned build check. This fork's README does not imply a separately published app release.

## License & attribution

[CC0 1.0 Universal](./LICENSE), as declared in the existing license file. Original project: [SoFriendly/2fhey](https://github.com/SoFriendly/2fhey). The README banner incorporates the repository's existing 2FA speech-bubble logo; application assets and branding are unchanged.
