---
name: agent-device-mobile-automation
description: Control iOS, Android, TV, and desktop devices for AI-driven mobile app testing and verification
triggers:
  - test my mobile app with an AI agent
  - automate iOS simulator for testing
  - inspect Android app UI programmatically
  - verify mobile app changes on real devices
  - capture screenshots and logs from mobile app
  - debug mobile app with AI assistance
  - create mobile e2e tests with agent-device
  - profile React Native app performance
---

# agent-device Mobile Automation

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

`agent-device` is a CLI tool designed for AI agents to control and verify mobile apps on iOS, Android, TV, web, and desktop platforms. It provides token-efficient snapshots, semantic element references, and evidence capture for AI-driven testing workflows.

## What agent-device Does

- **Inspect** app UI through structured accessibility snapshots with interactive refs (`@e1`, `@e2`, etc.)
- **Interact** with apps: open, tap, type, scroll, gestures, wait, assert, handle alerts
- **Capture** evidence: screenshots, videos, logs, traces, network traffic, performance samples
- **Replay** workflows by recording `.ad` scripts for CI and repeatable e2e checks
- **Cross-platform** support for iOS Simulator, Android Emulator, physical devices, tvOS, macOS, Linux

## Installation

### Global CLI Installation

```bash
npm install -g agent-device@latest
```

### Verify Installation

```bash
agent-device doctor
agent-device --version
agent-device help workflow
```

### Prerequisites by Platform

- **All platforms**: Node.js 22+
- **iOS/tvOS/macOS**: Xcode with command-line tools, macOS Accessibility permissions
- **Android**: Android SDK, ADB configured
- **Web**: Node.js 24+
- **Linux desktop**: AT-SPI accessibility support

Check setup:

```bash
agent-device doctor --platform ios
agent-device doctor --platform android
```

## Key Commands Reference

### Discovery & Session Management

```bash
# List available apps
agent-device apps --platform ios
agent-device apps --platform android

# Start session with an app
agent-device open "AppName" --platform ios
agent-device open com.example.app --platform android

# Close current session
agent-device close
```

### Inspection

```bash
# Full accessibility snapshot
agent-device snapshot

# Interactive elements only (recommended for agents)
agent-device snapshot -i

# With component tree (React Native apps)
agent-device snapshot --components

# With selectors visible
agent-device snapshot --selectors
```

**Snapshot Output Example:**

```
@e1 [heading] "Settings"
@e2 [button] "Sign In"
@e3 [text-field] "Email"
@e4 [button] "Submit"
```

### Interaction Commands

```bash
# Tap element by ref
agent-device tap @e2

# Tap by selector
agent-device tap 'button[label="Sign In"]'

# Fill text field
agent-device fill @e3 "user@example.com"

# Type text (supports keyboard input)
agent-device type "Hello World"

# Scroll
agent-device scroll down
agent-device scroll up --amount 0.5

# Swipe gesture
agent-device swipe left
agent-device swipe right --from @e5

# Wait for element
agent-device wait @e6 --timeout 5000

# Wait for time
agent-device wait 2000

# Assert element exists
agent-device assert @e7 --exists

# Assert element has text
agent-device assert @e8 --text "Welcome"

# Handle alerts
agent-device alert accept
agent-device alert dismiss
```

### Evidence Capture

```bash
# Take screenshot
agent-device screenshot ./artifacts/screen.png

# Record video (start/stop)
agent-device video start
# ... perform actions ...
agent-device video stop ./artifacts/recording.mp4

# Capture logs
agent-device logs ./artifacts/app.log

# Capture network traffic
agent-device network start
# ... perform actions ...
agent-device network stop ./artifacts/network.har

# Performance profile
agent-device profile start
# ... perform actions ...
agent-device profile stop ./artifacts/perf.json

# React Native profile
agent-device react-profile start
# ... interact with app ...
agent-device react-profile stop ./artifacts/react-profile.json
```

### Replay & Recording

```bash
# Record commands to .ad script
agent-device record start ./scripts/login-flow.ad

# Execute commands...
agent-device open MyApp --platform ios
agent-device tap @e1
agent-device fill @e2 "test@example.com"

# Stop recording
agent-device record stop

# Replay script
agent-device replay ./scripts/login-flow.ad

# Export to Maestro YAML
agent-device export maestro ./scripts/login-flow.ad --output ./maestro/login.yaml
```

## Configuration

### Session Options

```bash
# Specify device
agent-device open MyApp --platform ios --device "iPhone 15 Pro"

# Launch with arguments
agent-device open MyApp --platform ios --args "--env=staging"

# Use specific bundle ID
agent-device open com.company.myapp --platform android
```

### Global Flags

```bash
# Verbose output for debugging
agent-device snapshot -i --verbose

# JSON output for programmatic use
agent-device snapshot --format json

# Specify timeout
agent-device wait @e1 --timeout 10000
```

### Environment Variables

```bash
# Platform preference
export AGENT_DEVICE_PLATFORM=ios

# Device preference
export AGENT_DEVICE_DEVICE="iPhone 15 Pro"

# Artifacts directory
export AGENT_DEVICE_ARTIFACTS=./test-artifacts
```

## Real Code Examples

### Example 1: Login Flow Verification

```bash
#!/bin/bash
# Verify login flow on iOS

# Start session
agent-device open "MyApp" --platform ios

# Take initial screenshot
agent-device screenshot ./artifacts/01-launch.png

# Get interactive elements
agent-device snapshot -i

# Assuming snapshot shows:
# @e1 [text-field] "Email"
# @e2 [text-field] "Password"
# @e3 [button] "Log In"

# Fill credentials
agent-device fill @e1 "$TEST_EMAIL"
agent-device fill @e2 "$TEST_PASSWORD"

# Screenshot before submit
agent-device screenshot ./artifacts/02-filled.png

# Submit
agent-device tap @e3

# Wait for home screen
agent-device wait 'heading[label="Welcome"]' --timeout 5000

# Verify success
agent-device snapshot -i
agent-device screenshot ./artifacts/03-logged-in.png

# Close session
agent-device close
```

### Example 2: Android E2E Test with Evidence

```bash
#!/bin/bash
# Android checkout flow with full evidence capture

agent-device open com.example.shop --platform android

# Start evidence capture
agent-device video start
agent-device network start
agent-device logs start ./artifacts/app.log

# Navigate to product
agent-device snapshot -i
agent-device tap 'button[text="Shop"]'
agent-device wait 2000

agent-device snapshot -i
agent-device tap '@e5' # Assuming @e5 is a product

# Add to cart
agent-device wait 'button[text="Add to Cart"]'
agent-device tap 'button[text="Add to Cart"]'
agent-device screenshot ./artifacts/added-to-cart.png

# Go to cart
agent-device tap 'button[content-desc="Cart"]'
agent-device wait 2000
agent-device snapshot -i

# Checkout
agent-device tap 'button[text="Checkout"]'
agent-device wait 3000

# Stop evidence capture
agent-device video stop ./artifacts/checkout-flow.mp4
agent-device network stop ./artifacts/network.har

agent-device close
```

### Example 3: React Native Component Inspection

```typescript
// From Node.js/TypeScript context
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function inspectReactNativeApp() {
  // Open app
  await execAsync('agent-device open MyRNApp --platform ios');
  
  // Get component tree
  const { stdout } = await execAsync('agent-device snapshot --components --format json');
  const snapshot = JSON.parse(stdout);
  
  // Find specific component
  const loginButton = snapshot.components.find(
    (c: any) => c.type === 'Button' && c.props?.title === 'Log In'
  );
  
  if (loginButton) {
    console.log('Found login button:', loginButton.ref);
    await execAsync(`agent-device tap ${loginButton.ref}`);
  }
  
  // Take screenshot
  await execAsync('agent-device screenshot ./artifacts/after-tap.png');
  
  // Close
  await execAsync('agent-device close');
}
```

### Example 4: Replay Script (.ad format)

Create `login-test.ad`:

```
# Login flow replay script
open MyApp --platform ios
wait 2000
snapshot -i
fill @e1 "test@example.com"
fill @e2 "password123"
screenshot ./artifacts/login-filled.png
tap @e3
wait heading[label="Dashboard"] --timeout 5000
assert heading[label="Dashboard"] --exists
screenshot ./artifacts/dashboard.png
close
```

Execute:

```bash
agent-device replay ./login-test.ad
```

### Example 5: Performance Profiling

```bash
#!/bin/bash
# Profile app launch and navigation

agent-device open MyApp --platform ios

# Start performance profiling
agent-device profile start

# Navigate through screens
agent-device tap @e1
agent-device wait 1000
agent-device scroll down
agent-device wait 1000
agent-device tap @e5
agent-device wait 2000

# Stop and save profile
agent-device profile stop ./artifacts/navigation-perf.json

# React Native specific profiling
agent-device react-profile start
agent-device tap 'button[label="Load Data"]'
agent-device wait 3000
agent-device react-profile stop ./artifacts/react-render.json

agent-device close
```

## Common Patterns

### Pattern 1: Conditional Flow Based on Snapshot

```bash
# Take snapshot and parse
SNAPSHOT=$(agent-device snapshot -i)

if echo "$SNAPSHOT" | grep -q "Sign In"; then
  echo "User not logged in, performing login..."
  agent-device tap 'button[label="Sign In"]'
  agent-device fill @e1 "$USER_EMAIL"
  agent-device fill @e2 "$USER_PASSWORD"
  agent-device tap 'button[label="Submit"]'
  agent-device wait 3000
else
  echo "User already logged in"
fi
```

### Pattern 2: Retry with Evidence

```bash
retry_with_screenshot() {
  local max_attempts=3
  local attempt=1
  
  while [ $attempt -le $max_attempts ]; do
    echo "Attempt $attempt..."
    
    if agent-device tap "$1" 2>/dev/null; then
      echo "Success"
      return 0
    fi
    
    agent-device screenshot "./artifacts/retry-$attempt.png"
    agent-device snapshot -i > "./artifacts/snapshot-$attempt.txt"
    
    attempt=$((attempt + 1))
    sleep 2
  done
  
  return 1
}

retry_with_screenshot '@e5'
```

### Pattern 3: Multi-Platform Test

```bash
#!/bin/bash
test_on_platform() {
  local platform=$1
  
  echo "Testing on $platform..."
  agent-device open MyApp --platform "$platform"
  
  agent-device snapshot -i
  agent-device tap @e1
  agent-device wait 2000
  agent-device screenshot "./artifacts/${platform}-result.png"
  
  agent-device close
}

test_on_platform ios
test_on_platform android
```

### Pattern 4: Selector-Based Interaction (More Reliable Than Refs)

```bash
# Refs (@e1, @e2) are session-specific and change after scrolling
# Use selectors for stable references

# By label/text
agent-device tap 'button[label="Continue"]'
agent-device fill 'text-field[label="Username"]' "myuser"

# By test ID (best practice)
agent-device tap '[testID="submit-button"]'
agent-device assert '[testID="success-message"]' --exists

# By role + text
agent-device tap 'button[text="Sign Up"]'

# Complex selector
agent-device tap 'button[label="Submit"][enabled="true"]'
```

## Troubleshooting

### Issue: "No devices found"

```bash
# Check available devices
xcrun simctl list devices # iOS
adb devices # Android

# Boot simulator if needed
xcrun simctl boot "iPhone 15 Pro"

# Check agent-device can see devices
agent-device apps --platform ios
```

### Issue: "Element not found"

```bash
# Always take fresh snapshot after screen changes
agent-device tap @e1
agent-device wait 2000
agent-device snapshot -i  # Get new refs

# Use selectors instead of refs for stability
agent-device tap 'button[label="Next"]'

# Increase wait time
agent-device wait @e5 --timeout 10000
```

### Issue: "Session timeout"

```bash
# Increase timeout globally
export AGENT_DEVICE_TIMEOUT=30000

# Or per command
agent-device wait @e1 --timeout 30000
```

### Issue: Accessibility data missing

```bash
# Ensure accessibility is enabled on iOS Simulator
# System Settings > Accessibility > VoiceOver

# For Android, ensure TalkBack is available (doesn't need to be on)

# Add accessibility labels in your app code:
# iOS: accessibilityLabel, accessibilityIdentifier
# Android: contentDescription, testTag
# React Native: accessibilityLabel, testID
```

### Issue: Commands not being recorded

```bash
# Verify recording is active
agent-device record start ./test.ad

# Check file is being written
ls -lh ./test.ad

# Ensure commands are session commands (not all flags are recorded)
# Recordable: open, tap, fill, wait, snapshot, screenshot
# Not recorded: record, replay, export, doctor
```

### Debugging Tips

```bash
# Use verbose mode
agent-device snapshot -i --verbose

# Capture full logs
agent-device logs ./debug.log

# Take frequent screenshots to understand state
agent-device screenshot ./debug-$(date +%s).png

# Use JSON output for parsing
agent-device snapshot --format json | jq '.elements[] | select(.role=="button")'
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Mobile QA
on: [pull_request]

jobs:
  test-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '22'
      
      - name: Install agent-device
        run: npm install -g agent-device@latest
      
      - name: Run iOS tests
        run: |
          agent-device doctor --platform ios
          ./scripts/test-ios.sh
      
      - name: Upload artifacts
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: test-artifacts
          path: ./artifacts/
```

### EAS Workflow (Expo)

```yaml
# .eas/workflows/agent-qa-mobile.yml
build:
  name: Agent QA
  steps:
    - run:
        name: Install agent-device
        command: npm install -g agent-device@latest
    
    - run:
        name: Run tests
        command: |
          agent-device open MyExpoApp --platform ios
          agent-device replay ./e2e/smoke-test.ad
```

## Additional Resources

- **Documentation**: https://oss.callstack.com/agent-device/
- **Agent-readable docs**: https://oss.callstack.com/agent-device/llms-full.txt
- **CLI help**: `agent-device help workflow`
- **Platform-specific help**: `agent-device help <command>`
- **GitHub**: https://github.com/callstack/agent-device

## Best Practices for AI Agents

1. **Always run `agent-device snapshot -i` after screen changes** to get fresh element refs
2. **Prefer selectors over refs** for stable automation: `'button[label="Submit"]'` vs `@e5`
3. **Use testID in React Native** for reliable element targeting
4. **Capture evidence before assertions** to help debug failures
5. **Use `--verbose` flag** when troubleshooting unclear behavior
6. **Start recordings at workflow beginning** to capture full context
7. **Check `agent-device doctor`** before starting automation on new environments
8. **Use `agent-device apps`** to verify app is installed and discoverable
9. **Set reasonable timeouts** (default 5s may be too short for slow screens)
10. **Close sessions explicitly** with `agent-device close` to free resources
