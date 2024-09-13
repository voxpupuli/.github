# Contribution guidelines

## Table of contents

* [Contributing](#contributing)
* [Writing proper commits - short version](#writing-proper-commits-short-version)
* [Writing proper commits - long version](#writing-proper-commits-long-version)
  * [Make separate commits for logically separate changes](#make-separate-commits-for-logically-separate-changes)
  * [Sending your patches](#sending-your-patches)
  * [Update the related GitHub issue](#update-the-related-gitHub-issue)
* [Puppet development and testing](#puppet-development-and-testing)
  * [Dependencies](#dependencies)
    * [Note for OS X users](#note-for-os-x-users)
  * [The test matrix](#the-test-matrix)
  * [Syntax and style](#syntax-and-style)
  * [Running the unit tests](#running-the-unit-tests)
  * [Integration tests](#integration-tests)
* [Gem development and testing](#gem-development-and-testing)

This project has grown over time based on a range of contributions from people using it.
If you follow these contributing guidelines your patch will likely make it into a release a little more quickly.

## Contributing

Please note that this project is released with a Contributor Code of Conduct.
By participating in this project you agree to abide by its terms [Contributor Code of Conduct](https://voxpupuli.org/coc/).

* Fork the repo.
* Create a separate branch for your change.
* We only take pull requests with passing tests, and documentation. [GitHub Actions](https://docs.github.com/en/actions) run the tests for us. You can also execute them locally. This is explained [in a later section](#the-test-matrix).
* For Puppet Code: Checkout [our docs](https://voxpupuli.org/docs/reviewing_pr/) we use to review a module and the [official styleguide](https://puppet.com/docs/puppet/latest/style_guide.html). They provide some guidance for new code that might help you before you submit a pull request.
* Add a test for your change. Only refactoring and documentation changes require no new tests. If you are adding functionality or fixing a bug, please add a test.
* Squash your commits down into logical components. Make sure to rebase against our current HEAD branch. You don't have to squash everything into a single commit.
* Push the branch to your fork and submit a pull request.

Please be prepared to repeat some of these steps as our contributors review your code.

If this is a Puppet Module: Also consider sending in your profile code that calls this component module as an acceptance test or provide it via an issue. This helps reviewers a lot to test your use case and prevents future regressions!

## Writing proper commits - short version

* Make commits of logical units.
* Check for unnecessary whitespace with `git diff --check` before committing.
* Commit using Unix line endings (check the settings around "crlf" in git-config(1)).
* Do not check in commented out code or unneeded files.
* The first line of the commit message should be a short description (50 characters is the soft limit, excluding ticket number(s)), and should skip the full stop.
* The body should provide a meaningful commit message, which:
  * uses the imperative, present tense: `change`, not `changed` or `changes`.
  * includes motivation for the change, and contrasts its implementation with the previous behavior.
  * Make sure that you have tests for the bug you are fixing, or feature you are adding.
  * Make sure the test suites passes after your commit:
  * When introducing a new feature, make sure it is properly documented in the README.md
  * A detailed explanation is also available at [cbea.ms/git-commit](https://cbea.ms/git-commit/)

## Writing proper commits - long version

### Make separate commits for logically separate changes

Please break your commits down into logically consistent units which include new or changed tests relevant to the rest of the change.
The goal of doing this is to make the diff easier to read for whoever is reviewing your code.
In general, the easier your diff is to read, the more likely someone will be happy to review it and get it into the code base.

If you are going to refactor a piece of code, please do so as a separate commit from your feature or bug fix changes.

We also really appreciate changes that include tests to make sure the bug is not re-introduced, and that the feature is not accidentally broken.

Describe the technical detail of the change(s). If your description starts to get too long, that is a good sign that you probably need to split up your commit into more finely grained pieces.

Commits which plainly describe the things which help reviewers check the patch and future developers understand the code are much more likely to be merged in with a minimum of bike-shedding or requested changes.
Ideally, the commit message would include information, and be in a form suitable for inclusion in the release notes for the version of Puppet that includes them.

Please also check that you are not introducing any trailing whitespace or other "whitespace errors".
You can do this by running `git diff --check` on your changes before you commit.

### Sending your patches

To submit your changes via a GitHub pull request, we _highly_ recommend that you have them on a topic branch, instead of directly on your default HEAD branch (usually `main` or `master`).
It makes things much easier to keep track of, especially if you decide to work on another thing before your first change is merged in.
Vox Pupuli Maintainers have the option to rebase the source branch of a pull request, and you don't want that to happen in your HEAD branch.

GitHub has some pretty good [general documentation](http://support.github.com/) on using their site.
They also have documentation on [creating pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request).

In general, after pushing your topic branch up to your repository on GitHub, you can switch to the branch in the GitHub UI and click "Pull Request" towards the top of the page in order to open a pull request.

### Update the related GitHub issue

If there is a GitHub issue associated with the change you submitted, then you should update the ticket to include the location of your branch, along with any other commentary you may wish to make.

## Puppet development and testing

The following section is only relevant for Puppet modules:

The testing and development tools have a bunch of dependencies, all managed by [bundler](http://bundler.io/).

By default the tests use the latest version of Puppet.

If you want a specific version of Puppet, you must set an environment variable such as:

```sh
export PUPPET_VERSION="~> 8.8.1"
```

You can install all needed gems for spec tests into the modules directory by running:

```sh
bundle config set --local path 'vendor'
bundle config set --local without 'development system_tests release'
BUNDLE_JOBS="$(nproc)" bundle install
```

If you also want to run acceptance tests, don't exclude the `system_tests` group.

If you don't know if you need to install or update gems, you can just add `bundle update && bundle clean` as an additional command.

### The test matrix

#### Syntax and style

The test suite will run [puppet-lint](https://puppetlabs.github.io/puppet-lint) and [Puppet Syntax](https://github.com/voxpupuli/puppet-syntax) to check various syntax and style things.
[puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper/blob/main/lib/puppetlabs_spec_helper/rake_tasks.rb) provides the `validate` and `check` rake task.
The `validate` task also calls the `syntax` and `strings:validate:reference` rake tasks.
You can run these locally with:

```sh
bundle exec rake validate lint check
```

`puppet-lint` also supports some autofixing:

```sh
bundle exec rake lint_fix
```

It will also run some [RuboCop](https://rubocop.org/) tests against it.
You can run those locally ahead of time with:

```sh
bundle exec rake rubocop
```

RuboCop also supports save autofixing:

```sh
bundle exec rake rubocop:autocorrect
```

And another autofix option that can fix more, but might break your code:

```sh
bundle exec rake rubocop:autocorrect_all
```

#### Running the unit tests

The unit test suite covers most of the code, as mentioned above please add tests if you're adding new functionality.
If you've not used [rspec-puppet](http://rspec-puppet.com/) before then feel free to ask about how best to test your new feature.

To run the linter, the syntax checker and the unit tests:

```sh
bundle exec rake test
```

[voxpupuli-test](https://github.com/voxpupuli/voxpupuli-test?tab=readme-ov-file#voxpupuli-test-gem) is the Vox Pupuli gem for unit testing and static code analysis that pulls all the dependencies together.
Check the README.md from voxpupuli-test for more options.

#### Integration tests

The unit tests just check the code runs, not that it does exactly what we want on a real machine.
For that we're using [beaker](https://github.com/voxpupuli/beaker).

This fires up a new virtual machine or container and runs a series of simple tests against it after applying the module.
You can run this on your own with:

```sh
BEAKER_setfile=debian12-x64 bundle exec rake beaker
```

Integration (often also called acceptance) tests require the `system_tests` gem group:

```sh
bundle install --with system_tests
```

For extensive information and tips & tricks, see [voxpupuli-acceptance's documentation](https://github.com/voxpupuli/voxpupuli-acceptance#running-tests).
This is the Vox Pupuli gem for acceptance testing that pulls all the dependencies together.

## Gem development and testing

At the moment Vox Pupuli does not have a unified test setup for gems.
The majority of them use [rspec](https://rspec.info/) tests.
Some also [cucumber](https://cucumber.io/).

Our goal is that running `bundle exec rake` always executes the whole test matrix.
If this isn't yet the case for a project you're working on, please [let us know](https://voxpupuli.org/contributing/).
Usually there's also a rake task named `test` or `spec`.
You can list all available rake tasks with `bundle exec rake --tasks`.
All our gems run tests in GitHub Actions.
You can always check `.github/workflows/` and check what the CI does.
