# paket-git-github-sources

## Feature exercised

This probe exercises Paket's `git` and `github` source types, which allow dependencies to be sourced directly from a git repository or a specific file on GitHub, in addition to standard NuGet packages.

## Probe metadata

- **Pattern**: `github-source` / `git-source`
- **Target framework**: `net8.0`
- **Storage mode**: `storage: none`
- **Generated**: 2026-04-23T07:48:17Z

## Sources declared

| Type   | Reference                                                             |
|--------|-----------------------------------------------------------------------|
| nuget  | `https://api.nuget.org/v3/index.json`                                 |
| git    | `https://github.com/fsprojects/Chessie.git` branch `master`          |
| github | `fsprojects/Chessie` file `src/Chessie/ErrorHandling.fs`             |

## Expected dependency tree

### Packages that must appear

| artifactId                         | version / commit                           | dependencyType | Source section |
|------------------------------------|--------------------------------------------|----------------|----------------|
| `Newtonsoft.Json`                  | `13.0.3`                                   | `NUGET`        | NUGET          |
| `Serilog`                          | `3.1.1`                                    | `NUGET`        | NUGET          |
| `Chessie` (git repo)               | commit `1de397e9a95aba37ec9399c0eb7f83d851f43b01` | `GIT`   | GIT            |
| `src/Chessie/ErrorHandling.fs`     | commit `1de397e9a95aba37ec9399c0eb7f83d851f43b01` | `GITHUB` | GITHUB       |

### Key assertions for Mend detection

- The `NUGET` section resolves `Newtonsoft.Json 13.0.3` and `Serilog 3.1.1` with no transitive dependencies (neither has runtime deps in this pinned version set).
- The `GIT` section must be parsed: Mend UA is expected to report the git repository URL and pinned commit SHA as a detected dependency. If Mend silently drops the GIT section, the tree will be missing two entries.
- The `GITHUB` section must be parsed: Mend UA is expected to report the single file reference (`src/Chessie/ErrorHandling.fs`) with its pinned commit SHA. If Mend silently drops the GITHUB section, the tree will be missing one entry.
- No transitive dependencies are expected from the git or github sources (Paket does not resolve transitive deps for these source types).
- All four entries belong to the default (Main) group — no named groups are used in this probe.

## Mend detection failure modes targeted

- **GIT section silently skipped**: UA only reads NUGET sections and ignores GIT/GITHUB blocks entirely, producing a tree with only 2 of 4 expected entries.
- **GITHUB section silently skipped**: Same as above but specifically for the single-file GitHub reference.
- **Commit SHA not captured**: Mend records the dependency name but not the pinned SHA, making version comparison impossible.
