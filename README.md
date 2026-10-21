# DMClike

UE 5.6 第三人称高速动作战斗 Demo（Windows 64 位）。

## 下载

到 [Releases](https://github.com/15266/DMClike/releases) 下载**全部**分卷文件：

```
DMClike_win64.zip.001   (1.07 GiB)
DMClike_win64.zip.002   (710 MiB)
```

两个文件必须放在**同一个目录**里，缺一个都解不开。

## 安装与启动

1. 用 7-Zip 打开 `DMClike_win64.zip.001`，解压（7-Zip 会自动串上 `.002`）
2. 进入解压出来的目录
3. 双击 **`Play_DMClike.bat`**

> 不要直接双击 `DMClike.exe`。少了 `-unattended` 参数，
> 可能被显卡驱动警告框挡住，或者在少数情况下崩在出生点生成阶段。

首次启动要编译着色器，约 **20～40 秒**，之后会快很多。

## 系统要求

| 项目 | 要求 |
|---|---|
| 系统 | Windows 10 / 11 64 位 |
| 显卡 | 支持 DirectX 12 的独立显卡 |
| 驱动 | 建议 560.81 或更新 |
| 磁盘 | 约 2.6 GB（解压后） |

驱动过旧时 UE 会弹 `WARNING: Known issues with graphics driver`。
用 `Play_DMClike.bat` 启动可以跳过这个框。

## 操作

完整键位与连招见 **[出招表](docs/出招表.md)**。

摘要：`WASD` 移动、鼠标左键普攻、右键远程/瞄准、`E` 格挡弹反、
鼠标侧键锁定、滚轮换武器；方向键 + 攻击派生升龙 / 次元突进 / 破空斩等。

## 已知问题

**启动时的 ensure（不影响游玩）**

部分蓝图用了 `ActorSequenceComponent`，其动画序列在 cook 阶段没有生成预编译数据，
启动时 UE 会报：

```
Ensure condition failed: PrecompiledData->FindDataID(this).IsValid()
[File: .../Runtime/MovieScene/Private/MovieSceneSequence.cpp] [Line: 130]
No precompiled movie scene data is present for sequence 'Sequence'.
```

这是 **Handled ensure**，引擎会继续运行，用 `Play_DMClike.bat` 启动即可正常进图。
实测：地图加载 9.4 秒完成，渲染正常，无致命错误。

彻底修复需要在 UE 5.6 编辑器里删掉相关蓝图上的 `ActorSequenceComponent` 后重新打包。

## 关于内部目录名 `fdz`

外层的启动器叫 `DMClike.exe`，但里面的数据目录、主程序、pak 文件名仍叫 `fdz`。
这不是漏改：

- UE 打包时会把项目名编译进可执行文件，用于拼 pak 路径
  （`<root>/fdz/Content/Paks/fdz-Windows.utoc`）
- 启动器 `BootstrapPackagedGame` 里硬编码了 `fdz\Binaries\Win64\fdz.exe`

强行把这些也改成 `DMClike`，实测会直接弹「错误」对话框起不来。

### 为什么没有从源码层彻底改名

本机装了 UE 5.6.1（`D:\ue\UE_5.6`，CL-44394996，与本包编译版本一致），
理论上可以改名重打包。做法是 `DMClike.uproject` + `DMClike.Target.cs`
（C++ 模块名保持 `fdz`，这样资产里引用的 `/Script/fdz.*` 不会失效）。

实际试了一次，卡在编译量上：**Target 名一换，引擎模块的中间产物目录就变了**
（`Intermediate/Build/Win64/x64/<TargetName>/`），
于是 1000 多个引擎模块要按新目标名整个重编一遍。

这台机器只有 16 GB 内存，编译时可用约 2.4 GB，UBT 因此把并行度限制成 1：

```
Requested 1.5 GB memory per action, 2.37 GB available:
limiting max parallel actions to 1
```

1015 个动作串行执行，预计 4～6 小时。所以最终没有重编，
改为在打包产物层改名（启动器 / 窗口标题 / 包名），这些已经是 `DMClike`。

要在内存充足的机器上做彻底改名，需要的正是上面那两步文件改动。


窗口标题、启动器名、包名已经全部是 `DMClike`。

## 未包含的内容

- `fdz.pdb`（413 MB 调试符号）——普通游玩不需要，只在解析崩溃 minidump 时用得上
- 游戏源码与资产工程
