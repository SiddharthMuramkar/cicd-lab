# Learning notes

Terms from Brad Traversy's CI/CD crash course, in plain English.

## Ticket 1: Baseline and first Vercel deploy

**CI (continuous integration)**: merging everyone's changes into the shared code often, and having a machine check each change automatically (type check, tests, build). In this project that check is **Verify**.

**CD (continuous delivery vs continuous deployment)**: both mean every change that passes the checks is ready to go live. Continuous *delivery* still waits for a human to approve the release; continuous *deployment* releases automatically once the checks pass. This project uses continuous deployment.

**Production**: the real, live version of the app that users see, as opposed to your laptop or a test copy.

**Dev server**: a copy of the app running on your laptop (`npm run dev`) that restarts by itself when you save a file. Stop it with Ctrl+C.

**Health endpoint**: a tiny address (`/api/health`) that only answers "I'm running" (`{"status":"ok"}`), so people and tools can check that the app is alive.

**npm script**: a named shortcut in the `scripts` section of `package.json`. `npm run verify` runs whatever command is saved under `verify`.

**Verify (`npm run verify`)**: one command that runs three checks in order: the type check (`tsc --noEmit`, are the types consistent?), the tests (does `/api/health` answer correctly?), and the build (`tsc`, can the app compile into `dist`?). `&&` means "only go on if the last one passed", so the first failure stops it.

**Repository (repo)**: a project folder that Git tracks. The hidden `.git` folder inside it holds the whole history and the links to GitHub. `rm -rf .git` deletes that memory but leaves your files alone; `git init` creates a fresh, empty `.git`.

**Commit**: a saved snapshot of the project, with a message saying what changed. The history is a chain of commits, and a branch such as `main` is a label pointing at the latest one.

**Remote**: a copy of the repo that lives somewhere else, usually on GitHub. Your local repo knows each remote by a nickname and a URL; `git remote -v` lists them.

**origin and upstream**: the usual nicknames for two remotes. `origin` is your own repo on GitHub, where you push. `upstream` is the repo you started from (here, Brad's starter), which you only read from.

**git fetch and git reset**: `git fetch` downloads commits from a remote without touching your files. `git reset <commit>` moves your branch label to that commit and also leaves your files alone; adding `--hard` would overwrite them.

**Push**: sending your new commits from your laptop up to a remote (`git push`). VS Code's **Sync Changes** does the same.

**Pre-push hook**: a small script Git runs automatically before every push. Yours blocks pushes that change `main` on GitHub, so work goes through pull requests. `git push --no-verify` skips hooks for one command.

**Pull request (PR)**: a request to merge a branch into `main`, which can be reviewed and checked before it lands. Coming in ticket 2.

**Environment variable**: a named setting (like `VERCEL_EXPERIMENTAL_BACKENDS=1`) that a platform hands to your build or app without it being written in the code. Each one can be switched on per environment: Production, Preview, Development.

**Build Command**: the command Vercel runs to build your app. Left empty, Vercel uses `package.json`'s `build` script and adds its own steps; set explicitly (we use `npm run build`), it trusts your command. We set it in ticket 1 while working around TypeScript 7; it's harmless now that we're on TypeScript 6 (see "Why we dropped it" below).

**Deployment**: one build of one commit, served at its own unique URL (like `cicd-cme8nxrgd-acme-ef4b.vercel.app`). Every push or redeploy makes a new one; old ones stay in the list.

**Production deployment**: a **Deployment** of a commit on `main`. Our first one is `fe056c6` (the two failed builds were Production deployments too, they just never went live).

**Production URL**: the project's main address (`cicd-lab-vert.vercel.app`), which always serves the most recently promoted **Production deployment**. Its `-vert` suffix is random because `cicd-lab.vercel.app` was taken.

### Quick check gaps (ticket 1)

- **Verify's three checks**: type check (`tsc --noEmit`), tests (`npm test`), build (`tsc` into `dist`). It's not a frontend thing: it's the exact command GitHub Actions runs on every PR in ticket 2.
- **Deployment vs Production URL**: a Deployment is one build of one commit, from any branch (only builds of `main` are Production deployments). The Production URL is the free `*.vercel.app` address that always serves the latest live Production deployment; a bought custom domain is a separate, optional extra on top.

**Builder / backends build mode**: a builder is the Vercel program that turns your repo into something it can run. `VERCEL_EXPERIMENTAL_BACKENDS=1` swaps Express's default builder for a newer, opt-in one (better imports, logs filterable by route). Source: https://vercel.com/changelog/experimental-build-mode-hono-express

**Why we dropped it (ticket 2)**: both Vercel builders break on TypeScript 7. The default one fails with `TS2688: Cannot find type definition file for 'node'`, and the experimental one crashes in its own type check (vercel/vercel#17390). The variable was only on for Production, so the first Preview deployment failed. We removed the variable and moved TypeScript from 7 to 6, which the default builder handles.

## Ticket 2: GitHub Actions CI

**GitHub Actions**: GitHub's built-in automation. When something happens in your repo (a push, a pull request), it runs the tasks you wrote down, such as **Verify**. Here it does the CI checks; Vercel still does the building and deploying.

**Runner**: the computer a job runs on. GitHub gives each job a fresh Ubuntu virtual machine (a temporary computer in the cloud) and throws it away afterwards, so the checks prove the app works on a clean machine, not just on your laptop.

**Workflow**: a YAML file in `.github/workflows/` that tells GitHub Actions *when* to run (for us: a pull request into `main`, or a push to `main`) and *what* to do (check out the code, set up Node 24, `npm ci`, `npm run verify`).

**Branch**: a separate line of work. Creating one adds a new label at your current commit; new commits move only that label, so `main` stays untouched until you merge. Uncommitted changes don't belong to any branch yet, so they come along to a branch you create.

**YAML**: a plain-text settings format. `key: value` sets something, a leading `-` starts a list item, and indentation (spaces, never tabs) shows what belongs inside what.

**Trigger (`on`)**: the events that start a workflow. Ours starts on a push to `main`, and on a pull request into `main` (opened, or updated by a new push).

**Permissions (`contents: read`)**: each run gets a temporary GitHub key. Ours can only read the code, not push, merge or change settings: give a workflow only what it needs.

**Job**: a list of steps that runs on one **runner**. Ours has the ID `verify` and the display name **Verify**, which is the check name on every PR and the name both gates look for in ticket 3. `runs-on` picks the runner (`ubuntu-latest`); `timeout-minutes: 10` stops a stuck job instead of letting it run for GitHub's default of 6 hours.

**Step, `uses` vs `run`**: steps run top to bottom, and the first failure stops the job. `uses` runs a ready-made **action** (a packaged step someone else wrote, like `actions/checkout@v7`, where `@v7` is the version). `run` is a shell command, the same as typing it in your terminal.

**npm cache (`cache: npm`)**: `setup-node` saves npm's downloaded packages after a run (the "Post Set up Node.js" log line) and restores them next time, so `npm ci` doesn't download everything again. Changing `package-lock.json` starts a fresh cache.
