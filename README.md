# Dimensional Radio 插件清单

次元电台（Dimensional Radio）读取本仓库的 [`plugins.json`](plugins.json)，在「插件中心」里列出可用插件。
客户端只读清单；**下载插件包时会校验 MD5**，对不上就丢弃。

## 开发插件

想写自己的插件（歌词源、视频源、扫码登录），看 [sdk/README.md](sdk/README.md)：
契约 NuGet 包、能力接口、能干什么/不能干什么、打包发布一条龙都在那。

## 清单格式

```json
{
  "version": 2,
  "plugins": [
    {
      "id": "demo-theme",
      "name": "示例样式包",
      "description": "一句话说明，显示在卡片上。",
      "author": "作者",
      "version": "0.1.0",
      "kind": "theme",
      "homepage": "https://github.com/作者/仓库",
      "downloadUrl": "https://github.com/作者/仓库/releases/download/v0.1.0/demo-theme.zip",
      "md5": "57bb911b477a7be7315c6378ab93dc9f",
      "size": 445
    }
  ]
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | 是 | 唯一标识，小写短横线 |
| `name` | 是 | 显示名称 |
| `description` | 是 | 一句话说明 |
| `author` | 是 | 作者 |
| `version` | 是 | 插件版本，客户端只展示 |
| `kind` | 是 | `wallpaper` 壁纸源 / `work` 作品源 / `lyrics` 歌词源 / `theme` 样式包 / `plugin` 扩展功能；`tool`、`runtime` 是依赖包，装独立目录不显示分类 |
| `license` | 是 | 分发协议，如 `MIT`、`Apache-2.0`。插件中心卡片会显示，必须与插件实际授权一致 |
| `homepage` | 是 | 主页或发布页，没有下载地址时点按钮会打开它 |
| `downloadUrl` | 否 | 插件包直链。给了才会出现「下载」按钮 |
| `md5` | 给了下载地址时必填 | 插件包 zip 的 MD5。缺失或对不上，客户端会拒绝下载 |
| `size` | 否 | 字节数，仅用于展示 |
| `file` | DLL 插件必填 | 插件本体 DLL 文件名 |
| `fileMd5` | 跟 `file` 一起 | 本体 DLL 的 MD5，加载前核验 |
| `deps` | DLL 插件有依赖时 | 依赖 DLL 清单，每项 `file` + `md5`，同样逐一核验 |
| `installTo` | 仅 `tool`/`runtime` | 装到数据目录下的哪个子目录 |

## MD5 怎么算

```bash
md5sum demo-theme.zip      # Linux / macOS
certutil -hashfile demo-theme.zip MD5   # Windows
```

只管到"包有没有被换过"。客户端不签名、不自动安装，装什么由用户自己判断。

## 提交插件

开 PR，在 `plugins.json` 的 `plugins` 数组里追加一项。清单为空时客户端显示「暂无插件」。

## 许可与分发

| 内容 | 许可证 | 说明 |
|---|---|---|
| 本仓库（清单、文档、主题包） | [MIT](LICENSE) | 跟主程序同源 |
| 官方插件包（哔哩哔哩视频源、在线歌词、桌面频谱） | MIT | 随主程序 MIT 分发 |
| libmpv 运行时包（`packages/mpv-runtime.zip`） | **LGPL-2.1+** | mpv 的 LGPL 构建。`libmpv-2.dll` 由用户放置到程序目录、宿主经 C API 动态调用，不修改、不分发其源码；仓库仅分发热心的社区构建产物，使用者须自行遵守 LGPL 的授权要求 |
| 第三方提交的插件 | 随作者声明 | 开 PR 时须在条目或插件包内注明许可证；未声明的按仓库默认 MIT 处理 |

注意：MIT、Apache-2.0 允许商用和闭源分发，要求保留版权与许可证声明；LGPL 对二进制的再分发有对应义务（随附许可文本、标注修改、允许用户替换该组件）。
