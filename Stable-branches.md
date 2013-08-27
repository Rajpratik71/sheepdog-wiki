# What is this?
The stable branches of sheepdog are long term supported versions for production users. A maintainer of the stable branches will backport latest bug fixes to the branches.
Each major release has its stable branch. e.g. stable branch of 0.6 is [stable-0.6](https://github.com/collie/sheepdog/tree/stable-0.6)

Because the release cycle of sheepdog is corresponding to the cycle of QEMU, each stable version has its corresponding one. e.g. stable-0.6 of sheepdog corresponds to stable-1.5 of QEMU.
These stable branches of sheepdog will be maintained until corresponding stable branches of QEMU become obsolete.

# Maintenance policy of the stable branches
## Basic maintenance policy
A maintainer of the stable branches sends backported patches to sheepdog@lists.wpkg.org and sheepdog-users@lists.wpkg.org. The subject of the emails will contain a prefix like [PATCH stable-x.y].
- If the patches are bugfixes, they will be pushed to the branches immediately.
- If the patches are for implementing new features, they will not be pushed to the branches directly. If no one disagrees with backporting them in 7 days, they will be pushed to the branches officially. Basically, backporting features should be avoided.

## Emergent bug fixes
- When emergent bugfixes are merged to the master branch, emergent releases of stable branches can be created. In such a case, 2 days are allocated for reviewing window. If no one complains about the emergent releases in the window, tags for the releases will be created.

# Release plans of stable-0.6 in the near future

| Date       | Event                                                                           |
|:-----------|:--------------------------------------------------------------------------------|
| 2013-08-29 | Apply [the bugfix of recovery progress] (https://github.com/mitake/sheepdog/tree/stable-0.6-for-2013-08-29) and tag v0.6.2.       |
| 2013-09-06 | Tag v0.6.3       |

# Release plans of stable-0.7 in the near future

| Date       | Event                                                                           |
|:-----------|:--------------------------------------------------------------------------------|
| 2013-09-06 | Tag v0.7.3       |