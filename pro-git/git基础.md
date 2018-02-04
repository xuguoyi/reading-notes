### 获取git仓库
- **git init**
>在现有目录中初始化仓库,可通过`git add`命令来实现对指定文件的跟踪

- **git clone https://github.com/1v5/reading-notes.git**
>克隆现有的仓库

### 记录每次更新到仓库
- **git status**
>查看哪些文件处于什么状态

  - `-s`或`--short`
  >状态简览

- **git add**
>跟踪新文件,暂存已修改文件

- **git diff**
>查看尚未暂存的文件更新了哪些部分

  - `--cached`或`--staged`
  >查看已暂存的将要添加到下次提交的内容

- **git commit**
>提交更新

  - `-m`
  >可顺带补充提交信息

  - `-a`
  >跳过`git add`步骤

- **git rm**
>移除文件

  - `-f`
  >强制删除修改过且已放到暂存区域的文件

  - `-r`
  >删除整个文件夹

  - `--cached`
  >从仓库及暂存区中删除,但在磁盘中保留文件

- **git mv file_from file_to**
>移动文件

### 查看提交历史
