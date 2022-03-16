# Leiningen Dependabot support via Maven

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
