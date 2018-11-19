# Using the maven-release-plugin to automate releases

In the best-case scenario, your project is supported by developer operations and a Continuous Integration server that can track commits, run tests, and manage release and deployment of new versions of the software.

If this infrastructure is not in place, there are still ways to automate some processes. Assuming your project uses maven for building, the maven-release-plugin can handle the release process described [here](README.md).

The plugin's `prepare` step will update pom versions, create a release tag, and then edit all the poms again to reflect the new snapshot version.

## One-Time Setup Instructions

### Set Up a GPG Public Key in Github

Part of the release process is to "sign" the released artifacts so that downloading developers can verify them. The release plugin uses the maven-gpg-plugin under the covers for signing. To use it, you have to generate a key locally and provide the public key to Github.

Here are Github's instructions for generating a new key. Take care to remember the passphrase you set. You will need both this and your Github credentials to complete the release.

https://help.github.com/articles/generating-a-new-gpg-key/

Once it is generated, follow the link in the instructions to add the key to Github.

### Add Release Plugin and SCM to Pom

The following lines should be added to the main pom.xml file, replacing the developerConnection with a pointer to the repo you want to release. Submodule poms do not need to be updated. 

```
<project>
  ...
  <scm>
    <developerConnection>scm:git:https://github.com/aeyates/phenol.git</developerConnection>
  </scm>
   
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-release-plugin</artifactId>
        <version>2.5.3</version>
      </plugin>
    </plugins>
    ...
  </build>
  ...
</project>
```

This change must be committed and pushed to master before the release process begins, since a release cannot occur on a branch with uncommitted changes.

Other common options are listed here: http://maven.apache.org/maven-release/maven-release-plugin/usage.html

Use the "dryRun" option first if you want to see what will happen without making changes.

## Release Setup

Now that the one-time setup is complete, it's time to release. If you encounter errors during this process, you can use mvn release:rollback to start over. Or just start from a clean checkout.

There is one issue with the prepare step that we have to deal with first.

### Command-Line GPG Passphrase Entry

Long term, you probably want to set the GPG passphrase automatically instead of through the command line. Instructions for that are here: http://maven.apache.org/plugins/maven-gpg-plugin/usage.html

But if you use this command-line approach, you may encounter the following error:

```"gpg: signing failed: Inappropriate ioctl for device".```

This is because some part of the process is trying to open an interactive window and doesn't have permission:

https://issues.apache.org/jira/browse/MGPG-59?attachmentOrder=asc

Note that this does not happen with all versions of gpg, but the latest version from Homebrew throws this error. Before running the prepare step, run the following command as advised in the ticket:

```gpg --use-agent --armor --detach-sign --output $(mktemp) pom.xml```

This essentially signs the pom so that the later interactive step can be skipped. You may need to do this each time before running "prepare".

## Release

And now finally, you should be able to run the release:prepare step. A few options are necessary. The flag autoVersionSubmodules will do the work of setting all the versions the same in all the poms, so if you have modules in your project use this. The prepare step also builds everything for release and packages the javadoc. If there are errors in the javadoc, this can stop the release from happening. Fix these if you can, but if not another flag will disable this part of the build.

```mvn release:prepare -DautoVersionSubmodules=true -Darguments="-Dgpg.passphrase=<passphrase> -Dmaven.javadoc.skip=true"```

You'll have three interactive choices to make for versions. Maven provides sane defaults, and if they look good, you can just hit enter:

```
What is the release version for "org.monarchinitiative.phenol:phenol"? (org.monarchinitiative.phenol:phenol) 1.2.5: : 1.2.5
What is SCM release tag or label for "org.monarchinitiative.phenol:phenol"? (org.monarchinitiative.phenol:phenol) phenol-1.2.5: : v1.2.5
What is the new development version for "org.monarchinitiative.phenol:phenol"? (org.monarchinitiative.phenol:phenol) 1.2.6-SNAPSHOT: 
```

The prepare step pushes changes to the remote repository. Once it's finished you will have a tag and release named v1.2.5. All of the poms in that tag will have a version of 1.2.5. The master branch poms will be versioned at 1.2.6-SNAPSHOT.

## Cleanup

The prepare step creates a couple of new files in the main directory - release.properties and pom.xml.releaseBackup. You can set your repository to ignore these in .gitignore to avoid committing them. Or you can clean them up by running:


```mvn release:clean```

It appears that `mvn release:perform` essentially does the same thing, but we have not used this.



