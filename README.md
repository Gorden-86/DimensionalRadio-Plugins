# Dimensional Radio 插件清单

次元电台（Dimensional Radio）读取本仓库的 [`plugins.json`](plugins.json)，在「插件中心」里列出可用插件。
客户端只读清单；**下载插件包时会校验 MD5**，对不上就丢弃。

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
| `kind` | 是 | `wallpaper` 壁纸源 / `work` 作品源 / `theme` 样式包 / `extension` 扩展功能 |
| `homepage` | 是 | 主页或发布页，没有下载地址时点按钮会打开它 |
| `downloadUrl` | 否 | 插件包直链。给了才会出现「下载」按钮 |
| `md5` | 给了下载地址时必填 | 插件包的 MD5。缺失或对不上，客户端会拒绝下载 |
| `size` | 否 | 字节数，仅用于展示 |

## MD5 怎么算

```bash
md5sum demo-theme.zip      # Linux / macOS
certutil -hashfile demo-theme.zip MD5   # Windows
```

只管到"包有没有被换过"。客户端不签名、不自动安装，装什么由用户自己判断。

## 提交插件

开 PR，在 `plugins.json` 的 `plugins` 数组里追加一项。清单为空时客户端显示「暂无插件」。
