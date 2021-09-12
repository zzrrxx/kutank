### 指定 revision range

---

> ref: https://git-scm.com/docs/git-rev-parse.html#_specifying_ranges

像 `git log` 这样的历史记录遍历操作一组 commits，而不仅仅是单个 commit。

对于这些命令，指定单个 commit 意味着给定 commit 的“可到达的” commit 集合。

指定多个 commit 意味着可以从任何给定的 commit 获得一组 commit。

commit 的可达集是 commit 本身及其祖先链中的 commit。

有几种表示法可以指定一组连续的 commit (称为“修订范围”)，如下所示。

---



#### Commit Exclusions

---

##### \^\<rev> (caret) Notation

---

排除从 \<rev> 可达的 commits

例如， ^r1 r2 表示从 r2 可到达的，但排除从r1可到达的(即r1及其祖先)。

---



#### Dotted Range Notations

---



##### The .. (two-dot) range Notation

---

`^r1 r2` 指定使用的比较频繁，因此他有一个简写。假设你有两个 commit r1 和 r2， 你可以通过`^r1 r2`选择那些从 r2 可达，但是排序从r1 可达的 commit，它可以写成 `r1..r2`。

----



##### The ... (three-dot) Symmetric Difference Notation

---

`r1...r2` 称为 r1，r2 的 symmetric difference. 定义为 *r1 r2 --not \$(git merge-base --all r1 r2)*

它表示一组 commits，可以从 r1 (左边)或 r2 (右边)访问，但不能从两者都访问。



> 上述两个命令的左边可以省略，默认为 HEAD。

例如， origin.. 表示 origin..HEAD, ..origin 表示 HEAD..origin, .. 表示 HEAD..HEAD

---



#### Other \<rev>^ Parent Shorthand Notations

---

还有其他三种简写方式，特别适用于合并 commit，用于指定由 commit 及其父 commit 组成的集合。

r1^@ 表示 r1 的所有父 commit

r1^！表示包含 r1 自身但是排除它的所有父 commit.

\<rev>^-[\<n>] 表示包含 \<rev> 但是排除前 n 个父 commit， 是 \<rev>^\<n>..\<rev> 的简写。

---



#### Revision Range Summary

---

- \<rev> : 包括从\<rev>(即\<rev>及其祖先)可达的 commits。
- ^\<rev>: 排除从\<rev>(即\<rev>及其祖先)可达的 commits。
- \<rev1>..\<rev2>: 包括从\<rev2>可以到达的 commit，但排除从\<rev1>可以到达的 commit。 当 \<rev1> 或者 \<rev2> 省略时，默认为 HEAD
- \<rev1>...\<rev2>: 包括从\<rev1>或\<rev2>可以到达的提交，但排除从两者都可以到达的提交。当 \<rev1> 或者 \<rev2> 省略时，默认为 HEAD
- \<rev>^@: 列出\<rev>的所有父 commits (意思是包括从其父节点可访问的任何内容，但不包括提交本身)
- \<rev>^!: A suffix *^* followed by an exclamation mark is the same as giving commit *<rev>* and then all its parents prefixed with *^* to exclude them (and their ancestors).
- \<rev>^-\<n>: 等同于 \<rev>^\<n>..\<rev>



下面看一些例子：

```bash
Args   Expanded arguments    Selected commits
D                            G H D
D F                          G H I J D F
^G D                         H D
^D B                         E I J F B
^D B C                       E I J F B C
C                            I J F C
B..C   = ^B C                C
B...C  = B ^F C              G H D E B C
B^-    = B^..B
	   = ^B^1 B              E I J F B
C^@    = C^1
	   = F                   I J F
B^@    = B^1 B^2 B^3
	   = D E F               D G H E F I J
C^!    = C ^C^@
	   = C ^C^1
	   = C ^F                C
B^!    = B ^B^@
	   = B ^B^1 ^B^2 ^B^3
	   = B ^D ^E ^F          B
F^! D  = F ^I ^J D           G H D F
```

