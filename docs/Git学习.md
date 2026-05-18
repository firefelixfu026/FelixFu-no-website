# 自学笔记

## Git
### 版本控制（前情提要）
#### 版本控制系统
- 人肉VCS：复制粘贴大法
- LVCS：Local，打补丁，e.g.RCS（Revision Control System）
- CVCS：Centralized，依赖中央服务器，e.g.SVN（Subversion），CVS
- DVCS：Distributed，e.g.Git,Mercurial

#### 分支模型
- 常驻分支：master，dev
- 活动分支：feature，hotfix，release

### Git 结构
![](https://zju-xlab.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTBlNjU5NTZiZjY5OWUwOGQ1YThkMjM0MGNkMjRlZGNfSDZyVGFGMktvWmVaczlQQU5RVGFSU0N4OXhTeGFjenFfVG9rZW46Rmx0Q2JsQ2xnb0JoTGh4bHpCNWM0bUY3bkpnXzE3Nzc1MTgyNDA6MTc3NzUyMTg0MF9WNA)

工作区->暂存区（stage，实现分批按逻辑提交）->本地仓库->远程仓库

### 基本命令

```Bash
#git账号配置（设置用户名与邮箱）
git config --global user.name "YourName"
git config --global user.email "email@example.com" #全局配置

git config user.name "YourNmae"
git config user.email "email@example.com" #针对某一版本库专门配置

#查看当前所有配置
git config --list
```

```Bash
#创建本地git仓库
git init     #让当前文件夹变成git仓库（创建.git文件夹）
git init folder  #创建一个新的文件夹并初始化为git仓库
```

```Bash
#查看当前工作区与暂存区状态
git status  #Untracked、Tracked、Ignored

#将文件加入暂存区
git add file/folder #添加指定文件
git add .      #添加当前目录下的所有修改

#将暂存内容提交到本地仓库，生成一个新节点
git commit               #默认编辑器编辑提交信息
git commit -m "message"  #内容自定（提交附带信息）

#删除文件
git rm           #删除物理文件和git索引
git rm --cached  #仅删除git索引

#查看提交历史
git log             #完整日志
git log --oneline   #每个提交一行
git log --graph     #显示分支结构
git log --stat      #显示文件删改信息
git log -p          #显示详细的修改内容
```

```Bash
#查看分支（*表示当前分支）
git branch               #本地分支
git branch -r            #远程分支
git branch -a            #所有分支
git show-branch          #详细查看
git branch -vv           #查看本地分支与远程分支的追踪关系

#创建分支
git branch <BRANCH>      #基于当前HEAD
git branch <BRANCH> id   #基于id

#切换分支
git checkout <BRANCH>
git checkout -b <BRANCH>  #创建并切换
git checkout -            #回到上一个分支

#删除分支
git branch -d <BRANCH>    #删除本地分支

#内容比较
git diff branch1 branch2  #比较两个分支
git diff <BRANCH>         #比较工作区和分支
git diff                  #比较工作区和暂存区
git diff --cached [<reference>] #暂存区与某次提交的差异，默认为HEAD

#合并分支
git checkout <BRANCH>        #切回目标分支
git merge branch1 branch2... #将多个分支的更改都合并到当前分支
```

```Bash
#查看仓库
git remote -v   # 查看当前关联的远程仓库（显示名称与 URL）
git remote add origin https://github.com/username/repo.git  # 添加远程仓库关联（通常命名为 origin）
git remote set-url origin https://github.com/username/new-repo.git # 修改远程仓库的 URL
git remote remove origin   # 删除远程仓库关联

# 克隆仓库
git clone https://github.com/user/repo.git     #HTTPS
git clone git@github.com:user/repo.git         #SSH
git clone https://github.com/user/repo.git dir #指定目录名

#拉取/抓取分支
git pull origin main     # 获取远程更新并自动合并到当前分支
git pull --rebase origin main  # 使用变基合并，保持提交历史线性整洁
git fetch origin         # 只下载远程更新，但不合并（安全，用于查看变化）

#推送分支
git push origin main     # 将本地提交推送到远程
git push -u origin main  # 初次推送并建立关联（以后只需 git push 即可）
git push -f origin main  # 用本地覆盖远程（危险操作，需谨慎使用）

# 如果 push 被拒绝，通常是因为远程有新提交，先 pull：
git pull origin main
# (解决冲突后...)
git add .
git commit -m "Fix conflicts"
git push origin main
```

```Python
# 修改最近一次提交
git commit --amend -m "new message"      # 修改最后一次提交的 message
git commit --amend --no-edit             # 将暂存区文件加入上次提交，但不修改 message
git add file.py && git commit --amend    # 漏传文件时，补加文件并合并到上次提交

# 撤销提交（通过产生新提交来抵消）
git revert <commit_id>                   # 产生一个新提交来撤销指定 id 的改动（安全，不破坏历史）
git revert HEAD                          # 撤销最近的一次提交
git revert -n <commit_id>                # 撤销改动但暂不自动生成新提交（需手动 commit）

# 重置历史（回退指针）
git reset --soft <commit_id>             # 撤销提交，代码保留在“暂存区”（推荐，不丢代码）
git reset --mixed <commit_id>            # 撤销提交，代码保留在“工作区”（默认模式）
git reset --hard <commit_id>             # 彻底回退到指定 id，该 id 之后的修改全部丢失（慎用）
git reset --hard HEAD^                   # 回退到上一个版本

# 交互式变基（整理、合并历史）
git rebase -i <commit_id>                # 对指定 id 之后的提交进行操作（打开编辑器）
git rebase -i HEAD~3                     # 对最近的 3 个提交进行合并、删除或修改
# 在编辑器内：
# pick -> 保持提交
# squash -> 将此提交合并到前一个提交
# drop -> 删除此提交

# 灾难恢复（查看所有操作记录）
git reflog                               # 列出所有分支变动记录（包括已被 reset 掉的提交）
git reset --hard <lost_id>               # 配合 reflog 找回因 reset --hard 丢失的代码
```

```Python
# 储藏操作
git stash                # 储藏当前未提交的修改（不包括未追踪文件）
git stash save "message" # 储藏并添加备注（推荐，方便查找）
git stash -u             # 储藏修改，并连同“未追踪文件”（新创建的文件）一起储藏
git stash -a             # 储藏所有修改，包括“忽略文件”中的变化

# 查看与对比
git stash list           # 列出所有储藏的记录（显示索引如 stash@{0}）
git stash show           # 显示最近一次储藏的改动详情
git stash show -p        # 显示最近一次储藏的具体代码差异

# 恢复储藏
git stash apply          # 应用最近一次储藏，但【不删除】储藏记录
git stash apply stash@{1} # 应用指定的某次储藏
git stash pop            # 应用最近一次储藏，并【删除】该储藏记录
git stash pop stash@{1}  # 应用指定的某次储藏并删除

# 清理储藏
git stash drop stash@{0} # 删除指定的某次储藏记录
git stash clear          # 清空所有的储藏记录
```

```Bash
# 1. 基础匹配规则 (Basic Rules)
secret.txt          # 忽略所有目录下名为 secret.txt 的文件
/root_only.txt      # 仅忽略根目录下的 root_only.txt
build/              # 忽略所有目录下名为 build 的文件夹及其内容

# 2. 通配符模式 (Wildcards)
*.log               # 忽略所有以 .log 结尾的文件
config.[0-9]        # 忽略 config.0 到 config.9
test[AB].js         # 忽略 testA.js 和 testB.js，不包括 testC.js
file-?.md           # 忽略 file-1.md, file-a.md 等（? 匹配单个字符）
**/logs/*.txt       # 忽略任何路径下 logs 目录内的所有 .txt 文件

# 3. 取反与排除 (Negation)
*.pdf               # 忽略所有 .pdf 文件
!important.pdf      # 但排除（不忽略）important.pdf
# 注意：若父目录已被忽略，其子文件的取反规则将失效

# 4. 转义字符 (Escaping)
\#README#.md        # 忽略以 # 开头的文件（# 原本是注释符）
\[data\].csv        # 忽略包含方括号的文件（[ ] 原本是匹配符）

# 5. 常见实战组合
bin/                # 忽略编译输出文件夹
*.exe               # 忽略特定后缀的可执行文件
.vscode/            # 忽略编辑器配置文件
.env                # 忽略敏感的环境变量文件
!.env.example       # 但保留环境变量的模板示例文件
```

---

# 思考题&作业题

## Git
1. 大家可以看到，我们在文中将 Git 与 GitHub/GitLab 并列介绍。版本控制系统与代码托管平台分别解决什么问题？为什么会用 Git 不等于会协作开发？

Git:版本控制工具，管理代码
GitHub/GitLab:协作平台，进行代码托管和共享，方便多人协作

会用Git只说明你能管理好自己的代码，但不一定保证你能和他人按照一套约定好的任务流程进行开发（如代码冲突，分支管理，流程规范，远程同步）

2. 什么是小驼峰命名？什么是snake_case命名？

小驼峰命名：第一个单词以小写字母开头，后续每个单词的首字母大写，其余字母小写，单词之间不使用任何分隔符。用于JavaScript、Java、C#。e.g.theFirstVariable
snake_case命名：所有字母均小写，单词之间用下划线分割，不使用其他任何分隔符。用于Python、Ruby、SQL、JSON。e.g.the_first_variable

3. Commit时，"feat"、"fix"等分别表示什么含义？

|          |      |                     |                       |
| -------- | ---- | ------------------- | --------------------- |
| 前缀       | 含义   | 使用场景                | 示例                    |
| feat     | 新功能  | 新增功能、模块、页面等         | feat：添加可视化模块          |
| fix      | 修复   | 修复bug、错误            | fix：修复提交页面不及时刷新问题     |
| docs     | 文档   | 仅文档更改               | docs：更新API接口文档        |
| style    | 代码风格 | 不影响代码逻辑的样式修改        | style：调整代码缩进          |
| refactor | 重构   | 既不是新功能也不是bug修复的代码修改 | refactor：优化用户查询逻辑     |
| perf     | 性能优化 | 提升性能的代码修改           | perf：优化图片加载性能         |
| test     | 测试   | 添加或修改测试用例           | test：添加用户付款测试         |
| chore    | 杂项   | 构建过程、工具、依赖的修改       | chore：更新webpack配置     |
| ci       | 持续集成 | CI/CD配置文件或脚本的修改     | ci：添加GitHub Actions配置 |
| build    | 构建系统 | 影响构建系统或外部依赖的修改      | build：升级Vue到3.0       |

4. 本地版本管理：    
    1. 新建一个 git 仓库
    2. 在 main 分支上进行一版更新
    3. 新建一个有辨识度的 branch
    4. 分别在 main 分支与 branch 分支上提交一版更新，保证这两个更新会产生冲突
    5. Merge 自建 branch 到 main 分支，处理 conflict
    6. 在 main 分支中提交一版更新
    7. 将 branch 同步到 main 分支的最新更新上
---

# 进度同步

## 自学同步
### 第一周（4月27日-5月3日）
- 学习内容：git使用（4.26线下教学+线上浏览、4.28-29笔记编写）
- 思考&经验：之前在学习中有使用过Git，可以使用一些简单的命令。通过本次详细的学习，尤其是课后的Practice，我对于分支管理等功能有了更深的理解。相信以后能处理好项目的协作。