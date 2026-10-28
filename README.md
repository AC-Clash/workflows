# AC-Clash Workflows

Reusable GitHub Actions workflows shared by the AC-Clash plugin projects. Each plugin repo calls one workflow here instead of keeping its own copy, so a fix to the release process only has to be made once.

## `java-gradle-release.yml`

Builds a Gradle project and publishes a GitHub release whenever its version number changes.

On every push it:

1. **Reads the version** from `build.gradle.kts`. `-SNAPSHOT` versions are skipped.
2. **Compares it with the latest tag.** If the version hasn't changed, the run stops here, so ordinary commits never create releases.
3. **Picks the JDK** from the project's `languageVersion` setting and installs that version of Amazon Corretto.
4. **Builds the JAR** with `./gradlew build`.
5. **On `main`/`master` only:** tags the commit `v<version>`, creates a GitHub release with notes generated from the commit messages since the last tag, and uploads the built JAR as a release asset.
6. **Emails the author** a link to the new release, if the `passwd` secret is set.

### Usage

Add a caller workflow to the project, for example `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    branches: ['**']
    tags-ignore: ['*']

jobs:
  release:
    permissions:
      contents: write
    uses: AC-Clash/workflows/.github/workflows/java-gradle-release.yml@main
    secrets:
      token: ${{ secrets.TOKEN }}
      passwd: ${{ secrets.PASSWD }} # optional
```

### Secrets

| Secret | Purpose |
| --- | --- |
| `token` | GitHub token with `contents: write`, used to create the release and upload the JAR |
| `passwd` | Optional. App password for the SMTP account that sends the release email. If it is not set, the email step is skipped. |

### Project requirements

- Gradle Kotlin DSL (`build.gradle.kts`) with a quoted `version = "..."`
- A Java toolchain set with `languageVersion.set(JavaLanguageVersion.of(N))`
- The Gradle wrapper (`gradlew`) committed to the repo
- The built JAR in `build/libs/`. JARs ending in `dev.jar` or `dev-all.jar` are ignored.
