---
name: figma-ios-build-review
description: Build new or review existing native iOS UIKit screens from Figma, using SnapKit or Auto Layout, then iterate until the live UI matches the design. Use for from-zero Figma-to-code implementation, visual review, UI polish, Figma node/screenshot comparison, Maestro navigation flows, Lookin MCP hierarchy/screenshot inspection, simulator screenshots, accessibility identifiers, or repository harness commands.
---

# Figma iOS Build Review

Use this workflow to build or review native iOS UI from Figma and iterate until
the live app matches the design. Prefer real runtime state, but local UI
verification may use a Local Direct Harness. Use Figma as the visual reference,
not as a reason to add fake production app paths.

## Activation Contract

Use this skill only for native iOS UIKit surfaces where Figma fidelity must be
validated against a live simulator or a documented local harness. Before
editing, classify the request:

- **Full build/review**: Figma source plus an iOS repo/surface are available.
  Proceed with this skill.
- **Static review only**: Figma source is available, but no runnable iOS target
  or repo context exists. Review code/design statically and state that visual
  acceptance is blocked until a live screen can be inspected.
- **Asset export only**: The task is to download icons, illustrations, logos, or
  custom artwork. Use `$figma-ios-asset-export` instead of recreating the asset
  by hand.
- **Build/test/debug only**: The task is not Figma fidelity work. Use the repo's
  normal iOS build workflow or `$xcodebuildmcp`.

Do not claim visual review is complete unless a Figma source and a live native
screen or explicit screenshot fallback were inspected.

## Inputs

Collect only what is needed:

- Figma URL, file key, node id, or screenshot.
- Target surface and state: launch, loading, permission, empty, populated, or
  error.
- Target UIKit files or view controllers.
- Navigation route: launch-visible, committed Maestro flow, local route, or
  Local Direct Harness.
- Required assets and intended asset catalog names.

If a repository has an `AGENTS.md` or local Figma-to-code SOP, read it first and
let it override generic details.

## Mode Decision

Choose the path before editing:

- **Build From Figma**: Use when the user asks to implement a new Figma screen,
  create a missing page/component/cell, or go from design to UIKit code.
- **Review Existing UI**: Use when a UIKit surface already exists and the user
  asks to compare, polish, audit, or improve Figma fidelity.

Both paths end with the same visual review loop.

## Companion Skills

Use focused skills when the request splits into a narrower task:

- `$figma-ios-asset-export`: export real Figma artwork into iOS assets before or
  during implementation.
- `$xcodebuildmcp`: build, run, test, screenshot, or automate iOS simulator
  flows when the repo does not provide a wrapper.
- `$plan-before-code`: create an explicit iOS implementation plan when the user
  asks to plan first or the repo requires one.

## Shared Setup

1. Read the Figma node with Figma MCP. Capture node id, screenshot, dimensions,
   states, colors, typography, icons, assets, and important frames.
2. Read the target repo rules, existing UIKit/SnapKit patterns, navigation
   style, data stores, and harness commands.
3. If the work is non-trivial and the repo requires plans, add or update an
   execution plan under the repo's planning location.

## Build From Figma

Use this for from-zero implementation:

1. Decompose the Figma node into screen structure, reusable views, cells,
   controls, assets, states, and interactions.
2. Decide the target integration point: new view controller, existing view
   controller, child view, table/collection cell, route, or asset catalog entry.
3. Implement with existing UIKit, SnapKit or Auto Layout, stores, services,
   color helpers, and asset naming patterns.
4. Connect real runtime state when implementing product behavior. Use a Local
   Direct Harness only for local UI verification.
5. Add stable `accessibilityIdentifier` values only for controls that Maestro
   flows tap or assert.
6. Add or update routing so durable product screens are reachable through
   normal app navigation. Use a Local Direct Harness for local-only visual
   review when durable routing is unnecessary.
7. Add or update a committed Maestro flow if the new screen introduces a durable
   user interaction path.
8. Run the build/launch gate, then enter the visual review loop.

## Review Existing UI

Use this for fidelity review or polish:

1. Read the existing UIKit/SnapKit files, view models, assets, and flow ids.
2. Identify expected deltas from Figma: safe area, data differences, native
   behavior, missing assets, and dynamic text pressure.
3. Run the build/launch gate, put the simulator on the target page, and enter
   the visual review loop.

## Local Direct Harness

Use this for focused local visual verification when real navigation or Maestro
would add noise:

- Create mock or fixture dependencies needed to render the target state.
- Directly set the app window's `rootViewController` to the view controller
  under review.
- Scope the entry to Debug, harness, or test targets so it cannot affect normal
  production paths.
- Restore or remove temporary mock entries, launch hooks, and
  `rootViewController` overrides after review unless the harness is a committed
  reusable test entry.
- Record the mock dependencies, root view controller, and cleanup status in the
  final handoff.

## Visual Review Loop

Repeat this loop until the changed areas pass the comparison checklist:

1. Build and launch the Debug app with the repo's launch gate, usually:

   ```sh
   scripts/verify.sh
   ```

2. Put the simulator on the target page:
   - Use the latest harness `launch.png` for launch-visible screens.
   - Use committed Maestro flows for durable navigation.
   - Use a temporary hold flow under `.harness/tmp/` for local Lookin review.
     Assert the target root or key control before the final hold sentinel.
   - Use a Local Direct Harness when direct visual verification is faster.
3. Use Lookin MCP to inspect the live UIKit hierarchy and relevant attributes.
   If Lookin MCP is unavailable in the current environment, use simulator
   screenshots plus code inspection, and record the missing hierarchy evidence
   as residual risk.
4. Capture a live native screenshot with Lookin. If Lookin screenshot times out,
   the hierarchy is shallow, or Lookin only returns non-actionable layer data,
   use:

   ```sh
   xcrun simctl io "$VERIFY_SIMULATOR_ID" screenshot .harness/tmp/<surface>.png
   ```

5. Compare the live native screen with the Figma source. Adjust UIKit/SnapKit,
   assets, colors, typography, icon orientation, spacing, and dynamic sizing.
6. Rebuild, navigate back to the page, and repeat Lookin/screenshot inspection
   for changed visual areas.
7. Update the Evidence Packet after each meaningful iteration so final handoff
   reflects what was actually inspected.
8. Run the repo's verification gates and record any blocker.

## Maestro And Lookin

Use Maestro as the click driver and durable black-box gate. Use Lookin as the
native UIKit inspection bridge for hierarchy, attributes, and screenshots.
For focused visual review, a Local Direct Harness may replace Maestro
navigation. Prefer committed Maestro flows when validating durable end-to-end
navigation, taps, or assertions.

For iOS system permissions, prefer Maestro `launchApp.permissions` or
`setPermissions` so normal visual-review and smoke flows do not depend on
system alert copy. Keep ATT/usertracking alert taps optional when the local
Maestro/Xcode/iOS Simulator stack still shows the prompt after
`usertracking: allow`. Only make system permission alerts the target of a flow
when the flow is explicitly testing permission behavior.

Run Maestro flows serially when they target the same simulator. A single iOS
simulator can only be driven reliably by one Maestro process at a time, so do
not launch multiple flow commands in parallel just to save time. If a repo
provides a wrapper such as `scripts/verify-flow.sh`, prefer that wrapper when it
serializes committed flows and passes the selected simulator UDID.

Before starting a second local flow, make sure a previous `maestro` or
`verify-flow` process is not still holding the same simulator.

Committed flow example:

```sh
FLOW_FILE=.harness/flows/<flow-name>.yaml scripts/verify-flow.sh
```

Temporary hold flow example:

```yaml
appId: <bundle-id>
---
- launchApp:
    permissions:
      photos: allow
      usertracking: allow
- tapOn:
    text: "Allow"
    optional: true
- tapOn:
    id: "<entry.accessibilityIdentifier>"
- assertVisible:
    id: "<target.accessibilityIdentifier>"
- extendedWaitUntil:
    visible:
      id: "__hold_for_lookin__"
    timeout: 120000
```

The hold flow's final timeout is expected. Treat the useful result as the
successful navigation/assertions before the sentinel, not the final process exit
code. Do not commit hold flows or report their timeout as the verification
result.

## Comparison Checklist

Check these before calling the review done:

- Screen size, simulator model, scale, and safe-area differences.
- Nav height, title position, button frame, icon size, tint, and direction.
- Content start y-position, horizontal margins, row/card height, row spacing,
  scroll insets, and bottom safe area.
- Typography size, weight, line height, color, alignment, truncation, and
  dynamic text pressure.
- Background color/image, card fill, radius, shadow, tint, and opacity.
- Image slots, placeholders, content mode, clipping, and asset names.
- Runtime data differences that are acceptable because Figma uses sample
  content.

## Quality Bar And Failure Handling

Consider the visual review complete only when:

- In Build From Figma mode, the target surface is implemented, reachable through
  an accepted navigation method, and wired to real runtime state or documented
  mock/fixture verification state.
- The Figma source and the live native screen were both inspected.
- The live page was reached through an accepted navigation method:
  launch-visible state, real app navigation, committed Maestro flow, local hold
  flow, or Local Direct Harness.
- At least one live hierarchy inspection was completed with Lookin MCP.
- A live screenshot was inspected through Lookin or the simulator screenshot
  fallback.
- Visual differences were either fixed or explicitly recorded as acceptable
  native/runtime-data differences.
- The repo's relevant launch and flow gates passed, or the blocker was recorded
  with the exact command and artifact path.
- The final handoff includes an Evidence Packet with enough detail for another
  iOS engineer to reproduce the reviewed state.

Handle common failures this way:

- If Figma MCP cannot read the node, ask for a screenshot or a node with access
  rather than guessing design measurements.
- If no committed Maestro flow reaches the page, either create a local hold flow
  under `.harness/tmp/` or use a Local Direct Harness for inspection. Propose a
  committed flow only if the interaction should become a durable gate.
- If Lookin hierarchy fails, relaunch the Debug app, confirm the app is in the
  foreground, and record the blocker if Lookin still cannot connect.
- If Lookin returns only shallow hierarchy or non-actionable layer data, keep
  any useful evidence and use the simulator screenshot fallback for visual
  comparison.
- If Lookin screenshot fails but hierarchy works, use the simulator screenshot
  fallback and record that fallback.
- If the page is reachable only under a real runtime configuration, record the
  route and config condition that produced the reviewed state.
- If runtime data differs from Figma sample content, keep real app data and
  compare layout resilience instead of hard-coding Figma content.
- If verification cannot run, do not call the review complete; report the
  skipped command, reason, and residual risk.

## Hard Rules

- Do not add fake production routes, services, or stores to match Figma.
- Keep Local Direct Harness code scoped to Debug, harness, or test targets, and
  clean up temporary entries after review.
- Do not hard-code Figma sample rows when real app stores own the data.
- Use fixture data or existing harness setup from outside the app when stable
  visual data is needed.
- Add stable `accessibilityIdentifier` values only for controls that flows tap
  or assert.
- Do not commit DerivedData, build output, simulator recordings, harness run
  artifacts, local hold flows, screenshots, or Lookin dumps.
- Prefer native safe-area and platform behavior when Figma conflicts with iOS;
  record the decision.

## Verification And Handoff

Run the relevant local gates, commonly:

```sh
git diff --check
scripts/verify-docs.sh
scripts/verify.sh
scripts/verify-flow.sh
```

Run flow gates serially. If you need multiple specific flows, invoke them one
after another or use the repo's wrapper that serializes the flow list; parallel
Maestro runs can fight over the only available simulator and produce false
failures.

In the final summary or execution plan, include this Evidence Packet:

```markdown
Evidence Packet
- Figma source: <URL/node id/screenshot path>
- Target surface/state: <screen, component, cell, state>
- Simulator: <model, OS, logical size, scale>
- Navigation method: <real route, committed flow, hold flow, Local Direct Harness>
- Harness cleanup: <not used / removed / retained and why>
- Runtime data: <real data, fixture, mock dependency, or accepted difference>
- Accessibility ids asserted: <ids or none>
- Lookin evidence: <hierarchy inspected / screenshot captured / blocker>
- Screenshot evidence: <Lookin screenshot path or simctl screenshot path>
- Visual deltas fixed: <spacing, typography, assets, safe area, etc.>
- Accepted differences: <native behavior or runtime data differences>
- Verification: <commands and pass/fail/blocker>
```
