# CI/CD Lab

A tiny Express app used to practise a CI/CD pipeline by following Brad Traversy's CI/CD crash course, with GitHub Actions running the checks and Vercel in place of Render for deployments.

## Checks

**Verify**:
The single combined check (type check, tests, production build) that must pass locally, on every pull request into `main`, and on every push to `main`.
_Avoid_: "the tests", "CI" as the name of the check

## Guardrails

**Merge gate**:
The GitHub rule that refuses to merge a pull request into `main` until **Verify** passes on it.
_Avoid_: GitHub layer, branch protection (as a name for this gate)

**Promotion gate**:
The Vercel rule that holds a new **Production deployment** back from the **Production URL** until **Verify** passes on its commit.
_Avoid_: Render layer, deployment layer, deploy gate

**Guardrails**:
The **Merge gate** and the **Promotion gate** together.
_Avoid_: using "guardrail" for just one of the two gates

## Deployments

**Deployment**:
One build of one commit, served at its own unique URL.
_Avoid_: "deploy" as a noun meaning "going live"

**Production deployment**:
A **Deployment** of a commit on `main`; a candidate for **Promotion**.

**Preview deployment**:
A **Deployment** of a pull request's branch, for trying a change before it reaches `main`.
_Avoid_: preview environment

**Promotion**:
Making a **Production deployment** the one the **Production URL** serves.
_Avoid_: "deploy" or "release" for this step

**Production URL**:
The project's main address, which always serves the most recently promoted **Production deployment**.
_Avoid_: custom domain, live site
