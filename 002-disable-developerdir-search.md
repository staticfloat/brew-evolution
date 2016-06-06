# An option to disable developer directory searching via `xcode-select`
## Introduction
It would be nice to have a method (environment variables, command-line flags, etc...) to stop `brew` from querying `xcode-select` on every operation.

## Motivation
When deploying Homebrew into an environment without `sudo` access (and no `Xcode`, no `CLT`), Homebrew can still be very useful, as a combination of Homebrew itself and environment variables can build a functional bottle installation and relocation system.  Given the appropriate level of determination, a user can set the necessary environment variables to get things like `git`, `cairo`, etc... running without `sudo`, `Xcode` or the `CLT`.

As a use case, we are currently revamping the [`Homebrew.jl` package](https://github.com/JuliaLang/Homebrew.jl/tree/sf/rewrite) for the Julia language to bootstrap itself completely without a commandline `git` or CLT installed.  We take the following steps:

* Download/extract `Homebrew` with `curl`/`tar`
* Tap `homebrew/core`
* Install `cctools`
* Install `git` and all of its dependencies from bottles (using `--force-bottle` on each dependency in order to avoid the fact that [`--force-bottle` doesn't propagate to dependencies](https://github.com/Homebrew/legacy-homebrew/issues/35100))
* Set `GIT_EXEC_PATH` to `#{opt}/libexec/git-core` and `GIT_TEMPLATE_DIR` to `#{opt}/share/git-core`

These steps are automated and work well, but as soon as one invokes `brew` it [checks for the active developer directory](https://github.com/Homebrew/brew/blob/2cd81e50513f96a657454031e52fa4aec773ea97/Library/Homebrew/os/mac.rb#L71-L73), which causes a window to popup every `brew` operation.  Other than those popups, Homebrew is functional for installing bottles.

## Proposed solution
I suggest adding an environment variable to change the behavior of the active developer directory autodetection code linked to above.

I propose to add the environment variable `HOMEBREW_DEVELOPER_DIR` to override the autoselection through `xcode-select`.  This would allow users to override the autodetected developer directory in the event of multiple installations.  I don't have a solid use case for this, I prefer it because it seems the most flexible choice, and provides an obvious logic path through the code, whereas attempting to "skip" the `Utils.popen_read()` function doesn't provide an obvious return value for `active_developer_dir`.

## Detailed design
The design is very simple: The `active_developer_dir` method checks for the existence of the `HOMEBREW_DEVELOPER_DIR` environment variable.  If it exists, it returns its value instead of calling `Utils.popen_read()`.
