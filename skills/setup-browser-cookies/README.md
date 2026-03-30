# setup-browser-cookies

> Import logged-in browser sessions from your real Chromium browser into the headless browse session for authenticated QA testing.

## What it does

The `setup-browser-cookies` skill locates the `browse` binary, then runs `cookie-import-browser` to detect installed Chromium-based browsers (Comet, Chrome, Arc, Brave, Edge) and open an interactive picker UI in your default browser. From the picker you can search domains, import specific domain cookies with a single click, and remove previously imported cookies. Cookies are decrypted and loaded directly into the Playwright session, persisting across all subsequent `browse` commands. After you confirm completion, it runs `$B cookies` to display a summary of imported domains and counts.

## How to use it

Invoke by asking Claude to set up browser cookies, or by specifying a domain directly.

**Interactive picker (select domains in your browser):**
```
/setup-browser-cookies
```

**Direct import for a specific domain and browser:**
```
/setup-browser-cookies github.com
```
This skips the UI and imports `github.com` cookies from Comet (or the browser you specify) immediately.

Note: The first import from a given browser may trigger a macOS Keychain dialog — click "Allow" or "Always Allow" to proceed.

## Why it's useful

QA testing authenticated pages requires a valid session, and manually exporting and importing cookies is error-prone and slow. This skill makes it a one-step operation that reuses the session you already have open in your real browser, so you can immediately QA any page that requires login without re-entering credentials or navigating through auth flows.
