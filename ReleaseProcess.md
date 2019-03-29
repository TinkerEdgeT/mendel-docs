# Release Process

1. Update all or some of the package version numbers
   - All of the package changelogs need to have the proper release listed in the
     changelog after the package number.
2. Create the release branch in all projects
3. Create the release branch in continuous-build
   - Add branch=release-$RELEASE for all projects in .gitmodules
   - git submodule update --remote --init
   - git add -A; git commit
   - git push origin HEAD:refs/heads/release-$RELEASE
4. All package repositories we have updated need to have git tags
   - Format of the tag is the same as the package version in the
     debian/changelog file.
   - git tag $VERSION aiy/release-$RELEASE
   - If the package has separate sources in another repo, you need to tag the
     same way in the source repo.
   - git push --tags aiy
5. Make a new release candidate for the appropriate release
   - If no proposals exist for the release, create a new Rollup proposal and
     approve it.
6. Tag the latest rapture with the $RELEASE tag.
   - rapture --universe cloud-apt settag mendel-core-unstable spacepark-eng.$RELEASE:true
7. Run the kokoro job for $RELEASE.

Cherry picks:
1. Commit to master
2. Cherry pick to next branch
3. All packages we have updated need to have tags
4. Checkout $current-release in continuous
5. `git submodule update --remote --init`
6. Check status, update git submodules, commit
7. Run the rapid release job on for $current-release
   - Looks at the next branch looking for the latest versions and updates that way
8. Tag the latest rapture with the next tag.
