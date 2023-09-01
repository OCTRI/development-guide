# Versioning and Release Best Practices using Maven and Git

When publishing software libraries that will be consumed downstream, it is beneficial to follow a few rules to help maintain consistency for the end user. For long-running software projects, many of these rules can and should be managed through a Continuous Integration environment like Jenkins.

### Rule 1: Use SNAPSHOT versions to indicate the library is not ready for publication and may change

The Maven convention for versioning software uses the suffix "-SNAPSHOT" to indicate that the branch may be unstable and that changes to the API may occur. The corresponding pom might look like:

	<groupId>org.example</groupId>
	<artifactId>mylibrary</artifactId>
	<version>1.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

The version should be set to the next targeted release of the software. In the case above, the next planned release is version 1.0.1. All branches, including "main" ("master" in older projects), should be packaged as a SNAPSHOT until the point in time that you are ready to release.

### Rule 2: Use Git branches for all development work

All experimental development work is done in branches and merged to "main" through pull requests reviewed by peers. Any branch merged should pass all tests and may benefit from other plugins to examine code quality.

### Rule 3: Always release from a single branch, usually "main"

One branch in the project should be designated as the branch to release from. Generally, this is how "main" is used. In earlier versions of Git, the convention was to call it the "master" branch, so you may see either depending on how old the project is. While this branch may have breaking changes compared to other release points, it should always be stable in terms of compile and run time errors and tests.

### Rule 4: Always create a Tag for releases, so end users can upgrade on their own schedule

Once the software is ready for a release, create a tag in Git for it. A tag is simply a marker for a certain commit, and it will indicate that the software is at a stable, releasable point. Follow these steps to release manually:

1.  Edit the pom (and all module sub-poms) with the new version number (e.g., 1.0.0). Commit and push this change to the repository.
2.  Tag the commit. In Github, you can do this from the releases page of the project (e.g., https://github.com/OCTRI/development-guide/releases), and this provides a space for adding release notes and instructions for upgrade. Alternatively, you can manually tag the commit - `git tag -a v1.0.1 -m "Release Version 1.0.1"` and then push it to the remote server: `git push origin v1.0.1`
3.  Finally, edit the pom (and all module sub-poms) with a new snapshot version based on the previous release (e.g., 1.0.2-SNAPSHOT). Commit and push.

Continuous integration servers or use of the [Maven release plugin](MAVEN-RELEASE-PLUGIN.md) can also automate much of this. With the tag pushed to the remote server, a user can check it out at any point and be confident that they are using a stable, unchanging version of the software. Checkout works just like a branch, except that the user isn't able to make changes to the tag:

```git checkout v1.0.0```

### Rule 5: Use semantic versioning (MAJOR.MINOR.PATCH) to indicate the impact of the changes.

The release version chosen should follow semantic versioning standards. API changes that are not backward-compatible require a MAJOR version bump. Features that are backward-compatible get a MINOR bump. And backward-compatible bug fixes merit a PATCH. [https://semver.org/](https://semver.org/)

Ideally, release notes are tracked and published, particularly any breaking changes to the API. Github has this functionality in the Release tab.

### Rule 6: Release early and often

Each project will have individual release schedules based on the unique properties of the team and stakeholders. In general, though, it is recommended that a first release occur soon after the software is being consumed and used by other projects. This allows those projects the promise of stability using tags while new features are added and bugs are fixed. It is also advised that tested and approved changes get released on a regular basis. This makes the process of upgrading smoother and prevents improvements from languishing before end users can take advantage of them. Releases are cheap and easy - use them!

### Rule 7: Never make direct changes to a tagged release, even if a critical bug is discovered

Even with careful attention to detail, sometimes critical bugs can be introduced in a release. While it may be tempting to delete the tag, this can have a negative downstream impact on users. Once a release is tagged, you have essentially created a contract with the end user promising that no changes will be made to that version. The best approach in this case is to create a new branch from the tag, fix the bug, and then create a new release indicating it should be used instead. Update release notes on previous versions.

```
git checkout -b v1.0.2 v1.0.1 (Checks out new branch v1.0.2 from tag v1.0.1)
...Make changes in branch including changing pom(s) version to "1.0.2"...
git commit -m "Critical bug fix for v1.0.1"
git tag -a v1.0.2 -m "Release Version 1.0.2"
git push -u origin v1.0.2
```

Note that in this instance, the release occurred from the branch and not from main as previously advised. Main may be on an entirely different release when the bug is discovered, so this release from the tag in question is important for downstream users. The bug must also be fixed in master and potentially in any other releases that occurred in the intervening period. The general rule of thumb is to update the most recent version in the major line, since MINOR and PATCH versions should not introduce any breaking changes. So if you've released Version 2.1.1 when you discover a critical bug in the 1.x.x line, only grab the most recent 1.x.x tag and fix it. If the bug is also in the 2.x.x line, create a branch 2.1.2 from tag 2.1.1 and release and also make sure the change is persisted in master going forward.

