# What is this?
The stable branches of sheepdog are long term supported versions for production users. A maintainer of the stable branches will backport latest bug fixes to the branches.
Each major release has its stable branch. e.g. stable branch of 0.6 is [stable-0.6](https://github.com/collie/sheepdog/tree/stable-0.6)

Because the release cycle of sheepdog is corresponding to the cycle of QEMU, each stable version has its corresponding one. e.g. stable-0.6 of sheepdog corresponds to stable-1.5 of QEMU.
These stable branches of sheepdog will be maintained until corresponding stable branches of QEMU become obsolete or as long as possible.

# Obtain stable release based on the stable branches
| Branch | Latest Release (tag)      | Corresponding stable version of QEMU | Release Date |
|:-----------|:-----------|:-----------|:-----------|
| [stable-0.9] (https://github.com/sheepdog/sheepdog/tree/stable-0.9) | [v0.9.0] (https://github.com/sheepdog/sheepdog/releases/tag/v0.9.0) | [stable-2.1.2] (http://git.qemu.org/?p=qemu.git;a=shortlog;h=refs/heads/stable-2.1.2) - [tar.bz2](http://wiki.qemu-project.org/download/qemu-2.1.2.tar.bz2) | 10/31/2014 |
| [stable-0.8] (https://github.com/sheepdog/sheepdog/tree/stable-0.8) | [v0.8.3] (https://github.com/sheepdog/sheepdog/releases/tag/v0.8.3) | [stable-1.7] (http://git.qemu.org/?p=qemu.git;a=shortlog;h=refs/heads/stable-1.7) - [tar.bz2](http://wiki.qemu-project.org/download/qemu-1.7.2.tar.bz2) | 08/21/2014 |
| [stable-0.7] (https://github.com/sheepdog/sheepdog/tree/stable-0.7) | [v0.7.8] (https://github.com/sheepdog/sheepdog/releases/tag/v0.7.8) | [stable-1.6] (http://git.qemu.org/?p=qemu.git;a=shortlog;h=refs/heads/stable-1.6) | missing |
| [stable-0.6] (https://github.com/sheepdog/sheepdog/tree/stable-0.6) (obsolete) | [v0.6.3] (https://github.com/sheepdog/sheepdog/releases/tag/v0.6.3) | [stable-1.5] (http://git.qemu.org/?p=qemu.git;a=shortlog;h=refs/heads/stable-1.5) | missing |

# Maintenance policy of the branches
## Basic maintenance policy
A maintainer of the stable branches sends backported patches to sheepdog@lists.wpkg.org and sheepdog-users@lists.wpkg.org (only a cover letter will be sent to the users list). The subject of the emails will contain a prefix like [PATCH stable-x.y].
- If the patches are bugfixes, they will be pushed to the branches immediately.
- If the patches are for implementing new features, they will not be pushed to the branches directly. If no one disagrees with backporting them in 7 days, they will be pushed to the branches officially. Basically, backporting features should be avoided.

## Emergent bug fixes
When emergent bugfixes are merged to the master branch, emergent releases of stable branches can be created. 

In such a case, 2 days are allocated for reviewing window. If no one complains about the emergent releases in the window, tags for the releases will be created.

# Release plans in the near future

## stable-0.6 based releases
| Date       | Event                                                                           |
|:-----------|:--------------------------------------------------------------------------------|
| ~~2013-08-29~~ | Apply [the bugfix of recovery progress] (https://github.com/mitake/sheepdog/tree/stable-0.6-for-2013-08-29) and tag v0.6.2.       |
| ~~2013-09-06~~ | Tag v0.6.3       |

## stable-0.7 based releases
| Date       | Event                                                                           |
|:-----------|:--------------------------------------------------------------------------------|
| ~~2013-09-06~~ | Tag v0.7.3       |
| ~~2013-10-18~~ | Tag v0.7.4_rc0       |
| ~~2013-10-31~~ | Tag v0.7.5_rc0       |
| ~~2013-12-17~~ | Tag v0.7.6_rc0       |
| ~~2014-02-02~~ | Tag v0.7.7_rc0       |
| ~~2014-03-03~~ | Tag v0.7.8_rc0       |

## stable-0.8 based releases
| Date       | Event                                                                           |
|:-----------|:--------------------------------------------------------------------------------|
| ~~2014-03-03~~ | Tag v0.8.1_rc0       |
| 2014-05-31 | Tag v0.8.2_rc0       |

## stable-0.9 based releases
| Date       | Event                                                                           |
|:-----------|:--------------------------------------------------------------------------------|
| ~~2014-09-29~~ | Tag v0.9.0_rc0       |
| 2014-10-06 | Tag v0.9.0       |
