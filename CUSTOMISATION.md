# Marlin Customisation Workflow

This documents the process for updating to a new Marlin release and carrying forward custom changes.

Assumtion in this doc: Taking the changes from `lts-2.1.1` and applying them to `lts-2.1.2.8`

---

## 1. Check out the latest release from remote

If `upstream` is not yet configured, add it:

```powershell
git remote add upstream https://github.com/MarlinFirmware/Marlin.git
```

Fetch all refs and tags from `upstream` (the official MarlinFirmware/Marlin remote), then create a new local branch at the release tag (e.g. `lts-2.1.2.8`):

```powershell
git fetch upstream --tags
git checkout -b lts-2.1.2.8-custom lts-2.1.2.8
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

Stage and commit them as a baseline, then tag that commit as the stock config anchor:

```powershell
git add Marlin/Configuration.h Marlin/Configuration_adv.h Marlin/_Bootscreen.h Marlin/_Statusscreen.h
git commit -m "Apply stock config files for <version>"
git tag lts-2.1.2.8-base
```

---

## 3. Generate a patch of changes from the previous branch

Both ends of the previous customisation are tagged — the stock config baseline (`lts-2.1.1-Ender3Config`) and the final tested state (`lts-2.1.1-Customisations`). Generate the patch as a diff between them:

```powershell
git diff lts-2.1.1-Ender3Config lts-2.1.1-Customisations > customisations.patch
```

This captures all commits made during customisation and testing, and is applied to the new branch in the next step.

---

## 4. Apply the patch (3-way merge)

Apply the patch to the new branch using 3-way merge so conflicts are marked rather than causing the apply to abort:

```powershell
git apply --3way customisations.patch
```

---

## 5. Resolve conflicts

If `git apply --3way` produces conflicts, resolve them manually in the affected files.

---

## 6. Build, test and commit

Build the firmware using PlatformIO (via Auto Build Marlin or the CLI):

```powershell
pio run
```

Verify the firmware behaves correctly on hardware, then commit:

```powershell
git add -u
git commit -m "Customised <version> — built and tested"
```

Tag the final tested state so both ends are labelled for next time:

```powershell
git tag lts-2.1.2.8-custom
```

The stock config anchor (`lts-2.1.2.8-base`) was tagged in step 2. Together these two tags are all that is needed to regenerate the patch in a future update.

Push the branch and both tags to origin:

```powershell
git push origin lts-2.1.2.8-custom
git push origin refs/tags/lts-2.1.2.8-base
git push origin refs/tags/lts-2.1.2.8-custom
```

---

## 7. Archive the built firmware

Copy the compiled firmware from the PlatformIO build output into the `firmware/` folder with a date-stamped name:

```powershell
$date = Get-Date -Format "yyyyMMdd-HHmmss"
Copy-Item "Marlin\.pio\build\STM32F103RC_creality\firmware.bin" "firmware\firmware-$date.bin"
```

Commit the archived firmware:

```powershell
git add firmware/
git commit -m "Archive firmware build for lts-2.1.2.8-custom ($date)"
git push origin lts-2.1.2.8-custom
```
