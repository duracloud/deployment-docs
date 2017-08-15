# DuraCloud Release Steps

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

## Verify automated deployment
* Ensure tag build in Travis-CI completes successfully (~25 minutes): https://travis-ci.org/duracloud/duracloud/builds
* Ensure new version is available in maven central: https://search.maven.org/#search%7Cga%7C1%7Corg.duracloud
* Ensure new version is created on github releases (with all the expected artifacts): https://github.com/duracloud/duracloud/releases

## Deploy to Production
* Download the duracloud-beanstalk-vX.X.X-*.zip from github releases
* Create a new DuraCloud version in Elastic Beanstalk
* Deploy new version

## Update documentation
* Add release notes to github release: https://github.com/duracloud/duracloud/releases
* Update download links to point to latest release in github: https://wiki.duraspace.org/display/DURACLOUD/DuraCloud+Downloads
* Update documentation based on changes in code: https://wiki.duraspace.org/display/DURACLOUD
