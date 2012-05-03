All contributions to Sheepdog are sent as patches to the sheepdog mailing list <sheepdog@lists.wpkg.org>.

* Follow the Linux kernel coding style [[http://www.kernel.org/doc/Documentation/CodingStyle]]
* Add Signed-off-by line by the committer
* Send patches inline so they are easy to reply to with review comments. Do not put patches in attachments.
* Patches should be against current git master
* Split up longer patches into a patch series of logical code changes
* Don't include irrelevant changes
* When resending patches add a v2/v3 suffix (eg [PATCH v2])

## Submitting patches via git

The Sheepdg project uses git for version control and it is helpful for submitted patches to follow the git work flow to optimize the opportunity for review and merge.

### Setup git

install git and git-email via your distribution tools. In RPM distributions, this can be done via
```
 # yum install git
 # yum install git-email
```
Add the following to your ~/.gitconfig so that commits appear to come from you and are signed with your name and email address.
```
[user]
        name = yourname
        email = yourlogin@youremail
```

### Setup repository

```
$ git clone git://github.com/collie/sheepdog.git
```

### Make your suggested changes

```
$ cd sheepdog
Make changes to the repository which represent the bug or feature you want to add.
$ git commit -s -a
```
After the editor opens with the list of changed files, enter a commit log.

The commit log should follow this format:

Enter a one line description of the change

<enter>

Enter a multiple line description of the patch and rationale

Save the results, and send the commit to the mailing list:
```
$ git send-email --to=sheepdog@lists.wpkg.org --smtp-server=YOURSMTPSERVERHERE -1
```
This will send the patch to the list and format it properly for merging into the tree

### If changes are requested to patch

Please don't take personal offense if changes are asked in a patch. We have many users and we don't want to break the tree with changes.
```
Make suggested changes in the repository
$ git commit -s -a --amend
```
Add amended description to commit log
```
$ git send-email --to=sheepdog@lists.wpkg.org --smtp-server=YOURSMTPSERVERHERE -1
```
