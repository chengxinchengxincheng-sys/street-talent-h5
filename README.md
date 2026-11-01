# 测测你的路边从业天赋（Street Vending Talent H5）

从木疙瘩（Mugeda）导出的独立 H5 互动测试，纯静态实现，零依赖，可直接部署到 GitHub Pages / 任意静态托管。

## 玩法

- 5 道情景选择题，A=1 分 / B=2 分 / C=3 分（第 2 题各选项均 1 分）
- 总分 5~15，映射四种「路边从业天赋」结果：
  | 分数段 | 结果 |
  |---|---|
  | 5–7 | 手艺精湛型 |
  | 8–9 | 便利零售型 |
  | 10–11 | 交际与情绪价值型 |
  | 12–15 | 操作与灵活型 |
- 支持重测、背景音乐开关（右上角）、选中音效

## 本地预览

```bash
# 任意静态服务器，如：
python3 -m http.server 8000
# 打开 http://localhost:8000
```

直接双击 `index.html` 也可以（file:// 协议下音频可能受浏览器自动播放策略限制）。

调试模式：`index.html?debug=1`，左下角显示当前页码与计分。

## 部署到 GitHub Pages

```bash
git init
git add -A && git commit -m "init"
git branch -M main
git remote add origin git@github.com:<你的用户名>/<仓库名>.git
git push -u origin main
```

然后仓库 Settings → Pages → Source 选 `main` 分支 `/ (root)`，即可通过
`https://<你的用户名>.github.io/<仓库名>/` 访问。

`.nojekyll` 已包含，无需额外配置。

## 目录结构

```
index.html      # 全部页面结构 + 交互引擎（自包含，无外部依赖）
images/         # 47 张 WebP 无损素材 + 1 张 GIF（合计 10.3MB，压缩前 21.9MB）
audios/         # 1 段 BGM + 5 个音效（10MB，未压缩）
webp-map.json   # 原始文件名 → 压缩后文件名的映射
```

## 技术说明

页面结构由木疙瘩发布的场景数据（`scene.js`）解析生成：每个元素的坐标（精确到 0.01px）、
富文本样式、入场动画（打字机 / raise_in / move_in / closer_in / fade_in / move 过场，含延迟与时长）、
交互行为（click / appear / preset_end 触发的 nextPage / gotoPage / changeElement / play / audio）
均按原始数据还原。交互引擎以事件委托实现，行为数据内嵌于元素的 `data-beh` 属性。

动效采用「进入页面时启动」的策略：`data-anim` 存 CSS 动画、`data-typer` 存打字机参数，
翻页时才播放，并重放上一页的初始状态，因此支持反复重测；被 `play()` 引用的元素
（选中标记、结果生成中）不随页面自动播放，等点击触发。

音频与音效 ID 的映射为按时长推断（BGM=653s 长音频，选项音=0.42s 短音频），
如与原版听感不符，可在 `index.html` 顶部的 `AUDIO` 映射表中调整。

## 调整计分 / 文案

- 计分：改各题选项热区（`data-nm="a1"~"c5"`）`data-beh` 里 `changeElement` 的 `element_value`
- 结果分档：改「结果生成中」元素 `data-beh` 里四个 `gotoPage` 的 `_logic.expression`
- 结果页内容：直接编辑对应结果页（P7~P10）的 HTML
