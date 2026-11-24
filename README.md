# TNCF Curaçao / Juliana

EuroScope sector packages for the Curaçao FIR (TNCF).

- **Curacao** profile: Aruba, Curaçao and Bonaire.
- **Juliana** profile: St Maarten TMA.

The provider ships two packages, one per profile. Apart from the `.sct`/`.ese`/`.rwy`, the shared files in both are identical.

## Installing through EuroScope

EuroScope can download and update this package itself.

1. In EuroScope, open **Open SCT → Download Sector Files…**.
2. Fill in the fields below the provider list, then click **New**:
   - **Name:** `TNCF Curacao FIR - VATCAR`
   - **URL:** `https://github.com/East-Caribbean-Zone/TNCF-Release-Package/releases/latest/download/TNCF_Sector_Provider.txt`
   - **Filename:** a file in the folder where you keep this package, e.g. `C:\EuroScope\TNCF\TNCF_Sector_Provider.txt`. The package is unpacked into that folder.
3. Tick **A** on the new provider and click **Download**.
4. In **Provided Sector Files**, tick **A** on the package and click **Download**.

From then on EuroScope checks for a new version at start-up. When it finds one, it asks whether to switch your profiles and ASRs to the new sector file. Each update overwrites the package files in that folder, so keep personal changes in your own copies.

## Releasing an update

Releases are published by the **EuroScope release** GitHub Action whenever a tag is pushed. This repo is private, so the release goes to the public download repo [TNCF-Release-Package](https://github.com/East-Caribbean-Zone/TNCF-Release-Package) (set as `repository` in `tools/euroscope-release.json`), using the `RELEASE_TOKEN` secret. That token needs Contents read/write on the public repo and has to be renewed before it expires. The workflow builds the package from the tagged commit and attaches `TNCF-Curacao-Package.7z`, `TNCF-Juliana-Package.7z` and `TNCF_Sector_Provider.txt` to the release (one archive per sector file, because EuroScope only accepts one `.sct` per archive). EuroScope only sees a new version if the sector file name changes, so every release needs a new one:

1. Commit your changes.
2. Raise the version and commit the renames:
   ```powershell
   powershell -ExecutionPolicy Bypass -File tools\EuroScopeRelease.ps1 -Bump
   ```
   This raises the last counter (`…-261101-0001` → `…-261101-0002`), renames the `.sct`/`.ese`/`.rwy` and updates every profile and ASR. It prints the commands for the next step.
3. Tag the commit and push the tag, e.g. `git tag 261101-0002` then `git push origin main 261101-0002`.

To try a build locally without publishing, run the script with `-Build`. It writes to `dist\`, which is not committed. The archive is packed with plain LZMA because EuroScope unpacks it with its bundled 7-Zip 4.65, which can't read newer formats. `.rwy` files are left out so controllers keep their runway selections.

## Updating to a new AIRAC

This repo has no unmodified provider commit, so updates are merged per file with `git merge-file`. The provider's previous package (its zip) is the base, our file is "ours" and the new package is "theirs". Keep the zips of the AIRAC you're on.

1. **Rename first.** Rename the sector files to the new AIRAC names. Update `Settings sector` in `Curacao.prf` and `Juliana.prf`, and `SECTORTITLE` in every ASR. TopSky's `SCTFILEPATH` uses a wildcard and needs no change.
2. **Copy unmodified files.** Take NavData, `TNCF/ICAO` and `aeronav_copyright.txt` straight from the new package.
3. **Merge the `.ese` section by section.** A whole-file merge tends to collapse into one big conflict.
   - `HEADER`, `POSITIONS` and `RADAR` come from the package.
   - `FREETEXT` and `GROUND` are ours.
   - `SIDSSTARS` and `AIRSPACE` get a 3-way merge.
4. **Merge the `.sct`, then re-check our changes to provider data:**
   - **Curaçao VFR points:** Lighthouse, Refinery, Willemstoren, Crystal, Sint Joris, Coral, Bridge and Villas are in `[FIXES]` as the package has them. SIMON is ours, at N012.25.11.940 W070.02.06.640. The `Beatrix Approach` and `Curacao Terminal` ASRs show them as fixes.
   - **Juliana MVA borders** (`Saba`, `Statia`, `Saint Barth`, `Juliana MVA Border`): ours. They're redrawn from AIP AD 2 TNCM-23 around PJM at 14.5°W and must match `TopSky_Juliana/TopSkyMSAW.txt`. Keep our version.
   - **Juliana `.ese`:** the disabled TFFG/TFFJ TAR sector lines are commented out with `;`. EuroScope rejects `//` in `.ese` files.
5. **Check before committing:** run `tools\EuroScopeRelease.ps1 -Check`, which flags any profile or ASR still pointing at an old sector file name. Also check:
   - No conflict markers are left.
   - No broken characters (U+FFFD).
   - Every ASR item, including runways, still exists in its sector file.

## Editing

Open `.sct`, `.ese` and the `TNCF/Settings` files as **windows-1252**. Saving them as UTF-8 breaks `·` and the `ç` in Curaçao.
