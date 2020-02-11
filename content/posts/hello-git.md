+++
title = "Hello Git"
date = 2020-02-11T11:51:00+08:00
draft = false
tags = []
categories = []
+++

Git 学习。

<!--more-->

先来看一个视频[Git内部是如何工作的？Git的内部数据结构](https://www.youtube.com/watch?v=EejUbjFXm-g)

## What is git?

> Git is a fast, scalable, distributed revision control system with an unusually rich command set that provides both high-level operations and full access to internals.

`ref: https://github.com/git/git/blob/master/README.md`

- Porcelain (厕所的瓷砖, 指露在外面的意思, 或者说已经组合好的，面向用户的指令)
  - user-friendly commands: init, add, commit, branch, merge, etc. 
- Plumbing (修水管的工具, 指藏在后面的意思, 面向开发者的组件)
  - bunch of verbs that do low level work: hash-object, update-index, write-tree, etc.

补充:
- 定义: git 本身并不单纯是revision control system， 更主要的使用来做content tracker, 只是很多人，包括作者本身用它来做revision control
- Code Review: 需要工具加持

发展过程:
- diff and patch
- Tranditional SCM, Server Repo
- Distributed SCM, Git 

## Git Database

- Content addressable filesystem
- Simple key-value data store
- Key: SHA-1 hash (Everything is hashed)
  - 20 bytes, 40 hex, **160 bit**, 2.9e48 distinct keys
- Value: binary files
  - **Commits**: Actual git commits
  - **Trees**: Structure of file system
  - **Blobs**: content of files/data


### git init

```sh
➜  hello-git mkdir demo1
➜  hello-git cd demo1 
➜  demo1 git init 
Initialized empty Git repository in /Users/nonocast/Desktop/hello-git/demo1/.git/
➜  demo1 git:(master) git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
➜  demo1 git:(master) tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── ...
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

8 directories, 15 files
```

等同于
```sh
➜  demo2 mkdir .git
➜  demo2 echo 'ref: refs/heads/master' > .git/HEAD
➜  demo2 mkdir -p .git/objects/
➜  demo2 mkdir -p .git/refs/
➜  demo2 git:(master) git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

手动的方式你会发现，其实最小最小的git repo是1个文件加2个目录。

### git add

```sh
➜  demo1 git:(master) echo 'Hello World' > hello.txt
➜  demo1 git:(master) git add hello.txt
```

你可以观察到.git/objects下会多出一个目录和一个文件，这个称为刚才文件内容的blob, 目录加上文件名40个字符构成SHA1 hash, 可以根据这个SHA1 hash查看对应文件的内容,

```sh
➜  demo1 git:(master) ✗ git cat-file -p 557d
Hello World
```

也可以尝试根据文件内容获取SHA1 hash, 与文件名和文件信息无关
```sh
➜  demo1 git:(master) ✗ echo "Hello World" | git hash-object --stdin
557db03de997c86a4a028e1ebd3a1ceb225be238
```


文件信息和内容是分开的，
```sh
➜  demo1 git:(master) git ls-files --stage
100644 557db03de997c86a4a028e1ebd3a1ceb225be238 0	hello.txt
```


### Area & Workflow

Area:
- Working Directory
- Staging Area
- .git directory (Repository)

Workflow:
- git add: Working > Staging
- git commit: Staging > Repository
- git checkout: Working < Repository

Staging Area对应到.git/index
```sh
➜  demo1 git:(master) ✗ file .git/index
.git/index: Git index, version 2, 1 entries
➜  demo1 git:(master) ✗ git ls-files --stage
100644 557db03de997c86a4a028e1ebd3a1ceb225be238 0	hello.txt
```

如果需要从操作index, 即操作Staging Area
```sh
git rm --cached hello.txt
git update-index --add --cacheinfo 100644 <hash> hello.txt
```

现在回过来看如何手动git add

```sh
➜  demo2 git:(master) tree .git
.git
├── HEAD
├── objects
│   └── 55
│       └── 7db03de997c86a4a028e1ebd3a1ceb225be238
└── refs

3 directories, 2 files
➜  demo2 git:(master) git update-index --add --cacheinfo 100644 557db03de997c86a4a028e1ebd3a1ceb225be238 hello.txt
➜  demo2 git:(master) ✗ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   hello.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    hello.txt

➜  demo2 git:(master) ✗ git checkout -- hello.txt
➜  demo2 git:(master) ✗ cat hello.txt 
Hello World
```

显示deleted: hello.txt, 是因为Staging存在的hello.txt不存在于Working Area中, 然后通过checkout把这个文件从Repository中写到Working Area中。

## git commit

git commit 后会生成两个文件, 一个tree, 一个commit

```sh
➜  demo1 git:(master) git log                          
commit 1a4dac33dcf0f20af514f0302a439765162cc75b (HEAD -> master)
Author: nonocast <nonocast@gmail.com>
Date:   Wed Feb 12 02:50:28 2020 +0800

    initial
➜  demo1 git:(master) git cat-file
usage: ...

<type> can be one of: blob, tree, commit, tag
    -t                    show object type
    -s                    show object size
    -e                    exit with zero when there's no error
    -p                    pretty-print object's content
    ...

➜  demo1 git:(master) git cat-file -t 1a4d
commit
➜  demo1 git:(master) git cat-file -p 1a4d
tree 97b49d4c943e3715fe30f141cc6f27a8548cee0e
author nonocast <nonocast@gmail.com> 1581447028 +0800
committer nonocast <nonocast@gmail.com> 1581447028 +0800

initial

➜  demo1 git:(master) git cat-file -t 97b4
tree
➜  demo1 git:(master) git cat-file -p 97b4
100644 blob 557db03de997c86a4a028e1ebd3a1ceb225be238	hello.txt
```

查看所有objects,
```sh
➜  demo1 git:(master) git cat-file --batch-check --batch-all-objects      
1a4dac33dcf0f20af514f0302a439765162cc75b commit 166
557db03de997c86a4a028e1ebd3a1ceb225be238 blob 12
97b49d4c943e3715fe30f141cc6f27a8548cee0e tree 37
```

git logs 的起点是 `HEAD -> refs/master -> commit`

## Commit again

```sh
➜  demo1 git:(master) mkdir src
➜  demo1 git:(master) echo "echo 'hello world'" > src/hello.sh
➜  demo1 git:(master) ✗ tree .
.
├── hello.txt
└── src
    └── hello.sh

1 directory, 2 files
➜  demo1 git:(master) ✗ git add .
➜  demo1 git:(master) ✗ git ls-files --stage 
100644 557db03de997c86a4a028e1ebd3a1ceb225be238 0	hello.txt
100644 59c1fdfa8cd0b24760f0577903d0ac6cbb9b35e2 0	src/hello.sh
➜  demo1 git:(master) ✗ git commit -m 'add shell script'
[master 5708e5b] add shell script
 1 file changed, 1 insertion(+)
 create mode 100644 src/hello.sh

➜  demo1 git:(master) git log 
commit 5708e5b09d2d36cc1f0452932925266e3dd9d4a5 (HEAD -> master)
Author: nonocast <nonocast@gmail.com>
Date:   Wed Feb 12 03:10:35 2020 +0800

    add shell script

commit 1a4dac33dcf0f20af514f0302a439765162cc75b
Author: nonocast <nonocast@gmail.com>
Date:   Wed Feb 12 02:50:28 2020 +0800

    initial

➜  demo1 git:(master) git cat-file -p 5708
tree 46dc4ca4ec01d621e4a9e81f114a7b85474b0c22
parent 1a4dac33dcf0f20af514f0302a439765162cc75b
author nonocast <nonocast@gmail.com> 1581448235 +0800
committer nonocast <nonocast@gmail.com> 1581448235 +0800

add shell script
➜  demo1 git:(master) git cat-file -p 46dc
100644 blob 557db03de997c86a4a028e1ebd3a1ceb225be238	hello.txt
040000 tree c139cb17fbafa806c82102a32f640e94e54aa93d	src

➜  demo1 git:(master) git cat-file -p c139
100644 blob 59c1fdfa8cd0b24760f0577903d0ac6cbb9b35e2	hello.sh

➜  demo1 git:(master) git ls-files --stage
100644 557db03de997c86a4a028e1ebd3a1ceb225be238 0	hello.txt
100644 59c1fdfa8cd0b24760f0577903d0ac6cbb9b35e2 0	src/hello.sh
```
index需要等到下次add会做lazy update, commit并不会clean index, 这个不很重要, 知道就行。

所以:
- File content -> blob
- Directory -> tree (藏在tree的文件路径中，不单独保存)
- snapshot of current file system -> commit

Directed Acyclic Graph (有向无环图)
- N-ary tree (N叉树)
- Linked List (链表)

## Rollback

git revert会增加一个commit表示rollback, 重点是可以rollback这个rollback

```sh
➜  demo1 git:(master) git revert HEAD

➜  demo1 git:(master) git log   
commit 774072600e135ba902565c17cdfe23645dc17214 (HEAD -> master)
Author: nonocast <nonocast@gmail.com>
Date:   Wed Feb 12 03:24:52 2020 +0800

    Rollback Second Commit
    Revert "add shell script"
    
    This reverts commit 5708e5b09d2d36cc1f0452932925266e3dd9d4a5.

commit 5708e5b09d2d36cc1f0452932925266e3dd9d4a5
Author: nonocast <nonocast@gmail.com>
Date:   Wed Feb 12 03:10:35 2020 +0800

    add shell script

commit 1a4dac33dcf0f20af514f0302a439765162cc75b
Author: nonocast <nonocast@gmail.com>
Date:   Wed Feb 12 02:50:28 2020 +0800

    initial
➜  demo1 git:(master) 
```

留下git reset的三种方式, mixed, soft和hard。

## Branch

branch就是一个refs, 非常cheap.

## Merge (Important!)

merge的过程:
- Find intersection of 2 linked lists (找到两个linked list共同的base(commit节点))
- 找到两个轨迹中hash不同的文件，然后做diff, 然后apply到Working Area
- 然后产生Conflicts, 这时候Repository不会发生任何变化
- merge成功后会产生一个commit, 这个commit有两个parent


# Git is the stupid content tracker

> In many ways you can just see git as a filesystem - it's content-
addressable, and it has a notion of versioning, but I really really
designed it coming at the problem from the viewpoint of a _filesystem_
person (hey, kernels is what I do), and I actually have absolutely _zero_
interest in creating a traditional SCM system.  
>
> --- **Linus Torvalds**

[https://marc.info/?l=linux-kernel&m=111314792424707](https://marc.info/?l=linux-kernel&m=111314792424707)