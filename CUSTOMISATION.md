# Marlin Customisation Workflow

This documents the process for updating to a new Marlin release and carrying forward custom changes.

Branch/tag structure per version:
- `lts-2.1.2.8-ender3` — upstream release + Ender 3 config files applied (the base branch)
- `lts-2.1.2.8-custom` — branched from above, with customisations
- `customisations/lts-2.1.2.8` **tag** — marks the customisation commit on the custom branch (before firmware/docs commits)

Patch generation for the next update, from the previous base Ender3 version to the last of the customisation commmits: `git diff lts-2.1.2.8-ender3 customisations/lts-2.1.2.8`

---

## 1. Check out the latest release from remote

If `upstream` is not yet configured, add it:

```powershell
git remote add upstream https://github.com/MarlinFirmware/Marlin.git
```

Fetch all refs and tags from `upstream` (the official MarlinFirmware/Marlin remote), then create the Ender 3 base branch at the release tag (e.g. `lts-2.1.2.8`):

```powershell
git fetch upstream --tags
git checkout -b lts-2.1.2.8-ender3 lts-2.1.2.8
```

---

## 2. Get the matching config files

Download the matching example configuration files from the [Marlin Configurations repository](https://github.com/MarlinFirmware/Configurations/).

The branch/tag in that repo should match the Marlin version you checked out. Copy the relevant example config files into `Marlin/`:

```
Marlin/Configuration.h
Marlin/Configuration_adv.h
Marlin/_Bootscreen.h
Marlin/_Statusscreen.h
```

Stage, commit, and push the base branch:

```powershell
git add Marlin/Configuration.h Marlin/Configuration_adv.h Marlin/_Bootscreen.h Marlin/_Statusscreen.h
git commit -m "Apply Ender 3 config files for <version>"
git push origin lts-2.1.2.8-ender3
```

---

## 3. Generate a patch of changes from the previous version

The previous version's customisations are captured between its ender3 base branch and its customisations tag. Generate the patch:

```powershell
git diff lts-2.1.1-ender3 customisations/lts-2.1.1 > customisations.patch
```

This captures all customisation commits, excluding any subsequent firmware/docs commits on the previous branch.

---

## 4. Create the custom branch

Branch off from the new ender3 base:

```powershell
git checkout -b lts-2.1.2.8-custom lts-2.1.2.8-ender3
```

---

## 5. Apply the patch (3-way merge)

Apply the patch to the new custom branch using 3-way merge so conflicts are marked rather than causing the apply to abort:

```powershell
git apply --3way customisations.patch
```

---

## 6. Resolve conflicts

If `git apply --3way` produces conflicts, resolve them manually in the affected files.

---

## 7. Build, test and commit

Build the firmware using PlatformIO (via Auto Build Marlin or the CLI):

```powershell
pio run
```

Verify the firmware behaves correctly on hardware, then commit and tag the customisation commit:

```powershell
git add -u
git commit -m "Customised <version> — built and tested"
git tag customisations/lts-2.1.2.8
```

Push the branch and tag to origin:

```powershell
git push origin lts-2.1.2.8-custom
git push origin customisations/lts-2.1.2.8
```

---

## 8. Archive the built firmware

Copy the compiled firmware from the PlatformIO build output into the `firmware/` folder with a date-stamped name:

```powershell
$date = Get-Date -Format "yyyyMMdd-HHmmss"
Copy-Item "Marlin\.pio\build\STM32F103RC_creality\firmware.bin" "firmware\firmware-$date.bin"
```

Commit and push:

```powershell
git add firmware/
git commit -m "Archive firmware build for lts-2.1.2.8-custom ($date)"
git push origin lts-2.1.2.8-custom
```
