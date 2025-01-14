In a typical Expo setup, **if you have both `app.json` and an `app.config.js/ts` in your project**, the `app.config.*` file will take precedence. In other words, **Expo will read from `app.config.*` first**, merging any fields it finds there into the final config. Anything in `app.json` that is _not_ overwritten by `app.config.ts` remains in effect.

### **Why are `name` and `slug` repeated?**
- **Option A:** You plan to remove or ignore `app.json` and rely solely on `app.config.*`.  
- **Option B:** You keep `app.json` for some defaults but override (or reinforce) certain fields in `app.config.*`.  
- **Option C:** You unify everything by copying what’s in `app.json` into `app.config.ts` and then delete `app.json`.  

Ultimately, **Expo** will generate its final config from **both** files, with `app.config.ts` overriding any duplicates. That means **if you include `name` and `slug` in `app.config.ts`, the values you specify there will override** those in `app.json`. If they’re the same values (like `"rn_Protected_Routez"`), it’s purely redundant. 

---

## **So which approach should you use?**
1. **Keep using `app.json`**:  
   - Don’t bother with `name` and `slug` in `app.config.ts`. You only need to add fields _not_ in `app.json` (for example, the `extra` object for environment variables).  

2. **Move to `app.config.js/ts` fully**:  
   - Copy everything from `app.json` into `app.config.ts`, remove (or ignore) `app.json`.  
   - Your new `app.config.ts` is the single source of truth.

3. **Hybrid**:  
   - Keep `app.json` for older references or partial config.  
   - Add `app.config.ts` only for environment variable injection or special plugin logic not easily expressed in JSON.  
   - If you specify `name` and `slug` in both, the `app.config.ts` version overrides.

---

## **Best Practice for Env Variables**
If your main goal is to inject environment variables at build time for **web** _and_ native:

- **Keep** your existing `app.json` for all the standard fields (name, slug, icon, etc.).  
- **Add** `app.config.ts` only for the `extra: {...}` object where you read your `.env` keys (and do any runtime Node logic you need, like `import 'dotenv/config'`).  

In **`app.config.ts`**:

```ts
import 'dotenv/config';

export default () => ({
  // "expo": {} is optional. If you leave it out, it merges with what's in app.json
  // but you can override or add new fields:
  expo: {
    extra: {
      ENV_PUBLIC_GREETING: process.env.ENV_PUBLIC_GREETING,
      // ...
    },
  },
});
```

Since **`app.json`** already has `name`, `slug`, etc., you can **omit** them from `app.config.ts`. Expo merges them behind the scenes. That way, you’re not duplicating or clobbering what you already defined. If you add them to `app.config.ts`, you’re effectively overriding the values in `app.json`.

---

## **Bottom Line**
There’s **no requirement** to repeat `name` and `slug` if you prefer to keep them in `app.json`. You only do so if you plan on migrating entirely to `app.config.ts` (or if you specifically want to override them).  

In short:
1. **Leaving `name`/`slug` only in `app.json`** is fine.  
2. **Adding them to `app.config.ts`** duplicates or overrides them.  

It all depends on whether you want to keep `app.json` in the long run or not.


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
cd rn_Protected_Routez
npm run reset-project
```

```bash
npx expo install nativewind tailwindcss
npx tailwindcss init
touch global.css
touch babel.config.js
npx expo customize metro.config.js
touch nativewind-env.d.ts
touch .env .env.example
npm install @supabase/supabase-js
npm install react-native-dotenv

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
     ]
    // plugins: [
    //   ["module:react-native-dotenv"], // Plugin for accessing .env variables
    // ],
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

## .env  & .env.example ##

```env
```
