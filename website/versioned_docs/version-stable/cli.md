---
id: cli
title: CLI
---

Use the `prettier` command to run Prettier from the command line.

```bash
prettier [options] [file/dir/glob ...]
```

:::note

To run your locally installed version of Prettier, prefix the command with `npx`, `yarn exec`, `pnpm exec`, or `bunx`, i.e. `npx prettier --help`, `yarn exec prettier --help`, `pnpm exec prettier --help`, or `bunx prettier --help`.

:::

To format a file in-place, use `--write`. (Note: This overwrites your files!)

In practice, this may look something like:

```bash
prettier . --write
```

This command formats all files supported by Prettier in the current directory and its subdirectories.

It’s recommended to always make sure that `prettier --write .` only formats what you want in your project. Use a [`.prettierignore`](ignore.md) file to ignore things that should not be formatted.

A more complicated example:

```bash
prettier docs package.json "{app,__{tests,mocks}__}/**/*.js" --write --single-quote --trailing-comma all
```

:::warning

Don’t forget the **quotes** around the globs! The quotes make sure that Prettier CLI expands the globs rather than your shell, which is important for cross-platform usage.

:::

:::note

It’s better to use a [configuration file](configuration.md) for formatting options like `--single-quote` and `--trailing-comma` instead of passing them as CLI flags. This way the Prettier CLI, [editor integrations](editors.md), and other tooling can all know what options you use.

:::

## File patterns

Given a list of paths/patterns, the Prettier CLI first treats every entry in it as a literal path.

- If the path points to an existing file, Prettier CLI proceeds with that file and doesn’t resolve the path as a glob pattern.

- If the path points to an existing directory, Prettier CLI recursively finds supported files in that directory. This resolution process is based on file extensions and well-known file names that Prettier and its [plugins](plugins.md) associate with supported languages.

- Otherwise, the entry is resolved as a glob pattern using the [glob syntax from the `fast-glob` module](https://github.com/mrmlnc/fast-glob#pattern-syntax).

Prettier CLI will ignore files located in `node_modules` directory. To opt out from this behavior, use `--with-node-modules` flag.

Prettier CLI will not follow symbolic links when expanding arguments.

To escape special characters in globs, one of the two escaping syntaxes can be used: `prettier "\[my-dir]/*.js"` or `prettier "[[]my-dir]/*.js"`. Both match all JS files in a directory named `[my-dir]`, however the latter syntax is preferable as the former doesn’t work on Windows, where backslashes are treated as path separators.

## `--check`

When you want to check if your files are formatted, you can run Prettier with the `--check` flag (or `-c`).
This will output a human-friendly message and a list of unformatted files, if any.

```bash
prettier . --check
```

Console output if all files are formatted:

```console
Checking formatting...
All matched files use Prettier code style!
```

Console output if some of the files require re-formatting:

```console
Checking formatting...
[warn] src/fileA.js
[warn] src/fileB.js
[warn] Code style issues found in 2 files. Run Prettier with --write to fix.
```

The command will return exit code `1` in the second case, which is helpful inside the CI pipelines.
Human-friendly status messages help project contributors react on possible problems.
To minimise the number of times `prettier --check` finds unformatted files, you may be interested in configuring a [pre-commit hook](precommit.md) in your repo.
Applying this practice will minimise the number of times the CI fails because of code formatting problems.

If you need to pipe the list of unformatted files to another command, you can use [`--list-different`](cli.md#--list-different) flag instead of `--check`.

### Exit codes

| Code | Information                         |
| ---- | ----------------------------------- |
| `0`  | Everything formatted properly       |
| `1`  | Something wasn’t formatted properly |
| `2`  | Something’s wrong with Prettier     |

## `--debug-check`

If you're worried that Prettier will change the correctness of your code, add `--debug-check` to the command. This will cause Prettier to print an error message if it detects that code correctness might have changed. Note that `--write` cannot be used with `--debug-check`.

## `--find-config-path` and `--config`

If you are repeatedly formatting individual files with `prettier`, you will incur a small performance cost when Prettier attempts to look up a [configuration file](configuration.md). In order to skip this, you may ask Prettier to find the config file once, and re-use it later on.

```console
$ prettier --find-config-path path/to/file.js
path/to/.prettierrc
```

This will provide you with a path to the configuration file, which you can pass to `--config`:

```bash
prettier path/to/file.js --write --config path/to/.prettierrc
```

You can also use `--config` if your configuration file lives somewhere where Prettier cannot find it, such as a `config/` directory.

If you don’t have a configuration file, or want to ignore it if it does exist, you can pass `--no-config` instead.

## `--ignore-path`

Path to a file containing patterns that describe files to ignore. By default, Prettier looks for `./.gitignore` and `./.prettierignore`.\
Multiple values are accepted.

## `--list-different`

Another useful flag is `--list-different` (or `-l`) which prints the filenames of files that are different from Prettier formatting. If there are differences the script errors out, which is useful in a CI scenario.

```bash
prettier . --single-quote --list-different
```

You can also use [`--check`](cli.md#--check) flag, which works the same way as `--list-different`, but also prints a human-friendly summary message to stdout.

## `--no-config`

Do not look for a configuration file. The default settings will be used.

## `--config-precedence`

Defines how config file should be evaluated in combination of CLI options.

**cli-override (default)**

CLI options take precedence over config file

**file-override**

Config file take precedence over CLI options

**prefer-file**

If a config file is found will evaluate it and ignore other CLI options. If no config file is found, CLI options will evaluate as normal.

This option adds support to editor integrations where users define their default configuration but want to respect project specific configuration.

## `--no-editorconfig`

Don’t take `.editorconfig` into account when parsing configuration. See the [`prettier.resolveConfig` docs](api.md) for details.

## `--with-node-modules`

Prettier CLI will ignore files located in `node_modules` directory. To opt out from this behavior, use `--with-node-modules` flag.

## `--write`

This rewrites all processed files in place. This is comparable to the `eslint --fix` workflow. You can also use `-w` alias.

## `--log-level`

Change the level of logging for the CLI. Valid options are:

- `error`
- `warn`
- `log` (default)
- `debug`
- `silent`

## `--stdin-filepath`

A path to the file that the Prettier CLI will treat like stdin. For example:

_abc.css_

```css
.name {
  display: none;
}
```

_shell_

```console
$ cat abc.css | prettier --stdin-filepath abc.css
.name {
  display: none;
}
```

## `--ignore-unknown`

With `--ignore-unknown` (or `-u`), prettier will ignore unknown files matched by patterns.

```bash
prettier "**/*" --write --ignore-unknown
```

## `--no-error-on-unmatched-pattern`

Prevent errors when pattern is unmatched.

## `--cache`

If this option is enabled, the following values are used as cache keys and the file is formatted only if one of them is changed.

- Prettier version
- Options
- Node.js version
- (if `--cache-strategy` is `metadata`) file metadata, such as timestamps
- (if `--cache-strategy` is `content`) content of the file

```bash
prettier . --write --cache
```

Running Prettier without `--cache` will delete the cache.

Also, since the cache file is stored in `./node_modules/.cache/prettier/.prettier-cache`, so you can use `rm ./node_modules/.cache/prettier/.prettier-cache` to remove it manually.

:::warning

Plugins version and implementation are not used as cache keys. We recommend that you delete the cache when updating plugins.

:::

## `--cache-location`

Path to the cache file location used by `--cache` flag. If you don't explicit `--cache-location`, Prettier saves cache file at `./node_modules/.cache/prettier/.prettier-cache`.

If a file path is passed, that file is used as the cache file.

```bash
prettier . --write --cache --cache-location=path/to/cache-file
```

## `--cache-strategy`

Strategy for the cache to use for detecting changed files. Can be either `metadata` or `content`.

In general, `metadata` is faster. However, `content` is useful for updating the timestamp without changing the file content. This can happen, for example, during git operations such as `git clone`, because it does not track file modification times.

If no strategy is specified, `content` will be used.

```bash
prettier . --write --cache --cache-strategy metadata
```
