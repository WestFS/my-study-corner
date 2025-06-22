#tag: EN/React Native
# Native Components in React Native

## Table of Contents
1. [What are Native Components?](#what-are-native-components)
2. [Component Mapping Table](#component-mapping-table)
3. [The View Component](#the-view-component)
4. [Best Practices & Tips](#best-practices--tips)
5. [References & Further Reading](#references--further-reading)

---

## What are Native Components?

A **component** is a reusable, independent part of an application's interface. In **React Native**, your app is built from many components, each representing a specific part of the UI—such as buttons, text, forms, images, etc.

This approach improves **organization**, **maintenance**, and **scalability**, allowing each part of the app to be developed in isolation but work together seamlessly.

---

## Component Mapping Table

The table below shows how React Native's core components map to their native counterparts on Android, iOS, and the web:

| **React Native** | **Android (Java/Kotlin)** | **iOS (Swift)** | **HTML**              |
|------------------|--------------------------|-----------------|-----------------------|
| `<View>`         | `ViewGroup`              | `UIView`        | `<div>`               |
| `<Text>`         | `TextView`               | `UITextView`    | `<p>`                 |
| `<Image>`        | `ImageView`              | `UIImageView`   | `<img>`               |
| `<TextInput>`    | `EditText`               | `UITextField`   | `<input type="text">`|
| `<ScrollView>`   | `ScrollView`             | `UIScrollView`  | `<div>`               |

---

## The <View> Component

The `<View>` is the primary building block for UI in React Native. It acts as a container to group and organize other components (text, images, buttons, etc.).

- Integrates with **flexible layouts** (Flexbox)
- Supports **custom styles**
- Handles **touch gestures**
- Can be **nested** to create complex structures

**Example:**
```tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

export default function Example() {
  return (
    <View style={styles.container}>
      <Text>Hello from inside a View!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
  },
});
```

---

## Best Practices & Tips
- Use `<View>` for layout and grouping, not for semantic meaning (use `<Text>` for text)
- Prefer Flexbox for responsive layouts
- Keep components small and focused
- Use TypeScript for type safety
- Test on both Android and iOS for consistent appearance

---

## References & Further Reading
- [React Native Components](https://reactnative.dev/docs/components-and-apis)
- [Flexbox in React Native](https://reactnative.dev/docs/flexbox)
- [RFC 8259: JSON](https://tools.ietf.org/html/rfc8259)
- [RFC 3339: Date/Time](https://tools.ietf.org/html/rfc3339)

---

## Expo Tips
- You can use all native components out of the box with Expo—no extra setup needed.
- Use the Expo Go app to preview your components instantly on your phone.

---

## Integrating with TanStack Query
While native components are for UI, you often need to display data from an API. Use TanStack Query to fetch and cache data, then render it with your components.

```tsx
import { useQuery } from '@tanstack/react-query';
import { View, Text } from 'react-native';

type User = { id: string; name: string };

async function fetchUser(userId: string): Promise<User> {
  const res = await fetch(`https://api.example.com/users/${userId}`);
  if (!res.ok) throw new Error('Failed to fetch user');
  return res.json();
}

export function UserCard({ userId }: { userId: string }) {
  const { data, isLoading, error } = useQuery<User>({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId,
  });

  if (isLoading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error.message}</Text>;

  return (
    <View>
      <Text>{data?.name}</Text>
    </View>
  );
}
```

---

## Validating Data with Zod
Use Zod to validate API data before rendering:

```tsx
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
});

async function fetchUser(userId: string) {
  const res = await fetch(`https://api.example.com/users/${userId}`);
  const data = await res.json();
  return UserSchema.parse(data); // Throws if data is invalid
}
```

---

## Summary
- Native components are the foundation of every React Native app.
- Use TypeScript for type safety.
- Use Expo for fast development.
- Fetch and validate data with TanStack Query and Zod.
- Keep components focused, well-named, and easy to reuse. 