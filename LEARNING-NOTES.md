# Learning notes

Terms from Brad Traversy's CI/CD crash course, in plain English. Words in **bold** are defined in `GLOSSARY.md`.

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
