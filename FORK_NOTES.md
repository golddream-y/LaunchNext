# 个人 Fork 说明（golddream-y/LaunchNext）

上游：https://github.com/RoversX/LaunchNext

这个 fork 只做一件事：**修掉“一直显示正在加载应用、列表永远空”的卡死**，其余保持与上游一致，方便随时 rebase。

## 本地补丁

分支：`fix/scan-tolerate-inaccessible-subdirs`

问题：`AppStore.performApplicationReconciliation` 里，`FileManager.enumerator` 的
`errorHandler` 返回 `false` 并把来源标记为失败。只要 `/Applications` 下任意一个子目录
不可读（例如 XAMPP 的 `var/mysql/*` 是 `0700 _mysql`），整轮扫描直接作废，于是
`isInitialLoading` 永远不会变回 `false`，首屏就一直转圈，`Data.store` 里 0 条记录。

改动：

1. `errorHandler` 改为返回 `true`，跳过不可读子树继续枚举；只把结果标记为“不完整”。
2. 结果不完整时 `processScannedApplications(allowRemovals: false)`：**只增不删**，
   不会把“这次没扫到”的应用从持久化布局里误删。
3. 扫描整体失败（来源根目录不可读）时退回宽松扫描，先解锁界面，不再无限转圈。

## 维护

- 同步上游：`git fetch upstream && git rebase upstream/main`
- 如果上游以后自己修了这段，直接丢掉本分支即可。

## 注意

- 上游应用内更新会下载官方包并覆盖本地构建，补丁会丢失。要用自己的构建就别点应用内更新。
- 本机构建使用 ad-hoc 签名（`CODE_SIGN_IDENTITY=-`），每次重新编译后系统权限（辅助功能等）可能需要重新授予。
