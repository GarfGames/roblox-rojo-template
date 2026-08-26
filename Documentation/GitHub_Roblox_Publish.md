# GitHub → Roblox publish (GarfGames)

Hand this file to an agent (or follow it yourself) in any **private** GarfGames game repo. Mallipelago is the working example: [GarfGames/mallipelago](https://github.com/GarfGames/mallipelago).

## Already done for the org

- Org Actions secret `ROBLOX_API_KEY` exists on **GarfGames**, visibility **private repositories**. Do **not** add a repo secret of the same name — it would hide the org secret.
- CI reads `${{ secrets.ROBLOX_API_KEY }}`. Public repos and forks do not get this secret.

## Per game (do this in the repo)

### 1. Open Cloud allowlist

At [create.roblox.com/dashboard/credentials](https://create.roblox.com/dashboard/credentials), the shared key needs **universe-places:write** on **this** experience. If CI 403s, the key is still mallipelago-only.

### 2. Copy publish tooling from mallipelago

From [mallipelago](https://github.com/GarfGames/mallipelago):

| File | What to change |
| --- | --- |
| `.github/workflows/publish.yml` | Keep as-is (triggers on `release`, not `main`) |
| `tool/publish.sh` | `OUT=build/<repo>.rbxl` and the Keychain account name (`mallipelago` → this repo) |
| `tool/publish.env` | This game’s universe + place ids (not secret) |

Add to `.gitignore` if missing:

```
.env.publish
/build/
```

`tool/publish.env`:

```bash
# Creator Dashboard URL:
#   .../dashboard/creations/experiences/<UNIVERSE_ID>/places/<PLACE_ID>/configure
UNIVERSE_ID=0
PLACE_ID=0
```

Replace the zeros. Ids are public; the API key is not.

### 3. `release` branch

```bash
git checkout main
git pull
git checkout -b release        # skip if it already exists
git push -u origin release
```

Creating `release` from current `main` is the production snapshot. Further `main` pushes do **not** go live.

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
- Publish a `rojo build` of a game that still has `UNIVERSE_ID=0`.

## If it fails

| Symptom | Likely cause |
| --- | --- |
| `secrets.ROBLOX_API_KEY is not set` | Public repo, or a fork PR. Org secret is private-repos only. |
| HTTP 401 / 403 | Key missing **universe-places:write** for this universe, or IP allowlist. |
| HTTP 404 | Wrong ids in `tool/publish.env`. |
| Push to `release` does nothing | Workflow file on **that** branch still says `branches: [main]`. Merge main into release. |
