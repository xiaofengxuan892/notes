[TOC]



#### Git常用指令：

拉取**远程仓库指定分支**内容到**<font color=red>本地仓库当前分支</font>**：`git pull origin 分支名`

推送**本地分支**内容到**远程仓库指定分支**：`git push origin 本地分支:远程分支` 

- 如果远程仓库没有该同名分支，则该指令会自动在远程仓库创建同名分支

查看本地仓库所有分支：`git branch` (**“*”代表本地仓库当前所在分支**)

查看**<font color=blue>本地已保存的远程仓库所有分支</font>**：`git branch -a` 

- **需要先拉取远程仓库**，才能展示**<font color=red>最新的远程仓库所有分支情况</font>**

在本地仓库创建新分支：`git branch 分支名`

切换本地分支：`git checkout 分支名` 

- 如果本地没有该分支，则会**<font color=red>自动在本地仓库创建该分支，并拉取远程仓库同名分支到本地，同时关联两个同名分支</font>**；如果本地有该分支，则只会将本地仓库切换到目标分支，并不会自动拉取远程仓库代码



##### git flow分支开发相关指令：

**初始化git flow**： `git flow init` 

- 如果本地仓库与远程仓库的默认分支名不一致，如前者为main，后者为master，则初始化可能失败。此时需要使用 `git config --global init.defaultbranch 分支名` 设置本地仓库默认分支名与远程仓库一致即可

**开启feature功能分支**：`git flow feature start 分支名`

**关闭feature功能分支**：`git flow feature finish 分支名`

- 该指令会将本地仓库该分支删除，同时切换到develop分支，同时远程仓库该分支也会删除。且**<font color=red>该feature分支已提交的内容都会保存到本地仓库的develop分支，需要push到远程仓库才能生效</font>**。故执行关闭feature分支的指令后，还需要 `git push origin develop:develop` 将修改push到远端

**推送feature分支已修改内容到远程**：`git push origin develop:develop`

- 由于项目联合开发，develop分支可能有其他新的提交记录，因此在推送之前先需要使用 **<font color=blue>git pull origin develop 拉取最新内容到本地，之后再push</font>**



##### 使用Git时需要注意的问题：

**1)**.**<font color=red>git创建的分支名首字母不能大写</font>，否则push失败**。如果使用git flow开启feature分支，则**分支名为“feature/分支名”，则必然为小写开头**；但如果直接**使用 git branch 开启新分支**，则**分支命名需要注意不要以大写字母开头**

**2)**.项目根目录下的“.git文件夹”保存该项目本地仓库以及远程仓库的相关信息。如果需要重新关联或初始化，则可删除该文件夹

**3)**.在关联本地项目与远程仓库时调用 `git remote add origin 远程仓库链接` 指令，其中**<font color=red>“origin”指代远程仓库别名</font>**，之后使用其他指令如 `git pull/push origin ....` 等指令时，**<font color=blue>origin即为远程仓库</font>**



##### 其他常用Git指令：

**查看与本地工程相关联的远程仓库地址**：`git remote -v`

**设置git仓库默认分支名字为main**：`git config --global init.defaultbranch main`

**将本地指定分支与远程指定分支关联**：`git branch --set-upstream-to=origin/develop develop`

**下载项目的submodule**：`git submodule update --init --recursive`

**查看git配置信息**： `git config --list`

**查看用户名**：`git config user.name`

**查看邮箱**：`git config user.email`

**设置全局用户名**: `git config --global user.name "xxxx"(用户名)`

**设置全局邮箱**：`git config --global user.email "xxxx"(邮箱)`



#### 退出Git命令窗口的“编辑模式”：

当在Git命令行窗口中输入“git log”或“git config --list”等指令时，会出现“编辑状态”且无法退出，导致不能继续在同一命令行窗口中输入其他指令

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240328103712474.png" alt="image-20240328103712474" style="zoom:80%;" />

**解决方法**：在当前窗口中**<font color=red>按下“q”键</font>**即可退出编辑状态

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240328103821445.png" alt="image-20240328103821445" style="zoom:80%;" />



#### 分支关联

当推送本地仓库指定分支的内容到远程时，如果该分支没有与远程指定分支相关联，则每次都需要手动选定“远程推送分支”，操作比较麻烦。可采用如下方式关联：

**1)**.先将本地仓库切换到需要关联的分支，如main分支等

**2)**.在当前分支中执行git指令：`git branch --set-upstream-to origin/remote_branch`

**查看本地分支与远程分支的关联情况**：`git branch -vv`

**撤销本地当前分支与远程分支的关联**：`git branch --unset-upstream`

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240409123002723.png" alt="image-20240409123002723" style="zoom:80%;" />



#### 分支删除

**删除本地仓库指定分支**：`git branch -d 分支名` 或  `git branch -D 分支名`

**删除远程仓库指定分支**：`git push origin --delete 分支名`  或  `git push origin :分支名`



#### 分支合并

**丢弃本次分支合并**：`git merge --abort`

在合并分支时，有时会遇到冲突或其他情况，此时可能需要丢弃该次合并，即可执行该指令，将本地分支恢复到“合并分支之前的状态” —— 相当于没有执行本次分支合并的操作

​                                       

#### 版本回退

##### 一次性将仓库还原到指定版本，且不保留该版本之后的提交记录

该情况下**<font color=red>目标版本后所有的提交记录都不存在了，完全还原到目标版本的时期</font>**

**操作步骤**：

**1)**.在项目根目录下鼠标右键选择“Open Git Bash here”打开Git客户端，并输入“`git log`”指令查看提交记录：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240327114922162.png" alt="image-20240327114922162" style="zoom:80%;" />

**2)**.使用指令回退至指定版本：

- `git reset --soft`：回退到指定版本，但保留已提交内容到本地

  ```
  git reset --soft acf6abf01d622a89dd14db10a7d03ae842ec9b4f
  ```

  该指令会将本地仓库还原至目标版本A，并将**<font color=red>版本A后提交的内容全部保存到本地文件中。开发人员可在此基础上修改代码后重新提交</font>**

- `git reset --hard`：回退到指定版本，且不保留已提交内容

  ```
  git reset --hard acf6abf01d622a89dd14db10a7d03ae842ec9b4f
  ```

  该指令会将本地仓库还原至版本A，且**<font color=red>该版本后续提交的内容全部被丢弃</font>**

**执行以上指令后，再使用“git log”查看**时可看到**<font color=red>当前最新的提交记录为版本A</font>**

**3)**.使用 `git push origin 本地分支:远程分支` 或 `git push -f` 推送到远程仓库

由于**<font color=red>该操作会将远程仓库还原至目标版本，且该版本之后的提交记录都会丢失</font>**，因此会自动触发分支的“ProtectionRules”导致无法推送，如：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230109160323386.png" alt="image-20230109160323386" style="zoom:67%;" />

此时需要**<font color=blue>先在远程仓库暂时解除该分支的保护规则，在推送完成后再次添加“ProtectionRules”即可</font>**



##### 只还原单次提交的内容，且保留所有提交记录

该情况下**<font color=red>只针对某次提交的内容做还原，且保留所有的提交记录</font>**

**操作步骤**：

**1)**.使用 `git revert xxx` 指令，还原单次修改内容

```
git revert HEAD      //撤销最近一次提交
git revert HEAD~1    //撤销上上次的提交，注意：数字从0开始依次递增
git revert offaacc   //撤销offaacc这次提交(offaacc是该次的commit id，即SHA1码)
```

执行以上指令后则会还原“本地仓库”该次提交的内容**<font color=red>并为本次还原操作生成新的“提交记录”</font>**。此时使用“TortoiseGit -> show log”可查看最新提交记录：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20230109161214281.png" alt="image-20230109161214281" style="zoom: 80%;" />

**2)**.使用 `git push origin 本地分支:远程分支` 或 `git push -f` 推送到远程仓库即可



##### 丢弃本次还原：

部分情况下，当还原某次代码后又需要将其回退，此时可使用 `git revert --abort` 丢弃本次还原

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240327180217735.png" alt="image-20240327180217735" style="zoom:80%;" />





#### 将新项目上传到Github或Gitlab

**操作步骤**：

**1)**.在Github或Gitlab上新建仓库，并勾选“ReadMe”文件

**2)**.在本地项目根目录下执行 `git init` 初始化仓库

**3)**.将该项目与远程仓库关联：`git remote add origin 远程仓库链接`

**PS**：若要断开本地项目与远程仓库的链接可使用 `git remote remove origin`；若要查看本地项目关联的远程仓库地址可使用 `git remote -v`，如下所示：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20240327173143600.png" alt="image-20240327173143600" style="zoom:80%;" />

**4)**.拉取远程仓库最新内容到本地：`git pull --rebase origin 远程仓库分支Alias`

**PS**：**<font color=red>第一次拉取远程仓库</font>**时直接使用 `git pull origin 分支Alias` 可能会失败，在命令中添加“--rebase”即可。后续即可正常使用“git pull origin 分支Alias”拉取远程仓库代码

**5)**.执行以上步骤后，则本地项目与远程仓库即正常关联。之后按照平常流程commit并push到远程仓库即可

**注意**：Github新建仓库默认分支名为“**master**”，Gitlab新建仓库默认分支名为“**main**”，可通过 `git config --global init.defaultbranch 分支名` 来修改本地项目的默认分支名，使得两者保持一致，避免出错







