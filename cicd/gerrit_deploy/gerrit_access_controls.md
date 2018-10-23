# gerrit权限说明

- Abandon

此权限允许用户丢弃一个提交的change。如果用户有push权限，给用户分配此权限的同时用户也被分配了restore a change的权限。

- Create Reference

此权限管理用户是有可以创建references，branches，tags。此权限一般与普通的push权限一起被分配。

- Forge Author

伪造发起人权限，此权限允许用户绕过提交时的身份验证（Gerrit默认会匹配提交信息中author或者committer行中的email地址，如果 Email地址不匹配，则不允许提交）。

- Forge Committer

伪造提交者权限，此权限允许用户绕过提交时的身份验证（Gerrit默认会匹配提交信息中author或者committer行中的email地址，如果 Email地址不匹配，则不允许提交 ）。

- Forge Server

伪造Gerrit服务器权限，此权限允许在committer行中使用server owner和email

- Owner

此权限允许用户修改香项目的配置，具体如下：

修改项目描述
通过ssh的"create-branch"命令创建分支
在web UI界面创建/删除branch
允许/撤销任何访问权限，包括Owner权限。

- Push

此分类控制用户被允许怎样推送新commit到Gerrit。

- Direct Push

所有已存在的branch可以快进到新的commit。创建新分支受“Create Reference”控制，不允许删除已存在的分支，这是最安全的模式（因为commit不可以被丢弃）。

- Force option

允许已存在的branch被删除。开启此选项可以从项目历史中删除提交记录。
此权限主要用来给那些只想用Gerrit的访问控制，不需要Gerrit的代码审查功能的工程使用。

- Upload To Code Review

此push权限分配在refs/for/refs/heads/BRANCH命名空间上，允许用户提交一个未合并(non-merge)的commit到refs/for/BRANCH命名空间，创建一个新的代码审查change。
用户必须能够clone和fetch一个工程才可以提交change，所以用户还必须拥有Read权限。

- Push Merge Commits

此权限允许用户提交merge commits，它是Push权限的附属物，如果想只允许通过Gerrit做merge操作，那么应该只分配Push仅限而不分配此权限。

- Push Annotated Tag

此类权限允许用户向工程仓库提交一个annotated tag。通常使用以下两种方式提交：

git push ssh://USER@HOST:PORT/PROJECT tag v1.0
或者:
git push https://HOST/PROJECT tag v1.0
Tags必须被注释（使用git tag -a），必须在refs/tags/下存在，而且必须是新的。
一般在工程达到了稳定且可发布的时候会打一个Tag。
此权限允许创建一个未签名的Tag。打Tag者的email地址必须与当前用户的一致。
如果要提交不是自己打的Tag，则必须同时分配Forge Committer - - Identity权限

如果要提交轻标签(lightweight tags)分配Create Reference权限给引用/refs/tags/*
如果要删除或覆盖一个已存在的tag，分配Push权限并开启Force option。

- Push Signed Tag

此类权限允许用户向工程仓库提交一个PGP签名的 tag。通常使用以下两种方式提交：

git push ssh://USER@HOST:PORT/PROJECT tag v1.0
或者:
git push https://HOST/PROJECT tag v1.0
Tags必须被注释（使用git tag -a），必须在refs/tags/下存在，而且必须是新的。

- Read

此类权限控制工程的changes， comments，和code diffs可见性，和是否可通过SSH或HTTP访问Git。
如果在单独工程的ACL中设置的此权限，那么全局ACL中的设置将不起作用。

- Rebase

此类仅限允许用户通过web页面的“Rebase Change”按钮衍合（Rebase）修改

- Remove Reviewer

此类权限允许用户在一个change的reviewers list中移除其他用户。
change所属者可以移除0分或负分的reviewers（即使没有此权限）。
项目所有者和网站管理员可以移除所有reviewers（即使没有此权限）。
没有此权限的用户只可以移除自己。

- Review Labels

// TODO

- Submit

此类权限允许用户提交changes。
提交一个change会使该change尽可能快的合并到目的分支，使其作为项目历史永久的一部分。
为了提交change,所有的labels都必须允许提交，并且不能block它。
如果要快速提交一个push上的change，用户需要在refs/for/<ref>(e.g. on refs/for/refs/heads/master)有此权限。

- Submit(On Behalf Of)

此类权限允许有Submit权限的用户代表其他用户提交change。
在project.config文件中，此权限被命名为submitAs。

- View Drafts

此类权限允许用户查看其他用户提交的drafts changes
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）

- Publish Drafts

此类权限允许用户发布其他用户提交的drafts changes
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）

- Delete Drafts

此类权限允许用户删除其他用户提交的drafts changes
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）

- Edit Topic Name

允许用户编辑提交到review的change的话题名。
change所用者，分支所用者，项目所用者和网站管理员都可以编辑此话题名（即使没有此权限）。
“Force Edit”标识控制是否可以编辑已关闭的change标题，如果此标识设置只能编辑open changes，则不可以编辑已关闭的change 标题。

- Edit Hashtags

允许用户在提交到reviews的changes上添加或移除hashtags。
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）