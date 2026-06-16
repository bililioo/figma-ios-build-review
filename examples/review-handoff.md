# Example Review Handoff

This is the expected final shape after using `$figma-ios-build-review`.

## Summary

Reviewed `SettingsViewController` against the Figma populated-state frame and
updated nav spacing, row typography, icon asset names, and bottom safe-area
insets. Runtime account data differs from the Figma sample text; layout
resilience was compared with real data.

## Evidence Packet

- Figma source: `https://www.figma.com/design/FILE_KEY/App?node-id=12-34`
- Target surface/state: `SettingsViewController`, populated state
- Simulator: `iPhone 15 Pro`, iOS 18.5, `393x852`, `3x`
- Navigation method: committed flow `.harness/flows/settings.yaml`
- Harness cleanup: not used
- Runtime data: real account fixture from existing harness seed
- Accessibility ids asserted: `settings.root`, `settings.profileRow`
- Lookin evidence: hierarchy inspected; profile row frame and labels checked
- Screenshot evidence: `.harness/tmp/settings-after.png`
- Visual deltas fixed: title y-position, row height, icon tint, separator inset
- Accepted differences: real account name and email replace Figma sample copy
- Verification:
  - `git diff --check`: pass
  - `scripts/verify.sh`: pass
  - `FLOW_FILE=.harness/flows/settings.yaml scripts/verify-flow.sh`: pass

## Residual Risk

None for the reviewed state. Dark mode was outside the requested Figma frame.
