# Professional Portfolio Website

Editorial portfolio (cream + oxblood) with an admin dashboard.  
Built to be published on **GitHub Pages** (or any static host).

No frameworks, no build step. Works with **no backend at all**, and
gains real sign-in and multi-device sync if you connect a database.

## Features

- Public site: Home, Projects, About, Skills, Contact
- Only **public** projects are visible to visitors
- Admin dashboard (`admin.html`, deliberately not linked from the public nav)
  - Add / edit / delete projects
  - Toggle Public ↔ Private
  - Edit all site content (name, about, skills, links…)
  - Pick images from your computer for the profile and for projects
  - Set / change the password
  - Export content as JSON, or upload a file back
- **Images live in the `img/` folder** beside the pages, and the admin
  dashboard walks you through putting a new one there. No image hosting,
  no bucket, no upload. See [Images](#images).
- **Two modes.** With no database configured everything is stored in
  your browser, exactly as before. Connect Supabase and edits follow
  you to every device, and private projects stop being readable from
  devtools.
- Responsive from 390px to 1440px, with visible keyboard focus rings

## Quick Start

Open `index.html` in a browser.

To use the dashboard, open `admin.html` directly. The first time you
open it there is **no password**: the login form turns into a
"Set Up Admin Access" form and asks you to create one. Pick something
you have not used elsewhere — it is stored in this browser only, and
there is no way to recover it if you forget it.

To reset everything, clear this site's data in your browser
(devtools → Application → Local Storage → Clear site data).

---

## Connect a database (optional but recommended)

This is the difference between "works on the browser I edited it in"
and "works everywhere, and private really means private". It takes
about ten minutes and costs nothing until you go over the free tier.

### 1. Create the project

1. Sign up at [supabase.com](https://supabase.com) and create a project.
2. Wait for the database to finish provisioning.

### 2. Run the schema

1. In Supabase, open **SQL Editor → New query**.
2. Paste the whole of [`supabase/schema.sql`](supabase/schema.sql) and run it.
3. Then set your owner address, because the file ships with a
   placeholder:

   ```sql
   update public.app_config
      set value = 'you@yourdomain.com'
    where key = 'owner_email';
   ```

   It must match the email you use in step 4.

### 3. Create exactly one account

**Authentication → Users → Add user**, with your email and a
password. This is the only account that can change anything.

Then **Authentication → Providers → Email**, and turn **Enable email
signup** off. The policies already stop a stranger from writing, but
an open signup form is a hole you do not need.

### 4. Put your keys in `js/config.js`

**Project Settings → API** gives you two values:

| Supabase shows | Put it in |
|---|---|
| Project URL | `SUPABASE_URL` |
| `anon` `public` key | `SUPABASE_ANON_KEY` |

```js
const SUPABASE_URL = 'https://xxxxxxxxxxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOi...';
```

That is the whole integration. Reload `admin.html`, sign in with your
email and password, and the dashboard starts writing to the database.

### 5. Move your existing content up

If you already had projects saved in a browser, the database is
empty. Sign in, then on the **Dashboard** panel use
**Content Transfer → Upload Content**:

- **Download Content** writes `portfolio-content.json` from whatever
  the browser has, so you can keep it or edit it first.
- **Upload Content** reads that file back and replaces the database
  contents with it.

---

## Images

Every image on this site is an ordinary file in the **`img/` folder**
next to `index.html` and `admin.html`. There is no image hosting, no
storage bucket, and nothing is uploaded anywhere.

```
img/
├── profile/     the picture in About
├── projects/    software, websites, AI, creative work
├── games/       games
└── films/       films and videos
```

Folders exist so that a growing portfolio stays findable. Nothing in the
code refers to any individual project by name, so you can add as many as
you like without touching it.

### Adding an image

**Admin → Site Settings**, or **Admin → Projects → edit a project**:

1. Press **Choose Image** and pick a file from your computer.

   It is checked for real: the file's own contents have to be a JPG,
   PNG or WEBP, not just its name, and it has to be under 5 MB. A `.gif`,
   a `.pdf`, or a `.txt` renamed to `.jpg` is refused with the reason.

2. The picture appears immediately, with its filename and size.

3. The line underneath shows **exactly where the file has to go**, for
   example `img/games/run-from-the-light.jpg`. Press **Download Image**
   and your browser saves it under that exact filename.

4. Put that file in that folder, next to `index.html`.

5. Press **Save**. The path is stored and the site uses it.

A browser cannot do step 4 for you. No web page is allowed to write into
a folder on your disk, so the dashboard gives you the file back under
the right name and tells you where it goes rather than pretending it has
already copied it.

### Choosing the folder

The project editor has a **Where should it go?** menu, which decides
the folder in the path:

| Menu entry | Folder |
| --- | --- |
| software, websites, AI, creative work | `img/projects/` |
| games | `img/games/` |
| films and videos | `img/films/` |

Open a project that already has a picture in `img/games/` and Games is
already selected. Renaming a project never moves its picture, and
nothing about any specific project is hard-coded.

### Existing images

Anything already stored keeps working, untouched:

- External addresses such as `https://images.unsplash.com/photo-…` are
  still used exactly as they are. Paste one into
  **Enter a path or URL instead** if you want to keep it.
- Paths already inside `img/` are still used as they are.
- The seed data ships with Unsplash links, so the site looks complete
  before you add anything.

### If an image does not appear

The site falls back to `assets/img-placeholder.svg`. It is almost always
a path with a typo, or a file that has not been copied into the folder
yet. The dashboard says so directly when a stored path will not load.

### Removing an image

**Remove Image** clears the path. The file stays in `img/` — delete it
yourself once you are sure nothing points at it.

---

### How the two modes fit together

The public page never waits for the network. It paints from the local
copy immediately, then quietly swaps in the database copy when it
arrives and caches that for next time. So:

- First visit is instant.
- Offline still shows your content.
- A database outage or bad credentials degrade to the local copy
  rather than showing an error page.
- A visitor's browser caches **only public projects**, because that
  is all the database will send an anonymous browser.

### What the anon key is, and is not

`SUPABASE_ANON_KEY` is shipped in `config.js` and every visitor
downloads it. That is by design — it is the *public* key, and Supabase
publishes it on purpose. It is not a secret.

The thing protecting your data is **row level security** in
`supabase/schema.sql`:

- Anonymous browsers can only read rows where `is_public = true`.
- Only the account named in `app_config.owner_email` can write.
- A rejected `UPDATE` or `DELETE` returns **zero rows and no error**,
  so the dashboard checks the returned rows rather than just the
  error, and tells you when you are signed in as the wrong account.

## The admin password is not real security — in browser-only mode

With no database connected, this is a static site, so there is no
server to authenticate against. Everything, including the password,
lives in your own browser.

- The password is salted and hashed (60,000 iterations of SHA-256)
  before it is stored, so it is not kept as plain text.
- That is **obfuscation, not protection**. Anyone with access to the
  browser can read the stored data, replace the password, or edit the
  projects directly from devtools.
- It stops a casual visitor from clicking through to your drafts. It
  does not stop anyone who is determined.
- There is no "private" project that cannot be discovered by someone
  reading `localStorage`.

**Connect a database and this section stops applying**: the password
is checked by Supabase, and private projects are never sent to a
visitor's browser at all. The dashboard says which mode it is in, on
the login screen and in the sidebar.

## Data handling

`js/core.js` is the single source of truth for validation and for
output safety. Both `app.js` and `admin.js` load it first and use it,
and `js/db.js` routes every read and write through one place.

- One storage contract, with a schema version in `portfolio_meta`.
- Settings and projects are validated and coerced on every read, so
  the page still renders if stored data is missing, truncated, the
  wrong shape, or not valid JSON at all.
- Anything unreadable is copied to `<key>.unreadable-backup` before
  defaults are restored, so bad data is never silently destroyed.
- **Values coming back from the database are treated as untrusted and
  go through the same normalisers as local data.** The database is
  not assumed to be well-formed just because you own it.
- All dynamic values are sanitised on the way out: text is assigned
  with `textContent`, URLs must be `http:`, `https:` or (for the
  contact address) `mailto:`, and the About field is filtered through
  a tag and attribute allow-list.
- Project images fall back to `assets/img-placeholder.svg` when the
  remote image is missing or the URL is unusable.
- A save that the server refuses is reported in the dashboard, and
  kept locally, rather than appearing to succeed and being lost.

To start over from the seed data, clear the site's localStorage — or,
in database mode, re-run `supabase/schema.sql` and upload a fresh
export.

## Customizing

### Through the dashboard (recommended)
1. Open `admin.html` and create your password.
2. **Site Settings** → name, about, skills, links, profile image.
3. **Projects** → add your real projects, set Public or Private.
4. For each image, follow [Images](#images) to put the file in `img/`.

Values that are not valid URLs or email addresses are cleared rather
than saved, and the dashboard tells you which fields were affected.

### In the code
- `js/config.js` → the only file you normally need to edit.
- `js/core.js` → `DEFAULT_SETTINGS` and `DEFAULT_PROJECTS` (used only
  on a first visit, or after clearing site data).
- `css/style.css` → `:root` holds the colour and spacing tokens.
- `assets/img-placeholder.svg` → the local image fallback.

## Deploy to GitHub Pages

1. Create a new repository.
2. Upload everything from this folder, keeping the structure
   (`assets/`, `css/`, `js/`, `index.html`, `admin.html`).
3. **Settings → Pages** → Source: deploy from branch `main`, folder
   `/ (root)`.
4. Your site is live at `https://YOUR_USERNAME.github.io/REPO_NAME/`

### Notes about browser storage
- With no database connected, content is stored in each visitor's
  browser, not on the server, and every visitor starts from the same
  default seed data.
- With a database connected, everyone reads the same content, and
  private projects are withheld from anonymous browsers entirely.
- Either way, `admin.html` is not linked from the public navigation
  on purpose. Anyone can still reach it by typing the URL, which is
  why the sign-in exists at all.

## Project layout

```
index.html          public site
admin.html          admin dashboard
css/style.css       all styling
js/config.js        the only file you edit: database credentials
js/core.js          storage contract, defaults, validation, sanitising
js/db.js            data source: browser storage or the database
js/auth.js          sign-in, in either mode
js/app.js           public page rendering
js/admin.js         dashboard, auth, CRUD, content transfer
supabase/schema.sql tables, row level security, seed data
assets/img-placeholder.svg   local image fallback
img/                every image on the site: profile, projects, games, films
```

## Tech

- HTML, CSS, and vanilla JavaScript
- No frameworks, no build step, no package manager
- Optional backend: Supabase (Postgres + auth), loaded on demand
  from a CDN only when `js/config.js` is filled in
- Font Awesome icons and Google Fonts from a CDN
- Images from Unsplash in the seed data (replace with your own)

---

Made for your portfolio. You can take full credit.
