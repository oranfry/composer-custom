# Composer Custom

A tool to temporarily change custom composer repositories during development

## Use cases

This tool is useful when you're developing a project that uses composer, and some of the dependencies are also packages you are responsible for. Specifically, it eases development by automating the switching of `composer.json` between **remote** and **local** state.

When your project depends on other custom projects, you may have found it can be hard to make a quick change to a dependency project as you go. The copy of that dependency being used by the main project is a disposable one. If you're reading this, chances are you've had some headaches with editing files the `/vendor` directory, such as losing precious changes when you later run `composer install` or `composer update`.

My solution to this issue was to put my project into a special, **local** state before doing development.

I keep permanent copies of all the the collected dependecies on my machine, off to the side (`~/deps/lib1`, `~/deps/lib2`, etc.). Before starting development on a main project, I change all the custom repositories in `composer.json` to `type=path`, the `url` to the the appropriate path (e.g., `/home/myuser/deps/lib1`), then run `composer update`. From this state, I can make changes directly in `~/deps/*` and they will apply immediately. I can commit and push changes in the dependencies freely as I develop, too.

Before commiting changes to the main project, I would change all the custom repository types and urls back to what they were (e.g., `vcs` for the type, `https://gitlab.com/myuser/lib1.git` for the url), and run `composer update` again. This reverts all traces of referring to dependencies on the local machine, ready to push your changes back to a general audience.

An alternative preparation for committing is to simply revert `composer.json` and `composer.lock` with your version control system. However, by doing this you may inadvertently wipe out other changes to `composer.json` intented to be permanent, or some desired updates to `composer.lock`. By following a process to convert `composer.json` between **remote** and **local** states and relying on `composer update`, you can freely work on all aspects of composer at any time.

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

Each `{...package config...}` contains at least `local`, and optionally `remote`.

```
{
    "local": {"type": "path", "url": "/Users/myuser/deps/lib1"},
    "remote": {"type": "vcs", "url": "https://github.com/myuser/deps/lib1.git"}
}
```

`local` should be an object, and so should `remote` if it is given. These object will be placed as-is into the `"repositories": [...]` of your `composer.json` as you switch between **local** and **remote** state.

Now, on the command line, `cd` to your main project (whereever `composer.json` resides, usually the project root) and run `composer-custom local`.

You will see that repositories are modified and added as necessary in `composer.json` to use the dependencies locally. Run `composer update` and you are now using the local copies and you can hot-change almost as much as you want. If you change a dependencies `composer.json` file as you go, just run `composer update` again in your main project.

When your ready to commit, back on the command line, go to your main project and run `composer-custom remote`.

That's it! Enjoy!

## Notes on version constraints and minimum stability

To keep composer happy when using `type=path` repositories, the tool temporarily sets version constraints to `*` and `minimum-stability` to `dev` in the local state. So it can later restore the original values, it keeps backups in your `composer.json` file under `extra->composer_custom`, using a predicatble format.

It also recurses to dependencies of your project to switch the versions of downstream known packages to `*` in their respective `composer.json` files. As usual it keeps a backup for when it comes time to switch back to remote state. This is experimental and could cause an issue when using `composer-custom` on multiple packages in random order. If this behaviour is not desired, please switch back to version `1.1.0`.