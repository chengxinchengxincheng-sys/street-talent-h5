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

音效按场景数据 `sm` 资源清单精确映射（`audio_id -> url`），全部由 `behavior` 中的
`audio` 行为驱动：`appear` 事件触发答题卡文字出现音与翻页声，`click` 事件触发选项选中音：

| audio_id | 文件 | 触发时机 |
| --- | --- | --- |
| `15cn0e0a7mx` | background.mp3 | 背景音乐（加载结束自动播放） |
| `15cnzte177a` | writing-263642.mp3 | 各题「文字1#x」appear —— 答题卡文字打字时出现 |
| `15cn3uiajlz` | 卡片翻页声.mp3 | 各页背景图 appear —— 页面入场 |
| `15co0xdnbe5` | ui-pop-sound.mp3 | 选项 a1~c5 click —— 选中选项 |
| `15co5ewjgbr` | 洗牌声.mp3 | 第5题选项 click 附加音 |
| `15co03qswy0` | drumsticks.mp3 | 首页 START click |

音效采用克隆节点播放（可重叠、不截断上一次），并在加载结束 / 首次触摸时静音预热解锁，
以绕开移动端自动播放限制。

## 原版时序细节（已还原）

- **循环动效**：`presetLoop=1` 的元素无限循环 —— 第2题背景红绿灯的 ¥ 屏闪烁（fade_in 0.5s）、
  第3题两辆车的穿行（move 5s）、结果页「长按保存图片」闪烁提示（fade_in 1.5s）
- **升起方向**：`raise_in` 的 `raise=up` 从页面下方升入（分享海报 1.5s 从下往上出场），
  `raise=down` 从上方落入（答题卡等 9 处）
- **选中态**：`changeElement background` 作用于文字元素 = 文字变红 `rgba(233,30,99,1)`，
  同时 `play` 触发字母下方的红色标记横向扫出（0.3s），扫完由 `preset_end` 驱动翻页
- **第5题 loading**：`changeElement left` 为瞬时 overwrite —— 蒙层（alpha 0.8）与加载 gif
  原地出现覆盖在当前页，页面不滑动；「结果生成中...」打字机 3s 结束后按分值滑入结果页
- **淡入类入场动效元素**在播放前隐藏（木疙瘩语义），由 `play()` 触发后才显示

## 调整计分 / 文案

- 计分：改各题选项热区（`data-nm="a1"~"c5"`）`data-beh` 里 `changeElement` 的 `element_value`
- 结果分档：改「结果生成中」元素 `data-beh` 里四个 `gotoPage` 的 `_logic.expression`
- 结果页内容：直接编辑对应结果页（P7~P10）的 HTML
