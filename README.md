# Lakebase Workshop

A hands-on workshop for exploring **Databricks Lakebase** -- a fully managed PostgreSQL database that runs inside your Databricks workspace.


## Quick Start

### Step 1: Clone the repo and run setup

Open a terminal and run:

```bash
git clone <this-repo-url>
cd Lakebase-Workshop
bash setup.sh
```

The setup script will walk you through everything: installing dependencies, connecting to your Databricks workspace, and deploying the workshop content. Just follow the prompts.


##### Deploy the Lab Console (facilitator only)

The Lab Console is a **shared app** — the facilitator deploys it once and all
participants use it. During `setup.sh`, choose **option 2 (Labs + App)** to deploy
everything including the app.

The setup script will also:
- **Ensure your Lakebase project exists** — if you haven't already run the setup
  notebook, it auto-creates the project + seed schema (~2–3 min). If that fails,
  it prints the path to `00_Setup_Lakebase_Project` so you can run it and re-run
  `setup.sh`.
- **Attach a postgres resource** to the app (your Lakebase project), giving the SP the `postgres` OAuth scope
- **Grant the SP access** to your schema so the app can connect to your project

To deploy manually:

```bash
# Build the frontend
cd apps/lakebase-lab-console/frontend
npm install && npm run build
cd ../../..

# Deploy the bundle + start the app
databricks bundle deploy --target dev --profile lakebase-workshop
databricks bundle run lakebase_lab_console --target dev --profile lakebase-workshop
```

The app is named `lakebase-lab-console` and is accessible to all workspace users
at **Compute → Apps → lakebase-lab-console**.

> **The frontend build is not optional.** `frontend/dist/` is gitignored, so a fresh
> clone contains no UI. Skipping `npm run build` (or deploying from a machine without
> Node.js) produces an app that starts and reports healthy but serves no interface.
> Confirm the build landed with `curl <app-url>/api/health` — `frontend_built` must be
> `true`.

> **Per-user SP grant required.** Each participant must grant the app's Service
> Principal access to their Lakebase project. This happens automatically in Step 6
> of `00_Setup_Lakebase_Project`, or can be done from the App Deployment lab.


### Step 2: Run the setup notebook

In your Databricks workspace, Each participant opens the **`00_Setup_Lakebase_Project`** notebook and click **Run All**.

This creates your personal Lakebase database (`lakebase-lab-<username>`) and loads sample data, seeds a per-user schema (`lakebase_lab_<username>`), and grants the Lab Console app's SP access to their project It takes about 2-3 minutes. 

You can find the notebook at:
**Workspace > Users > *your email* > .bundle > lakebase-workshop > dev > files > notebooks**

### Step 3: Start exploring

You're all set. Pick any lab from the list below and dive in. Each lab is self-contained -- no need to follow a specific order.


### All Labs Reference

Every lab lives directly in `labs/`. No track folder structure — tracks are a facilitation guide, not a file hierarchy. Almost every folder has one notebook; **`development-experience/`** is the exception (three notebooks). 

| Lab | Location | What It Covers | Time |
|-----|----------|----------------|------|
|**Introduction**
| What is Lakebase?  | Slides + foundation |Architecture overview| 30 mins |
|**Fundamentals**
| Data Operations | `labs/data-operations/` | CRUD, JSONB, arrays, triggers, transactions | 20 mins |
| Development Experience | `labs/development-experience/` | Branching, autoscaling, scale-to-zero, high availability + read replicas | 40 mins |
| Authentication, Security & Compliance | `labs/authentication/` | OAuth tokens, roles, encryption/CMK, Private Link, compliance | 20 mins |
| Backup & Recovery | `labs/backup-recovery/` | Checkpoint branches, snapshots, PITR, restore, cross-region DR note | 20 mins |
| Observability | `labs/observability/` | pg_stat views, index analysis, monitoring | 20 mins |
| Reverse ETL | `labs/reverse-etl/` | Synced tables from Delta Lake | 20 mins |
| Unity Catalog Access | `labs/unity-catalog-access/` | Federated read-only UC catalog over Postgres | 20 mins |
| Lakehouse Sync *(Public Preview)* | `labs/lakehouse-sync/` | Lakebase → Unity Catalog Delta via Lakebase Change Data Feed (UI-configured) | 20 mins |
| Data API | `labs/data-api/` | PostgREST REST access, `authenticator` role, OAuth bearer, RLS | 30 mins |
| **Advanced**
| Agentic Memory | `labs/agentic-memory/` | Persistent agent memory pattern | 30 mins |
| Lakebase Search *(Beta)* | `labs/lakebase-search/` | Vector + BM25 + hybrid RRF search (enablement irreversible) | 30 mins |
| Online Feature Store | `labs/online-feature-store/` | Real-time ML feature serving with Lakebase | 30 mins |
| App Deployment | `labs/app-deployment/` | Full-stack Lab Console app (capstone) | 30 mins |



## Prerequisites

Before you begin, make sure you have:

- **Your own Databricks workspace** with Lakebase enabled
- **Python 3.11+** on your computer (`python3 --version`)
- **Databricks CLI ≥ 0.294.0**, logged in via `databricks auth login` (do this ahead of time)
- **Node.js 18+** — only needed if you deploy the **Lab Console app**. `setup.sh` installs it for you (Homebrew, nvm, or apt) when you choose option 2. You can also install it ahead of time (`node --version`). Not needed for the notebook labs.

Check **[Prerequisites Guide](docs/PREREQUISITES.md)** (GitHub access, bring-your-own workspace, and auth login).

All labs are in the `labs/` folder, organized by topic.

## Lab Console App

If you deployed a shared **Lab Console** web app during setup.sh. This app mirrors all the labs in a visual interface -- you can use it alongside (or instead of) the notebooks.

Open it at: **Compute > Apps > lakebase-lab-console**

The setup notebook (Step 2 above) automatically grants the app access to your database, so it will show your data as soon as you log in.

## Troubleshooting

| Problem | What to Do |
|---------|------------|
| `setup.sh` fails during login | Run `databricks auth login --host <your-workspace-url> --profile lakebase-workshop` manually |
| Setup notebook hangs on "Waiting for endpoint" | This is normal -- it can take 2-3 minutes. Let it finish. |
| "password authentication failed" | Your database token expired (they last 1 hour). Re-run the connection cell in your notebook. |
| Opening the app shows `{"detail":"Not Found"}` | You're on an older deployment whose UI was never built. Re-run `bash setup.sh`, choose **option 2**, and accept the Node.js install when prompted. Current builds show an explanatory page instead. |
| Opening the app shows "the app is running, but its user interface was not deployed" | The app was deployed without a frontend build. Re-run `bash setup.sh` → **option 2** (setup installs Node.js if needed). The notebook labs are unaffected. |
| Lab Console shows "Project Not Found" | You haven't run the setup notebook yet. Go back to Step 2. |
| `function databricks_create_role(...) does not exist` | The `databricks_auth` extension isn't installed in your Postgres database. Run `CREATE EXTENSION IF NOT EXISTS databricks_auth;` once per database — the setup notebook now does this automatically in Step 6. |
| Deploying the app errors with "No endpoints" before running the setup notebook | Re-run `bash setup.sh` and choose **option 2** — the script auto-creates your Lakebase project before deploying the app. If auto-create fails, run `00_Setup_Lakebase_Project` and re-run setup. |


## Cleanup
Participants can delete their projects by uncommenting the cleanup cell at the bottom of the foundation notebook, or by running:
```
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
w.postgres.delete_project(name="projects/lakebase-lab-<username>")

# Delete the app (this also stops it if running)
w.apps.delete(name="lakebase-lab-console")
print("App 'lakebase-lab-console' deleted successfully.")
```

## Resources

**Start here (canonical Lakebase docs):**
- [What is Lakebase?](https://docs.databricks.com/aws/en/oltp/projects/about)
- [Lakebase documentation (hub)](https://docs.databricks.com/aws/en/oltp/projects/)
- [Get started with Lakebase](https://docs.databricks.com/aws/en/oltp/projects/get-started)
- [June 2026 release notes](https://docs.databricks.com/aws/en/release-notes/product/2026/june) — latest launches & changes

**Feature docs (mapped to the labs):**
- [Authentication](https://docs.databricks.com/aws/en/oltp/projects/authentication) · [Roles & permissions](https://docs.databricks.com/aws/en/oltp/projects/roles-permissions) · [Private Link](https://docs.databricks.com/aws/en/oltp/projects/private-link)
- [Branches](https://docs.databricks.com/aws/en/oltp/projects/branches) · [Autoscaling](https://docs.databricks.com/aws/en/oltp/projects/autoscaling) · [Scale to zero](https://docs.databricks.com/aws/en/oltp/projects/scale-to-zero) · [High availability](https://docs.databricks.com/aws/en/oltp/projects/high-availability) · [Read replicas](https://docs.databricks.com/aws/en/oltp/projects/read-replicas)
- [Serve lakehouse data with synced tables (Reverse ETL)](https://docs.databricks.com/aws/en/oltp/projects/sync-tables) · [Register Lakebase in Unity Catalog](https://docs.databricks.com/aws/en/oltp/projects/register-uc) · [Lakebase Change Data Feed (Lakehouse Sync)](https://docs.databricks.com/aws/en/oltp/projects/lakehouse-sync)
- [Point-in-time restore](https://docs.databricks.com/aws/en/oltp/projects/point-in-time-restore) · [Monitor](https://docs.databricks.com/aws/en/oltp/projects/monitor)
- [Data API (PostgREST)](https://docs.databricks.com/aws/en/oltp/projects/data-api) · [Lakebase Search](https://docs.databricks.com/aws/en/oltp/projects/lakebase-search) *(Beta)*
- [Online Feature Store](https://docs.databricks.com/aws/en/machine-learning/feature-store/online-feature-store) · [Connect an application](https://docs.databricks.com/aws/en/oltp/projects/connect-application)

**Reference:**
- [Databricks Apps documentation](https://docs.databricks.com/en/dev-tools/databricks-apps/)
- [PostgreSQL documentation](https://www.postgresql.org/docs/)

## License

See [LICENSE.md](LICENSE.md).
