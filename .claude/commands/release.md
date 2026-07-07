---
description: Build a new Rembo Android beta APK and publish it as a GitHub release
argument-hint: [optional release notes]
---

# Rembo Release

Ship a new closed-beta Android build end to end: clean old artifacts, build a fresh
APK with EAS, publish it as a GitHub release, and update the local release tracker.

Repo layout assumption: this repo (`rembo-releases`) and `rembo-mobile` are sibling
directories. Resolve `rembo-mobile`'s path as `$(dirname "$(git rev-parse --show-toplevel)")/rembo-mobile`
rather than assuming the shell's current directory.

## Steps

1. **Clean old builds.** Remove any `*.apk` in `builds/` (this repo). They're
   gitignored build artifacts, safe to delete.

2. **Determine the next version.** Run `gh release list --limit 1` (or
   `git -C <this repo> tag --sort=-v:refname | head -1`) to find the latest
   `v1.0.0-beta.N` tag and increment N by 1.

3. **Build the APK.** In `rembo-mobile`, run:
   ```
   npx eas build --profile preview --platform android --local --non-interactive \
     --output <rembo-releases repo path>/builds/Rembo_Secure_Beta.apk
   ```
   Building straight to that path/name avoids a separate move+rename step.
   This can take several minutes — let it run to completion and check the exit code.

4. **Verify the artifact.** Confirm `builds/Rembo_Secure_Beta.apk` exists and is a
   plausible size (the last few builds were 150–280MB). If the build failed or the
   file is missing/tiny, stop and report the error — do not proceed to publish.

5. **Show a summary and confirm before publishing.** Publishing a GitHub release is
   public and visible to beta testers, so pause here and show the user:
   - the new tag (`v1.0.0-beta.N`)
   - the release title (`Rembo v1.0.0-beta.N`)
   - the draft release notes (template below)
   - the APK size

   Wait for explicit confirmation before running `gh release create`.

6. **Create the GitHub release**, uploading the APK as the asset:
   ```
   gh release create v1.0.0-beta.N builds/Rembo_Secure_Beta.apk \
     --repo ctrl-raj/rembo-releases \
     --title "Rembo v1.0.0-beta.N" \
     --prerelease \
     --notes "<notes>"
   ```

   Release notes template (fill in N, keep the rembo.space mention):
   ```
   ## Rembo Player App — Beta N

   Thanks for being part of the Rembo closed beta!

   Learn more about Rembo at https://rembo.space

   **Installation steps:**
   1. Download the APK file below
   2. On your Android device, enable *Install from unknown sources* in Settings
   3. Open the downloaded APK and install
   4. Launch Rembo and sign in

   > This is a pre-release build intended for closed beta testers only.
   ```
   If the user passed extra notes as `$ARGUMENTS`, fold them into the template
   (e.g. a "What's new" section) instead of replacing it.

7. **Update the release tracker.** Insert a new row at the top of the table in
   `builds/releases.md` (newest first):
   ```
   | <UTC timestamp, YYYY-MM-DD HH:MM> | v1.0.0-beta.N | https://github.com/ctrl-raj/rembo-releases/releases/download/v1.0.0-beta.N/Rembo_Secure_Beta.apk |
   ```
   Get the timestamp from `date -u +"%Y-%m-%d %H:%M"`, not from memory.

8. **Commit the tracker update.** `builds/releases.md` is git-tracked (the APK
   itself is gitignored). Stage and commit just that file:
   ```
   git add builds/releases.md
   git commit -m "Track release v1.0.0-beta.N"
   ```
   Ask before pushing, same as any other push.

9. **Report back** with the release URL and the direct APK download link so the
   user can share it immediately.
