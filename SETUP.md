# Setup: new repo with the bot and the website

This folder is the whole project: the playtest bot, the website (`docs/`) and the
proxy that lets the website's Start button run the bot (`worker/`).

## 1. Make the new repo

1. On github.com, click New repository. Name it, set it to **Public** (free GitHub
   Pages needs a public repo), and create it.
2. Click "uploading an existing file", then drag in everything from this folder:
   `index.js`, `package.json`, `env.example`, `README.md`, `SETUP.md`, the `docs`
   folder and the `worker` folder. Commit to `main`.

## 2. Add the workflow file

Folders that start with a dot are often hidden when you drag files, so add this one
by hand.

1. In the new repo click Add file, then Create new file.
2. In the name box type `.github/workflows/playtest.yml`. Typing the slashes
   makes the folders.
3. Open `workflow-playtest.yml` from this folder, copy all of it, and paste it in.
4. Commit.

## 3. Add your Anthropic key

Settings, Secrets and variables, Actions, New repository secret.
Name `ANTHROPIC_API_KEY`, value your key. Without it the bot still runs and saves
screenshots, but writes no report.

## 4. Make a GitHub token

GitHub Settings (your profile menu, not the repo), Developer settings, Personal
access tokens, Fine-grained tokens, Generate new token.

- Repository access: only the new repo
- Permissions: Actions set to Read and write
- Pick an expiry date and write it down. When it expires, Start stops working
  until you make a new token and replace it in step 5.

Copy the token. GitHub shows it only once.

## 5. Create the proxy on Cloudflare

1. In the Cloudflare dashboard, open Workers & Pages and create a Worker.
2. Edit its code. Delete what is there, paste in all of `worker/worker.js`, and
   deploy.
3. Open the Worker's Settings, then Variables and Secrets, and add these:

| Name | Type | Value |
| --- | --- | --- |
| `GITHUB_REPO` | Text | your repo as `owner/name`, for example `snapplerap-c00l/my-playtester` |
| `ALLOWED_ORIGIN` | Text | `https://YOUR-GITHUB-NAME.github.io` with no path and no slash at the end |
| `GITHUB_TOKEN` | Secret | the token from step 4 |
| `ACCESS_CODE` | Secret | any passcode you choose |

4. Deploy again so the settings take effect, then copy the Worker's
   `.workers.dev` address.

If your default branch is not `main`, also add `GITHUB_BRANCH` as Text.

## 6. Point the site at the proxy

In the new repo, open `docs/index.html`, click the pencil, find the line that starts
with `var API_URL`, and replace the address with the one you copied. Commit.

## 7. Turn on the site

Settings, Pages, Build and deployment, Deploy from a branch, branch `main`,
folder `/docs`, Save. After a minute or two the site is at
`https://YOUR-GITHUB-NAME.github.io/YOUR-REPO-NAME/`.

## 8. Use it

Open the site, enter your access code and an allout.game link, and press Start.
The page shows Queued, Running, then Finished. The report is posted on the run's
summary page on GitHub, and the screenshots are attached to that run for 14 days.

## What the proxy checks

- Every request needs the access code.
- Only `https://allout.game` links, using plain characters only.
- Minutes 1 to 30, seconds between shots 5 to 120, bots 1 to 10.
- The job timeout is set to minutes plus 15.
- A new run is refused while another is queued or running.

Runs use your GitHub Actions minutes and your Anthropic key, so keep the access code
private.

## If something goes wrong

- "Wrong access code": the code in the page differs from the `ACCESS_CODE` secret.
- "Failed to fetch" or nothing happens: `ALLOWED_ORIGIN` does not exactly match the
  site's address, or `API_URL` is still the placeholder.
- "GitHub refused the token": the token expired, or it lacks Actions read and write
  on this repo.
- "GitHub could not find the workflow": `GITHUB_REPO` is misspelled, or the file at
  `.github/workflows/playtest.yml` is missing.
