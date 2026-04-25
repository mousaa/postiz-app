This project is Postiz, a tool to schedule social media and chat posts to 28+ channels.
You can add posts to the calendar, they will be added into a workflow and posted at the right time.
You can find things like:
- Schedule posts
- Calendar view
- Analytics
- Team management
- Media library

This project is a monorepo with a root only package.json of dependencies.
Made with PNPM.
We have 3 important folders

- apps/backend - this is where the API code is (NESTJS)
- apps/orchestrator - this is temporal, it's for background jobs (NESTJS) it contains all the workflows and activities
- apps/frontend - this is the code of the frontend (Vite ReactJS)
- /libraries contains a lot of services shared between backend and orchestrator and frontend components.

We are using only pnpm, don't use any other dependency manager.
Never install frontend components from npmjs, focus on writing native components.

The project uses tailwind 3, before writing any component look at:
- /apps/frontend/src/app/colors.scss
- /apps/frontend/src/app/global.scss
- /apps/frontend/tailwind.config.js

All the --color-custom* are deprecated, don't use them.

And check other components in the system before to get the right design.

When working on the backend we need to pass the 3 layers:
Controller >> Service >> Repository (no shortcuts)
In some cases we will have
Controller >> Mananger >> Service >> Repository.

Most of the server logic should be inside of libs/server.
The backend repository is mostly used to write controller, and import files from libs.server.

For the frontend follow this:
- Many of the UI components lives in /apps/frontend/src/components/ui
- Routing is in /apps/frontend/src/app
- Components are in /apps/frontend/src/components
- always use SWR to fetch stuff, and use "useFetch" hook from /libraries/helpers/src/utils/custom.fetch.tsx

When using SWR, each one have to be in a seperate hook and must comply with react-hooks/rules-of-hooks, never put eslint-disable-next-line on it.

It means that this is valid:
const useCommunity = () => {
   return useSWR....
}

This is not valid:
const useCommunity = () => {
  return {
    communities: () => useSWR<CommunitiesListResponse>("communities", getCommunities),
    providers: () => useSWR<ProvidersListResponse>("providers", getProviders),
  };
}

- Linting of the project can run only from the root.
- Use only pnpm.

# Docker / Local Setup

We run from a locally built image (not the upstream prebuilt one) so code changes are reflected.

To apply backend or frontend code changes:
```bash
docker compose build postiz && docker compose up -d --force-recreate postiz
```

Secrets and credentials live in `.env` (gitignored). docker-compose.yaml references them via `${VAR}` syntax. Never hardcode secrets in docker-compose.yaml.

For X (Twitter) OAuth to work locally, ngrok is required since X blocks localhost callback URLs:
- `NGROK_URL` in `.env` sets the allowed CORS origin and X callback base URL
- Run ngrok with: `ngrok http --url=crevice-job-etching.ngrok-free.dev 4007`
- The app itself is used via `localhost:4007`. Ngrok only handles the X OAuth callback redirect.

If Temporal crashes on startup, restart it then restart postiz:
```bash
docker compose restart temporal && docker compose restart postiz
```