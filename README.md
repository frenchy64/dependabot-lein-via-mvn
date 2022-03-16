# Leiningen Dependabot support via Maven

Unfortunately, as of March 2022, plans have stalled to add official support Clojure support to Dependabot.

Jurre Stender from GitHub has been working closely with the Clojure community (via Clojurians Slack channel `#dependabot-core`):

> Jurre Stender: It’s not necessarily a technical issue, it’s just that our team (at GitHub) is not ready to accept more ecosystems to support.

He explains dependabot is prohibitively unpleasant to integrate with, with no resources to improve it at GitHub.

> Jurre Stender: We’ve ran an experiment with the Dart team who have implemented most of this functionality in their package manager (pub), but it turned out to be a pretty complicated implementation that we didn’t feel comfortable asking other maintainers to follow. Given our teams priorities we won’t have time to make dependabot-core easier to implement against at this time.

In Clojure, we're used to piggiebacking off the success of Java, so
in that spirit let's lower expectations slightly and access dependabot via Maven.

## Overview

This repository demonstrates how you use dependabot to help
manage Leiningen project dependencies. The basic idea is to use `lein pom`
to generate a `pom.xml` for dependabot to inspect.

The downside is that the PR's sent by dependabot are not mergable, as they
modify the `pom.xml` instead of the `project.clj`.
To compensate, this setup will make GitHub Actions fail in dependabot's PRs until you fix them yourself.

This way, you get many of the benefits of dependabot (rich diffs, security notices), with a few guard rails
to prevent otherwise easy-to-make mistakes that come with this approach.

## Usage

Assuming you have a Leiningen project in a git repo ready to go, adding dependabot support is a matter of
copying some files and code from this repo.

1. Copy [.github/dependabot.yml](.github/dependabot.yml) to your repo. Notice that we've configured the `dependabot` directory as a Maven project--this is where our fake Maven project will live.
2. Copy [script/sync-dependabot](script/sync-dependabot) and [script/check-dependabot](script/check-dependabot) to your repo.
3. Call `./script/sync-dependabot` in your repository root. Commit the generated [dependabot/pom.xml](dependabot/pom.xml).
4. Add a call to `./script/check-dependabot` in your CI build. It will fail if [dependabot/pom.xml](dependabot/pom.xml) is out of date.
   - see [.github/workflows/build.yml](.github/workflows/build.yml) for an example
5. Commit and push.

CI failures will now force you to keep [project.clj](project.clj) and [dependabot/pom.xml](dependabot/pom.xml) synchronized (via [script/sync-dependabot](script/sync-dependabot)).

Finally, to activate dependabot on your repo, go to `https://github.com/<user>/<repo>/settings/security_analysis` and activate
the security and analysis tools you are interested in.

To force dependabot to run the first time, go to `https://github.com/<user>/<repo>/network/updates`, click on the "Last checked" link to the right of `dependabot/pom.xml`, and then `Check for Updates`.
