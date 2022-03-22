# Leiningen Dependabot support via Maven

Unfortunately, as of March 2022, plans have stalled to officially support Clojure in Dependabot.

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
   - make sure they're executable: run `chmod +x script/sync-dependabot script/check-dependabot`
3. Call `./script/sync-dependabot` in your repository root. Commit the generated [dependabot/pom.xml](dependabot/pom.xml).
   - requires [babashka](https://github.com/babashka/babashka)
4. Add a call to `./script/check-dependabot` in your CI build. It will fail if [dependabot/pom.xml](dependabot/pom.xml) is out of date.
   - requires [babashka](https://github.com/babashka/babashka)
   - see the end of the `setup` job in [.github/workflows/build.yml](.github/workflows/build.yml) for an example, which uses a babashka installer
5. Commit and push.

CI failures will now force you to keep [project.clj](project.clj) and [dependabot/pom.xml](dependabot/pom.xml) synchronized (via [script/sync-dependabot](script/sync-dependabot)).

Finally, to activate dependabot on your repo, follow [these](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates#enabling-or-disabling-dependabot-security-updates-for-an-individual-repository) instructions. In short, go to `https://github.com/<USER>/<REPO>/settings/security_analysis` and activate
the security and analysis tools you are interested in. You must be an admin--if you're preparing a pull-request for a repo that you're not an admin of, you can first fork the repo to test out dependabot on your own fork, and then I suggest asking the maintainer to enable dependabot themselves.

To simulate the "admin" experience or to test out dependabot, you can fork this repo and go to `https://github.com/<USER>/<REPO>/settings/security_analysis`.

To force dependabot to run the first time, go to `https://github.com/<YOUR USER>/<YOUR REPO>/network/updates`, click on the "Last checked" link to the right of `dependabot/pom.xml`, and then `Check for Updates`.

## Workflow hints

Most of the time it will be easy to update the dependabot PR directly from GitHub's UI to be mergable.

1. At the top of the dependabot PR, it will show a link to the `tree` view of the upstream branch. Click on it.
  - eg., in [this](https://github.com/frenchy64/dependabot-lein-via-mvn/pull/2) PR, click on the link at the end of `dependabot wants to merge 2 commits into main from dependabot/maven/dependabot/org.clojure-clojure-1.10.3`
2. You should be at (for example) `https://github.com/frenchy64/dependabot-lein-via-mvn/tree/dependabot/maven/dependabot/org.clojure-clojure-1.10.3`.
3. To edit the file, go to `https://github.com/frenchy64/dependabot-lein-via-mvn/edit/dependabot/maven/dependabot/org.clojure-clojure-1.10.3/project.clj`.
  - some ways to do this:
    1. press `t` and type `project.clj` and press enter, then click the "Edit this file" button, or
    2. change `tree` to `edit` and add `/project.clj` to the end of the URL.
4. Now make the `project.clj` reflect the new `dependabot/pom.xml` version, and press `Commit changes` at the bottom of the page to commit directly to the branch.
