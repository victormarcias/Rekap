# React Native — Fundamentos

Ya se vio en [React (fundamentos)](../../frontend-react/react-fundamentos.md#react-native) por qué React Native comparte el mismo core y el mismo JSX que React web, solo con un renderer distinto (vistas nativas en vez de DOM). Acá el detalle práctico de cómo se arma una app con eso.

## Componentes core (vs HTML)

No hay `<div>`/`<span>`/`<img>` — son elementos nativos propios:

| HTML/Web | React Native | Nota |
|---|---|---|
| `<div>` | `<View>` | contenedor genérico, sin semántica |
| `<p>` / `<span>` | `<Text>` | **todo** texto tiene que estar envuelto acá |
| `<img>` | `<Image>` | |
| `<div style="overflow:auto">` | `<ScrollView>` | renderiza todos los hijos de una, sin virtualización |
| lista larga | `<FlatList>` | virtualizada — solo renderiza lo visible en pantalla |
| `<input>` | `<TextInput>` | |
| `<button onClick>` | `<Pressable>` / `<TouchableOpacity>` | |

```jsx
import { View, Text, StyleSheet } from 'react-native';

function Saludo({ nombre }) {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Hola, {nombre}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { padding: 16 },
  titulo: { fontSize: 20, fontWeight: 'bold' },
});
```

A diferencia de HTML (donde un string suelto dentro de un `<div>` se muestra igual), React Native **tira error** si hay texto plano fuera de un `<Text>` — no hay forma de renderizar texto sin ese wrapper.

## Styling: objetos JS, no CSS

No hay CSS ni `className` — los estilos son objetos JS, un subconjunto de Flexbox. Todo el layout es Flexbox por default, con una diferencia importante respecto a la web: `flexDirection: 'column'` es el default acá (en CSS web el default de un flex container es `'row'`).

```jsx
// ❌ funciona, pero recrea el objeto en cada render
<View style={{ padding: 16, flexDirection: 'row' }} />

// ✅ StyleSheet.create valida los estilos una sola vez y les da una referencia liviana
const styles = StyleSheet.create({ fila: { padding: 16, flexDirection: 'row' } });
<View style={styles.fila} />
```

## Expo vs React Native CLI (bare)

- **Expo**: capa sobre RN que resuelve el tooling — no hace falta tocar Xcode/Android Studio para desarrollar, trae un SDK propio de APIs nativas ya integradas (cámara, ubicación, notificaciones), y "Expo Go" para probar en el celular sin compilar nada local.
- **React Native CLI (bare)**: acceso directo a los proyectos nativos (`ios/` y `android/` son carpetas Xcode/Gradle reales) — control total, pero el tooling nativo (Pods, Gradle) lo manejás vos.

**Regla práctica**: arrancar con Expo salvo que se sepa de antemano que hace falta un módulo nativo puntual que Expo no cubre — hoy la mayoría de las apps arrancan con Expo, incluidas las que terminan necesitando código nativo propio vía su mecanismo de "prebuild".

## Navegación

No hay URL ni historial de browser — la navegación la maneja una librería aparte, típicamente **React Navigation**, con su propio stack de pantallas en memoria.

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Detalle" component={DetalleScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function HomeScreen({ navigation }) {
  // navigate() empuja una pantalla nueva al stack nativo, no cambia una URL
  return <Button title="Ver detalle" onPress={() => navigation.navigate('Detalle')} />;
}
```

## El bridge — cómo el JS habla con código nativo

El JS de la app corre en su propio motor (JavaScriptCore o Hermes), separado del hilo principal de UI, y se comunica con el lado nativo (Swift/Kotlin) a través de un **bridge** que serializa mensajes entre los dos mundos de forma asincrónica. Llamar a una API nativa (cámara, GPS) implica: el JS manda un mensaje al lado nativo, el código nativo real lo ejecuta, y el resultado vuelve por el mismo camino — todo async.

**La Nueva Arquitectura (Fabric + TurboModules)** reemplaza ese bridge por comunicación **síncrona** vía JSI (JavaScript Interface) — reduce la latencia de esos cruces JS↔nativo, relevante porque el bridge async viejo era un cuello de botella real en animaciones/gestos que necesitan responder dentro del mismo frame.

## Código específico por plataforma

Cuando iOS y Android necesitan comportarse distinto, dos formas de resolverlo:

```jsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  header: {
    paddingTop: Platform.OS === 'ios' ? 44 : 24, // altura de la status bar difiere por plataforma
  },
});
```

O separar en archivos por extensión — React Native elige el correcto automáticamente según la plataforma que se está compilando, sin ningún `if` explícito:
```
Boton.ios.tsx
Boton.android.tsx
```

---
Relacionado: [React (fundamentos)](../../frontend-react/react-fundamentos.md#react-native), [Hooks](../../frontend-react/hooks.md).
