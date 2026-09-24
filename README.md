# THORVIX — Website

Marketing website for **THORVIX**, a Lahore-based AI and engineering agency that sells AI agents, automation, dedicated development teams and staff augmentation. It is a single-page React app: one long landing page with anchor navigation, animated effects and a "book a strategy call" form.

![Home page](https://mujtabaasif.vercel.app/assets/projects-screenshots/thorvix/home.webp)

**Live site:** https://thorvix.com/ · **Portfolio:** https://mujtabawd.vercel.app/

## Tech stack

| Area | Choice |
| --- | --- |
| Framework | React 18 + TypeScript |
| Build tool | Vite 5 |
| Styling | Tailwind CSS 3 plus CSS variables in `src/index.css` |
| Fonts | Barlow Condensed (headings), Barlow (body), DM Mono (labels), Bebas Neue (numbers), all from Google Fonts |
| Images | Logo and team photos are hosted on Cloudinary |

`@supabase/supabase-js` and `lucide-react` are installed but not used.

## Getting started

Requires Node.js 18 or newer.

```bash
npm install
npm run dev        # http://localhost:5173
```

| Script | What it does |
| --- | --- |
| `npm run dev` | Start the Vite dev server with hot reload |
| `npm run build` | Production build into `dist/` |
| `npm run preview` | Serve the production build locally |
| `npm run typecheck` | Type-check with `tsc` (passes) |
| `npm run lint` | Run ESLint. It currently crashes; see "Known issues" |

## Project structure

```
src/
├── App.tsx                 SiteContentProvider → MainLayout → HomePage
├── main.tsx                Entry point
├── index.css               Colour variables, bolt-effect classes, reduced-motion rules
├── context/
│   └── SiteContentContext.tsx   ALL site text, links, stats, team, form fields + their TypeScript types
├── layouts/MainLayout.tsx  Navbar + BookingModal + page + Footer; owns the modal open state
├── pages/HomePage.tsx      Stacks the page sections in order
├── sections/               One folder per page section (Hero, Services, Why, How, Solutions, Team, …)
├── components/
│   ├── Navbar/             Fixed header and full-screen mobile menu
│   ├── Footer/
│   ├── BookingModal/       "Schedule your strategy call" form
│   └── BoltEffect/         Lightning bolt that draws down the page as you scroll
├── ui/                     Button, SectionLabel, SectionHeading, StatBlock
├── hooks/                  useSiteContent (used); useScrollAnimation, useMediaQuery (unused)
└── utils/                  cn() class joiner; constants.ts (unused)
docs/                       Project documentation PDF and screenshots
```

## Page layout

There is only one page (`/`). The header links scroll to anchors on it:

| Order | Section | Anchor | Component |
| --- | --- | --- | --- |
| 1 | Hero: "Break every bottleneck", two buttons, three stats | none | `sections/Hero` |
| 2 | Service ticker (scrolling strip) | none | `sections/Ticker` |
| 3 | "Trusted by" logo strip | none | `sections/Trusted` |
| 4 | What we do: six service cards | `#services` | `sections/Services` |
| 5 | Lime marquee strip | none | `sections/Marquee` |
| 6 | The Thorvix edge: five reasons | `#about` | `sections/Why` |
| 7 | The process: three steps | none | `sections/How` |
| 8 | AI solutions: five tabs with a chat mock-up | `#solutions` | `sections/Solutions` |
| 9 | Team: three people | `#team` | `sections/Team` |
| 10 | Case studies: three metrics | `#cases` | `sections/Cases` |
| 11 | Testimonial | none | `sections/Testimonial` |
| 12 | Closing call to action | none | `sections/CTA` |

Every "Book…" or "Schedule…" button opens the same booking modal. So does the team section's "Meet our extended engineering team" link.

## Editing content

All text, links, numbers, team members and form fields live in one object, `siteContent`, in [src/context/SiteContentContext.tsx](src/context/SiteContentContext.tsx). Edit it and redeploy. There is no admin panel or CMS.

Some text fields accept a small set of tags, which the components turn into styling:

| Tag | Effect | Supported in |
| --- | --- | --- |
| `<em>…</em>` | Lime-coloured word | `hero.heading` |
| `<outline>…</outline>` | Outlined (hollow) text | `hero.heading` and every section `heading` |
| `<br>` | Line break | `hero.heading` and section `heading`s. Section headings accept only `<br>`, not `<br/>` |
| `<br/>` | Line break | `why.quote`, `cta.heading`, `solutions.panels[].heading` |
| `<strong>…</strong>` | Bold / highlighted words | `why.quote`, `testimonial.quote` |

Any other tag is shown as literal text.

## Booking form

The modal asks for name, company, email, primary interest and an optional message. **Submissions are not sent anywhere.** `handleSubmit` in [BookingModal.tsx](src/components/BookingModal/BookingModal.tsx) waits 1.2 seconds, shows "Message sent!" and closes. The code comment mentions EmailJS as the intended service. Until a real service (EmailJS, Formspree, a serverless function, …) is connected, enquiries are lost.

## Animations

- **Hero network**: a full-screen `<canvas>` of 60 drifting lime dots joined by lines when close (`HeroCanvas.tsx`). It runs on every frame for the whole visit.
- **Bolt effect**: an SVG lightning path that draws down the page as you scroll, with a glowing tip that makes nearby cards flash (`BoltEffect.tsx`).
- **Tickers**: the service ticker, trusted-logo strip and lime marquee loop horizontally using CSS animations.
- **Reduced motion**: when the OS asks for reduced motion, the bolt is removed and the tickers stop. The hero canvas still animates.

## Deployment

The build output is a static folder (`dist`). It has no client-side routes, so it can go on any static host (Vercel, Netlify, GitHub Pages, S3) with no rewrite rules. The build command is `npm run build` and the output folder is `dist`.

## Known issues

Each of these was confirmed in the running site or in the build output.

| # | Issue | Where |
| --- | --- | --- |
| 1 | Booking form does not send anything (see above) | `components/BookingModal/BookingModal.tsx` |
| 2 | Team heading shows a literal `<BR/>` ("MINDS BEHIND<BR/>THE MACHINES") because `SectionHeading` only splits on `<br>` | `team.heading` in `SiteContentContext.tsx` / `ui/SectionHeading.tsx` |
| 3 | The featured "AI Agents & Chatbots" card should be lime but renders dark with black text, which is almost unreadable. `bg-[var(--bg2)]` comes later in the CSS than `bg-[var(--accent)]`, so the dark background wins | `sections/Services/BentoCard.tsx` |
| 4 | On phones the two hero buttons sit on one row and "View Services" is cut off the right edge | `sections/Hero/Hero.tsx` (button row needs `flex-wrap`) |
| 5 | The booking modal has no scroll area, so on screens shorter than about 870px its top or bottom is cut off while the page behind is locked | `BookingModal.tsx` |
| 6 | The bolt effect looks for 8 card classes, but only `.team-card` exists in the markup, so only the 3 team cards react. It also looks for `.hero-section` and `#aiTabs`, which don't exist, so the bolt starts at the very top of the page | `components/BoltEffect/BoltEffect.tsx` |
| 7 | `p-[100px]_lg` and `lg:h-fit-content` in the Why section are not valid Tailwind classes, so no CSS is generated and the intended desktop padding / sticky left column don't apply | `sections/Why/Why.tsx` |
| 8 | `npm run lint` crashes: the installed ESLint 9.39 is incompatible with typescript-eslint 8.3. Upgrading `typescript-eslint` fixes it | `package.json` |
| 9 | Four of five footer service links and both policy links (Privacy, Terms) point to `#`, so no privacy or terms pages exist | `footer` in `SiteContentContext.tsx` |
| 10 | The favicon points to `/vite.svg`, which doesn't exist (there is no `public/` folder) | `index.html` |
| 11 | The mobile menu button stays as three bars when open (the `open` class has no styles) | `components/Navbar/Navbar.tsx` |

Housekeeping: the package is still named `vite-react-typescript-starter`, and `useScrollAnimation`, `useMediaQuery`, `utils/constants.ts`, `@supabase/supabase-js` and `lucide-react` are unused. `index.html` also has no meta description or social-share (Open Graph) tags.

## Brand

| Token | Hex | Use |
| --- | --- | --- |
| `--bg` | `#080808` | Page background |
| `--bg2` / `--bg3` | `#0e0e0e` / `#141414` | Cards, footer, hover states |
| `--border` / `--border-bright` | `#1e1e1e` / `#2a2a2a` | Grid lines and outlines |
| `--text` | `#f0ece4` | Main text (warm off-white) |
| `--muted` / `--muted2` | `#666` / `#444` | Secondary text |
| `--accent` | `#c8f032` | Lime: buttons, highlights, bolt |
| `--red` | `#ff3e3e` | Form error state |

The same colours are also available as Tailwind colours in `tailwind.config.js` (`bg`, `accent`, `muted`, …), but the components use the CSS variables.
