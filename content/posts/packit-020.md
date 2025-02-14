---
title: "Packit 0.2.0 is here!"
date: 2019-03-19T18:25:39+01:00
draft: false
---

Our sprint nears an end which means we have released [a new version of packit -
0.2.0](https://github.com/packit-service/packit/releases/tag/0.2.0)! You can
expect a new release after every sprint (i.e. every 2 weeks).

The 0.2.0 version has a bunch of new features and improvements: you can find
a complete list [in the
changelog](https://github.com/packit-service/packit/blob/master/CHANGELOG.md#020).
We also have a [detailed
documentation](https://github.com/packit-service/packit#workflows-covered-by-packit)
for all the workflows packit covers.
<!--more-->

Let's get through what's new:

1. We have decided to rename two keys in our config file so they are more
   descriptive. Old names still work but they are deprecated:
    * `package_name` → `downstream_package_name`
    * `upstream_name` → `upstream_project_name`
2. You don't need to touch dist-git at all when getting your new upstream
   release into Fedora, you can stay in your upstream repository and just fire
   off a bunch of packit calls:
    * `packit propose-update` to create a pull request in Fedora dist-git with
      the selected upstream release
    * `packit build` to build the new upstream release once the pull request is
      merged
    * and finally, `packit create-update` creates a new bodhi update (if you
      chose a stable Fedora release)
3. Packit now has a `srpm` command which creates an SRPM out of the local
   content of your upstream repository.
4. You can now use packit to sync files from your dist-git repo back into
   upstream (mainly to keep spec files in sync). `sync-from-downstream` is the
   command.
5. Command `propose-update` received numerous improvements:
    * You can pick upstream version to use.
    * Packit will NOT check out the git ref with the upstream release if you
      specify `--local-content`
    * It's possible to force packit to execute `fedpkg new-sources` using
      `--force-new-sources` and bypass the caching mechanism.


## Installation

Please make sure you are installing `0.2.0`:
```
$ dnf install --enablerepo=updates-testing packit
```

Or

```
$ pip3 install --user packitos
```

You can also install packit from master branch, if you are brave enough:

```
$ pip3 install --user git+https://github.com/packit-service/packit
```


## How are we using packit?

I'd like to show you how we used packit to bring a new upstream
release of ogr into Fedora, a library which packit is using.

Once we have performed an upstream release of ogr, we can propose an update in
dist-git:
```
$ git clone https://github.com/packit-service/ogr && cd ogr/

$ packit propose-update
INFO: Running 'anitya' versioneer
Version in upstream registries is '0.0.3'.
Version in spec file is '0.0.3'.
Picking version of the latest release from the upstream registry over spec file.
Checking out upstream version 0.0.3
Using 'master' dist-git branch
Cloning repo: https://src.fedoraproject.org/rpms/python-ogr.git -> /tmp/tmpb9xlvdhj
Syncing /home/tt/g/user-cont/ogr/python-ogr.spec
Archive ogr-0.0.3.tar.gz found in lookaside cache (skipping upload).
ERROR    Cmd('git') failed due to: exit code(1)
  cmdline: git commit -s -m [packit] 0.0.3 upstream release -m Upstream tag: 0.0.3
Upstream commit: 059d21080a7849acff4626b6e0ec61830d537ac4

  stdout: 'On branch 0.0.3-master-update
nothing to commit, working tree clean'
```

Whoops, it seems that I have messed up, I forgot to bump the spec file in the
upstream repo when doing the release. I will bump it locally and utilize
`--local-content` argument:
```
$ rpmdev-bumpspec -n 0.1.0 -c 'New upstream release: 0.1.0' *.spec

$ packit propose-update --local-content
INFO: Running 'anitya' versioneer
Version in upstream registries is '0.0.3'.
Version in spec file is '0.1.0'.
Picking version of the latest release from the upstream registry over spec file.
Using 'master' dist-git branch
Cloning repo: https://src.fedoraproject.org/rpms/python-ogr.git -> /tmp/tmpd9j4se27
Syncing /home/tt/g/user-cont/ogr/python-ogr.spec
Archive ogr-0.1.0.tar.gz found in lookaside cache (skipping upload).
INFO: Downloading file from URL https://files.pythonhosted.org/packages/source/o/ogr/ogr-0.1.0.tar.gz
100%[=============================>]    20.25K  eta 00:00:00
Downloaded archive: '/tmp/tmpd9j4se27/ogr-0.1.0.tar.gz'
About to upload to lookaside cache
won't be doing kinit, no credentials provided
PR created: https://src.fedoraproject.org/rpms/python-ogr/pull-request/6
```

Once the scratch build is done and tests passed we merged and built it:
```
$ packit build
Using 'master' dist-git branch
Cloning repo: https://src.fedoraproject.org/rpms/python-ogr.git -> /tmp/tmprp3cmdjy
Building python-ogr-0.1.0-1.fc31 for rawhide
Created task: 33616980
Task info: https://koji.fedoraproject.org/koji/taskinfo?taskID=33616980
```

We have done the same for F30 and F29.

The previous commands were ran in the directory of the upstream repository. Packit also accepts path to your upstream clone, or even URL. So let's create a bodhi update for python-ogr by specifying the upstream repo URL:
```
$ cd $HOME

$ packit create-update --dist-git-branch f29 https://github.com/packit-service/ogr
Cloning repo: https://github.com/packit-service/ogr -> /tmp/tmpdkdadmn_
Koji builds for package python-ogr and koji tag f29-updates-candidate:
 - python-ogr-0.1.0-1.fc29
Cloning repo: https://src.fedoraproject.org/rpms/python-ogr.git -> /tmp/tmpn1809ec9
Bodhi update FEDORA-2019-78948e62d2:
- https://bodhi.fedoraproject.org/updates/FEDORA-2019-78948e62d2
- stable_karma: 3
- unstable_karma: -3
- notes:
New upstream release: 0.1.0
```

And that's it, no need to access dist-git any more.

Please give packit a try and let us know what you think.
