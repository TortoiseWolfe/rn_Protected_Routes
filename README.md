# **Expo Router + Supabase Auth + NativeWind Tutorial**

## **1) Create & Reset the Expo Project**

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```
- Remove `app-example` if it appears.  
- Keep your **`app.json`** for `name` and `slug`.

*(No test yet—just scaffolding.)*

---

## **2) Install Core Deps + Set Up Config Files**

```bash
npx expo install nativewind tailwindcss
npx tailwindcss init
touch global.css
touch babel.config.js
npx expo customize metro.config.js
touch nativewind-env.d.ts

npm install dotenv
npm install @supabase/supabase-js
touch app.config.ts
```

### **What Each Command Does**

- **`npx expo install nativewind tailwindcss`**: Adds NativeWind + Tailwind.  
- **`npx tailwindcss init`**: Creates `tailwind.config.js`.  
- **`touch global.css`**, **`babel.config.js`**, **`metro.config.js`**: will be configured for NativeWind.  
- **`touch nativewind-env.d.ts`**: TypeScript definitions for NativeWind classes.  
- **`dotenv`**: so we can load `.env` in `app.config.ts`.  
- **`@supabase/supabase-js`**: Supabase client for authentication.  
- **`app.config.ts`**: our dynamic Expo config (we’ll also add `"expo-font"` here as a plugin).

*(Still no test—Babel & Metro not wired, no screens yet.)*

---

## **3) Create `.env` with Four Variables**

```bash
touch .env
```

Open **`.env`** in your editor. **Paste all 4** variables (example below, adjust to your real keys):

```
ENV_PUBLIC_GREETING="Hello from .env!"
ENV_PUBLIC_VERSION="1.2.3"
SUPABASE_URL="https://YOURSUBDOMAIN.supabase.co"
SUPABASE_ANON_KEY="YOUR_SUPABASE_ANON_KEY"
```

*(No test yet—will read them in `app.config.ts`.)*

---

## **4) `app.config.ts`** (One Place for Env Vars **and** `"expo-font"`)

Since you got the “Cannot automatically write to dynamic config … Please add the following to your Expo config: plugins: [ 'expo-font' ]” message, we’ll **manually** add that to the plugins array. We **do not** repeat `name`/`slug` if they’re in `app.json`.

```ts
import 'dotenv/config';

export default () => ({
  expo: {
    // If desired, you can unify name/slug here, or rely on app.json
    // name: "rn_Protected_Routez",
    // slug: "rn_Protected_Routez",
    extra: {
      ENV_PUBLIC_GREETING: process.env.ENV_PUBLIC_GREETING,
      ENV_PUBLIC_VERSION: process.env.ENV_PUBLIC_VERSION,
      SUPABASE_URL: process.env.SUPABASE_URL,
      SUPABASE_ANON_KEY: process.env.SUPABASE_ANON_KEY,
    },
    plugins: [
      "expo-font" // Tells Expo to configure the expo-font plugin
    ],
    // (Optional) userInterfaceStyle to remove the color scheme warning:
    userInterfaceStyle: "automatic",
  },
});
```

*(Now Expo knows to apply the `expo-font` plugin from your dynamic config.)*

---

## **5) Configure Tailwind, Babel & Metro**

### **5.1. `tailwind.config.js`**  
*(from `npx tailwindcss init`)*
```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./app/**/*.{js,jsx,ts,tsx}"],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### **5.2. `global.css`**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### **5.3. `babel.config.js`**
```js
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

### **5.4. `metro.config.js`**
```js
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, {
  input: "./global.css",
});
```

*(No test yet—still need the app folder + screens.)*

---

## **6) Base App + Supabase Client**

```bash
touch nativewind-env.d.ts
mkdir -p lib
touch lib/supabaseClient.ts
```

### 6.1. **`nativewind-env.d.ts`**
```ts
/// <reference types="nativewind/types" />
```

### 6.2. **`app/_layout.tsx`** (Root Layout)
```tsx
import { Stack } from "expo-router";
import "../global.css"; // Tailwind

export default function RootLayout() {
  return <Stack />;
}
```

### 6.3. **`app/index.tsx`** (Home Screen)
```tsx
import React from 'react';
import { View, Text } from 'react-native';
import Constants from 'expo-constants';

export default function HomeScreen() {
  const greeting = Constants.expoConfig?.extra?.ENV_PUBLIC_GREETING || "No greeting";
  const version = Constants.expoConfig?.extra?.ENV_PUBLIC_VERSION || "0.0.0";

  return (
    <View className="flex-1 items-center justify-center bg-white p-4">
      <Text className="text-xl font-bold text-blue-500 mb-2">
        Hello from NativeWind + Expo Router!
      </Text>
      <Text className="text-base text-gray-700">
        {greeting} (v{version})
      </Text>
    </View>
  );
}
```

### 6.4. **`lib/supabaseClient.ts`**
```ts
import { createClient } from "@supabase/supabase-js";
import Constants from "expo-constants";

const supabaseUrl = Constants.expoConfig?.extra?.SUPABASE_URL;
const supabaseAnonKey = Constants.expoConfig?.extra?.SUPABASE_ANON_KEY;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error("Supabase URL or ANON KEY not set in env variables.");
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### **Test** (Baseline)

```bash
npx expo start --clear
```
- Should see **“Hello from .env! (v1.2.3)”** in blue.  
- No errors => success so far.

---

## **7) Create Sign-Up & Login Pages (No placeholders)**

1. **`(auth)/signUp.tsx`**: sign up with Supabase.  
2. **`(auth)/signIn.tsx`**: sign in with Supabase.

```bash
mkdir -p "app/(auth)"
touch "app/(auth)/signUp.tsx"
touch "app/(auth)/signIn.tsx"
```

### 7.1. **`app/(auth)/signUp.tsx`**
```tsx
import React, { useState } from 'react';
import { View, Text, TextInput, Button } from 'react-native';
import { supabase } from '../../lib/supabaseClient';
import { useRouter } from 'expo-router';

export default function SignUpScreen() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const router = useRouter();

  async function handleSignUp() {
    const { error } = await supabase.auth.signUp({
      email,
      password,
    });
    if (error) {
      alert("Error signing up: " + error.message);
    } else {
      alert("Sign-up successful! Please sign in.");
      router.replace("/(auth)/signIn");
    }
  }

  return (
    <View className="flex-1 items-center justify-center bg-white p-4">
      <Text className="text-2xl font-bold mb-4">Sign Up</Text>

      <TextInput
        style={{ borderWidth: 1, borderColor: "#ccc", width: "80%", marginBottom: 12, padding: 8 }}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
      />
      <TextInput
        style={{ borderWidth: 1, borderColor: "#ccc", width: "80%", marginBottom: 12, padding: 8 }}
        placeholder="Password"
        secureTextEntry
        value={password}
        onChangeText={setPassword}
      />
      <Button title="Sign Up" onPress={handleSignUp} />
    </View>
  );
}
```

### 7.2. **`app/(auth)/signIn.tsx`**
```tsx
import React, { useState } from 'react';
import { View, Text, TextInput, Button } from 'react-native';
import { supabase } from '../../lib/supabaseClient';
import { useRouter } from 'expo-router';

export default function SignInScreen() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const router = useRouter();

  async function handleSignIn() {
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    if (error) {
      alert("Error signing in: " + error.message);
    } else {
      // If sign-in succeeded, go home or wherever
      router.replace("/");
    }
  }

  return (
    <View className="flex-1 items-center justify-center bg-white p-4">
      <Text className="text-2xl font-bold mb-4">Sign In</Text>

      <TextInput
        style={{ borderWidth: 1, borderColor: "#ccc", width: "80%", marginBottom: 12, padding: 8 }}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
      />
      <TextInput
        style={{ borderWidth: 1, borderColor: "#ccc", width: "80%", marginBottom: 12, padding: 8 }}
        placeholder="Password"
        secureTextEntry
        value={password}
        onChangeText={setPassword}
      />

      <Button title="Sign In" onPress={handleSignIn} />
    </View>
  );
}
```

---

## **8) Protected Route via Higher-Level Layout**

We’ll add `(protected)/_layout.tsx` that checks Supabase for a session. If not logged in, redirect to `(auth)/signIn`.

```bash
mkdir -p "app/(protected)"
touch "app/(protected)/_layout.tsx"
touch "app/(protected)/profile.tsx"
```

### 8.1. **`app/(protected)/_layout.tsx`**
```tsx
import React, { useEffect, useState } from 'react';
import { Stack, useRouter } from 'expo-router';
import { supabase } from '../../lib/supabaseClient';

export default function ProtectedLayout() {
  const router = useRouter();
  const [checked, setChecked] = useState(false);
  const [session, setSession] = useState<any>(null);

  useEffect(() => {
    // On mount, check current session
    supabase.auth.getSession().then(({ data }) => {
      setSession(data.session);
      setChecked(true);
      if (!data.session) {
        router.replace("/(auth)/signIn");
      }
    });

    // Also subscribe to any auth changes
    const { data: sub } = supabase.auth.onAuthStateChange((_event, session) => {
      if (!session) {
        router.replace("/(auth)/signIn");
      } else {
        setSession(session);
      }
    });

    return () => {
      sub?.subscription.unsubscribe();
    };
  }, [router]);

  if (!checked) {
    // We haven't finished checking session => show nothing or a loader
    return null;
  }

  if (!session) {
    // We already redirected, but if not, show nothing
    return null;
  }

  return <Stack />;
}
```

### 8.2. **`app/(protected)/profile.tsx`**
```tsx
import React from 'react';
import { View, Text, Button } from 'react-native';
import { supabase } from '../../lib/supabaseClient';

export default function ProfileScreen() {
  async function handleSignOut() {
    await supabase.auth.signOut();
  }

  return (
    <View className="flex-1 items-center justify-center bg-white p-4">
      <Text className="text-xl font-bold text-green-600 mb-4">
        Welcome to the Protected Profile!
      </Text>
      <Button title="Sign Out" onPress={handleSignOut} />
    </View>
  );
}
```

### 8.3. Link to Profile from Home

Open **`app/index.tsx`** again and add link buttons:

```tsx
import React from 'react';
import { View, Text, Button } from 'react-native';
import { Link } from "expo-router";
import Constants from "expo-constants";

export default function HomeScreen() {
  const greeting = Constants.expoConfig?.extra?.ENV_PUBLIC_GREETING || "No greeting";
  const version = Constants.expoConfig?.extra?.ENV_PUBLIC_VERSION || "0.0.0";

  return (
    <View className="flex-1 items-center justify-center bg-white p-4">
      <Text className="text-xl font-bold text-blue-500 mb-2">
        Hello from NativeWind + Expo Router!
      </Text>
      <Text className="text-base text-gray-700 mb-4">
        {greeting} (v{version})
      </Text>

      <Link href="/(protected)/profile">
        <Button title="Go to Profile" onPress={() => {}} />
      </Link>

      <Link href="/(auth)/signUp">
        <Button title="Sign Up" onPress={() => {}} />
      </Link>
      <Link href="/(auth)/signIn">
        <Button title="Sign In" onPress={() => {}} />
      </Link>
    </View>
  );
}
```

### **Test** (Sign Up, Sign In, Protected Route)

```bash
npx expo start --clear
```
1. **Sign Up** a new user.  
2. **Sign In** with that user.  
3. **Tap “Go to Profile”** => loads protected screen if session is valid.  
4. Sign out => subscription triggers => back to Sign In.  
5. **No** “navigate before mounting” error.

---

## **9) (Optional) Dark Theme & Steampunk Fonts**

If you want the dark theme & custom fonts:

1. **Install** fonts:
   ```bash
   npx expo install @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab expo-font
   ```
2. **Add** them in `_layout.tsx` with `useFonts`.  
3. **Create** a theme context to toggle darkMode.

*(This is optional, see previous code blocks if you need details.)*

---

# **Final Verification**

- **4 environment variables**: `ENV_PUBLIC_GREETING`, `ENV_PUBLIC_VERSION`, `SUPABASE_URL`, `SUPABASE_ANON_KEY` in `.env`.  
- **`app.config.ts`** references them all (plus `"plugins": ["expo-font"]`)—so the “cannot automatically write to dynamic config” error is handled.  
- **Home screen** displays the greeting and version.  
- **SignUp** + **SignIn** pages let you create/log in with Supabase.  
- **Protected** route in `/(protected)/profile`, guarded by higher-level `_layout.tsx`.  
- **No** partial placeholders or missing env vars.  
- **Zero** early navigation crashes.  

Now you have a **production-ready** Expo Router app with:

- **Supabase** auth (sign-up, sign-in, sign-out).  
- **NativeWind** (Tailwind) + TypeScript.  
- **All** environment variables in **one** `.env`, read in `app.config.ts`, with `"expo-font"` added to `plugins`.  
- Optionally, a **dark steampunk** theme with custom fonts if you wish.

Enjoy coding!
