### 指定 revision

---

> ref: https://git-scm.com/docs/git-rev-parse.html#_specifying_revisions

git 命令中的 \<rev> 参数通常是指一个 commit object. 它使用的语法被称为 `extended SHA-1`。这里有多种方式指定对象名称的方法。

---



#### \<sha1>

---

> e.g.:  dae86e1950b1277e545cee180551750029cfe735 或者 dae86e

可以通过 git object 的 SHA-1 值或者这个哈希值的前缀来指定一个 object. 使用前缀时需要确保这个仓库中有多个 object 拥有相同的前缀

---



#### \<describeOutput>

---

>e.g.:   v1.7.4.2-679-g3bee7fb

这里的格式和 `git describe` 相同。

---



#### \<refname>

---

>e.g.: 
>
>​    master
>
>​    heads/master
>
>​    refs/heads/master

使用符号名(A symbolic ref name)来指定某个 git object。

例如， **master** 通常指向 refs/heads/master 引用的 commit object。

如果你碰巧同时有 heads/master 和 tags/master，你可以显式地使用 heads/master 来告诉 Git 你指的是哪一个

当你指定的符号名存在二义性时， Git ;通过以下规则中的第一个匹配来消除歧义:

1. 当 \$GIT_DIR/\<refname> 存在时，便会匹配到此项。这通常只对一下 object 有用：HEAD，FETCH_HEAD, ORIG_HEAD， MERGE_HEAD， CHERRY_PICK_HEAD

2. 尝试匹配 refs/\<refname>

3. 尝试匹配 refs/tags/\<refname>

4. 尝试匹配 refs/heads/\<refname>

5. 尝试匹配 refs/remotes/\<refname>

6. 尝试匹配 refs/remotes/\<refname>/HEAD

   **HEAD** 指向 working tree 中的修改所基于的 commit object

   **FETCH_HEAD** 记录你最后一次调用 `git fetch` 从远程存储库中获取数据的分支。

   **ORIG_HEAD** 它由那些会移动 HEAD 的命令创建。 它用户记录 HEAD移动之前的位置，使得您可以轻松地将分支回退到修改 HEAD 之前的位置.

   **MERGE_HEAD** 当你运行 `git merge` 时，MERGE_HEAD 会记录那些你将要合并到目标分支的 commit objects.

   **CHERRY_PICK_HEAD** 当你运行 `git cherry-pick` 时，CHERRY_PICK_HEAD 记录那些 cherry-pick 的 commit objects

   注意，上述描述的匹配 refs/* 的规则，匹配到的结果可能来自 \$GIT_DIR/refs 或者 \$GIT_DIR/packed-refs。

----



#### @

---

单个 @ 等同于 HEAD

---



#### [\<refname>]@{\<data>}

---

>e.g.: 
>
>​    master@{yesterday}
>
>​    HEAD@{5 minutes ago}
>
>​    HEAD@{1 month 2 weeks 3 days 1 hour 1 second ago}
>
>​    HEAD@{1979-02-26 18:30:00}

这种形式用来指定 ref 在之前时间点的值。 这个后缀只能在 ref 名称后面使用，而且 ref 必须有一个已存在的日志(	\$GIT_DIR/logs/\<ref>)。

注意，这将在给定时间查找 local ref 的状态。 例如，上周你们 local master 分支上有什么? 如果您想查看在特定时间内的 commits，请参阅 --since 和 --until。

---



#### [\<refname>]@{\<n>}

----

>e.g.: master@{1}

这种形式用于指定 ref 之前的第 n 个值。

例如， master@{1} 表示于 master 紧邻的前一个值。 master@{5} 表示在 master 之前的第 5 个值。

这个后缀只能在 ref 名称后面使用，而且 ref 必须有一个已存在的日志(	\$GIT_DIR/logs/\<ref>)。

----



#### @{\<n>}

---

> e.g.: @{1}

这种形式用来指定当前分支的 reflog。

例如， 如果你当前处于 blabla 分支，那么 @{1} 等同于 blabla@{1}

----



#### {-\<n>}

---

> e.g.: @{-1}

The construct *@{-\<n>}* means the \<n>-th branch/commit checked out before the current one.

---



#### [\<branchname>]@{upstream}

----

> e.g.: 
>
> ​    master@{upstream}
>
> ​    @{u}

The suffix *@{upstream}* to a branchname (short form *\<branchname>@{u}*) refers to the branch that the branch specified by branchname is set to build on top of (configured with `branch.<name>.remote` and `branch.<name>.merge`). A missing branchname defaults to the current one. These suffixes are also accepted when spelled in uppercase, and they mean the same thing no matter the case.

---



#### [\<branchname>]@{push}

---

> e.g.: 
>
> ​    master@{push}
>
> ​    @{push}

The suffix *@{push}* reports the branch "where we would push to" if `git push` were run while `branchname` was checked out (or the current `HEAD` if no branchname is specified). Since our push destination is in a remote repository, of course, we report the local tracking branch that corresponds to that branch (i.e., something in `refs/remotes/`).

看几个例子：

```bash
$ git config push.default current
$ git config remote.pushdefault myfork
$ git switch -c mybranch origin/master

$ git rev-parse --symbolic-full-name @{upstream}
refs/remotes/origin/master

$ git rev-parse --symbolic-full-name @{push}
refs/remotes/myfork/mybranch
```

---



#### \<rev>^[\<n>]

----

> e.g.:
>
> ​    HEAD^
>
> ​    v1.5.1^0

\<rev>^ 用来指定特定 commit object 的一个父 object。^\<n> 表示第 n 个父 object。

\<rev>^ 等同于 \<rev>\^1

\<rev>\^0 表示当前 commit object 自身。

---



#### \<rev>~[\<n>]

---

> e.g.:
>
> ​    HEAD~
>
> ​    master~3

\<rev>~ 用于指定特定 commit object 的第一个父 object。A suffix *~\<n>* to a revision parameter means the commit object that is the \<n>-th generation ancestor of the named commit object, following only the first parents. 

例如，\<rev>~3 等同于 \<rev>\^\^\^, 又等同于 \<rev>\^1\^1\^1

---



#### \<rev>~[\<type>]

---

> e.g.: v.099.8\^{commit}

后缀^后面跟着用大括号对括起来的对象类型名意味着在\<rev>处的 object 上递归解析，直到找到一个类型为\<type> 的object， 或找不到该 object。

例如， 如果 \<rev> 是一个 commit-ish, \<rev>^{tree} 解析为对应的 commit object。类似的，如果 \<rev> 是一个 tree-ish, \<rev>^{tree} 解析为对应的 tree object。 

\<rev>^0 是 \<rev>^{commit} 的简写

\<rev>^{object} 可以用于判断 \<rev> 是否代表一个 object。

\<rev>^{tag} 可以用于判断 \<rev> 是否代表了一个 tag object。

---



#### \<rev>~{}

----

> e.g.: v.099.8\^{}

后缀^后跟空括号对意味着该 object 可能是一个 tag，然后递归地解引用该标签，直到找到一个非标签对象。

---



#### \<rev>~{/\<text>}

---

> e.g.: HEAD^{/fix nasty bug}

此形式等同于 :/\<text>, 区别在于此形式可以指定一个 \<rev>。

---



#### :/\<text>

---

> e.g.: :/fix nasty bug

使用 \<text> 作为正则表达式进行匹配查找特定的 commit 。

这个正则表达式可以匹配 commit 消息的任何部分。要匹配以字符串开头的消息，可以使用例如*:/^foo*。

---



#### \<rev>:\<path>

---

> e.g.: 
>
> ​    HEAD:README
>
> ​    master:./README

在给定的命名对象中寻找指定路径下的 object。

以 ./ 或 .. 开头的路径表示相对于当前工作目录的路径。

----



#### :[\<n>:]\<path>

---

> e.g.:
>
> ​    0:README
>
> ​    :README

  \<n> 表示 stage number。 0-3.

这种形式用来指定在给定的路径下特定 index的对象。

在合并期间，stage 1 表示公共祖先，stage 2 是目标分支的版本(通常是当前分支)，stage 3 是来自被合并的分支的版本。



最后，我们来看一个例子：

```bash
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A
```

```c
A =      = A^0
B = A^   = A^1     = A~1
C =      = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```



