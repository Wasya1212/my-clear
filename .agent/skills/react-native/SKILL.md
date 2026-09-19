---
name: React Native + React Native Web Shared-Codebase Skill
description: Use this skill when building, modifying, or reviewing a project that must run from the same React Native codebase on both native platforms and the web.
---

## Instructions

The target architecture is:

                    Shared application code
                           │
              ┌────────────┴────────────┐
              │                         │
        React Native               React Native Web
              │                         │
        iOS / Android                  Web

The default goal is to maximize shared code and avoid maintaining separate web and native applications.

Core principles

React Native is the primary UI abstraction.
Prefer React Native primitives such as:

View

Text

Image

Pressable

ScrollView

TextInput

FlatList

ActivityIndicator

Modal

SafeAreaView or the project's chosen safe-area solution

React Native Web is the web renderer.
Do not create a separate web UI implementation unless a real platform difference requires it.

Share application code by default.
Components, screens, hooks, state, business logic, validation, API clients, models, and most styling should live in shared files.

Platform-specific code is an escape hatch, not the default architecture.
Use .native.*, .ios.*, .android.*, and .web.* files only when the platforms genuinely require different behavior.

Do not use DOM APIs directly in shared code.
Avoid window, document, HTML elements, and browser-only APIs in shared components unless they are isolated behind a web-specific adapter.

Do not use native-only APIs directly in shared web-compatible components.
Wrap platform-specific capabilities behind a small abstraction.

Keep UI behavior consistent across platforms.
A screen should have the same information architecture and interaction model on web, iOS, and Android unless platform conventions make a difference necessary.

Recommended project structure

Prefer a structure similar to:

src/
  app/
    App.tsx

  components/
    Button.tsx
    Card.tsx
    Screen.tsx
    Input.tsx

  screens/
    HomeScreen.tsx
    SettingsScreen.tsx

  hooks/
    useAuth.ts
    useProjects.ts

  state/
    ...

  services/
    api.ts
    storage.ts

  utils/
    ...

  theme/
    colors.ts
    spacing.ts
    typography.ts

  platform/
    storage.ts
    storage.web.ts
    storage.native.ts

Keep platform-specific implementations close to the abstraction they implement.

Avoid structures such as:

web/
  ...
native/
  ...

for the entire application. That usually causes the codebases to drift apart.

Entry points

Use platform-specific entry points only where necessary.

A common pattern is:

index.js
index.web.js

or the equivalent supported by the project's bundler.

The application itself should normally render the same root component:

<App />

Do not duplicate the complete application tree between web and native.

Component rules

Prefer React Native components

Good:

import { Pressable, Text, View } from 'react-native';

export function ActionButton() {
  return (
    <Pressable onPress={handlePress}>
      <Text>Continue</Text>
    </Pressable>
  );
}

Avoid:

<button onClick={handlePress}>Continue</button>

in shared application code.

Avoid:

<div>
  ...
</div>

in shared components.

React Native Web will translate React Native primitives to appropriate web elements.

Styling

Prefer React Native styles:

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
});

Use:

<View style={styles.container} />

Avoid CSS as the primary styling system for shared components.

Do not introduce CSS classes for ordinary application layout unless there is a specific web-only requirement.

Cross-platform styling

Remember that not every CSS feature maps perfectly to React Native.

Prefer portable properties:

flex

flexDirection

alignItems

justifyContent

padding

margin

gap when supported by the project's target versions

width

height

minWidth

maxWidth

borderRadius

backgroundColor

color

fontSize

fontWeight

lineHeight

When a visual requirement cannot be represented well using React Native styles, isolate the exception rather than converting the whole component to web-specific markup.

Platform-specific files

Use React Native's platform resolution when platform behavior genuinely differs.

Example:

storage.ts
storage.web.ts
storage.native.ts

Shared code:

import { storage } from './storage';

The bundler resolves the appropriate implementation.

For iOS/Android differences:

permissions.ts
permissions.ios.ts
permissions.android.ts

For web:

share.ts
share.web.ts
share.native.ts

Keep the public interface identical where possible.

Example:

export interface Storage {
  getItem(key: string): Promise<string | null>;
  setItem(key: string, value: string): Promise<void>;
  removeItem(key: string): Promise<void>;
}

Only the implementation should vary.

Browser APIs

Do not write this in shared code:

window.localStorage.setItem(...)

Instead create an abstraction:

storage.setItem(...)

Then provide:

storage.web.ts
storage.native.ts

The web implementation can use browser storage.

The native implementation can use the project's native storage solution.

This keeps the rest of the application platform-neutral.

Native APIs

If a feature needs a native capability such as:

camera

filesystem

push notifications

biometrics

native permissions

sensors

deep linking

native share

haptics

do not leak the native library throughout the application.

Create a small service/hook abstraction.

Example:

export interface DeviceAuth {
  authenticate(): Promise<boolean>;
}

Then provide platform implementations as needed.

The shared UI should consume:

const authenticated = await deviceAuth.authenticate();

rather than importing platform-specific implementation details everywhere.

Navigation

Use one navigation model for the application whenever possible.

Screens should be shared:

src/screens/
  HomeScreen.tsx
  ProjectScreen.tsx
  SettingsScreen.tsx

Do not create:

WebHomeScreen.tsx
NativeHomeScreen.tsx

unless the actual UX must differ.

Web-specific URL/deep-link behavior should be configured at the navigation layer rather than duplicating screens.

Responsive design

The same components must work on:

desktop web

mobile web

iOS

Android

Do not assume a fixed phone viewport.

Use:

useWindowDimensions()

when layout needs to react to viewport size.

Prefer responsive layouts over platform checks.

Bad:

if (Platform.OS === 'web') {
  // desktop layout
}

when the real requirement is simply responsive sizing.

Better:

const { width } = useWindowDimensions();

const isWide = width >= 768;

Use platform checks only when behavior, not screen size, is actually platform-specific.

Web-specific UI

Some web features genuinely need DOM behavior.

Examples:

keyboard shortcuts

browser clipboard APIs

drag and drop

file input

hover-specific behavior

browser history integration

browser-only accessibility behavior

Keep those implementations isolated.

Example:

KeyboardShortcuts.ts
KeyboardShortcuts.web.ts
KeyboardShortcuts.native.ts

The shared application should consume the abstraction.

Do not make every component aware of web-specific APIs.

Native-specific UI

Some native behaviors may not have meaningful web equivalents.

Examples:

native navigation transitions

haptics

native context menus

platform permission prompts

native share sheets

Prefer graceful web fallbacks.

Example:

shareProject(project)

rather than:

if (Platform.OS === 'ios') {
  ...
} else if (Platform.OS === 'android') {
  ...
} else {
  navigator.share(...)
}

scattered across the UI.

Images and assets

Use React Native's Image for shared images.

Avoid direct <img> in shared components.

For assets that require platform-specific handling, isolate the implementation.

Keep asset references compatible with both bundlers.

Fonts

Use a shared typography definition.

Native and web may require different font loading mechanisms, but application components should consume the same semantic typography tokens.

Example:

export const typography = {
  body: {
    fontSize: 16,
    lineHeight: 24,
  },
  title: {
    fontSize: 24,
    fontWeight: '700',
  },
};

Platform-specific font loading/configuration should live outside normal components.

Testing

Test shared components primarily as shared behavior.

Prioritize:

TypeScript correctness

shared component tests

business logic tests

web smoke test

native build/test

platform-specific behavior tests where required

A change to a shared component should be assumed to affect both web and native.

Dependencies

Before adding a package, check that it supports:

React Native

React Native Web

the project's React Native version

the project's React version

iOS/Android requirements

the project's web bundler

Do not introduce a web-only package into shared application code unless it is isolated behind a platform adapter.

Likewise, do not introduce a native-only dependency into code imported by the web bundle unless the bundler/project configuration explicitly supports that dependency being excluded or substituted on web.

Prefer mature cross-platform React Native libraries.

Package scripts

Maintain explicit scripts for both targets.

Example conceptually:

{
  "scripts": {
    "web": "...",
    "ios": "...",
    "android": "...",
    "test": "..."
  }
}

Do not assume that a successful web build means the native app will build.

Native dependencies, CocoaPods, Xcode configuration, and native modules must still be validated separately.

AI coding rules

When an AI agent is asked to implement a feature:

Default workflow

Inspect the existing project before changing dependencies.

Reuse existing components, theme, navigation, and services.

Implement the feature using React Native primitives.

Keep the implementation shared.

Run/typecheck/test the web target.

Run the native target when available.

Only introduce platform-specific files when required.

Explain why a platform-specific implementation is necessary.

Do not

create separate web and native screens by default;

replace React Native components with HTML;

add browser APIs to shared components;

add native APIs directly to shared components;

rewrite working architecture merely to add a feature;

install packages without checking cross-platform compatibility;

assume React Native Web means "React DOM with React Native-looking components."

Platform detection

Use Platform.OS only when runtime behavior genuinely differs.

Example:

import { Platform } from 'react-native';

if (Platform.OS === 'ios') {
  ...
}

Prefer platform-specific modules when an implementation becomes substantial.

Bad:

if (Platform.OS === 'web') {
  // 200 lines of browser implementation
} else {
  // 200 lines of native implementation
}

Better:

feature.ts
feature.web.ts
feature.native.ts

with a shared API.

When separate UI is justified

Separate UI implementations are acceptable when there is a real platform-specific UX requirement.

Examples:

desktop web requires a persistent sidebar while mobile uses a tab/navigation pattern;

iOS requires a native interaction unavailable on web;

browser drag-and-drop is fundamentally different from touch interaction;

a native module exposes a capability that has no web equivalent.

Even then, keep the business logic, state, data fetching, validation, and domain model shared.

Definition of done

A feature is considered complete when:

the implementation uses the shared React Native codebase by default;

web works through React Native Web;

native works through React Native;

no unnecessary platform-specific duplication was introduced;

platform-specific code is isolated behind a clear boundary;

TypeScript passes;

existing tests pass;

the relevant web/native build has been checked;

the feature does not depend on browser-only APIs in shared code;

the feature does not depend on native-only APIs in shared web code.

Guiding rule

When deciding how to implement something, ask:

"Can this be implemented once using React Native primitives and work on both React Native and React Native Web?"

If yes, implement it once.

Only diverge when the platform actually requires different behavior.
