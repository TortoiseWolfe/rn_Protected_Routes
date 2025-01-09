# react native Protected Routes #

```bash
app/
├── _layout.tsx
├── index.tsx

assets/
├── fonts/
│   └── SpaceMono-Regular.ttf
└── images/
    ├── adaptive-icon.png
    ├── favicon.png
    ├── icon.png
    ├── partial-react-logo.png
    ├── react-logo.png
    ├── react-logo@2x.png
    ├── react-logo@3x.png
    └── splash-icon.png

node_modules/

app.json
package.json
tsconfig.json
```

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protecte_Routez
npm run reset-project
```

```bash
npx expo install nativewind tailwindcss
npx tailwindcss init
touch global.css
touch babel.config.js
npx expo customize metro.config.js
touch nativewind-env.d.ts
```

## tailwind.config.js ##

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  // NOTE: Update this to include the paths to all of your component files.
  content: ["./app/**/*.{js,jsx,ts,tsx}"],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

## global.css ##

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## babel.config.js ##

```javascript
module.exports = function (api) {
    api.cache(true);
    return {
      presets: [
        ["babel-preset-expo", { jsxImportSource: "nativewind" }],
        "nativewind/babel",
      ],
    };
  };
```

## metro.config.js ##

```javascript
// Learn more https://docs.expo.io/guides/customizing-metro
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require("nativewind/metro");

/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: "./global.css" });
```

## app/_layout.js ##

```javascript
import { Stack } from "expo-router";
// Import your global CSS file
import "../global.css";

export default function RootLayout() {
  return <Stack />;
}
```

## app/index.tsx ##

```tyypscript

```

## nativewind-env.d.ts ##

```tyypscript
/// <reference types="nativewind/types" />
```
