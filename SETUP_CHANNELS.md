# Postiz Channel Setup Guide

How to connect social media channels to your local Postiz instance at `http://localhost:4007`.

Every channel that uses OAuth needs a "developer app" on the respective platform, which gives you a **Client ID** and **Client Secret**. You paste those into `docker-compose.yaml` and restart.

**Universal callback URL pattern:** `http://localhost:4007/integrations/social/<identifier>` (e.g. `.../social/x`, `.../social/linkedin`). Each channel has its own identifier — see the table at the bottom.

---

## Table of contents
1. [Prerequisites](#prerequisites)
2. [Part 1: Connect X (step-by-step)](#part-1-connect-x-step-by-step)
3. [Part 2: Applying env changes](#part-2-applying-env-changes)
4. [Part 3: All other channels — reference](#part-3-all-other-channels--reference)
   - [A. Zero-config channels (no env vars, no dev app)](#a-zero-config-channels)
   - [B. OAuth 2.0 channels (need a dev app)](#b-oauth-20-channels)
   - [C. Special cases](#c-special-cases)
5. [Part 4: Common gotchas](#part-4-common-gotchas)
6. [Channel identifier reference](#channel-identifier-reference)

---

## Prerequisites

- Postiz running locally: `docker compose up -d`
- Accessible at http://localhost:4007
- Registered + logged in (you're auto-activated because no email provider is configured)
- Edit target: `docker-compose.yaml` — env vars are set inline (lines 33–64), not via a separate `.env` file

---

## Part 1: Connect X (step-by-step)

X is the first channel we're tackling. It uses **OAuth 1.0a** (unlike most others which use OAuth 2.0). That changes which fields you copy from the developer portal.

### Step 1: Create an X Developer account
1. Go to **https://developer.x.com/en/portal/dashboard**
2. Sign in with the X account you want to post from
3. If prompted for a use case, pick "Making a bot" or "Building tools for X users" — both fine

### Step 2: Create a Project and App
1. In the dashboard, click **Create Project** if you don't have one
   - Name: anything, e.g. `Postiz Local`
   - Use case: any
2. Inside the project, you'll be prompted to create an **App** (or use an existing one)
   - Name: anything, e.g. `Postiz Dev`

### Step 3: Configure User Authentication
This is the critical step. Posting won't work if this is wrong.

1. Open your App → **Settings** tab
2. Scroll to **User authentication settings** → click **Set up** (or **Edit**)
3. Fill in:
   - **App permissions**: `Read and write` ← **must be this, not "Read only"**
   - **Type of App**: `Web App, Automated App or Bot`
   - **Callback URI / Redirect URL**: `http://localhost:4007/integrations/social/x`
   - **Website URL**: `http://localhost:4007` (or any URL)
4. Save

### Step 4: Get your API Key & Secret
1. Go to the **Keys and tokens** tab
2. Under **Consumer Keys** click **View Keys** or **Regenerate**
3. Copy:
   - **API Key** → this is `X_API_KEY`
   - **API Key Secret** → this is `X_API_SECRET`

> ⚠️ If you set up User Authentication *after* generating the keys, you **must regenerate** the Consumer Keys. Changing permissions doesn't retroactively update old keys, and posting will fail with 403. This trips up 90% of first-time setups.

### Step 5: Paste into `docker-compose.yaml`
Edit lines 34–35:

```yaml
X_API_KEY: 'paste_your_api_key_here'
X_API_SECRET: 'paste_your_api_key_secret_here'
```

### Step 6: Apply + connect
See [Part 2](#part-2-applying-env-changes) for the apply command. Then in the Postiz UI:
- Go to **Channels → Add Channel → X**
- Complete the OAuth flow on X's site
- You should be redirected back and the channel should appear in your list

### Step 7: Test posting
- Create a post in Postiz
- Pick the X channel
- Schedule it ~2 minutes in the future
- Watch it at **http://localhost:8080** (Temporal UI) to see the workflow execute
- Verify on X

---

## Part 2: Applying env changes

Every time you edit `docker-compose.yaml`, run:

```bash
docker compose up -d
```

This reapplies env vars and restarts only the containers whose config changed. Your data (users, posts, channels) is preserved via the named volumes.

**Note:** Connecting a channel in the UI also persists to Postgres, so your connected channels survive restarts too. You only need to reconfigure env vars if you rotate/change credentials.

---

## Part 3: All other channels — reference

### A. Zero-config channels

These require **no environment variables** and no developer app. You enter credentials (API key, username/password, instance URL, etc.) directly in the Postiz UI when you add the channel.

| Channel | What you'll need to provide in the UI |
|---|---|
| **Dev.to** | API key from https://dev.to/settings/account |
| **Hashnode** | Personal Access Token from https://hashnode.com/settings/developer |
| **Medium** | Integration token from https://medium.com/me/settings/security → "Integration tokens". ⚠️ Medium stopped issuing new tokens for most accounts around 2021; existing tokens still work |
| **WordPress** | Your blog URL, admin username, application password (not your login password — create one at Users → Profile → Application Passwords) |
| **Bluesky** | Handle (`you.bsky.social`) + app password from https://bsky.app/settings/app-passwords |
| **Lemmy** | Instance URL (e.g. `https://lemmy.world`) + username + password |
| **Nostr** | Your Nostr private key in **hex** format |
| **ListMonk** | Your ListMonk URL + admin username + password |
| **Moltbook** | API key from your Moltbook agent |
| **Telegram** (channel) | Chat/channel ID (see Telegram section below for bot token caveat) |
| **Skool** | Requires the Postiz Chrome extension to capture browser cookies |

**These are the easiest to start with if you want to test the end-to-end posting flow quickly.** Bluesky is especially fast (app password takes 10 seconds to create).

---

### B. OAuth 2.0 channels

All of these follow the same pattern:
1. Create a developer app on the platform
2. Set the callback URL to `http://localhost:4007/integrations/social/<identifier>`
3. Copy Client ID + Client Secret into `docker-compose.yaml`
4. `docker compose up -d`
5. Connect in the Postiz UI

#### LinkedIn (identifier: `linkedin`, `linkedin-page`)
- **Env vars**: `LINKEDIN_CLIENT_ID`, `LINKEDIN_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/linkedin`
- **Dev portal**: https://www.linkedin.com/developers/apps → Create App
- **Required products**: "Share on LinkedIn", "Sign In with LinkedIn using OpenID Connect", and (for company pages) "Marketing Developer Platform" — the last one requires LinkedIn approval
- **Scopes used by Postiz**: `openid profile w_member_social r_basicprofile rw_organization_admin w_organization_social r_organization_social`
- **Notes**: Both personal profile (`linkedin`) and company pages (`linkedin-page`) share the same credentials

#### Reddit (identifier: `reddit`)
- **Env vars**: `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/reddit`
- **Dev portal**: https://www.reddit.com/prefs/apps → "Create another app"
- **App type**: `web app`
- **Scopes**: `read identity submit flair`
- **Notes**: Rate limited to 1 request/sec. Posting to most subreddits requires karma — test with your own profile first

#### Facebook (identifier: `facebook`)
- **Env vars**: `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/facebook`
- **Dev portal**: https://developers.facebook.com/apps → Create app → type "Business"
- **Products to add**: "Facebook Login", "Pages API"
- **Scopes**: `pages_show_list business_management pages_manage_posts pages_manage_engagement pages_read_engagement read_insights`
- **Notes**: Only Facebook **Pages** can be posted to (not personal timelines — Meta removed that API years ago). You need admin access to a Page. App Review is required to post to Pages you don't own

#### Instagram (Business) (identifier: `instagram`)
- **Env vars**: reuses `FACEBOOK_APP_ID` + `FACEBOOK_APP_SECRET` (same Facebook app)
- **Callback URL**: `http://localhost:4007/integrations/social/facebook` (handled via the Facebook flow)
- **Dev portal**: same as Facebook. Add the "Instagram Graph API" product
- **Requirements**:
  - Your Instagram account must be **Business** or **Creator** type
  - It must be connected to a Facebook Page
- **Notes**: This is the "proper" Instagram integration. Most people want this one

#### Instagram (Standalone) (identifier: `instagram-standalone`)
- **Env vars**: `INSTAGRAM_APP_ID`, `INSTAGRAM_APP_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/instagram-standalone`
- **Dev portal**: https://developers.facebook.com/apps → a separate app type, or use "Instagram Basic Display"
- **Scopes**: `instagram_business_basic instagram_business_content_publish instagram_business_manage_comments instagram_business_manage_insights`
- **Notes**: Alternative flow that doesn't require a Facebook Page. Uses Meta's newer standalone Instagram API

#### Threads (identifier: `threads`)
- **Env vars**: `THREADS_APP_ID`, `THREADS_APP_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/threads`
- **Dev portal**: https://developers.facebook.com/apps → add "Threads API" product
- **Scopes**: `threads_basic threads_content_publish threads_manage_replies threads_manage_insights`
- **Notes**: Meta's Threads app is separate from Facebook/Instagram apps even though it's in the same portal

#### YouTube (identifier: `youtube`)
- **Env vars**: `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/youtube`
- **Dev portal**: https://console.cloud.google.com/ → Create project → APIs & Services → OAuth consent screen (external), then Credentials → Create OAuth Client ID → Web application
- **Enable these APIs**: YouTube Data API v3, YouTube Analytics API
- **Scopes**: `userinfo.profile userinfo.email youtube youtube.force-ssl youtube.readonly youtube.upload youtubepartner yt-analytics.readonly`
- **Notes**: Uploads/posting are subject to YouTube quota. For unverified apps, scope screen shows a warning — add yourself as a test user on the consent screen
- **Bonus**: These credentials are also used for Google login (the "Sign in with Google" button on the login page) AND as a fallback for Google My Business (GMB)

#### Google My Business (GMB) (identifier: `gmb`)
- **Env vars**: `GOOGLE_GMB_CLIENT_ID`, `GOOGLE_GMB_CLIENT_SECRET` — **or** fall back to `YOUTUBE_CLIENT_ID/SECRET` (same Google OAuth app works for both)
- **Callback URL**: `http://localhost:4007/integrations/social/gmb`
- **Dev portal**: same Google Cloud project as YouTube
- **APIs to enable**: My Business Business Information API, My Business Account Management API
- **Scopes**: `userinfo.profile userinfo.email business.manage`
- **Notes**: Requires a verified Google Business Profile location. Approval from Google needed for production

#### TikTok (identifier: `tiktok`)
- **Env vars**: `TIKTOK_CLIENT_ID`, `TIKTOK_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/tiktok`
- **Dev portal**: https://developers.tiktok.com/ → Manage apps → Create app
- **Products to add**: "Login Kit", "Content Posting API"
- **Scopes**: `video.list user.info.basic video.publish video.upload user.info.profile user.info.stats`
- **Notes**: ⚠️ **Content Posting API requires TikTok approval.** Until approved, your app can only post to **private** (unlisted) accounts and the post will be stuck in "draft" state. Approval takes weeks. For local testing, just use your own private test account

#### Pinterest (identifier: `pinterest`)
- **Env vars**: `PINTEREST_CLIENT_ID`, `PINTEREST_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/pinterest`
- **Dev portal**: https://developers.pinterest.com/apps/ → Create app
- **Scopes**: `boards:read boards:write pins:read pins:write user_accounts:read`
- **Notes**: Trial apps can be used by the app owner immediately. Production use requires app review

#### Dribbble (identifier: `dribbble`)
- **Env vars**: `DRIBBBLE_CLIENT_ID`, `DRIBBBLE_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/dribbble`
- **Dev portal**: https://dribbble.com/account/applications/new
- **Scopes**: `public upload`
- **Notes**: Dribbble requires you to have an existing Dribbble account with shot-posting privileges

#### Slack (identifier: `slack`)
- **Env vars**: `SLACK_ID`, `SLACK_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/slack`
- **Dev portal**: https://api.slack.com/apps → Create New App → From scratch
- **Scopes to add under "OAuth & Permissions"**: `channels:read chat:write users:read groups:read channels:join chat:write.customize`
- **Notes**: ⚠️ Slack traditionally **requires HTTPS callback URLs**. Postiz auto-wraps the local URL with `https://redirectmeto.com/http://localhost:4007/...` to work around this for local dev — no action needed from you, but be aware if you see a weird-looking redirect URL during OAuth

#### Discord (identifier: `discord`)
- **Env vars**: `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_BOT_TOKEN_ID`
- **Callback URL**: `http://localhost:4007/integrations/social/discord`
- **Dev portal**: https://discord.com/developers/applications → New Application
- **Setup steps**:
  1. In your application, go to **OAuth2** → copy Client ID + Client Secret
  2. Add the callback URL under **Redirects**
  3. Go to **Bot** → Add Bot → copy the Bot Token → this is `DISCORD_BOT_TOKEN_ID`
  4. Enable "Message Content Intent" on the Bot page
- **Scopes**: `identify guilds bot`
- **Notes**: The bot needs to be invited to any server you want to post to. Permission mask: `377957124096`

#### Twitch (identifier: `twitch`)
- **Env vars**: `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/twitch`
- **Dev portal**: https://dev.twitch.tv/console/apps/create
- **OAuth redirect URL**: must match the callback URL exactly
- **Scopes**: `user:write:chat user:read:chat moderator:manage:announcements`
- **Notes**: Twitch integration is **chat-focused**, not video/stream content. You're sending messages to chats, not scheduling streams

#### Kick (identifier: `kick`)
- **Env vars**: `KICK_CLIENT_ID`, `KICK_SECRET`
- **Callback URL**: `http://localhost:4007/integrations/social/kick`
- **Dev portal**: https://kick.com/developer (when available)
- **Scopes**: `chat:write user:read channel:read`
- **Notes**: Uses OAuth 2.0 with PKCE. Similar to Twitch — chat-focused

#### VK (identifier: `vk`)
- **Env vars**: `VK_ID` (just the app ID — no secret)
- **Callback URL**: `http://localhost:4007/integrations/social/vk`
- **Dev portal**: https://id.vk.com/about/business/go
- **Scopes**: `vkid.personal_info email wall status docs photos video`
- **Notes**: Uses OAuth 2.0 with PKCE. VK is primarily for Russian-speaking audiences

#### Whop (identifier: `whop`)
- **Env vars**: `WHOP_CLIENT_ID` (no secret — uses PKCE)
- **Callback URL**: `http://localhost:4007/integrations/social/whop`
- **Dev portal**: https://dev.whop.com/
- **Scopes**: `openid profile email forum:post:create forum:read company:basic:read`

#### MeWe (identifier: `mewe`)
- **Env vars**: `MEWE_APP_ID`, `MEWE_API_KEY`, optionally `MEWE_HOST` (default `https://mewe.com`)
- **Callback URL**: `http://localhost:4007/integrations/social/mewe`
- **Dev portal**: Contact MeWe directly — they don't have a public developer signup

---

### C. Special cases

#### Mastodon (identifier: `mastodon`, `mastodon-custom`)
Mastodon is federated, so there are two flavors:

**Default Mastodon (`mastodon`)** — uses a fixed instance (defaults to `mastodon.social`)
- **Env vars**: `MASTODON_CLIENT_ID`, `MASTODON_URL` (default `https://mastodon.social`)
- To get a Client ID: register an application at https://mastodon.social/settings/applications (or wherever your `MASTODON_URL` points)
- Redirect URI: `http://localhost:4007/integrations/social/mastodon`
- Scopes: `write:statuses profile write:media`

**Mastodon Custom (`mastodon-custom`)** — user picks any instance at connect time
- **No env vars needed**. The provider dynamically calls `POST /api/v1/apps` on whatever instance the user enters to register itself on-the-fly.
- You just set up your Postiz `FRONTEND_URL` correctly (already done) and connect via the UI

#### Telegram (identifier: `telegram`)
- **Env vars**: `TELEGRAM_TOKEN` (single bot token)
- **Callback URL**: Not used — bot polling
- **Setup**:
  1. Open Telegram → chat with [@BotFather](https://t.me/BotFather)
  2. Send `/newbot`, follow prompts
  3. BotFather gives you a bot token — paste into `TELEGRAM_TOKEN`
  4. In Postiz, connecting a channel means sending a command like `/connect <word>` to your bot from the Telegram channel/chat where you want to post
  5. The bot must be an admin of the channel for it to post there

#### Farcaster / Warpcast (identifier: `wrapcast`)
- **Env vars**: `NEYNAR_SECRET_KEY`, `NEYNAR_CLIENT_ID`
- Get these from https://neynar.com/ (the third-party Farcaster API provider used by Postiz)
- Uses Neynar's "signer" flow rather than standard OAuth

#### GitHub (login, not a channel)
- **Env vars**: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`
- This is not a social channel for posting — it's an OAuth login provider for signing into Postiz itself (like "Sign in with Google")
- Set up at https://github.com/settings/developers → OAuth Apps → New OAuth App
- Callback: `http://localhost:4007/auth/activate` (or wherever the auth provider sends it — check source if needed)
- Only needed if you want "Sign in with GitHub" to work on the login page

#### Beehiiv (newsletter subscription, not a channel)
- **Env vars**: `BEEHIIVE_API_KEY`, `BEEHIIVE_PUBLICATION_ID`
- This is **not** a social channel. It's used to automatically subscribe newly-registered Postiz users to a Beehiiv newsletter
- Optional and skippable for self-hosting
- Leave blank unless you actually run a Beehiiv newsletter and want signup-triggered subscriptions

---

## Part 4: Common gotchas

- **Callback URL must match EXACTLY.** `http://localhost:4007/integrations/social/x` is not the same as `http://localhost:4007/integrations/social/x/` (trailing slash) or `https://...` (wrong protocol). Copy-paste carefully.

- **App permissions vs keys:** On X (and some others), changing app permissions after you've already generated the Consumer Keys does NOT update the existing keys. You must **regenerate** them.

- **HTTPS requirement:** Some platforms (Slack, TikTok, Threads) historically require HTTPS callback URLs. Postiz auto-wraps these via `https://redirectmeto.com/http://localhost:4007/...` as a workaround for local dev. This is normal.

- **App review / approval:** Several platforms (TikTok, Facebook Pages posting, LinkedIn Marketing, Pinterest production) gate the posting APIs behind manual review. For local testing, you can usually post to **your own** account/page/account without approval — app review only matters if you want *other* users to use your Postiz instance.

- **Localhost IPs and mobile OAuth:** If your dev portal rejects `localhost` in the callback, try `http://127.0.0.1:4007/integrations/social/<id>` and update `FRONTEND_URL` and `NEXT_PUBLIC_BACKEND_URL` in `docker-compose.yaml` to match.

- **Rate limits:** X and Reddit have aggressive rate limits. The codebase sets `maxConcurrentJob = 1` for X to stay under limits. Don't be surprised if batch scheduling is slow.

- **Changes to `docker-compose.yaml` need `docker compose up -d` to apply.** The running container won't pick up edits otherwise. (Data is preserved — it only restarts the containers whose env changed.)

- **Check Temporal UI if a scheduled post doesn't fire.** http://localhost:8080 → find the workflow → you'll see the error directly instead of guessing.

---

## Channel identifier reference

Use this to construct the exact callback URL for any channel. Format: `http://localhost:4007/integrations/social/<identifier>`

| Channel | Identifier | Env vars required |
|---|---|---|
| X (Twitter) | `x` | `X_API_KEY`, `X_API_SECRET` |
| LinkedIn (profile) | `linkedin` | `LINKEDIN_CLIENT_ID`, `LINKEDIN_CLIENT_SECRET` |
| LinkedIn (page) | `linkedin-page` | `LINKEDIN_CLIENT_ID`, `LINKEDIN_CLIENT_SECRET` |
| Facebook | `facebook` | `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET` |
| Instagram (Business) | `instagram` | `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET` |
| Instagram (Standalone) | `instagram-standalone` | `INSTAGRAM_APP_ID`, `INSTAGRAM_APP_SECRET` |
| Threads | `threads` | `THREADS_APP_ID`, `THREADS_APP_SECRET` |
| YouTube | `youtube` | `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET` |
| Google My Business | `gmb` | `GOOGLE_GMB_CLIENT_ID/SECRET` (or YouTube fallback) |
| TikTok | `tiktok` | `TIKTOK_CLIENT_ID`, `TIKTOK_CLIENT_SECRET` |
| Pinterest | `pinterest` | `PINTEREST_CLIENT_ID`, `PINTEREST_CLIENT_SECRET` |
| Reddit | `reddit` | `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET` |
| Discord | `discord` | `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_BOT_TOKEN_ID` |
| Slack | `slack` | `SLACK_ID`, `SLACK_SECRET` |
| Dribbble | `dribbble` | `DRIBBBLE_CLIENT_ID`, `DRIBBBLE_CLIENT_SECRET` |
| Twitch | `twitch` | `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET` |
| Kick | `kick` | `KICK_CLIENT_ID`, `KICK_SECRET` |
| VK | `vk` | `VK_ID` |
| Whop | `whop` | `WHOP_CLIENT_ID` |
| MeWe | `mewe` | `MEWE_APP_ID`, `MEWE_API_KEY` |
| Mastodon (fixed instance) | `mastodon` | `MASTODON_CLIENT_ID`, `MASTODON_URL` |
| Mastodon (any instance) | `mastodon-custom` | none — dynamic registration |
| Telegram | `telegram` | `TELEGRAM_TOKEN` |
| Farcaster | `wrapcast` | `NEYNAR_SECRET_KEY`, `NEYNAR_CLIENT_ID` |
| Dev.to | `devto` | none — API key in UI |
| Hashnode | `hashnode` | none — API key in UI |
| Medium | `medium` | none — API key in UI |
| WordPress | `wordpress` | none — credentials in UI |
| Bluesky | `bluesky` | none — credentials in UI |
| Lemmy | `lemmy` | none — credentials in UI |
| Nostr | `nostr` | none — hex private key in UI |
| ListMonk | `listmonk` | none — credentials in UI |
| Moltbook | `moltbook` | none — API key in UI |
| Skool | `skool` | none — requires Postiz Chrome extension |

---

## Recommended order for testing

1. **Bluesky** — fastest, zero-config, 10 seconds to set up an app password
2. **Mastodon** (any instance) — also fast, dynamic registration, no env vars
3. **X** — follow Part 1 above, uses OAuth 1.0a (odd one out)
4. **LinkedIn** — standard OAuth 2.0, good reference for the common pattern
5. Everything else — same pattern as LinkedIn, just different portal

If the goal is just to verify the scheduling pipeline end-to-end, start with Bluesky. You'll have a test post scheduled within 5 minutes of finishing this guide.

---

# Part 5: Architecture crash course

Quick primer on the tech stack so you can navigate the code.

## The 30,000 foot view

```
         ┌─────────────────────┐
Browser ─┤  Frontend (Next.js) │
         └──────────┬──────────┘
                    │ HTTP (/api/*)
                    ▼
         ┌─────────────────────┐       ┌──────────┐
         │  Backend (NestJS)   │◄─────►│ Postgres │  (users, orgs, posts, channels)
         │  REST API           │       └──────────┘
         └──────┬──────────┬───┘
                │          │
                │          ▼
                │    ┌──────────┐
                │    │  Redis   │  (cache, rate limits, queues)
                │    └──────────┘
                ▼
         ┌─────────────────────┐       ┌──────────┐
         │  Temporal Server    │◄─────►│ Temporal │  (workflow state)
         │  (external service) │       │ Postgres │
         └──────────┬──────────┘       └──────────┘
                    │ task queue
                    ▼
         ┌─────────────────────┐
         │ Orchestrator        │
         │ (NestJS + Temporal  │
         │  worker)            │
         │ - runs workflows    │
         │ - calls X/LinkedIn/ │
         │   etc. APIs at the  │
         │   scheduled time    │
         └─────────────────────┘
```

Three processes run inside the single `postiz` container (bundled by `pm2` — a Node process manager):
1. **backend** — the NestJS HTTP API. Handles login, CRUD, creates scheduled posts.
2. **frontend** — Next.js server. Serves the UI, proxies `/api` requests to the backend.
3. **orchestrator** — a NestJS app that runs Temporal **workers** — when a scheduled post's time comes, this is what actually calls Twitter/LinkedIn/etc.

The `nginx` inside the container is just a reverse proxy that routes `/` → frontend, `/api` → backend.

## Monorepo layout

```
postiz-app/
├── apps/
│   ├── backend/         NestJS API (HTTP). Mostly thin controllers.
│   ├── frontend/        Next.js 16 app. Pages, components, UI.
│   ├── orchestrator/    NestJS app that hosts Temporal workers (no HTTP, just workers).
│   ├── commands/        CLI commands (nestjs-command) for one-off tasks
│   ├── extension/       Chrome extension (for Skool cookie capture) — Vite-built
│   └── sdk/             Externally published `@postiz/sdk` npm package (not used internally)
│
└── libraries/
    ├── helpers/                 Pure utility functions, auth helpers, crypto
    ├── nestjs-libraries/        Shared backend logic — Prisma, services,
    │                            social providers, email, AI, payments.
    │                            **Most of the real business logic lives here.**
    └── react-shared-libraries/  Shared React utilities, fetch hooks, components
```

Rule of thumb from `CLAUDE.md`:
- `apps/backend` = **controllers only** (thin HTTP layer)
- Real logic lives in `libraries/nestjs-libraries/src/services/*.service.ts` and `.../database/prisma/*/*.repository.ts`
- Pattern: **Controller → (Manager) → Service → Repository → Prisma → DB**

This is the NestJS "three-layer" pattern. Don't take shortcuts skipping layers.

## NestJS crash course

NestJS is a Node.js framework heavily inspired by Angular. If you know Spring or .NET, it'll feel familiar. Core ideas:

### 1. Dependency injection via constructors

Classes declare what they need, and NestJS injects it:

```ts
@Injectable()
export class AuthService {
  constructor(
    private _userService: UsersService,           // injected
    private _organizationService: OrganizationService,  // injected
    private _emailService: EmailService,           // injected
  ) {}

  async login(email: string, password: string) { /* ... */ }
}
```

You never manually instantiate services with `new`. NestJS builds a DI container at startup and wires everything up. The `private _foo` in the constructor creates a class property and injects it in one line.

### 2. Decorators declare what things are

```ts
@Injectable()                    // "this is a service, DI manages it"
@Controller('auth')              // "this is an HTTP controller rooted at /auth"
@Module({...})                   // "this is a NestJS module (a bundle)"
@Cron('0 * * * *')               // "run this method every hour" (from @nestjs/schedule)
@Post('register')                // "handle POST /register"
@Body() body: CreateUserDto      // "parse the request body into this DTO"
```

If you see a `@something` above a class or method, it's metadata that tells Nest what to do. Most of them come from `@nestjs/common` or a sub-package.

### 3. Modules tie it all together

Every bundle of related code is a `@Module`:

```ts
@Module({
  imports: [DatabaseModule],            // other modules this one needs
  controllers: [AuthController],        // HTTP endpoints
  providers: [AuthService, UsersService], // services/injectables
  exports: [AuthService],               // what other modules can use
})
export class AuthModule {}
```

The top-level `apps/backend/src/app.module.ts` imports all the feature modules. When the app boots, NestJS walks this tree and builds the DI graph.

### 4. DTOs with class-validator

Request bodies are typed classes with validation decorators:

```ts
export class CreateOrgUserDto {
  @IsEmail()
  email: string;

  @MinLength(3)
  password: string;
}
```

When you declare a controller method as `@Body() body: CreateOrgUserDto`, NestJS automatically validates the incoming JSON against the decorators. Invalid → 400 response, never reaches your handler.

### 5. Guards & middleware

```ts
@UseGuards(AuthGuard)        // runs before the handler; throws 401 if not logged in
@UseInterceptors(LoggingInterceptor)  // wraps the handler
```

Postiz uses middleware to verify JWTs from cookies (`apps/backend/src/services/auth/auth.middleware.ts`) and inject the current user/org into the request.

## Prisma (the ORM)

Schema lives at `libraries/nestjs-libraries/src/database/prisma/schema.prisma`. Prisma reads that, generates a fully-typed TypeScript client, and you query with it:

```ts
await prisma.user.findFirst({ where: { email } });
await prisma.user.update({ where: { id }, data: { activated: true } });
```

Postiz wraps Prisma with one repository class per table (e.g. `users.repository.ts`, `organizations.repository.ts`). These repositories are the ONLY place where Prisma is called. Services consume repositories; controllers consume services.

Schema changes flow: edit `schema.prisma` → run `pnpm run prisma-db-push` → Prisma regenerates client and pushes schema to Postgres. No manual SQL migrations in this project.

## Temporal (the "why didn't my post go out" part)

**Temporal is a workflow engine.** It's the hardest piece to understand but the most important for this app.

### The problem it solves

A normal approach to "post this tweet at 3pm" would be:
- Save a row `{scheduled_for: 2026-04-15 15:00}`
- Cron job runs every minute: "any rows due now? post them"

This works for 99% of cases but is fragile:
- What if the worker crashes mid-post?
- What if X's API is down — how do you retry without double-posting?
- What if the schedule changes after the job has been queued?
- How do you cleanly cancel a scheduled post?

Temporal solves all of these. You describe your process as a **workflow** — a regular-looking async function — and Temporal:
- Persists every step to its own database
- Survives crashes (resumes from where it left off)
- Handles retries, timeouts, cancellation, versioning
- Gives you a UI to inspect what every workflow is doing (http://localhost:8080)

### Two key concepts

**Workflow** = the "script" (deterministic code that describes what should happen):

```ts
// simplified from apps/orchestrator/src/workflows/autopost.workflow.ts
export async function sendPostWorkflow(postId: string) {
  await sleep('until scheduled time');        // Temporal remembers this
  const result = await postToSocialMedia(postId);  // calls an activity
  if (!result.ok) throw new Error('retry me'); // Temporal auto-retries
  await markPostAsSent(postId);
}
```

Workflow code must be **deterministic** — no `Math.random()`, no `Date.now()`, no direct I/O. All side effects happen inside activities.

**Activity** = the "actual work" (a regular function that can do I/O):

```ts
// apps/orchestrator/src/activities/post.activity.ts
export async function postToSocialMedia(postId: string) {
  const post = await db.getPost(postId);
  const client = new TwitterApi({ /* creds */ });
  return client.v2.tweet(post.content);
}
```

When a workflow calls an activity, Temporal schedules it on a task queue, a worker picks it up, runs it, and reports the result back. If it fails, Temporal retries based on configured policies.

### The parts running in docker-compose

- **`temporal`** container — the Temporal Server itself (receives work, stores state)
- **`temporal-postgresql`** — Temporal's own database (separate from Postiz's DB)
- **`temporal-elasticsearch`** — search index for workflow history
- **`temporal-ui`** — http://localhost:8080 — inspector for workflows
- **`orchestrator` process** (inside the main `postiz` container) — this is the **worker**. It knows how to execute the workflows and activities defined in `apps/orchestrator/src/`

### How Postiz uses it

Workflows in `apps/orchestrator/src/workflows/`:
- `autopost.workflow.ts` — runs a scheduled post
- `send.email.workflow.ts` — sends an email (retry-safe)
- `refresh.token.workflow.ts` — refreshes OAuth tokens before expiry
- `missing.post.workflow.ts`, `digest.email.workflow.ts`, `streak.workflow.ts` — periodic tasks

Activities in `apps/orchestrator/src/activities/` — the actual "call the Twitter API" / "send via Resend" / etc.

When you create a scheduled post in the UI:
1. Backend saves it to Postgres
2. Backend tells Temporal "start a `sendPostWorkflow` for this post ID, scheduled for X"
3. Temporal persists the workflow state
4. At the scheduled time, Temporal dispatches it to an orchestrator worker
5. Worker runs the workflow → calls activities → hits X's API → reports success
6. Workflow marks the post as sent in Postgres

If you reboot your laptop at 2:58pm for a 3:00pm scheduled post:
- Postiz containers go down
- At 3:00pm nothing happens (no worker running)
- You power back on at 3:02pm
- Temporal server comes up, sees an overdue workflow, dispatches it to the new worker
- Post goes out at 3:02pm (or gets skipped depending on policy — check workflow definition)

**This is why "does my laptop need to stay on" is yes** — no workers, no posting.

### NestJS ↔ Temporal glue

Postiz uses `nestjs-temporal-core` (see `package.json`) which is a thin wrapper that lets NestJS services call Temporal workflows via normal DI:

```ts
// in a NestJS service
await this._temporalService.client.getRawClient()?.workflow.signalWithStart(...)
```

## Redis (the fast stuff)

Redis is used for things that don't need to survive forever:

- **Rate limiting** — `@nestjs/throttler` with `@nest-lab/throttler-storage-redis` stores per-user/per-IP request counts in Redis
- **Caching** — various service-level caches
- **Bull/BullMQ queues** — you might see some older queue code that predates Temporal; anything new goes through Temporal

You don't usually need to touch Redis directly. The libraries handle it. `REDIS_URL` in `docker-compose.yaml` points to the `postiz-redis` container.

Inspect with **RedisInsight** at http://localhost:5540 if you need to see what's in it.

## Postgres (Postiz's main DB)

One Postgres container (`postiz-postgres`) holds everything the app cares about: users, organizations, posts, channels, integrations, media references, analytics, etc. Schema is defined in `libraries/nestjs-libraries/src/database/prisma/schema.prisma`.

Inspect with **pgAdmin** at http://localhost:8081 (login: `admin@admin.com` / `admin`), or `psql` directly:

```bash
docker exec -it postiz-postgres psql -U postiz-user -d postiz-db-local
```

Note: the Temporal containers have their own **separate** Postgres (`temporal-postgresql`). Don't confuse the two. Postiz never touches the Temporal DB directly; Temporal Server owns it.

## Frontend (Next.js + React)

The frontend is **Next.js 16 with the App Router** (the file-based routing system where folders under `app/` become URL segments). It's a React app with server-side rendering.

### File layout

```
apps/frontend/src/
├── app/                    Next.js app router — file-based routing
│   ├── (app)/              Route group: authenticated pages
│   ├── auth/               /auth/login, /auth/register, etc.
│   └── layout.tsx          Root layout (wraps everything)
│
└── components/             Reusable React components
    ├── ui/                 Base design-system components (buttons, inputs, etc.)
    ├── auth/               Login/register/google-provider components
    ├── launches/           The post composer, calendar, scheduling UI
    ├── new-launch/providers/  Per-channel posting UI (one folder per channel)
    └── layout/             Top menu, sidebars, shell
```

### Data fetching: SWR

The rule from `CLAUDE.md`: **always use SWR, one hook per useSWR call, never inline multiple useSWR calls inside a returned object**.

```ts
// ✅ correct
const useCommunities = () => {
  return useSWR<CommunitiesListResponse>('communities', getCommunities);
};

// ❌ wrong (breaks rules-of-hooks)
const useCommunity = () => ({
  communities: () => useSWR(...),
  providers: () => useSWR(...),
});
```

SWR caches responses, deduplicates, and auto-revalidates on focus. Mutations use `mutate()` to invalidate the cache.

### Fetch helper

All API calls go through `useFetch` from `libraries/react-shared-libraries/src/utils/custom.fetch.tsx`. Don't use raw `fetch()` — the helper handles auth cookies, base URLs, and errors consistently.

### Styling

Tailwind CSS 3 (note: package.json also has Tailwind 4 bits for some tooling, but the main config is v3). Colors come from:
- `apps/frontend/src/app/colors.scss`
- `apps/frontend/src/app/global.scss`
- `apps/frontend/tailwind.config.js`

The `CLAUDE.md` says "all `--color-custom*` are deprecated — don't use them."

## pnpm workspaces (the monorepo glue)

Postiz uses **pnpm** (fast, disk-efficient npm replacement) with **workspaces** (one monorepo, multiple packages, shared `node_modules`).

Single root `package.json` declares all dependencies. Each `apps/*` and `libraries/*` has its own smaller `package.json` that references the shared ones.

Run things with:
```bash
pnpm dev                # all apps in parallel
pnpm dev:backend        # just backend
pnpm dev:frontend       # just frontend
pnpm dev:orchestrator   # just temporal worker
pnpm build              # build everything
pnpm prisma-db-push     # push Prisma schema to DB
```

The `--filter` flag runs a script in a specific package:
```bash
pnpm --filter ./apps/backend run dev
```

**Never use npm or yarn in this repo.** pnpm's lockfile is authoritative.

## Patterns you'll see constantly

### 1. Three-layer backend (enforced)

```
Controller (HTTP)
    ↓
Service (business logic)
    ↓
Repository (Prisma queries)
    ↓
Postgres
```

Don't call Prisma from a controller. Don't call Prisma from a service either. Always go through the repository.

### 2. Per-provider abstraction

Every social channel implements a common interface:

```ts
// libraries/nestjs-libraries/src/integrations/social/social.integrations.interface.ts
interface SocialProvider {
  identifier: string;
  name: string;
  generateAuthUrl(): Promise<{ url; state; codeVerifier }>;
  authenticate(params): Promise<AuthTokenDetails>;
  post(id, accessToken, postDetails): Promise<PostResponse[]>;
  // ...
}
```

To add a channel: create `my.provider.ts` in `libraries/nestjs-libraries/src/integrations/social/` implementing this interface, register it somewhere, and the rest of the app (UI, scheduling, posting) just works.

### 3. Decorator-based plugins

Providers use `@Plug`, `@PostPlug`, `@Rules` decorators to register scheduled actions / rules that the UI surfaces automatically:

```ts
@Plug({
  identifier: 'x-autoRepostPost',
  title: 'Auto Repost Posts',
  runEveryMilliseconds: 21600000,
  // ...
})
async autoRepostPost(integration, id, fields) { /* ... */ }
```

These get picked up at startup and appear in the UI without manual wiring.

### 4. Env-driven feature gating

Almost every optional feature is gated on an env var:

```ts
if (!process.env.RESEND_API_KEY) { /* fallback */ }
disabled: !!process.env.DISABLE_X_ANALYTICS,
```

If a feature isn't working, first check whether it's gated on an env var you haven't set.

### 5. Separation of auth providers vs social providers

Two folders, easy to confuse:
- `apps/backend/src/services/auth/providers/` — "Sign in with X" providers (Google, GitHub, generic OAuth). For **logging into Postiz itself**.
- `libraries/nestjs-libraries/src/integrations/social/` — "Post to X" providers. For **the actual social channels you schedule posts to**.

## TL;DR for reading the code

1. **Start in `apps/backend/src/app.module.ts`** — that's the top of the dependency tree. Follow imports to find feature modules.
2. **For business logic, jump to `libraries/nestjs-libraries/src/`** — that's where the interesting stuff lives.
3. **For "what happens when a post fires" → `apps/orchestrator/src/workflows/autopost.workflow.ts`** — follow it into activities → into the social provider classes.
4. **For frontend pages → `apps/frontend/src/app/`** — file-based routes.
5. **For frontend components → `apps/frontend/src/components/`** — look in the subfolder named after the feature.
6. **For Prisma/DB → `libraries/nestjs-libraries/src/database/prisma/schema.prisma`** plus the repositories next to it.

If you're stuck finding where something lives, grep for a unique string from the UI (like a form label) and work backwards from the component to the API call to the controller to the service to the repository.
