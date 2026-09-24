# React Native — Fundamentals

We already saw in [React (fundamentals)](../../frontend-react/react-fundamentals.md#react-native) why React Native shares the same core and the same JSX as web React, just with a different renderer (native views instead of DOM). Here's the practical detail of how an app gets built with that.

## Core components (vs HTML)

There's no `<div>`/`<span>`/`<img>` — they're their own native elements:

| HTML/Web | React Native | Note |
|---|---|---|
| `<div>` | `<View>` | generic container, no semantics |
| `<p>` / `<span>` | `<Text>` | **all** text has to be wrapped here |
| `<img>` | `<Image>` | |
| `<div style="overflow:auto">` | `<ScrollView>` | renders all children at once, no virtualization |
| long list | `<FlatList>` | virtualized — only renders what's visible on screen |
| `<input>` | `<TextInput>` | |
| `<button onClick>` | `<Pressable>` / `<TouchableOpacity>` | |

```jsx
import { View, Text, StyleSheet } from 'react-native';

function Greeting({ name }) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello, {name}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 16 },
  title: { fontSize: 20, fontWeight: 'bold' },
});
```

Unlike HTML (where a loose string inside a `<div>` displays fine), React Native **throws an error** if there's plain text outside a `<Text>` — there's no way to render text without that wrapper.

## Styling: JS objects, not CSS

There's no CSS or `className` — styles are JS objects, a subset of Flexbox. All layout is Flexbox by default, with one important difference from the web: `flexDirection: 'column'` is the default here (in web CSS, a flex container's default is `'row'`).

```jsx
// ❌ works, but recreates the object on every render
<View style={{ padding: 16, flexDirection: 'row' }} />

// ✅ StyleSheet.create validates the styles once and gives them a lightweight reference
const styles = StyleSheet.create({ row: { padding: 16, flexDirection: 'row' } });
<View style={styles.row} />
```

## Expo vs React Native CLI (bare)

- **Expo**: a layer on top of RN that handles the tooling — no need to touch Xcode/Android Studio to develop, comes with its own SDK of already-integrated native APIs (camera, location, notifications), and "Expo Go" to test on your phone with nothing compiled locally.
- **React Native CLI (bare)**: direct access to the native projects (`ios/` and `android/` are real Xcode/Gradle folders) — full control, but you manage the native tooling (Pods, Gradle) yourself.

**Practical rule**: start with Expo unless you already know you need a specific native module Expo doesn't cover — today most apps start with Expo, including ones that end up needing their own native code via its "prebuild" mechanism.

## Navigation

There's no URL or browser history — navigation is handled by a separate library, typically **React Navigation**, with its own in-memory screen stack.

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Detail" component={DetailScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function HomeScreen({ navigation }) {
  // navigate() pushes a new screen onto the native stack, doesn't change a URL
  return <Button title="View detail" onPress={() => navigation.navigate('Detail')} />;
}
```

## The bridge — how JS talks to native code

The app's JS runs on its own engine (JavaScriptCore or Hermes), separate from the main UI thread, and communicates with the native side (Swift/Kotlin) through a **bridge** that serializes messages between the two worlds asynchronously. Calling a native API (camera, GPS) means: the JS sends a message to the native side, the real native code executes it, and the result travels back the same way — all async.

**The New Architecture (Fabric + TurboModules)** replaces that bridge with **synchronous** communication via JSI (JavaScript Interface) — reduces the latency of those JS↔native crossings, relevant because the old async bridge was a real bottleneck in animations/gestures that need to respond within the same frame.

## Platform-specific code

When iOS and Android need to behave differently, two ways to solve it:

```jsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  header: {
    paddingTop: Platform.OS === 'ios' ? 44 : 24, // status bar height differs by platform
  },
});
```

Or split into files by extension — React Native automatically picks the right one based on which platform is being compiled, with no explicit `if`:
```
Button.ios.tsx
Button.android.tsx
```

---
Related: [React (fundamentals)](../../frontend-react/react-fundamentals.md#react-native), [Hooks](../../frontend-react/hooks.md).
