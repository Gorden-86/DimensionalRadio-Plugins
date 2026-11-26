# Dimensional Radio 插件清单

次元电台（Dimensional Radio）启动后会读取本仓库的 [`plugins.json`](plugins.json)，
在「插件中心」里列出可用插件。客户端只读这份清单，不下载、不加载任何代码。

## 清单格式

```json
{
  "version": 1,
  "plugins": [
    {
      "id": "bilibili-wallpaper",
      "name": "哔哩哔哩视频源",
      "description": "搜索视频，生成网页场景交给 Lively 当壁纸。",
      "author": "你的名字",
      "version": "0.1.0",
      "kind": "wallpaper",
      "homepage": "https://github.com/you/repo"
    }
  ]
}
```

| 字段 | 说明 |
|---|---|
| `id` | 唯一标识，小写短横线 |
| `name` | 显示名称 |
| `description` | 一句话说明，会显示在卡片上 |
| `author` | 作者 |
| `version` | 插件版本，客户端只做展示，不做兼容判断 |
| `kind` | `wallpaper`（壁纸源）或 `work`（作品数据源） |
| `homepage` | 插件主页或发布页，客户端会打开这个链接 |

## 提交插件

开一个 PR，在 `plugins.json` 的 `plugins` 数组里追加一项即可。清单保持为空时，客户端显示「暂无插件」。

## 说明

客户端不校验、不签名、不自动安装插件。装什么由用户自己判断，这个仓库只承担"列表"的职责。
