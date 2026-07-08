The AutoPilot Deploy Pipeline lets you deploy your codebase straight from a git repository onto your AutoPilot deployment.
It builds on our source control integration, so once your account is connected you can either push to a branch and have AutoPilot deploy it for you, or trigger a deploy by hand from the dashboard.
Under the hood, every deploy runs a small pipeline of scripts on your jump host.
We ship a sensible default pipeline, and you can override any part of it by dropping your own scripts into your repository.

This guide walks you through connecting your source control provider, configuring a deployment, triggering deploys, and customizing the pipeline.
We assume you already have a deployment provisioned on JetRails [AutoPilot](https://autopilot.jetrails.com) and a [GitHub](https://github.com) repository containing your project.

## Connect Your Source Control Provider

Right now GitHub is the only source control provider we support, but more are on the way.
The integration lives at the organization level.
Head to your organization's **Integrations** page and open the **GitHub** integration, then click **Install GitHub App**.
This kicks off the standard GitHub install flow where you pick the organization or user account to install on and choose which repositories to grant access to.

Once the app is installed, the integration page shows the connected account along with a list of granted repositories and their default branches.
Any deployment in this organization can now deploy from any of these granted repositories.

## Configure The Repository And Branch

Open the deployment you want to deploy to and go to the **Deploy Pipeline** tab, then the **Deploy Settings** section.
Under **Repository**, pick the repository you want to deploy from and set the **Deploy Branch**.
The branch defaults to the repository's default branch, but you can point it at any branch you want (it doesn't even have to exist yet).

When you save the mapping, AutoPilot generates a dedicated deploy key, stores the private key on your jump host (where we run the deploy process), and registers the public key with your repository on your source control provider.
This is the key we use to clone your repository during a deploy, so you do not have to manage any credentials yourself.

## Choose A Deploy Strategy

Still under **Deploy Settings**, the **Deploy Strategy** setting controls what happens when a commit lands on your deploy branch.
You get two options.

**Automatically Trigger On Push** gives you a GitOps style workflow.
Every push to your deploy branch kicks off a new deploy on its own, so merging a pull request is all it takes to ship.
This is the one you want for most setups where the deploy branch is your source of truth for what should be live.

**Manual Triggers Only** ignores pushes entirely.
Commits still land on the branch, but nothing deploys until you say so.
Reach for this when you want to control exactly when a deploy goes out, for example on a production deployment where you would rather ship on your own schedule than on every merge.
You trigger these deploys yourself from the dashboard, which the next section covers.

## Trigger Manual Deploy

For a manual deploy, go to the **Run History** section and click **Manually Trigger**.
A dialog shows you the exact commit that will be used, pulled from the tip of your deploy branch.
Confirm it and the deploy starts, dropping you on the run details page where you can watch each step run in real time.

For automatic deploys, just push to your deploy branch.
As long as your strategy is set to trigger on push and the integration is not suspended, AutoPilot picks up the webhook and starts a deploy for that commit.

Every run shows up in **Run History** with its trigger type, the pusher, the commit, and its status.
Click into a run to see the individual pipeline steps, expand their logs, download them, or re-run the whole thing.

## How The Pipeline Works

When a deploy fires, AutoPilot clones your repository onto the jump host in a temporary directory, checks out the target commit, and runs a pipeline of scripts against it.
The steps run as your deployment's cluster user, with the working directory set to the root of your checked-out repository.
The default pipeline scripts live on your jump host at `/opt/jrc/confs/steps.d/`.
Out of the box we ship four steps, and what each one actually does depends on the files it finds in your repository:

| Step | What it does |
| ---- | ------------ |
| `10-show-environment-variable` | Prints the deploy environment variables for the run |
| `20-install-dependencies` | Runs `composer install` if a `composer.json` is present |
| `30-deploy-release` | Runs PHP Deployer against the `autopilot` host if a `deploy.php` is present |
| `40-restart-services` | Restarts php-fpm and varnish and flushes redis-cache on the relevant nodes |

Steps run in order based on their numeric prefix, and if any step exits non-zero the deploy stops there and skips everything after it.
Each step name has to match the pattern `^[0-9]+(-[a-zA-Z0-9]+)+$`, so a leading number followed by hyphen-separated words, for example `50-example-step-name`.

## Override The Pipeline

You can extend or replace any part of the pipeline by adding your own scripts to a `.jetrails/steps.d/` folder in the root of your repository.
During a deploy we copy the default steps into this folder without clobbering anything already there.
That means a script in your repository always wins over a default with the same filename.

So there are two things you can do.
Add a new script with a fresh prefix like `50-example-step-name` and it runs alongside the defaults in numeric order.
Or drop in a script named exactly like one of the defaults, say `30-deploy-release`, and your version replaces ours entirely.

You will also want to make sure you `chmod +x` your scripts so they are executable, since we use `run-parts` to run them in sequential order.

!!! Common Mistake
By default, git tracks the executable bit of the file.
If your custom step is being skipped, you will want to make sure your local git is tracking the executable bit.
!!!

## The Deployer Host

If you use PHP Deployer, there is one detail you have to get right.
Our default `30-deploy-release` step deploys against a host named `autopilot`:

```shell
dep deploy --no-ansi --no-interaction autopilot
```

So your `deploy.php` has to define that host, otherwise the step fails.
The pipeline runs on the jump host itself, so `autopilot` is a localhost host and not a remote SSH host:

```php
localhost("autopilot");
```

If you would rather not use Deployer at all, just override the `30-deploy-release` step with your own script and deploy however you like.

## Environment Variables

Each step runs with a set of environment variables describing the deploy, so your scripts can key off the branch, commit, or how the deploy was triggered.

| Variable | Description |
| -------- | ----------- |
| `AUTOPILOT_DEPLOY_BRANCH` | The branch being deployed, for example `master` |
| `AUTOPILOT_DEPLOY_COMMIT_MESSAGE` | The first line of the commit message |
| `AUTOPILOT_DEPLOY_DELIVERY` | A unique ID for this deploy run |
| `AUTOPILOT_DEPLOY_DEPLOYMENT_ID` | The ID of the deployment being deployed to |
| `AUTOPILOT_DEPLOY_ENV_FILE` | Path to the file holding these environment variables (deleted after the run) |
| `AUTOPILOT_DEPLOY_FORCED` | Whether the push was a force push, either `true` or `false` |
| `AUTOPILOT_DEPLOY_KEY_FILE` | Path to the SSH deploy key used to clone the repository |
| `AUTOPILOT_DEPLOY_LOG_PATH` | Path to the log file for this deploy run |
| `AUTOPILOT_DEPLOY_PUSHER` | Who triggered the deploy, the GitHub user when triggered via a push or the AutoPilot user when triggered manually |
| `AUTOPILOT_DEPLOY_REF` | The full git ref, for example `refs/heads/master` |
| `AUTOPILOT_DEPLOY_REMOTE` | The git remote URL we clone from |
| `AUTOPILOT_DEPLOY_REPO` | The repository full name, for example `owner/repo` |
| `AUTOPILOT_DEPLOY_SHA` | The full commit SHA |
| `AUTOPILOT_DEPLOY_SHA_SHORT` | The short commit SHA |
| `AUTOPILOT_DEPLOY_TRIGGER` | How the deploy was started, either `push`, `manual`, or `re-run` |
| `AUTOPILOT_DEPLOY_WORKSPACE_DIR` | The temporary directory your repository is cloned into (deleted after the run) |

## Debugging

Every deploy run writes a log file on the jump host under `/mnt/jrc-logs/deploys/`.
Each file is named after the run's delivery ID (the `AUTOPILOT_DEPLOY_DELIVERY` value) with a `.log` extension.
This is the same output you see in the run details on the dashboard, but combined and in its raw form, so if you have SSH access to the jump host you can tail a run live or go back and read an old one.

Our default steps also get more verbose when the commit message contains `[DEBUG]`.
Both `composer install` and `dep deploy` switch to verbose output in that case, which surfaces a lot more detail about what each one is doing.
This is handy when a deploy is failing and the normal output does not tell you enough.
You can put `[DEBUG]` anywhere on the first line of the commit message.

If a push or manual trigger does not start a deploy at all, check two things.
The deployment has to be awake, since a hibernating deployment ignores pushes and blocks manual triggers until it is running again.
The integration also has to be active, since a GitHub integration that has been suspended will not trigger deploys until you unsuspend it.

## Disconnect

If you want to stop deploying to a deployment, use the **Disconnect** button in **Deploy Settings**.
This removes the deploy trigger, deletes the deploy key from your source control provider, and removes the private key file from the jump host.
Your organization level integration stays intact, so your other deployments keep working.
