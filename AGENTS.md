# Postiz local setup — agent notes

## Working environment

- Use **Node 22** from `C:\Users\Marti\scoop\apps\nodejs22\22.23.2` for builds/runs.  
  Node 25 fails to start the backend because the Sentry native profiler binary for module version 141 is missing.
- Package manager: **pnpm**.
- Start the backend:
  ```powershell
  $env:PATH = 'C:\Users\Marti\scoop\apps\nodejs22\22.23.2;' + $env:PATH
  pnpm --filter ./apps/backend start
  ```
- Start the frontend:
  ```powershell
  $env:PATH = 'C:\Users\Marti\scoop\apps\nodejs22\22.23.2;' + $env:PATH
  pnpm --filter ./apps/frontend dev
  ```
- The frontend runs on `http://localhost:4200` and the backend on `http://localhost:3000`.
- `.env` is gitignored.  It contains LinkedIn client credentials, DB connection strings, etc.  Never commit or push it.

## Verification after changes

1. Restart the backend (and rebuild if `apps/backend` source changed).
2. Test login at `http://localhost:4200/auth/login`.
3. Test LinkedIn OAuth generation:
   ```
   GET http://localhost:3000/integrations/social/linkedin
   ```
   The returned URL should contain the scopes in `.env` `LINKEDIN_SCOPES`.
4. Test the CopilotKit CORS preflight:
   ```
   OPTIONS http://localhost:3000/copilot/chat
   Origin: http://localhost:4200
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: content-type,x-copilotkit-runtime-client-gql-version
   ```
   Expect `Access-Control-Allow-Credentials: true` and `Access-Control-Allow-Origin: http://localhost:4200`.

## Upstream repository

- Original code: `https://github.com/gitroomhq/postiz-app.git`
- Local fork: `https://github.com/martinconstantineau/postiz-app-main.git`
- Upstream remote name: `upstream`

## Monthly upstream sync process

The user wants the original Postiz code merged into their fork every month, without breaking their local customizations.

1. **Fetch upstream:**
   ```
   git fetch upstream main
   ```
2. **Create a sync branch:**
   ```
   git checkout -b sync/upstream-YYYY-MM main
   ```
3. **Merge upstream changes:**
   ```
   git merge upstream/main
   ```
4. **Preserve local customizations.**  Files that are commonly modified locally:
   - `.env` (gitignored — will not be touched)
   - `.gitignore`
   - `libraries/nestjs-libraries/src/integrations/social/linkedin.provider.ts`
   - `apps/backend/src/main.ts`
   Resolve conflicts in favor of the local customizations unless the upstream change is a security or required bug fix.
5. **Install/build/test:**
   ```powershell
   $env:PATH = 'C:\Users\Marti\scoop\apps\nodejs22\22.23.2;' + $env:PATH
   pnpm install
   pnpm --filter ./apps/backend build
   pnpm --filter ./apps/frontend build
   pnpm test
   ```
6. **Start backend + frontend and verify:**
   - login
   - LinkedIn OAuth URL uses `LINKEDIN_SCOPES`
   - CopilotKit CORS preflight works
7. **Only fast-forward `main` after verification.**
8. **Push the updated `main`.**

Important: no automated process can guarantee zero breakage. The only way to keep the fork safe is to run the build, tests, and manual checks above before merging to `main`.
