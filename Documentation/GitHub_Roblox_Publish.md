# GitHub → Roblox publish (GarfGames)

This template already ships the publish kit:

- `.github/workflows/publish.yml` — runs on push to `release`, not `main`
- `tool/publish.sh` — lint, `rojo build`, Open Cloud upload
- `tool/publish.env` — this game’s universe / place ids (not secret)

Hand this file to an agent (or follow it) in a **private** GarfGames game created from this template.

## Already done for the org

- Org Actions secret `ROBLOX_API_KEY` exists on **GarfGames**, visibility **private repositories**. Do **not** add a repo secret of the same name — it would hide the org secret.
- CI reads `${{ secrets.ROBLOX_API_KEY }}`. Public repos and forks do not get this secret.

## Per game

### 1. Open Cloud allowlist

At [create.roblox.com/dashboard/credentials](https://create.roblox.com/dashboard/credentials), the shared key needs **universe-places:write** on **this** experience. If CI 403s, this universe is not on the key yet.

### 2. Fill `tool/publish.env`

Creator Dashboard URL:

`.../dashboard/creations/experiences/<UNIVERSE_ID>/places/<PLACE_ID>/configure`

```bash
UNIVERSE_ID=0          # replace
PLACE_ID=0             # replace
PLACE_FILE=build/my-game.rbxl
# PLACE_NAME=my-game   # optional macOS Keychain account
```

Ids are public; the API key is not. Leave zeros until the experience exists — `publish.sh` will refuse to upload.

### 3. `release` branch

```bash
git checkout main
git pull
git checkout -b release        # skip if it already exists
git push -u origin release
```

That branch is the production snapshot. Further `main` pushes do **not** go live.

### 4. First ship (this is the deploy)

```bash
git checkout release
git merge --ff-only main       # or open a PR targeting release
git push origin release
```

That push publishes **live** to players. Manual run: Actions → **Publish to Roblox** → Published (live) or Saved (history only).

Local (optional): `tool/publish.sh`, `tool/publish.sh --saved`, or `tool/publish.sh --build-only`. Key from `$ROBLOX_API_KEY`, macOS Keychain `roblox-open-cloud`, or gitignored `.env.publish`.

## Don’t

- Trigger publish on `main` (agents commit there).
- Commit `.env.publish` or any API key.
- Put `ROBLOX_API_KEY` on the repo if the org secret already covers private repos.
- Publish while `UNIVERSE_ID` is still `0`.

## If it fails

| Symptom | Likely cause |
| --- | --- |
| `secrets.ROBLOX_API_KEY is not set` | Public repo, or a fork PR. Org secret is private-repos only. |
| HTTP 401 / 403 | Key missing **universe-places:write** for this universe, or IP allowlist. |
| HTTP 404 | Wrong ids in `tool/publish.env`. |
| Push to `release` does nothing | Workflow file on **that** branch still says `branches: [main]`. Merge main into release. |
