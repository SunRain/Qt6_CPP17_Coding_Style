# Gamepad UI Toolkit / Linux Input and Gamepad Library Comment Guidelines

## Core Principles

Comments should explain only the information that names, types, and local code cannot express reliably: input semantics, state-machine boundaries, platform differences, device lifecycles, mapping contracts, error recovery, and diagnostic contracts.

Do not use comments to restate literal code behavior. If the code is directly understandable, do not add a comment.

## Comment Form

Use Doxygen style for public types, public functions, and stable contracts:

```cpp
/// @brief Packs key and relevant modifiers into a stable chord code.
static quint64 makeChordCode(int key, Qt::KeyboardModifiers modifiers);
```

Use short comments for key logic inside `.cpp` files:

```cpp
// Bump epoch before replacing mappings so stale releases can be rejected.
setMappingEpoch(m_mappingEpoch + 1);
```

Keep each comment short. Split complex meaning into multiple short sentences. Comments explain intent; they do not explain basic C++, Qt, or Linux API syntax.

## Content That Must Be Commented

### 1. Input Event Semantics

Comment the following:

- press / repeat / release state machines;
- action event lifecycles;
- consumed semantics;
- synthetic event semantics;
- duplicate press/release suppression;
- recovery strategy when release is lost;
- held-state cleanup strategy when a device disconnects.

Example:

```cpp
// Suppress duplicate press/release pairs from repeated or paired chords.
return;
```

### 2. Gamepad Mapping and Action IDs

Comment the following:

- mapping rules from physical buttons to action IDs;
- mapping rules from axes/triggers to actions;
- whether an action ID is canonical;
- whether an action ID is part of a stable contract;
- default semantics of mapping presets;
- keyboard-to-gamepad mirror rules;
- player index / device ID assignment rules.

Example:

```cpp
/// @brief Builds a named built-in mapping normalized for players/devices.
static KeyboardActionMapping preset(const QString &presetName,
                                    int playerCount,
                                    int deviceIdBase);
```

### 3. Mapping Epochs and Hot Switching

Comment the following:

- the role of mapping epochs/generations;
- when mapping reloads are allowed;
- how held actions are handled before release;
- how stale releases are identified;
- synthetic event semantics for forced releases;
- whether mapping switches wait for a safe point.

Example:

```cpp
// Apply mapping changes only after the final release event is delivered.
maybeSchedulePendingKeyboardMappingReload();
```

### 4. Axes / Triggers / Deadzones

Comment the following when relevant:

- where the default deadzone value or policy comes from;
- the purpose of hysteresis;
- the business meaning of trigger thresholds;
- axis range normalization rules;
- diagonal policy;
- ramp / interpolation strategy;
- handling rules when opposite axis directions are pressed at the same time.

Example:

```cpp
// Ramps interpolate from the current value to avoid direction jumps.
state->startValue = state->value;
```

### 5. Priority Between Real Gamepads and Keyboard Emulation

Comment the following:

- which source wins when a real gamepad and keyboard emulation both exist;
- behavior of auto / keyboard-only / real-only / both modes;
- whether keyboard mirroring is paused during text entry;
- whether keyboard events pass through;
- conditions for exclusive capture;
- event source filtering scope.

Example:

```cpp
// Auto mode gives real devices priority and disables the mirror keyboard.
syncKeyboardActionSourceRunningState();
```

### 6. Linux Input Backends

When code involves evdev, libinput, udev, hidraw, SDL, Steam Input, or similar backends, comment the following:

- device discovery and filtering rules;
- hotplug handling strategy;
- device identity stability rules;
- permission failure and fallback paths;
- exclusive grab behavior;
- whether kernel repeat is ignored;
- axis / hat / trigger normalization;
- vendor-specific quirks;
- cleanup strategy when a gamepad disconnects.

Focus comments on platform behavior and boundaries. Do not explain the literal usage of Linux APIs.

### 7. Multiple Devices, Multiple Players, and Routing

Comment the following when relevant:

- device ID generation strategy;
- player index assignment rules;
- multi-gamepad sorting strategy;
- active device selection rules;
- route target selection rules;
- how overlays, pages, and focus affect input routing;
- whether modal overlays consume input.

Example:

```cpp
// Overlay routes stay modal; macro routing only applies to base content.
```

### 8. Gamepad UI Navigation and Focus Restore

Comment the following:

- focus restore strategy;
- differences between stable-key and index-based restore;
- focus lock;
- queued move direction;
- spatial fallback;
- special handling for view delegate recycling;
- manual focus edges;
- route boundaries / macro routers.

Example:

```cpp
// Preserve the stable key while delegates are being recycled by the view.
storeKey->insert(QStringLiteral("itemKey"), existingItemKey);
```

### 9. Input Diagnostics and Observability

Comment the following when relevant:

- meaning of input trace fields;
- drop reasons;
- deny reasons;
- macro router degrade reasons;
- stable fields in debug state;
- relationship between raw device events and normalized action events.

Example:

```cpp
/// Macro-router trace codes used by diagnostics and static contract gates.
enum class MacroRouterCode
```

### 10. Security and User Experience Boundaries

Comment the following when relevant:

- why certain input is ignored;
- why duplicate events are blocked;
- why certain input does not pass through;
- why keyboard mapping is disabled during text entry;
- why release is synthesized on disconnect;
- why fallback priority is fixed.

These comments prevent future maintainers from deleting protective logic that looks redundant.

## Content That Must Not Be Commented

Do not add comments that:

- restate literal code behavior;
- explain basic C++ / Qt / Linux API syntax;
- document simple getters/setters;
- repeat meaning already made clear by variable names;
- narrate every line of code;
- add TODOs without an owner, condition, or action;
- preserve historical chat context;
- contain design notes that are not kept in sync with the code;
- say things like "create object here", "emit changed signal here", or "loop over list".

Bad examples:

```cpp
// Set enabled.
m_enabled = enabled;

// Emit changed signal.
emit enabledChanged();
```

## Decision Standard

Before adding a comment, ask:

1. Does the caller need to know a hidden contract?
2. Could a maintainer accidentally delete key protective logic?
3. Does this involve input state, platform differences, device lifecycle, error recovery, or a stable diagnostic contract?

If any answer is "yes", add a short comment. If all answers are "no", do not add a comment.
