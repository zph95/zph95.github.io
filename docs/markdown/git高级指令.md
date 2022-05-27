# Git 迁移子模块，保持文件历史记录
git filter-branch --subdirectory-filter ./submodule -- --all


Git filter-branch 重写历史
  我们已经知道了如何将A仓库全部迁移到B仓库，如果我们只需要将A仓库中的部分子目录文件迁移，就需要用到git重写历史的功能。git filter-branch 是git提供的重写历史的命令，我们可以用subdirectory-filter来实现重写git项目子路径的git历史记录。

--subdirectory-filter <directory> 重写子路径的git历史记录

   Only look at the history which touches the given subdirectory. The result will contain that directory (and only that) as its project root.

  subdirectory-filter可以从git项目中过滤出给定子目录的git历史记录，同时会把给定的子目录作为git项目的根目录。

# Git 合并两个不相干的分支
Git 从 2.9.0 版本开始，预设行为不允许合并没有共同祖先的分支，需要加上 --allow-unrelated-histories 进行 pull 操作才不会出现此类错误信息。

这个预设行为是为了避免，人为误操作合并了两个不相干的分支，加参数–allow-unrelated-histories 你要自己确定你真的要合并这个两个没有共同祖先的分支。

demo示例

git pull origin master --allow-unrelated-histories

