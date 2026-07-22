---
layout: default
title: React Native Developer Roadmap
---

# 📱 React Native Developer Roadmap

A structured path from beginner to professional React Native developer. Check off items as you progress.

---

## Phase 1: Foundations (4–6 weeks)

### JavaScript & TypeScript Fundamentals
- [ ] Variables, data types, scope (`var`, `let`, `const`)
- [ ] Functions, arrow functions, closures
- [ ] Array/object methods (`map`, `filter`, `reduce`, `find`, spread/rest)
- [ ] Destructuring
- [ ] Promises, `async`/`await`, event loop basics
- [ ] ES Modules (`import`/`export`)
- [ ] TypeScript basics: types, interfaces, generics
- [ ] TypeScript with React (typing props, state, hooks)

### Git & Tooling
- [ ] Git basics: commit, branch, merge, rebase
- [ ] GitHub/GitLab workflows (PRs, code review)
- [ ] Terminal / command line basics
- [ ] Node.js and npm/yarn/pnpm package management

---

## Phase 2: React Core Concepts (3–4 weeks)

- [ ] JSX syntax and rendering
- [ ] Components (function components, props)
- [ ] State management with `useState`
- [ ] Side effects with `useEffect`
- [ ] Component lifecycle concepts
- [ ] Conditional rendering & lists (`key` prop)
- [ ] Context API (`useContext`)
- [ ] Refs (`useRef`)
- [ ] Custom hooks
- [ ] Performance hooks: `useMemo`, `useCallback`
- [ ] Error boundaries

---

## Phase 3: React Native Fundamentals (6–8 weeks)

### Core Concepts
- [ ] Understanding the RN architecture (bridge vs. New Architecture/Fabric & TurboModules)
- [ ] Expo vs. React Native CLI — when to use which
- [ ] Core components: `View`, `Text`, `Image`, `ScrollView`, `FlatList`, `SectionList`
- [ ] `StyleSheet` and Flexbox layout for mobile
- [ ] Handling touch: `Pressable`, `TouchableOpacity`, gestures
- [ ] Platform-specific code (`Platform.OS`, `.ios.js`/`.android.js` files)
- [ ] Safe area handling (notches, status bars)
- [ ] Keyboard handling (`KeyboardAvoidingView`)

### Navigation
- [ ] React Navigation setup (Stack, Tab, Drawer navigators)
- [ ] Passing params between screens
- [ ] Deep linking
- [ ] Authentication flow (protected routes)

### Forms & Input
- [ ] TextInput handling and validation
- [ ] Form libraries (React Hook Form or Formik)
- [ ] Schema validation (Zod or Yup)

---

## Phase 4: Data & State Management (4–6 weeks)

- [ ] Local component state vs. global state
- [ ] State management libraries: Zustand, Redux Toolkit, or Jotai
- [ ] Fetching data with `fetch`/`axios`
- [ ] Server state management: React Query (TanStack Query)
- [ ] Local persistence: AsyncStorage, MMKV
- [ ] Offline-first strategies and caching
- [ ] SQLite / WatermelonDB for local databases
- [ ] REST API integration
- [ ] GraphQL basics (Apollo Client or urql) — optional but valuable

---

## Phase 5: Native Device Features (3–4 weeks)

- [ ] Camera and image picker
- [ ] Push notifications (Firebase Cloud Messaging / Expo Notifications)
- [ ] Location and maps (`react-native-maps`)
- [ ] Permissions handling (camera, location, notifications)
- [ ] Biometric authentication (Face ID/Touch ID)
- [ ] Local storage of files/media
- [ ] Deep linking & universal links
- [ ] In-app purchases (basic understanding)
- [ ] Background tasks

---

## Phase 6: Native Modules & Bridging (Advanced, 3–5 weeks)

- [ ] Understanding Native Modules (iOS: Swift/Objective-C, Android: Kotlin/Java)
- [ ] Writing a simple native module/bridge
- [ ] Understanding TurboModules & Fabric (New Architecture)
- [ ] Linking third-party native libraries
- [ ] Debugging native build issues (Xcode, Android Studio)
- [ ] Working with CocoaPods and Gradle

---

## Phase 7: Testing (2–3 weeks)

- [ ] Unit testing with Jest
- [ ] Component testing with React Native Testing Library
- [ ] Mocking native modules and APIs
- [ ] End-to-end testing with Detox or Maestro
- [ ] Snapshot testing
- [ ] Test-driven development basics

---

## Phase 8: Performance & Optimization (2–3 weeks)

- [ ] Identifying re-render issues (React DevTools Profiler)
- [ ] Optimizing `FlatList`/`SectionList` (windowing, `getItemLayout`, `keyExtractor`)
- [ ] Image optimization and caching (`react-native-fast-image`)
- [ ] Reducing bundle size
- [ ] Hermes engine understanding
- [ ] Memory leak detection
- [ ] Native performance profiling (Flipper, Xcode Instruments, Android Profiler)
- [ ] Reanimated for smooth animations (v3+)
- [ ] Gesture Handler for native-feeling gestures

---

## Phase 9: App Architecture & Best Practices (3–4 weeks)

- [ ] Project folder structure & feature-based architecture
- [ ] Design patterns (container/presenter, custom hooks pattern)
- [ ] Environment configuration (`.env` files, multiple environments — dev/staging/prod)
- [ ] Code quality: ESLint, Prettier, Husky pre-commit hooks
- [ ] Monorepo tools (Turborepo, Nx) — for larger teams
- [ ] Accessibility (a11y) best practices
- [ ] Internationalization (i18n)
- [ ] Theming and dark mode support
- [ ] Design systems / component libraries (e.g., Tamagui, NativeWind, React Native Paper)

---

## Phase 10: CI/CD & Deployment (3–4 weeks)

- [ ] App signing basics (iOS provisioning profiles, Android keystores)
- [ ] Building with EAS Build (Expo) or Fastlane
- [ ] Over-the-air (OTA) updates (Expo Updates / CodePush concepts)
- [ ] CI/CD pipelines (GitHub Actions, Bitrise, or EAS Workflows)
- [ ] App Store submission process (iOS)
- [ ] Google Play submission process (Android)
- [ ] Crash reporting & monitoring (Sentry, Firebase Crashlytics)
- [ ] Analytics integration (Firebase Analytics, Amplitude, PostHog)
- [ ] Feature flags and A/B testing

---

## Phase 11: Professional & Soft Skills (Ongoing)

- [ ] Reading and contributing to open-source RN libraries
- [ ] Writing technical documentation
- [ ] Code review etiquette
- [ ] Agile/Scrum workflow familiarity
- [ ] Communicating technical tradeoffs to non-technical stakeholders
- [ ] Building a portfolio with 2–3 polished apps
- [ ] Contributing to or maintaining a public GitHub repo
- [ ] Staying current: following React Native blog, Expo changelog, and release notes

---

## 🧭 Suggested Learning Order (Summary)

1. JavaScript/TypeScript → 2. React → 3. React Native Core → 4. Navigation → 5. State/Data → 6. Native Device Features → 7. Testing → 8. Performance → 9. Architecture → 10. CI/CD → 11. Ongoing professional growth

---

## 🛠️ Recommended Practice Projects

- [ ] To-do list app with local persistence (AsyncStorage/MMKV)
- [ ] Weather app consuming a public REST API
- [ ] Social media feed clone with infinite scroll (FlatList + pagination)
- [ ] E-commerce app with cart, auth, and payment flow
- [ ] Chat app with real-time updates (Firebase or WebSockets)
- [ ] Full production app published to both App Store and Google Play

---

## 📚 Key Resources to Explore

- [ ] Official React Native docs (reactnative.dev)
- [ ] Official Expo docs (docs.expo.dev)
- [ ] React Navigation docs
- [ ] TanStack Query docs
- [ ] React documentation (react.dev)
- [ ] "This Week In React Native" newsletter
- [ ] Kadi Kraman / William Candillon YouTube (animations & advanced RN)
