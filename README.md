# Figma iOS Build Review

> Build or review UIKit screens from Figma with a live simulator evidence loop.

[![skills.sh](https://skills.sh/b/bililioo/figma-ios-build-review)](https://skills.sh/b/bililioo/figma-ios-build-review)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Use this skill when an iOS UIKit screen, cell, or component must match a Figma
source and the result needs more than static code inspection. The workflow
combines Figma MCP, UIKit/SnapKit or Auto Layout, Maestro navigation, Lookin
hierarchy inspection, simulator screenshots, and a final Evidence Packet.

## When To Use It

- Implement a new UIKit screen from a Figma node.
- Review an existing UIKit surface against Figma and fix visual drift.
- Build a focused Local Direct Harness for visual inspection without adding fake
  production routes.
- Capture proof that the live native screen was inspected, not only compiled.

Do not use it for asset-only downloads; use `$figma-ios-asset-export`. Do not
use it for generic iOS build/test work; use the repo harness or `$xcodebuildmcp`.

## Quick Start

Install from GitHub:

```sh
npx skills add bililioo/figma-ios-build-review
```

Ask with a Figma node and target iOS surface:

```text
Use $figma-ios-build-review to review this UIKit screen against Figma:
<figma-url>
Target: SettingsViewController, populated state.
Navigation: .harness/flows/settings.yaml
```

The agent should return an Evidence Packet that records the Figma source, live
simulator state, Lookin result, screenshot path, visual changes, accepted
runtime differences, and verification commands.

## Output Example

![Evidence Packet demo](assets/showcase.gif)

See `examples/review-handoff.md` for the expected final handoff shape and
`examples/local-hold-flow.yaml` for a temporary Maestro hold flow template.

## Common Gates

Use the target repository's own wrappers when available. Typical local gates:

```sh
git diff --check
scripts/verify.sh
FLOW_FILE=.harness/flows/<flow-name>.yaml scripts/verify-flow.sh
```

## Verification Prompts

`test-prompts.json` contains sample prompts that should trigger build,
review, and static-review modes.

## Files

- `SKILL.md`: agent-facing UIKit/Figma visual review workflow.
- `examples/review-handoff.md`: expected final Evidence Packet shape.
- `examples/local-hold-flow.yaml`: temporary Maestro hold flow template.
- `assets/showcase.tape`: reproducible demo recording script for the showcase.
- `test-prompts.json`: trigger and behavior regression prompts.

## Safety Boundaries

- Do not add fake production routes, services, or stores to match Figma.
- Keep Local Direct Harness code scoped to Debug, harness, or test targets.
- Do not commit screenshots, Lookin dumps, simulator recordings, DerivedData,
  build products, or temporary hold flows.
- Do not claim visual review is complete without a Figma source and a live
  native screen or explicitly recorded screenshot fallback.
