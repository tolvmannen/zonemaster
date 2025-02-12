Release process - Release
=========================

## Table of contents

* [1. Overview](#1-overview)
* [2. Applicable components](#2-applicable-components)
* [3. Updates to repositories](#3-updates-to-repositories)
* [4. Determine the new version number](#4-determine-the-new-version-number)
* [5. Update the Changes files]
* [6. Set version number for Perl modules](#6-set-version-number-for-perl-modules)
* [7. Set version number for GUI](#7-set-version-number-for-gui)
* [8. Update Makefile.PL with required version](#8-update-makefilepl-with-required-version)
* [9. Create a clean Git working area of *develop branch*](#9-create-a-clean-git-working-area-of-develop-branch)
* [10. Produce distribution tarballs](#10-produce-distribution-tarballs)
* [11. Produce distribution zip file](#11-produce-distribution-zip-file)
* [12. Update Zonemaster repository main _README.md_](#12-update-zonemaster-repository-main-readmemd)
* [13. Generate documents and check public documents](#13-generate-documents-and-check-public-documents)
* [14. Upload to CPAN](#14-upload-to-cpan)
* [15. Merge develop branch into master](#15-merge-develop-branch-into-master)
* [16. Create Docker images and upload image to Docker Hub](#16-create-docker-images-and-upload-image-to-docker-hub)
* [17. Tag the release with git](#17-tag-the-release-with-git)
* [18. Announce the release](#18-announce-the-release)
* [19. Merge master into develop](#19-merge-master-into-develop)
* [Appendix A on version number in Makefile.PL](#appendix-a-on-version-number-in-makefilepl)
* [Appendix B on reverting commits](#appendix-b-on-reverting-commits)


## 1. Overview

The steps in this document executes the actual release. They assume that
development, the preparation steps and the QA testing have been concluded.
To run the steps below a build system is needed. See 
[Build Environment Preparation] for how to set it up.

> **Note:** *Normally, the develop branch version of this document should be used.*

[(Top)](#table-of-contents)

## 2. Applicable components

Every applicable step below should be run for each component. If there are no
changes in a component (no release) it should be skipped. The main component
(Zonemaster/Zonemaster) can never be skipped and should always be released.
The components should be released in the following order:

 * zonemaster-ldns
 * zonemaster-engine
 * zonemaster-cli
 * zonemaster-backend
 * zonemaster-gui
 * Zonemaster/Zonemaster

[(Top)](#table-of-contents)

## 3. Updates to repositories

All updates to *develop branch* and *master branch* should go via pull
requests following the usual process. That is just assumed here.

[(Top)](#table-of-contents)

## 4. Determine the new version number

The version number of the new release should be chosen according to
[Versions and Releases] document.

Each component listed above should have its version number. The Zonemaster
Product (Zonemaster/Zonemaster) version number also refer to the version
of the other components.

[(Top)](#table-of-contents)

## 5. Update the Changes files

Any changes since the last release must be documented in the Changes files.
Refer to any GitHub issues or pull requests related to the change by the
issue number or pull request number or both.

The updates to the *Changes* file are done to the *develop branch*.

 * zonemaster-ldns - [Changes LDNS]
 * zonemaster-engine - [Changes Engine]
 * zonemaster-cli - [Changes CLI]
 * zonemaster-backend - [Changes Backend]
 * zonemaster-gui - [Changes GUI]
 * Zonemaster/Zonemaster
   - First update [Changes Zonemaster]
   - Then update the [RELEASE.md] page (use content of *Changes* file, only
     latest release) to match the upcoming [Github release page].
 
[(Top)](#table-of-contents)

## 6. Set version number for Perl modules

> This step does not apply to Zonemaster/Zonemaster and Zonemaster-GUI

The version numbers is to be set in these Perl modules in the *develop branch*:

 * zonemaster-ldns - [LDNS.pm]
 * zonemaster-engine - [Engine.pm]
 * zonemaster-cli - [CLI.pm]
 * zonemaster-backend - [Backend.pm]

[(Top)](#table-of-contents)

## 7. Set version number for GUI

> This step applies to Zonemaster-GUI only

Update the following files in the *develop branch* of **Zonemaster-GUI**:

 * [package.json][package.json GUI]
   - In the top of the file, the version is given after "version".
   - The file `package-lock.json` is ignored
 * [src/environments/version.ts][Version.ts GUI]
   - "version" should point at the version number.

Update the following file in the *develop branch* of **Zonemaster/Zonemaster**:

 * [public/installation/zonemaster-gui.md][Installation.md GUI]
   - The version is part of the download path (a directory). It is repeated
     several times, once per OS.

> The update of the installation document can preferably be done in the same
> pull request as the update of the `Changes` file for Zonemaster/Zonemaster

[(Top)](#table-of-contents)

## 8. Update Makefile.PL with required version

> This section is relevant for Zonemaster-Engine, Zonemaster-CLI and
> Zonemaster-Backend.

If needed, update Makefile.PL in the *develop branch*. See [Appendix A] for
how the version number should be specified in `Makefile.PL`.

Usually let Zonemaster-Engine, Zonemaster-CLI and Zonemaster-Backend,
respectively, require the version of other the Zonemaster components
(see below) included in this release, because that is what has been tested.

If a component will not be updated by this release, then it can continue to
require the previous version unless it must be change to resolve some issue,
and then the version of the requiring component must also be updated.

In **[Zonemaster-Engine Makefile.PL]** set the lowest version of Zonemaster-LDNS
that Zonemaster-Engine requires. E.g.

```
requires 'Zonemaster::LDNS'   => 2.001;
```

In **[Zonemaster-CLI Makefile.PL]** and **[Zonemaster-Backend Makefile.PL]**,
set the minimum required versions of Zonemaster-LDNS and Zonemaster-Engine.
E.g.

```
requires 'Zonemaster::Engine' => 4.000;
requires 'Zonemaster::LDNS'   => 2.001;
```

[(Top)](#table-of-contents)

## 9. Create a clean Git working area of *develop branch*

Make sure that you have checked out the `develop` branch and that your clone is
up-to-date.
```
git fetch --all
git branch
```
Check out the right commit of the submodule (LDNS). Zonemaster-LDNS only.
```
git submodule update
```
Make sure your working directory is clean.
```
git status --ignored
```
Or clean it.
```
git clean -dfx
git reset --hard
```

[(Top)](#table-of-contents)

## 10. Produce distribution tarballs

> This section is not relevant for Zonemaster-GUI or Zonemaster/Zonemaster

See [Release process - Create Test Distribution], sections 4-6 for more details
and what warnings that could be ignored.
```
perl Makefile.PL --no-ed25519 # For Zonemaster-LDNS:
```
```
perl Makefile.PL              # For other components
```
```
make all                      # For all
make distcheck
make dist
```
[(Top)](#table-of-contents)

## 11. Produce distribution zip file

> This section is relevant for Zonemaster-GUI only.

For this you need a [build environment for Node.js], on which you create
the zip file. See [Release process - Create Test Distribution], sections 7
for more details

Build the distribution zip file:
```
npm install
npm run release
```

> You can ignore warnings and security fixes at this stage, and do not run 
> any `npm audit fix`.

The distribution zip file is in the root level of the zonemaster-gui folder. 
Its name is `zonemaster_web_gui.zip`.

[(Top)](#table-of-contents)

## 12. Update Zonemaster repository main _README.md_

> This section is relevant for Zonemaster/Zonemaster only.

If needed, update the following section of the Zonemaster repository main
[README.md][Zonemaster main README] file in *develop branch*:

* Notable bugs and issues

[(Top)](#table-of-contents)

## 13. Generate documents and check public documents

> The two sub-sections here are relevant for Zonemaster/Zonemaster only.

### 13.1. Generate documents

If no files in neither Zonemaster/Zonemaster nor Zonemaster-Engine have been
updated this section can be skipped.

1. On a computer install Zonemaster-LDNS and Zonemaster-Engine using the
   distribution packages created for this release.  
2. Clone Zonemaster/Zonemaster and check out the *develop branch*. The
   working area should be clean (`git status --ignore`).
3. Go to the [utils][utils Zonemaster] directory and create the files as
   documented in the [README.pm][Zonemaster utils README] file in that
   directory.
4. If any of the created files has been updated (`git status`) then it
   should be added to the *develop branch* via a pull request.
5. No reviewer or approval is required for this change.

### 13.2 Check public documents

> The step above, 13.1, should be completed (merged to develop branch) before
> this step is run to avoid double errors.

Prior to this step `mdbook-linkcheck` must be installed (see
[Build environment preparation]). Run from the Zonemaster/Zonemaster
repository (develop branch checked out):

```
mdbook-linkcheck -s docs/public/
```

1. If any error is reported, correct the file or files.
2. Add changes to the *develop branch* via a pull request.
3. No reviewer or approval is required for this change.
4. Repeat the command after merging the changes to verify that the found errors
   have been resolved.

[(Top)](#table-of-contents)

## 14. Upload to CPAN

> This section is not relevant for Zonemaster-GUI or Zonemaster/Zonemaster.

For each component that is to be updated in this release, publish the
distribution tarball created following this document on CPAN.
Currently we use the organizational account [ZNMSTR] on [PAUSE] for doing
this.

[(Top)](#table-of-contents)

## 15. Merge develop branch into master

> For the steps in this section, it is assumed that the git "remote"
> which is called "origin" points at "zonemaster/zonemaster.git",
> "zonemaster/zonemaster-ldns.git" etc. This is default when making a
> git clone. Use `git remote -v` to verify that.

Make sure you're up to date and your working directory is completely clean:

    git fetch origin
    git status --ignored

> **Note:** To throw away any and all changes to tracked and untracked files you
> can run `git clean -dfx ; git reset --hard`.

Verify if there are commits on `master` since last release:

    TAG=$( git tag --list --sort=-creatordate | head -n 1 )
    # count the number of commits in master since last release and
    # if the count is not 0, look for commits belonging to master and develop
    [ $(git rev-list --count $TAG..master) -ne 0 ] && cat <( git rev-list $TAG^2..develop ) <( git rev-list $TAG..master ) | sort | uniq -d

If commits are returned by this previous command, some additional work is
needed. See [Appendix B] for that.

Finally once `master` branch is ready, create a pull request from `develop`
into `master` and merge it. No reviewer or approval is required for this
update.

[(Top)](#table-of-contents)

## 16. Create Docker images and upload image to Docker Hub

1. Follow the instructions in [Create Docker Image] to build a Docker images.
2. Upload the Zonemaster-CLI image to [Docker Hub] for Zonemaster using the
   instructions in [Create Docker Image].

[(Top)](#table-of-contents)

## 17. Tag the release with git

For each repository, go to "releases" in GitHub and select "draft a new release".
Use the version number as tag and create a new release description. Use the
section of Changes file for the relevant release and make links of everything 
that can have meaningful links, especially make links to issues and PRs.

For Zonemaster-GUI, add the *distribution zip file* as attached file to the
release description in GitHub.

Always release the Zonemaster Product (Zonemaster/Zonemaster) as the last step.

The releases pages:

* [Zonemaster-LDNS Releases]
* [Zonemaster-Engine Releases]
* [Zonemaster-CLI Releases]
* [Zonemaster-Backend Releases]
* [Zonemaster-GUI Releases]
* [Zonemaster Product Releases] - use the [RELEASE.md] created in
  [step 5][5. Update the Changes files].

[(Top)](#table-of-contents)

## 18. Announce the release

Send emails to the mailing lists `zonemaster-users` and `zonemaster-announce`.

[(Top)](#table-of-contents)

## 19. Merge master into develop

Create a pull request from `master` on github back into `develop` and merge
it. No review or approval is required for this update.

[(Top)](#table-of-contents)

## Appendix A on version number in Makefile.PL

As described above, `Makefile.PL` of Zonemaster-CLI and Zonemaster-Backend,
respectively, must be specified with the lowest supported version of
Zonemaster::LDNS and Zonemaster::Engine, respectively. Zonemaster-Engine
must have that of Zonemaster::LDNS.

The versions of Zonemaster::LDNS and Zonemaster::Engine are defined in the
format `vX.Y.Z` and it is important how this is written in `Makefile.PL`.

For lowest risk of error follow the following steps:

1. Expand the version number with leading zeros on second ("Y") and third
   ("Z") level. Remove the leading `v` and the second and third dots.
   E.g. `v2.1.2` = `2.001002`.
2. Decide if the third level can be skipped, e.g. `v2.1.2` > `2.001`.
3. Please note that if the version in `Makefile.PL` is set to `2.1` then
   the interpretation is that the "v" version is `v2.100.0` or greater.

When specifying the version of Zonemaster::LDNS and Zonemaster::Engine in
`Makefile.PL` always use the expanded format, e.g. `2.001002` or `2.001`
depending on the need.

For other libraries, other formats may be correct. If the version of the
library only has two levels ("X.Y") then other rules apply. Also note
that the version in Zonemaster::Backend is specified as `X.Y.Z` without
the "v", which may affect the version comparison.

[(Top)](#table-of-contents)

## Appendix B on reverting commits

When merging `develop` into `master` a merge conflict could occur.  This is due
to the fact that some commits belongs to both branches. Usually this is the
result of a merge request in `master` branch instead of `develop` during
development. Once this happens, the faulty merge is reverted on `master` and
merged into `develop`. Depending on the steps chosen, this could lead to having
the same commit in both branches.

To avoid merge conflict, it is necessary to revert the revert commits on
`master` (yes revert of the revert) before merging `develop` into `master`.

### How would the release officer find the commit to revert?

Usually a revert commit contains a predefined subject (or title) looking like:
`Revert "subject of the reverted commit"`.

One can looks for commits with this string using a command like:

    TAG=$( git tag --list --sort=-creatordate | head -n 1 )
    for c in $(cat <( git rev-list $TAG^2..develop ) <( git rev-list $TAG..master ) | sort | uniq -d); \
        do git log --oneline --grep="Revert \"$(git show -s --pretty=%s $c)\"" $c~..master ^$TAG ; \
        done

Or look for them visually with:

    git log --oneline --graph $TAG..master

Once found, either revert each commits (`git revert <commit> ...`) or the merge
commit.

### How would the release officer know the parent number to revert?

In case one reverts the merge commit, since it was merged into master, the
parent number is `1`.

    git revert -m 1 <merge commit>


[(Top)](#table-of-contents)

<!-- Zonemaster links point on purpose on the develop branch. -->
[5. Update the Changes files]:                   #5-update-the-changes-files
[Appendix A]:                                    #appendix-a-on-version-number-in-makefilepl
[Appendix B]:                                    #appendix-b-on-reverting-commits
[Backend.pm]:                                    https://github.com/zonemaster/zonemaster-backend/blob/develop/lib/Zonemaster/Backend.pm
[Build environment preparation]:                 ../distrib-testing/README.md
[Build environment for Node.js]:                 ../distrib-testing/Ubuntu-Node.js-build-environment.md
[CI]:                                            https://github.com/travis-ci/travis-ci
[CLI.pm]:                                        https://github.com/zonemaster/zonemaster-cli/blob/develop/lib/Zonemaster/CLI.pm
[CPAN]:                                          https://www.cpan.org/
[Changes Backend]:                               https://github.com/zonemaster/zonemaster-backend/blob/develop/Changes
[Changes CLI]:                                   https://github.com/zonemaster/zonemaster-cli/blob/develop/Changes
[Changes Engine]:                                https://github.com/zonemaster/zonemaster-engine/blob/develop/Changes
[Changes GUI]:                                   https://github.com/zonemaster/zonemaster-gui/blob/develop/Changes
[Changes LDNS]:                                  https://github.com/zonemaster/zonemaster-ldns/blob/develop/Changes
[Changes Zonemaster]:                            https://github.com/zonemaster/zonemaster-gui/blob/develop/Changes
[Create Docker Image]:                           ReleaseProcess-create-docker-image.md
[Docker Hub]:                                    https://hub.docker.com/u/zonemaster
[Engine.pm]:                                     https://github.com/zonemaster/zonemaster-engine/blob/develop/lib/Zonemaster/Engine.pm
[Github release page]:                           https://github.com/zonemaster/zonemaster/releases/latest
[Installation.md GUI]:                           ../../public/installation/zonemaster-gui.md
[LDNS.pm]:                                       https://github.com/zonemaster/zonemaster-ldns/blob/develop/lib/Zonemaster/LDNS.pm
[PAUSE]:                                         https://pause.perl.org/pause/query
[Package.json GUI]:                              https://github.com/zonemaster/zonemaster-gui/blob/develop/package.json
[RELEASE.md]:                                    ../../public/RELEASE.md
[Release process - Create Test Distribution]:    ReleaseProcess-create-test-distribution.md
[Utils Zonemaster]:                              ../../../utils/
[Version.ts GUI]:                                https://github.com/zonemaster/zonemaster-gui/blob/develop/src/environments/version.ts
[Versions and releases]:                         ../design/Versions%20and%20Releases.md
[ZNMSTR]:                                        https://metacpan.org/author/ZNMSTR
[Zonemaster Product Releases]:                   https://github.com/zonemaster/zonemaster/releases
[Zonemaster main README]:                        ../../../README.md
[Zonemaster utils README]:                       ../../../utils/README.md
[Zonemaster-Backend Makefile.PL]:                https://github.com/zonemaster/zonemaster-backend/blob/develop/Makefile.PL
[Zonemaster-Backend Releases]:                   https://github.com/zonemaster/zonemaster-backend/releases
[Zonemaster-CLI Makefile.PL]:                    https://github.com/zonemaster/zonemaster-cli/blob/develop/Makefile.PL
[Zonemaster-CLI Releases]:                       https://github.com/zonemaster/zonemaster-cli/releases
[Zonemaster-Engine Makefile.PL]:                 https://github.com/zonemaster/zonemaster-engine/blob/develop/Makefile.PL
[Zonemaster-Engine Releases]:                    https://github.com/zonemaster/zonemaster-engine/releases
[Zonemaster-GUI Releases]:                       https://github.com/zonemaster/zonemaster-gui/releases
[Zonemaster-LDNS Releases]:                      https://github.com/zonemaster/zonemaster-ldns/releases


