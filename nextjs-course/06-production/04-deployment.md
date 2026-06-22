# Deployment

So far your app has lived on your own computer at `http://localhost:3000`. Only
you can see it. **Deployment** is the act of putting your app on the internet so
the whole world can visit it at a real address.

Think of it like opening a shop. Until now you have been building furniture in
your garage. Deployment is moving everything into a storefront on a busy street,
unlocking the door, and putting up the sign. This lesson walks through that move
step by step.

## What You'll Learn

- How to build a production version of your app with `npm run build`
- How to run that production build locally with `npm start`
- How to deploy to Vercel in a few clicks (the easiest path)
- How preview deployments give every change its own test URL
- How to set environment variables on your host
- How to self-host with plain Node or Docker (briefly)
- What `output: "standalone"` does and when you need it
- How to read and sanity-check your build output

## Development vs Production Builds

While building, you run `npm run dev`. That mode is friendly for development: it
reloads on every save and shows detailed errors, but it is **not optimized for
speed**.

For real users you create a **production build** — a compressed, optimized version
of your app.

```bash
# Create the optimized production build
npm run build
```

```bash
# Run that production build locally to test it before deploying
npm start
```

Always run `npm run build` before deploying. If the build fails on your machine,
it will fail on the host too — better to find out now.

## Reading the Build Output

When `npm run build` finishes, it prints a summary of every route. Learning to
read it is a quiet superpower.

```text
Route (app)                     Size     First Load JS
┌ ○ /                           1.2 kB         92 kB
├ ○ /about                      0.5 kB         88 kB
└ ƒ /products/[id]              2.1 kB         95 kB

○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
```

- **Static (`○`)** pages are built once and served instantly to everyone — the
  fastest kind.
- **Dynamic (`ƒ`)** pages are rendered per request, usually because they read
  cookies, headers, or fresh data.
- **First Load JS** is how much JavaScript a visitor downloads. Smaller is faster;
  if a number looks huge, revisit the performance lesson.

## The Easiest Path: Deploy to Vercel

Vercel is the company that makes Next.js, so their hosting fits it like a glove.
For most people this is the smoothest way to ship.

### Step by step

1. **Push your code to GitHub** (or GitLab/Bitbucket). Make sure your project is
   in a Git repository and pushed to the remote.

```bash
git add .
git commit -m "Ready to deploy"
git push
```

2. **Create a Vercel account** at vercel.com and sign in with your Git provider.

3. **Import your repository.** Click "Add New Project", pick your repo, and Vercel
   auto-detects that it is a Next.js app — you usually do not change any settings.

4. **Add environment variables** (see the next section) if your app needs them.

5. **Click Deploy.** Vercel runs `npm run build` on its own servers and, a minute
   later, gives you a live URL like `https://your-app.vercel.app`.

### Auto-deploy on every push

After that first deploy, Vercel watches your Git repo. Every time you push to your
main branch, it rebuilds and redeploys automatically. You ship by pushing — no
manual upload, ever.

### Preview deployments

When you push to a **branch** or open a pull request, Vercel builds a separate
**preview deployment** with its own URL. This lets you (and teammates) click
around the change before it reaches your real users.

```text
push to main      ->  production:  https://your-app.vercel.app
push to a branch  ->  preview:     https://your-app-git-feature-x.vercel.app
```

## Environment Variables on the Host

Your app probably uses secrets like a database URL or an API key. You keep these
out of your code in a `.env.local` file — but that file is not pushed to Git, so
the host has never seen it. You must set the same variables in your host's
dashboard.

On Vercel: Project → Settings → Environment Variables. Add each key and value, the
same names you used locally.

```text
DATABASE_URL = postgres://...
STRIPE_SECRET_KEY = sk_live_...
```

Remember the rule from earlier: a variable is only visible in the browser if its
name starts with `NEXT_PUBLIC_`. Everything else stays server-only — which is
exactly what you want for secrets.

```ts
// lib/db.ts
// Read on the server only; never exposed to the browser.
const url = process.env.DATABASE_URL;
```

After adding or changing variables, redeploy so the new values take effect.

## Other Hosts and Self-Hosting

Vercel is the easy path, but Next.js is a normal Node app and can run almost
anywhere — Netlify, Render, AWS, or your own server.

### Self-hosting with Node

Any server with Node installed can run your app:

```bash
npm run build   # build it
npm start       # start the production server on port 3000
```

You would then put something like Nginx in front of it and point your domain at
the server. More moving parts than Vercel, but full control.

### Self-hosting with Docker

Docker packages your app and everything it needs into a single image that runs
the same everywhere. A minimal Dockerfile looks like this:

```text
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

```bash
# Build the image and run it
docker build -t my-next-app .
docker run -p 3000:3000 my-next-app
```

### `output: "standalone"`

For Docker and other self-hosting, a normal build expects all of `node_modules`
to be present, which makes the image large. Turning on **standalone output**
makes Next.js copy only the files actually needed into a small, self-contained
folder.

```ts
// next.config.ts
const nextConfig = {
  output: "standalone", // produces a slim, self-contained server
};

export default nextConfig;
```

After building, you get a `.next/standalone` folder you can copy into a tiny
Docker image and run with `node server.js`. This keeps your container small and
fast to start.

## Common Mistakes

- **Deploying without running `npm run build` first.** If the build breaks
  locally, it breaks on the host. Test the build before you push.
- **Forgetting environment variables on the host.** Your `.env.local` is not in
  Git, so the host needs the variables set in its dashboard.
- **Putting a secret in a `NEXT_PUBLIC_` variable.** That name ships the value to
  the browser. Only public values belong there.
- **Committing `.env.local` to Git.** Keep it in `.gitignore` so secrets never
  leave your machine.
- **Ignoring the build output.** A page marked dynamic when you expected static,
  or a huge First Load JS number, is a useful warning.

## Exercises

1. Run `npm run build` and then `npm start`. Visit `http://localhost:3000` and
   confirm the production build works.
2. Read your build output and list which routes are static (`○`) and which are
   dynamic (`ƒ`). Pick one dynamic route and guess why.
3. Push your project to GitHub, import it into Vercel, and deploy it to a live
   URL. Visit the URL from your phone.
4. Add an environment variable in the Vercel dashboard, redeploy, and read it on a
   server page with `process.env`.
5. Add `output: "standalone"` to your config, run `npm run build`, and confirm a
   `.next/standalone` folder appears.

## Key Takeaways

- `npm run build` makes the optimized production build; `npm start` runs it
  locally so you can test before deploying.
- The build output tells you which routes are static or dynamic and how much
  JavaScript each ships.
- Vercel is the easiest host: connect your Git repo, deploy, and it auto-deploys
  on every push with preview URLs for branches.
- Set environment variables in the host's dashboard; only `NEXT_PUBLIC_` ones
  reach the browser.
- You can self-host with Node or Docker, and `output: "standalone"` produces a
  slim, self-contained server for containers.

---

[← Previous: Testing Your Next.js App](03-testing.md) | [Back to Course Index](../README.md) | [Next: Best Practices and Project Structure →](05-best-practices.md)
