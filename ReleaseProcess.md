# Release Process

1. Update the packages kokoro job to point to the latest release
2. Update all or some of the package version numbers
   - All of the package changelogs need to have the proper release listed in the
     changelog after the package number.
3. Create the release branch in all projects
4. Create the release branch in continuous-build
5. All package repositories we have updated need to have git tags
   - Format of the tag is the same as the package version in the
     debian/changelog file.
6. Run the packages kokoro job on beaker
   - Looks at the beaker branch looking for the latest versions and updates that way
7. Tag the latest rapture with the beaker tag.

Cherry picks:
1. Commit to master
2. Cherry pick to next branch
3. All packages we have updated need to have tags
4. Run the packages kokoro job on next
   - Looks at the next branch looking for the latest versions and updates that way
5. Tag the latest rapture with the next tag.
