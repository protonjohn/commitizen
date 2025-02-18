![Bump version](images/bump.gif)

## About

The version is bumped **automatically** based on the commits.

The commits should follow the rules of the committer to be parsed correctly.

It is possible to specify a **prerelease** (alpha, beta, release candidate) version.

The version can also be **manually** bumped.

The version format follows [semantic versioning][semver].

This means `MAJOR.MINOR.PATCH`

| Increment | Description                 | Conventional commit map |
| --------- | --------------------------- | ----------------------- |
| `MAJOR`   | Breaking changes introduced | `BREAKING CHANGE`       |
| `MINOR`   | New features                | `feat`                  |
| `PATCH`   | Fixes                       | `fix` + everything else |

Prereleases are supported following python's [PEP 0440][pep440]

The scheme of this format is

```bash
[N!]N(.N)*[{a|b|rc}N][.postN][.devN]
```

Some examples:

```bash
0.9.0
0.9.1
0.9.2
0.9.10
0.9.11
1.0.0a0  # alpha
1.0.0a1
1.0.0b0  # beta
1.0.0rc0 # release candidate
1.0.0rc1
1.0.0
1.0.1
1.1.0
2.0.0
2.0.1a
```

`post` releases are not supported yet.

## Usage

```bash
$ cz bump --help
usage: cz bump [-h] [--dry-run] [--files-only] [--local-version] [--changelog]
               [--no-verify] [--yes] [--tag-format TAG_FORMAT]
               [--bump-message BUMP_MESSAGE] [--prerelease {alpha,beta,rc}]
               [--devrelease DEVRELEASE] [--increment {MAJOR,MINOR,PATCH}]
               [--check-consistency] [--annotated-tag] [--gpg-sign]
               [--changelog-to-stdout] [--retry] [--major-version-zero]
               [MANUAL_VERSION]

positional arguments:
  MANUAL_VERSION        bump to the given version (e.g: 1.5.3)

options:
  -h, --help            show this help message and exit
  --dry-run             show output to stdout, no commit, no modified files
  --files-only          bump version in the files from the config
  --local-version       bump only the local version portion
  --changelog, -ch      generate the changelog for the newest version
  --no-verify           this option bypasses the pre-commit and commit-msg
                        hooks
  --yes                 accept automatically questions done
  --tag-format TAG_FORMAT
                        the format used to tag the commit and read it, use it
                        in existing projects, wrap around simple quotes
  --bump-message BUMP_MESSAGE
                        template used to create the release commit, useful
                        when working with CI
  --prerelease {alpha,beta,rc}, -pr {alpha,beta,rc}
                        choose type of prerelease
  --devrelease DEVRELEASE, -d DEVRELEASE
                        specify non-negative integer for dev. release
  --increment {MAJOR,MINOR,PATCH}
                        manually specify the desired increment
  --check-consistency, -cc
                        check consistency among versions defined in commitizen
                        configuration and version_files
  --annotated-tag, -at  create annotated tag instead of lightweight one
  --gpg-sign, -s        sign tag instead of lightweight one
  --changelog-to-stdout
                        Output changelog to the stdout
  --retry               retry commit if it fails the 1st time
  --major-version-zero  keep major version at zero, even for breaking changes
```

### `--files-only`

Bumps the version in the files defined in `version_files` without creating a commit and tag on the git repository,

```bash
cz bump --files-only
```

### `--changelog`

Generate a **changelog** along with the new version and tag when bumping.

```bash
cz bump --changelog
```

### `--check-consistency`

Check whether the versions defined in `version_files` and the version in commitizen
configuration are consistent before bumping version.

```bash
cz bump --check-consistency
```

For example, if we have `pyproject.toml`

```toml
[tool.commitizen]
version = "1.21.0"
version_files = [
    "src/__version__.py",
    "setup.py",
]
```

`src/__version__.py`,

```python
__version__ = "1.21.0"
```

and `setup.py`.

```python
...
    version="1.0.5"
...
```

If `--check-consistency` is used, commitizen will check whether the current version in `pyproject.toml`
exists in all version_files and find out it does not exist in `setup.py` and fails.
However, it will still update `pyproject.toml` and `src/__version__.py`.

To fix it, you'll first `git checkout .` to reset to the status before trying to bump and update
the version in `setup.py` to `1.21.0`

### `--local-version`

Bump the local portion of the version.

```bash
cz bump --local-version
```

For example, if we have `pyproject.toml`

```toml
[tool.commitizen]
version = "5.3.5+0.1.0"
```

If `--local-version` is used, it will bump only the local version `0.1.0` and keep the public version `5.3.5` intact, bumping to the version `5.3.5+0.2.0`.

### `--annotated-tag`

If `--annotated-tag` is used, commitizen will create annotated tags. Also available via configuration, in `pyproject.toml` or `.cz.toml`.

### `--changelog-to-stdout`

If `--changelog-to-stdout` is used, the incremental changelog generated by the bump
will be sent to the stdout, and any other message generated by the bump will be
sent to stderr.

If `--changelog` is not used with this command, it is still smart enough to
understand that the user wants to create a changelog. It is recommened to be
explicit and use `--changelog` (or the setting `update_changelog_on_bump`).

This command is useful to "transport" the newly created changelog.
It can be sent to an auditing system, or to create a Github Release.

Example:

```bash
cz bump --changelog --changelog-to-stdout > body.md
```

### `--retry`

If you use tools like [pre-commit](https://pre-commit.com/), add this flag.
It will retry the commit if it fails the 1st time.

Useful to combine with code formatters, like [Prettier](https://prettier.io/).

### `--major-version-zero`

A project in its initial development should have a major version zero, and even breaking changes
should not bump that major version from zero. This command ensures that behavior.

If `--major-version-zero` is used for projects that have a version number greater than zero it fails.
If used together with a manual version the command also fails.

We recommend setting `major_version_zero = true` in your configuration file while a project
is in its initial development. Remove that configuration using a breaking-change commit to bump
your project’s major version to `v1.0.0` once your project has reached maturity.

## Avoid raising errors

Some situations from commitizen rise an exit code different than 0.
If the error code is different than 0, any CI or script running commitizen might be interrupted.

If you have special use case, where you don't want one of this error codes to be raised, you can
tell commitizen to not raise them.

### Recommended use case

At the moment, we've identified that the most common error code to skip is

| Error name        | Exit code |
| ----------------- | --------- |
| NoneIncrementExit | 21        |

There are some situations where you don't want to get an error code when some
commits do not match your rules, you just want those commits to be skipped.

```sh
cz -nr 21 bump
```

### Easy way

Check which error code was raised by commitizen by running in the terminal

```sh
echo $?
```

The output should be an integer like this

```sh
3
```

And then you can tell commitizen to ignore it:

```sh
cz --no-raise 3
```

You can tell commitizen to skip more than one if needed:

```sh
cz --no-raise 3,4,5
```

### Longer way

Check the list of [exit_codes](./exit_codes.md) and understand which one you have
to skip and why.

Remember to document somewhere this, because you'll forget.

For example if the system raises a `NoneIncrementExit` error, you look it up
on the list and then you can use the exit code:

```sh
cz -nr 21 bump
```

## Configuration

### `tag_format`

It is used to read the format from the git tags, and also to generate the tags.

Commitizen supports 2 types of formats, a simple and a more complex.

```bash
cz bump --tag-format="v$version"
```

```bash
cz bump --tag-format="v$minor.$major.$patch$prerelease.$devrelease"
```

In your `pyproject.toml` or `.cz.toml`

```toml
[tool.commitizen]
tag_format = "v$major.$minor.$patch$prerelease"
```

The variables must be preceded by a `$` sign.

Supported variables:

| Variable      | Description                                 |
| ------------- | --------------------------------------------|
| `$version`    | full generated version                      |
| `$major`      | MAJOR increment                             |
| `$minor`      | MINOR increment                             |
| `$patch`      | PATCH increment                             |
| `$prerelease` | Prerelease (alpha, beta, release candidate) |
| `$devrelease` | Development release                         |

---

### `version_files` \*

It is used to identify the files which should be updated with the new version.
It is also possible to provide a pattern for each file, separated by colons (`:`).

Commitizen will update it's configuration file automatically (`pyproject.toml`, `.cz`) when bumping,
regarding if the file is present or not in `version_files`.

\* Renamed from `files` to `version_files`.

Some examples

`pyproject.toml` or `.cz.toml`

```toml
[tool.commitizen]
version_files = [
    "src/__version__.py",
    "setup.py:version"
]
```

In the example above, we can see the reference `"setup.py:version"`.
This means that it will find a file `setup.py` and will only make a change
in a line containing the `version` substring.

---

### `bump_message`

Template used to specify the commit message generated when bumping.

defaults to: `bump: version $current_version → $new_version`

| Variable           | Description                         |
| ------------------ | ----------------------------------- |
| `$current_version` | the version existing before bumping |
| `$new_version`     | version generated after bumping     |

Some examples

`pyproject.toml` or `.cz.toml`

```toml
[tool.commitizen]
bump_message = "release $current_version → $new_version [skip-ci]"
```

---

### `update_changelog_on_bump`

When set to `true` the changelog is always updated incrementally when running `cz bump`, so the user does not have to provide the `--changelog` flag every time.

defaults to: `false`

```toml
[tool.commitizen]
update_changelog_on_bump = true
```

---

### `annotated_tag`

When set to `true` commitizen will create annotated tags.

```toml
[tool.commitizen]
annotated_tag = true
```

---

### `gpg_sign`

When set to `true` commitizen will create gpg signed tags.

```toml
[tool.commitizen]
gpg_sign = true
```

---

### `major_version_zero`

When set to `true` commitizen will keep the major version at zero.
Useful during the initial development stage of your project.

Defaults to: `false`

```toml
[tool.commitizen]
major_version_zero = true
```

## Custom bump

Read the [customizing section](./customization.md).

[pep440]: https://www.python.org/dev/peps/pep-0440/
[semver]: https://semver.org/
