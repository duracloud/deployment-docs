# DuraCloud Release Steps

## Preparing for release

* Assign a Release Manager, the person who will be responsible for the release process. The Release Manager must be a DuraCloud committer.
* Ensure all code commits have been pushed and merged
* Issue a code freeze statement ensuring all developers know not to push further changes

## Perform version update

Set release version
* `mvn versions:set -DnewVersion=X.X.X`
* `mvn versions:commit`

Commit release version update (Git)
* Commit version update with commit message: Updates version to X.X.X
* Create duracloud-X.X.X tag
* Checkout master branch, merge tagged commit from develop
* Checkout develop branch

Set snapshot version
* `mvn versions:set -DnewVersion=X.X.X-SNAPSHOT`
* `mvn versions:commit`

Commit snapshot version update (Git)
* Commit version update with commit message: Updates version to X.X.X-SNAPSHOT for continued development
* Push changes to github
  ```
    git push upstream develop --tags
    git push upstream main  
  ```

## Verify automated deployment
* Ensure main build in Git Actions CI completes successfully (~25 minutes): https://github.com/duracloud/duracloud/actions
* Ensure new version is available in maven central: https://search.maven.org/#search%7Cga%7C1%7Corg.duracloud
  * If deployment to Sonatype was not completed properly by the CI build, it can be executed locally.
  * Check out the release tag commit, then run: `mvn deploy -DreleaseBuild -DskipTests -DskipDeploy`
* Ensure new version is created on github releases (with all the expected artifacts): https://github.com/duracloud/duracloud/releases
  * If release artifacts aren't sent to the github release by the Travis CI build, you can build them locally and upload to github.
  * Check out the release tag commit, then run: `mvn clean install -DskipTests -Pinstallers`
  * Collect the release artifacts and upload to the github release
    * The contents of the top level /target directory will contain the files you will need to create the beanstalk ZIP.
    * The SyncTool installers (windows-installer.exe, osx-installer.zip, installer jar) will be under synctoolui/target.
    * The Retrieval Tool JAR will be under retrievaltool/target

## Deploy to Production
* Download the duracloud-beanstalk-vX.X.X-*.zip from github releases
* Create a new DuraCloud version in Elastic Beanstalk
* Deploy new version

## Update documentation
* Add release notes to github release: https://github.com/duracloud/duracloud/releases
* Update download links to point to latest release in github: https://wiki.duraspace.org/display/DURACLOUD/DuraCloud+Downloads
* Update documentation based on changes in code: https://wiki.duraspace.org/display/DURACLOUD
