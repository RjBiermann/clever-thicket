# AGENTS.md — builder (j-hc revanced-magisk-module fork)

GitHub repo: `RjBiermann/revanced-magisk-module` (fork of j-hc/revanced-magisk-module).
Produces the patched APK + Magisk module zip and publishes them as releases
(tag is a plain incrementing number, `1`, `2`, … used as NEXT_VER_CODE).

## Config (`config.toml`)

Single app table `[AIS]` (codename; do not spell the app name in this repo).

- `patches-source = "RjBiermann/brave-waffle"` (`patches-version = "latest"` pulls the
  newest `patches-*.mpp` release)
- `cli-source = "MorpheApp/morphe-desktop"` (get_prebuilts matches `*desktop-*.jar`)
- `direct-dlurl = "b64:…"` — **dlurl values with a `b64:` prefix are base64-decoded in
  build.sh**; keep the app's domain out of plain text
- `version = "6.7.1"` pinned — bump on app update. The site's `/download` endpoint
  always serves the latest APK.
- The app has no public per-variant URLs; one universal APK covers phone + TV
  (leanback feature, runtime-detected). Do not add a TV table.

## Local modifications vs upstream (keep on rebase/merge)

1. `build.sh`: after the existing dlurl trailing-strip logic, decode `b64:`-prefixed
   `*-dlurl` values (decode AFTER the strips — strips would mangle the decoded URL).
2. `utils.sh` `dl_direct()`: version-in-URL check removed (the URL is static; version
   comes from the patches' compatibility list).
3. `utils.sh` `build_rv()`: stock APK path keyed on `${app_name_l}` instead of
   `${pkg_name}` (both variants share package name + version).

## Gotchas

- Morphe CLI ≠ ReVanced CLI flags: `list-patches --patches=<file>` (`-p` there is a
  boolean `--with-packages`). j-hc's `patches_list()` first attempt fails, its fallback
  uses the space form and works. `list-versions` output matches j-hc's parser.
- `dlurl` config values get `%download`/`%/` suffix strips — never put the encoded
  URL such that decoding happens before the strips.
- j-hc repo has tags that collide with branch names — push with
  `git push origin HEAD:refs/heads/main`.
- CI: `build.yml` (workflow_dispatch + schedule 16:00 UTC daily), `ci.yml` checks only.
  Release assets: `ais-morphe-v<ver>-all.apk` and `-module-…-zip`.
- Signing: repo-root `ks.keystore` (alias jhc, pass 123456789) — keep these keys stable,
  otherwise users lose update compatibility.
- Local build needs `GITHUB_TOKEN` (gh auth token works) for prebuilts download;
  build outputs land in `build/` (gitignored), scratch in `temp/` (safe to rm).
