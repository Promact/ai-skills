# React Native Best Practices

- Avoid unnecessary re-renders in list components; use `FlatList`/
  `FlashList` with stable `keyExtractor` rather than `.map()` for long lists.
- Keep platform-specific code isolated (`.ios.tsx`/`.android.tsx` or a clear
  `Platform.select`) rather than scattering inline platform checks.
- Handle offline/poor-connectivity states explicitly for network calls —
  mobile networks are not desktop networks.
- Be deliberate about bundle size and native dependencies; each added native
  module has a real cost (build time, binary size, upgrade risk).
- Persist only what's needed locally (secure storage for tokens/secrets,
  not `AsyncStorage`).

<!-- TODO: replace/extend with Promact's internal React Native standards -->
