# 02 Navigation

## Routing

- Use hash routing from `react-router` v8 as the router (`createHashRouter`, because it is stable in a file://-based Electron renderer). Use the single `react-router` package, not `react-router-dom` from v7 and earlier
- Keep the main screen mounted at all times, and layer detail, settings, and similar screens on top as **child-route modals** (preserving the state of the screen behind them)

## Routed Modals

Screens opened as a modal (settings, detail, etc.) get their own URL route.

- Open: call `navigate("<parent path>/settings")` from a rail button or similar
- Close: remove the child route with `navigate("..")` (or `navigate` to the parent path). The modal must be closable with both the Esc key and a backdrop click
- Modal frame: `fixed inset-0 z-50` + overlay `bg-black/70 backdrop-blur-sm`

## Rail Placement and the Router

Choose the placement based on whether the rail implementation depends on route state.

- **Depends on route state**: Create a layout route and put the rail inside `<Route element={<AppLayout />}>`. `useParams` / `useNavigate` are available inside the rail

  ```tsx
  <Route element={<AppLayout />}>
    <Route path="/sheet/:id" element={<Sheet />}>
      <Route path="settings" element={<Settings />} />
    </Route>
  </Route>
  ```

- **Does not depend on route state (meguri style)**: The rail may be placed outside `RouterProvider` and navigate by manipulating `window.location.hash` directly

## Compatibility with Old URLs

When you change a screen's URL structure, keep a redirect route from the old URL (such as `<Route path="/settings" element={<Navigate to="/" replace />} />`).
