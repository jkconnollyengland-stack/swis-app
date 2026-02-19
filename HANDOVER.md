# SWIS App — AI Handover Document
> Last updated: February 2026  
> If you're a new Claude session, read this first before touching any code.

---

## What Is SWIS?

**SWIS = See What I See**  
A real-time group screen/content sharing web app. Users create or join a "room" using a short code, then share photos, links, or text messages that all room members can see instantly on their own devices — like a shared second screen for a group.

**Live URL:** `https://jkconnollyengland-stack.github.io/swis-app/`  
**GitHub repo:** `https://github.com/jkconnollyengland-stack/swis-app`  
**Structure:** Single file — everything is in `index.html`. No build step, no npm, no framework. Pure HTML/CSS/JS.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (ES modules) |
| Realtime DB | Firebase Realtime Database (europe-west1) |
| Hosting | GitHub Pages |
| QR codes | qrcodejs via cdnjs |
| Icons | Lucide Eye icon (inline SVG) |

### Firebase Config
```js
const firebaseConfig = {
    apiKey: "AIzaSyCeDlaBjRz-jPj6TZmEsg0yNvfR7fv0l4E",
    authDomain: "swis-9b0ce.firebaseapp.com",
    databaseURL: "https://swis-9b0ce-default-rtdb.europe-west1.firebasedatabase.app",
    projectId: "swis-9b0ce",
    storageBucket: "swis-9b0ce.firebasestorage.app",
    messagingSenderId: "25689930290",
    appId: "1:25689930290:web:a5370855e0dd7104071a2b"
};
```

---

## Firebase Data Structure

```
rooms/
  {ROOMCODE}/
    members/
      {userId}: { name, joined }
    shares/
      {userId}: { type, data, timestamp }
```

- `type` is one of: `photo`, `link`, `text`
- `data` for photos is a base64 data URL
- Room codes are random words + number e.g. `PUB42`, `CAFE7`
- Users are auto-named `User_xxxxxx` (random 6-char suffix)
- Users are cleaned up from Firebase on tab close / leave

---

## App Flow

1. **Splash screen** (3 seconds) → fades out
2. **Welcome screen** — create room or enter a code to join
3. **Room screen** — shows carousel of member cards, each with their current share
4. QR code button → overlay with scannable link to join the room directly

---

## UI Components

### Splash Screen
- Originated from Lovable.ai's `StartupLogo.tsx` component
- Orange gradient outer circle (128px) with `animate-ping` ripple ring
- Blue gradient inner circle (112px) with `animate-pulse` border ring
- Lucide Eye SVG (64px, strokeWidth 1.5) in white
- SWIS title: orange→blue gradient text
- "See What I See" + "Share your world instantly" subtitles
- 5 connection dots (alternating large orange / small blue) with staggered pulse

### Header Bar (room screen)
- White rounded card, `height: 56px`, `margin: 8px 12px`
- Logo: single div — orange border ring, blue gradient background, white eye SVG centred inside
  - **Important:** Use a single div (border = orange ring, background = blue) NOT two nested absolute-positioned divs — that approach broke in practice
- Room code in purple/blue (`#667eea`), bold
- Member count below in grey
- QR button (purple gradient) floated right, fully contained inside the white card

### Carousel (room screen)
- Instagram-style horizontal swipe carousel
- Each member gets a card (9:19.5 aspect ratio, phone shape)
- Cards show: member name, LIVE badge if sharing, and the shared content
- Touch swipe supported with smooth follow + snap
- Arrow buttons (left/right) appear when multiple members
- Dot indicators below carousel

### Control Bar
- Share button (purple gradient) — opens share modal
- Stop button (red) — only visible when YOU are sharing
- Leave button (grey) — confirms then cleans up Firebase and returns to welcome screen

### Share Modal
- Three options: Photo/Screenshot, Website Link, Message
- Photo → file input → reads as base64 → stores in Firebase
- Link/Text → textarea → stores as string

---

## Known Issues & History

### Fixed
- **Header bleed bug** — the purple gradient background was bleeding through/over the white header strip. Fixed by giving `.room-header` a fixed `height: 56px` and ensuring the QR button sits fully inside.
- **Logo rendering bug** — using two absolutely-positioned nested divs for the logo (orange ring outer + blue circle inner) failed in practice. Fixed by using a single div with `border` for the orange ring and `background` for the blue fill.

### Watch Out For
- Firebase `onValue` listeners are added inside `updateScreens()` which itself is called from other `onValue` callbacks — this creates nested listeners. It works but could cause extra reads. A future refactor could separate member and share listeners.
- `setupSwipe()` clones the container element to remove old listeners (`container.replaceWith(container.cloneNode(true))`). This means event delegation won't survive — always re-query after calling it.
- Photos are stored as base64 in Firebase — large images will hit Firebase's 10MB node limit. No compression is currently applied.
- Session is maintained via `sessionStorage` — closing and reopening the tab starts a fresh session.

---

## Design Language

- **Primary colour:** `#667eea` → `#764ba2` (purple gradient) — used for buttons, room code text
- **Accent orange:** `#f97316` (Tailwind orange-500) — logo ring
- **Accent blue:** `#3b82f6` (Tailwind blue-500) — logo fill
- **Background:** `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
- **Cards:** white, `border-radius: 20px`, subtle shadow
- **Font:** system font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI', Arial`)

---

## Deployment

The app is deployed via GitHub Pages from the `main` branch root.  
To deploy a change: just commit and push `index.html` to `main`. GitHub Pages auto-deploys within ~60 seconds.

QR codes point to:  
`https://jkconnollyengland-stack.github.io/swis-app/?room={ROOMCODE}`

The app reads the `?room=` URL param on load and auto-joins that room.

---

## What Was Built With Lovable.ai

The project was started in Lovable.ai which produced the initial React app including:
- `SplashScreen.tsx` component (3-second animated intro)
- `StartupLogo.tsx` component (the animated eye logo)

These were ported into vanilla HTML/CSS/JS when the app was consolidated into a single `index.html` for GitHub Pages hosting. The Lovable React components are no longer used directly but the visual design is faithfully recreated.

---

## Next Steps / Ideas (not yet built)

- Username customisation (let users set a display name)
- Room expiry / auto-cleanup of old rooms
- Image compression before upload to avoid Firebase size limits
- Ability to share the current URL/tab (browser share API)
- Notification when a new member joins or shares something
- Admin/host role (only host can approve shares)
