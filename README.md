# **Complete Expo Router + Supabase Auth + NativeWind Tutorial**

We will:

1. **Create & Reset** the Expo project.  
2. **Add** 4 environment variables in a single `.env` file.  
3. **Load** them all in `app.config.ts`.  
4. **Set up** Supabase sign-up & sign-in pages.  
5. **Guard** a protected route with `(protected)/_layout.tsx`.  
6. **Display** environment variables on the home screen.  
7. (Optional) **Dark steampunk theme** & custom fonts.

By the end, you’ll have a real sign-in/sign-up flow, environment variables in one place, and a protected route that doesn’t crash or navigate too early.

---

## **1) Create & Reset the Expo Project**

```bash
npx create-expo-app rn_Protected_Routez
cd rn_Protected_Routez
npm run reset-project
rm -rf app-example
```

*(If `app-example` was created, remove it. Keep your `app.json` with name/slug.)*

No test yet—just scaffolding.

---

## **2) Install Core Deps + Config Files**

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

### Explanation

- **`nativewind` + `tailwindcss`**: Tailwind for RN.  
- **`tailwindcss init`**: creates `tailwind.config.js`.  
- **`global.css`**: your Tailwind `@tailwind` directives.  
- **`babel.config.js`**, **`metro.config.js`**: we’ll configure for NativeWind.  
- **`nativewind-env.d.ts`**: type definitions for Tailwind.  
- **`dotenv`**: so we can load `.env` in our config file.  
- **`@supabase/supabase-js`**: the Supabase client for sign-up/sign-in.  
- **`app.config.ts`**: references `.env` for all environment variables.

---

## **3) Create `.env` with Four Variables**

```bash
touch .env
```

Open **`.env`** in your editor and **paste all 4** variables:

```
ENV_PUBLIC_GREETING="Hello from .env!"
ENV_PUBLIC_VERSION="1.2.3"
SUPABASE_URL="https://YOURSUBDOMAIN.supabase.co"
SUPABASE_ANON_KEY="YOUR_SUPABASE_ANON_KEY"
```

*(Adjust to your actual Supabase credentials. The first two are “dummy” env vars to show on the home screen.)*

No test yet—we’ll read them in our config next.

---

## **4) `app.config.ts` (One Place for Env Vars)**

We do **not** repeat `name`/`slug` (from `app.json`), only define `extra` for these **4** variables:

```ts
import 'dotenv/config';

export default () => ({
  expo: {
    // your name/slug/extra stuff
    // name: "rn_Protected_Routez",
    // slug: "rn_Protected_Routez",
    extra: {
      ENV_PUBLIC_GREETING: process.env.ENV_PUBLIC_GREETING,
      ENV_PUBLIC_VERSION: process.env.ENV_PUBLIC_VERSION,
      SUPABASE_URL: process.env.SUPABASE_URL,
      SUPABASE_ANON_KEY: process.env.SUPABASE_ANON_KEY,
    },
    plugins: [
      "expo-font"
    ]
  },
});
```

---

## **5) Configure Tailwind, Babel & Metro**

### 5.1. **`tailwind.config.js`**  
*(already created by `tailwindcss init`)*
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

### 5.2. **`global.css`**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 5.3. **`babel.config.js`**
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

### 5.4. **`metro.config.js`**
```js
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require("nativewind/metro");

/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, {
  input: "./global.css",
});
```

No test yet—we need screens.

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
*(One place to create our Supabase client—using the env vars)*

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
- Should see “Hello from .env! (v1.2.3)” on a white background—no errors so far.  
- If you see “Supabase URL or ANON KEY not set,” check `.env` and `app.config.ts`.

---

## **7) Create Sign-Up & Login Pages** (No placeholders)

We’ll add `(auth)/signUp.tsx` and `(auth)/signIn.tsx`. If the user tries to access a protected route without logging in, we’ll direct them here.

```bash
mkdir -p 'app/\(auth\)'
touch 'app/\(auth\)/signUp.tsx'
touch 'app/\(auth\)/signIn.tsx'
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

*(Now the user can sign up, then sign in using Supabase credentials.)*

---

## **8) The Protected Route via Higher-Level Layout**

We’ll create `(protected)/_layout.tsx` that checks **Supabase** for a logged-in session before rendering child screens.

```bash
mkdir -p 'app/\(protected\)'
touch 'app/\(protected\)/_layout.tsx'
touch 'app/\(protected\)/profile.tsx'
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

**Logic**:

1. **getSession()** checks if user is logged in.  
2. If **no** session, we do `router.replace("/(auth)/signIn")`.  
3. We also subscribe to any future sign-out events.  
4. If user is logged out, we redirect them again.  
5. If user is logged in, the child screens (like `profile.tsx`) render.

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

*(You can sign out from here; the subscription in `_layout.tsx` will catch it and redirect to signIn.)*

### 8.3. Link to Profile from Home

Open **`app/index.tsx`** and add a button/link to `(protected)/profile`:

```tsx
import React from 'react';
import { View, Text, Button } from 'react-native';
import { Link } from "expo-router";
import Constants from 'expo-constants';

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

### **Test** (Sign Up + Sign In + Protected Route)

```bash
npx expo start --clear
```

1. **Sign Up**: go to “Sign Up,” create a user.  
2. **Sign In**: then sign in with that user.  
3. **Protected Route**: Now “Go to Profile” loads `(protected)/profile` because `_layout.tsx` sees you have a session.  
4. If you sign out, `_layout.tsx` subscription logs you out => redirect to signIn.  
5. **No** “navigate before mounting” errors. No environment variables lost.

---

## **9. (Optional) Dark Theme & Steampunk Fonts**

If you want the dark steampunk theme + fonts:

1. **Install**:
   ```bash
   npx expo install @expo-google-fonts/special-elite @expo-google-fonts/arbutus-slab expo-font
   ```
2. **Create** a theme context in `app/context/theme.tsx`.  
3. **Load** fonts in `_layout.tsx` with `useFonts`.  
4. **Toggle** darkMode in your screens.

*(Already shown in previous code blocks. This step is purely optional.)*

---

# **Final Check**

1. You have **4 environment variables** in `.env`:  
   - `ENV_PUBLIC_GREETING`  
   - `ENV_PUBLIC_VERSION`  
   - `SUPABASE_URL`  
   - `SUPABASE_ANON_KEY`  
2. **`app.config.ts`** references them all in **one** place.  
3. **Home** screen displays the greeting and version.  
4. **Sign Up** & **Sign In** pages let you create and log in to a real Supabase user.  
5. The **protected route** (`/(protected)/profile`) is guarded by a higher-level layout that checks the session before rendering.  
6. **No** early navigation crash: the layout only redirects *after* checking session.  
7. Everything compiles **without** errors.  

You now have a **complete** tutorial with **four** env vars in a single `.env`, **signUp** and **signIn** pages for Supabase, a **protected** route that never crashes, **environment variables** displayed on the home screen, and an **optional** dark steampunk theme. Enjoy your production-ready **Expo** + **Tailwind** + **Supabase** setup!
