# Composer Custom

A tool to remporarily change custom composer repositories to local path ones

## Use cases

This tool is useful when you're developing a project that uses composer, and some of the dependencies are also packages you are responsible for. Specifically, it eases development by automating the switching of `composer.json` between **remote** and **local** state.

When your project depends on other custom projects, you may have found it can be hard to make a quick change to a dependency project as you go. The copy of that dependency being used by the main project is a disposable one. If you're reading this, chances are you've had some headaches with editing files the `/vendor` directory, such as losing precious changes when you later run `composer install` or `composer update`.

My solution to this issue was to put my project into a special, **local** state before doing development.

I keep permanent copies of all the the collected dependecies on my machine, off to the side (`~/deps/lib1`, `~/deps/lib2`, etc.). Before starting development on a main project, I change all the custom repositories in `composer.json` to `type=path`, the `url` to the the appropriate path (e.g., `/home/myuser/deps/lib1`), then run `composer update`. From this state, you can make changes directly in `~/deps/*` and they will apply immediately. You can commit and push changes in the dependencies freely as you develop, too.

Before commiting changes to the main project, I would change all the custom repository types and urls back to what they were (e.g., `vcs` for the type, `https://gitlab.com/myuser/lib1.git` for the url), and run `composer update` again. This reverts all traces of referring to dependencies on the local machine, ready to push your changes back to a general audience.

An alternative preparation for committing is to simply revert `composer.json` and `composer.lock` with your version control system. However, by doing this you may inadvertently wipe out other intentional changes to `composer.json`, or some desired updates to `composer.lock`. By following a process to convert `composer.json` between **remote** and **local** states and relying on `composer update`, you work on other aspects of composer at any time.

The process works just as well for dependencies that are public (i.e., listed on packagist.org). In this case, instead of changing the url of an existing repository back and forth, we're adding and removing a repository to stand in for (override) the public one during development.

The purpose of this tool is to automate the switching between **remote** and **local** state of `composer.json`. (You'll have to do the `composer update` but yourself!)

As a bonus, it also sorts your repositories, requires, and your overall `composer.json` file into alphabetical order.

## Installation

Copy or symlink the file `composer-custom` to a location on your PATH. So easy!

## How to use

The tool needs to know the local path for all dependency packages you wish to control with the tool. For truly custom packages (i.e., those not listed on packagist.org), you also need to know the remote URL (hint, you'll find them in `composer.json`), so that they can be restored when switching back to the **remote** state.

Create your config file at `~/custom-package.json` or `~/.custom-package.json`. If both exist, the first is used.

The config is in JSON format, like so

```
{
    "myuser/lib1": {...package config...},
    "myuser/lib2": {...package config...}
}
```

Each `{...package config...}` contains at least `local`, and optionally `remote` and `require`.

```
{
    "local": {"type": "path", "url": "/Users/myuser/deps/lib1"},
    "remote": {"type": "vcs", "url": "https://github.com/myuser/deps/lib1.git"},
    "require": ["myuser/lib2"]
}
```

`local` should be an object, and so should `remote` if it is given. These object will be placed as-is into the `"repositories": [...]` of your `composer.json` as you switch betwee **local** and **remote** state.

`require` is available because, as you know, all custom repository information must reside in the top-level `composer.json` file. If a dependency requires its own dependency, you'll have to replicate this information in this tool's config so the it knows to recurse. The tool does not have access to the dependency repositories themselves, so it can't work out where to recurse to by itself.

Now, on the command line, `cd` to your main project (whereever `composer.json` resides, usually the project root) and run `composer-custom local`.

You will see that repositories are modified and added as necessary in `composer.json` to use the dependencies locally. Run `composer update` and you are now using the local copies and you can hot-change almost as much as you want. If you change a dependencies `composer.json` file as you go, just run `composer update` again in your main project.

When your ready to commit, back on the command line, go to your main project and run `composer-custom remote`.

That's it! Enjoy!

## Note on Composer minimum-stability

The tool takes control of composer's `minimum-stability` setting.

- If you're not using that setting (i.e., accepting the default of `stable`), you won't have any issues.
- If you are explicitly setting a minimum stablility, the tool may remove the setting at some point.
    - If you were using the default (`stable`), you may choose to commit this change and you will have no further issues (unless composer changes the default one day!).
    - If you were using a non-default value, you will find yourself in competition with the tool for control of the setting! If this is you, please let me know your use case and I will consider aadding feature for shared control of that setting.

The reason for this behaviour is that composer can't see tags for repositories of type `path`. This makes composer believe the your local repositories have the lowest stability rating (`dev`) which would prevent doing a successful `composer update`.

To make things go smoothly, the tool injects `"minimum-stability": "dev"` into `composer.json` if any injected repository(ies) is of type `path`. The general intention is that this would only happen when switching to **local** state, but this is not enforced.

When switching back to a state that doesn't use `path`-type repositories (typically, the **remote** state), the tool has no record of any previous value for `minimum-stability`, which is why it has to make an assumption. Namely, it assumes you were not using the setting at all.
