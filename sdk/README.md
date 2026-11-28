# 次元电台插件开发 SDK

次元电台的插件 = 一个实现 `IRadioPlugin` 能力接口的 .NET 类库。引用这个 SDK 包，写完打包成 zip，就能被主程序加载。

- SDK 包：本目录的 `DimensionalRadio.Plugin.Abstractions.1.3.0.nupkg`
- 契约版本：1（契约号和包版本是两回事：契约是协议，包是工具；契约改了包就升一位）

## 快速开始

1. 建一个类库工程，目标框架 `net10.0-windows10.0.19041.0`：

   ```xml
   <Project Sdk="Microsoft.NET.Sdk">
     <PropertyGroup>
       <TargetFramework>net10.0-windows10.0.19041.0</TargetFramework>
       <Nullable>enable</Nullable>
       <!-- 版本号写这里，插件运行时读程序集版本，清单上的 version 要跟它对齐 -->
       <Version>1.0.0</Version>
       <!-- NuGet 依赖要复制进输出，打包才带得上 -->
       <CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>
     </PropertyGroup>
     <ItemGroup>
       <PackageReference Include="DimensionalRadio.Plugin.Abstractions" Version="1.3.0" />
     </ItemGroup>
   </Project>
   ```

2. 实现一个能力接口（见下），`Id` 全仓库唯一，小写短横线。
   **契约声明不用写**：SDK 包自带 build targets，编译时自动给程序集打上 `PluginContract` 声明。
3. csproj 里写 `<Authors>你的名字</Authors>`，开源许可窗口按它显示作者。
4. 用主程序仓库的 `scripts/pack-plugin.ps1` 打包发布（见文末）。

## 契约版本怎么运作

插件 DLL 的程序集上带一个契约声明，主程序 Abstractions.dll 上也有一个。
加载器在实例化**之前**比对两个数字：对不上直接拒载，DLL 里的代码一行都不执行——
所以老插件不会炸出一堆 `Method not found`，只会得到一句「契约版本不一致」。

- 声明数字由 SDK 自动注入（包里的 build targets），升级 SDK 包 = 声明跟着升级。
- **分发许可也是自动注入**：程序集默认带 `PluginLicense = MIT`，csproj 里写
  `<PluginLicense>Apache-2.0</PluginLicense>` 即可改成别的；它显示在插件中心卡片和开源许可窗口。
  仓库清单的 `license` 字段必须跟它一致。
- 给接口改签名、删成员 = 契约版本加一，同时升 SDK 包版本；旧插件被拒载，作者跟进升级。
- 加**可选**能力接口（像 `IVideoPanelPlugin` 那样单独一个接口）不用动契约版本，老插件照样装。

## 能力接口一览

主程序按接口认插件，实现了哪个就在哪生效，没实现的不管：

| 接口 | 干什么 | 宿主在哪用它 |
|---|---|---|
| `ILyricsPlugin` | 歌词源：换歌广播来了去找词，找到了主动提交 | 桌面歌词卡片、逐字扫描 |
| `ILyricsImporter` | 把本地字幕文件（ASS、SRT、LRC…）解析成歌词 | 正在播放页「导入字幕」，按声明的扩展名路由 |
| `IVideoSourcePlugin` | 搜索视频、下载成文件 | 场景页的视频源、下载建场景 |
| `IVideoPanelPlugin` | 在视频源基础上加列分 P、预览地址、按集下载 | 选片面板（检测不到就退化成整条下载） |
| `ILoginPlugin` | 生成二维码、轮询扫码状态 | 设置页的「扫码登录」按钮 |
| `IDesktopPanelPlugin` | 往桌面摆常驻装饰窗（频谱、氛围灯这类） | 启动、重载、设置保存后调 `RefreshPanel`，摆不摆插件自己定 |
| `IOpenSourcePlugin` | 声明插件引入的第三方开源库 | 设置页「开源许可」弹窗的「插件使用的开源项目」分组 |

可选接口不需要全实现：`IVideoPanelPlugin` 继承自 `IVideoSourcePlugin`，实现了它主程序自动升级成面板模式。

## 歌词源：广播怎么收、词怎么交

主程序只认一样东西：`LyricDocument`（带时间轴的行，有逐字表就逐字扫描）。
词从哪来、怎么找、找多快，主程序一概不管——网络搜的、本地文件解析的、缓存翻出来的都一样。

流程是**广播-提交**：

1. **收广播**：每次换歌，主程序给**每个**装着的歌词源调一次 `OnTrackChangedAsync(request, sink)`。
   `request` 里是歌名、歌手、专辑、时长；`sink` 是这次广播专用的提交通道。
   多个歌词源并存互不认识——A 搜 LRC、B 解析 ASS，主程序眼里都是歌词源。
2. **自己找**：宿主不等结果，你在方法里随便跑异步——慢工出细活随它去。
   找不到就什么都不交，返回 false；找到了返回 true。
3. **交词**：拿 `sink.Submit(document)` 交一份 `LyricDocument`。
   同一首歌里后交的顶先交的，切歌之后才交到的直接丢弃，所以早交晚交都安全。
   所有源都交完还没人交词，主程序显示"这首歌没有歌词"。

```csharp
public sealed class MyLyricsPlugin : ILyricsPlugin
{
    // Id / Name / Description / Version / Settings / Initialize 照基座接口写
    // 契约声明由 SDK 自动注入，不用手写

    public async Task<bool> OnTrackChangedAsync(LyricRequest request, ILyricsSink sink,
        CancellationToken cancellationToken = default)
    {
        var document = await SearchSomewhereAsync(request.Title, request.Artist, cancellationToken);
        if (document is null) return false;

        sink.Submit(document);
        return true;
    }
}
```

本地字幕导入（`ILyricsImporter`）不走广播——那是用户主动选文件，宿主按你声明的
`Extensions` 把路径递给你，`ParseAsync` 返回 `LyricDocument` 就行。ASS 的 `\k`
卡拉OK标签天然带逐字时长，映射成 `LyricWord` 就能吃到逐字扫描；SRT 没有逐字，整句常亮。

## 插件能干什么

- **实现能力接口**：宿主按需调用，结果交给宿主渲染。
- **发网络请求**：搜索、下载、登录轮询，都是插件自己的事，用自带 HTTP 客户端随便发。
- **带自己的 NuGet 依赖**：DLL 会一起进包、一起装进插件目录，加载时自动解析。
  **引入第三方开源库必须声明**：实现 `IOpenSourcePlugin`，把库名、版本、协议、主页、用途一条一条列进
  `OpenSourceNotices`，宿主汇总进开源许可列表（按插件分组标注）。SDK 包和系统运行时不用声明。
- **声明设置项**：`Settings` 里列出来，宿主照着画设置页，值持久化由宿主负责。
  类型有文本、开关、下拉（Choice）、颜色（Color，宿主渲染成调色板）；
  项可以带 `Group` 收进抽屉、`ShowWhen="key=值"` 做条件显示（选了自定义才亮出调色板这类）。
- **读自己的设置**：`IPluginContext.GetSetting(key)`，没配过返回默认值；`SetSetting` 直接写回，
  宿主负责落盘和同步设置页控件（桌面面板上的锁钮、右键菜单就靠它）。
- **拿播放上下文**：`IPluginContext.GetCoverImage()` 拿当前封面字节、`GetThemeAccent()` 拿程序强调色。
- **写日志**：`IPluginContext.Log(message)`，落在主程序日志里，带插件 Id 前缀。

生命周期就三步：加载（MD5 + 契约声明都过关）→ `Initialize(context)` 调一次 → 之后宿主按接口按需调方法。卸载/更新时整个加载上下文被回收，不存在后台常驻。

## 插件不允许干什么

这些不是建议，是加载器和宿主的硬边界：

- **没登记的 DLL 不执行**。插件目录里来路不明的文件直接拒绝加载，界面会明示「未在仓库登记」。
- **MD5 对不上不执行**。文件被换过、下坏了，一律「核验错误，请重新下载」。
- **契约声明不一致不执行**。你声明的版本和主程序对不上，在实例化之前就被拒载，代码一行不跑。
- **拿不到宿主内部对象**。数据库、壁纸后端、播放器、主窗口，都不在插件能摸到的范围里；插件和宿主的全部通道就是 `IPluginContext`。
- **没有文件系统自由**。唯一允许写的位置是宿主调 `DownloadAsync` 时传入的目标目录；要持久化只能走自己的设置项。
- **不能注入界面**。设置页、插件中心、选片面板都由宿主按接口渲染，插件给数据不给控件。
- **不能常驻后台**。宿主不调你，你就是一段躺在内存里的代码；没有事件订阅、没有定时回调。
- **不能抢 Id**。`Id` 冲突的插件互相顶替，取第一个，别用别人的。

设计上插件坏了也连累不了主程序：构造抛异常、加载失败只记一条日志，主程序照常跑。

## 打包发布

主程序仓库里有一条龙脚本：

```powershell
D:\...\DimensionalRadio\scripts\pack-plugin.ps1 `
    -Project D:\...\src\DimensionalRadio.Plugins.Lyrics\DimensionalRadio.Plugins.Lyrics.csproj
```

它会构建、收集 DLL、压成 `packages/<短名>-plugin.zip`、算好 zip 和每个 DLL 的 MD5，最后打印一段可以直接粘进本仓库 `plugins.json` 的条目。补齐尖括号里的显示信息和 `notes`，推送仓库即可。

注意装载端的规矩（都会被拒，别踩）：

- **换 DLL 前先退出主程序**，再同步 `plugins.json` 里对应的 `fileMd5`/`deps` MD5。
- 依赖 DLL 也逐一登记 MD5；下载端的包级 `md5` 是 zip 本体的校验。
- `version` 要和程序集版本一致，客户端拿它判断有没有更新。

## 插件分类

仓库清单里 `kind` 只有这几种，别发明新的：

| kind | 显示 | 装到哪 |
|---|---|---|
| `wallpaper` | 壁纸源 | 插件目录，实现 `IVideoSourcePlugin` |
| `lyrics` | 歌词源 | 插件目录，实现 `ILyricsPlugin` / `ILyricsImporter` |
| `plugin` | 扩展功能 | 插件目录，桌面面板等其余能力 |
| `theme` | 样式包 | 主题目录，纯 JSON 不是 DLL |
| `tool` / `runtime` | 不显示分类 | 装独立目录，不参与加载 |
