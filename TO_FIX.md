# Outstanding Issues to Fix

## From Original Issue List

### 1. No Offline Support
- App requires network connection to function
- Could implement Service Worker for caching
- Consider PWA manifest for install capability
- Cache static assets and JSON data for offline viewing

### 5. No Error Boundaries
- No graceful handling if data fails to load
- API errors would show broken UI
- Need Svelte error boundary components
- Should show user-friendly error messages with retry options

### 7. No Keyboard Navigation for Track Selector
- Track tabs lack arrow key navigation
- Should support Left/Right arrow keys to move between tabs
- Enter/Space to select focused tab
- Home/End keys to jump to first/last tab
- Add proper `tabindex` and focus management
