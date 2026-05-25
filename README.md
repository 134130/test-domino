# test-domino

Fixture repository for manual gh-domino end-to-end testing.

Run `./init` to reset this repository and create a reproducible GitHub PR
fixture. The script force-pushes `main`, closes open fixture PRs, and deletes
fixture branches whose names start with `domino-`. It also cleans up the legacy
`stack-1`, `stack-2`, and `stack-3` branches from the first fixture version.

## Usage

```sh
./init [basic|merged|diverged|projection|conflict|all]
```

The default scenario is `basic`.

Generated branch names use a timestamped prefix:

```text
domino-<run-id>-<logical-id>
```

The script prints a manifest at the end that maps each logical id to its branch,
base branch, PR URL, and expected gh-domino state.

## Scenarios

### basic

Creates a clean tree for TUI navigation and selection behavior:

```text
100
- 110
  - 111

200
- 210
  - 211
  - 212
- 220
  - 221
```

Use this for row rendering, cursor movement, node/subtree/chain toggles, and
clean update previews.

### merged

Creates child PRs whose parents are merged with different GitHub merge methods:

```text
M100 -> M110  merge commit
S100 -> S110  squash merge
R100 -> R110  rebase merge
```

Use this to verify `merged_base` detection and the upstream behavior used by
repair rebases.

### diverged

Creates `D100 -> D110`, then updates `D100` after `D110` was created.

Expected: `D110` is reported as `parent_diverged`.

### projection

Creates `P000 -> P100 -> P110`, then merges `P000`.

Expected:

- `P100` is a repair candidate because its base was merged.
- `P110` alone has no action.
- Selecting `P100` projects `P110` as `after parent repair` in the TUI.

### conflict

Creates `C000 -> C100`, merges `C000`, then edits the same line on `main`.

Expected: repairing `C100` should fail with a rebase conflict and gh-domino
should abort the rebase.

### all

Builds every scenario in one reset. Branches are created before the scenario
merge steps so independent roots are not accidentally based on another fixture's
merged commits.
