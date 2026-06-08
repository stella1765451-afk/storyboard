# 分镜分析工具 · GitHub Pages 版（v5）

把原来的 Storyboard Analyzer 从「浏览器本地存储」迁移到「仓库 JSON 文件」。
数据源 = 仓库里的 JSON，打开页面自动 fetch 加载，**不再依赖浏览器存储**。

---

## 一、目录结构

```
你的仓库/
├── index.html              ← 工具本体（单文件，无需构建）
├── .nojekyll               ← 关闭 GitHub Jekyll 处理（保留即可）
├── README.md
└── projects/
    ├── manifest.json       ← 项目清单（唯一需要手动登记项目的地方）
    ├── 伊丽川预告/
    │   ├── shots.json          ← 镜头数组（主数据，Excel 校准的就是它）
    │   ├── changes.json        ← 版本对比（可选）
    │   └── production.json     ← 制作评估（可选）
    └── 绝区零/
        └── shots.json
```

> 一个项目最少只要 `shots.json`，`changes.json` / `production.json` 没有就自动跳过，不会报错。

---

## 二、首次部署（约 5 分钟）

### 1. 建仓库 + 传文件
- GitHub 上新建仓库（例如 `storyboard`，public）。
- 把本文件夹**整个**内容（`index.html`、`.nojekyll`、`projects/`）传上去。
  - 网页传：仓库页 → **Add file → Upload files** → 拖进去 → Commit。
  - 命令行：
    ```bash
    git clone https://github.com/<你的用户名>/storyboard.git
    # 把文件拷进去后
    git add .
    git commit -m "init storyboard analyzer"
    git push
    ```

### 2. 开启 Pages
- 仓库 → **Settings → Pages**
- **Source** 选 `Deploy from a branch`
- **Branch** 选 `main`，目录选 `/ (root)` → **Save**
- 等 1–2 分钟，页面顶部给出地址：
  `https://<你的用户名>.github.io/storyboard/`

打开它，应当看到「伊丽川预告」项目和那几条示例镜头，说明加载成功。

---

## 三、四个迁移目标 · 已实现说明

### ① 数据持久化到仓库 JSON，自动加载
打开页面 → 自动读 `projects/manifest.json` → 按清单逐个项目读 `shots/changes/production.json` → 合并进工具。右上角显示 `✓ 已同步仓库`。

### ② 多项目文件夹结构
每个项目一个文件夹。新增项目只需两步：
1. 建 `projects/<新项目>/shots.json`
2. 在 `manifest.json` 的 `projects` 数组里加一行

### ③ 跨项目案例库全局搜索
「案例库」Tab 自动合并**所有**项目的镜头（有 `core_intent` / `creation_note` / `translation_note` 字段的会被收录），可按项目 / 情况 / 类型 / 复用性 / 关键词筛选。无需任何额外配置。

### ④ 本地编辑工作流：Excel 校准 → commit + push → 自动更新
见下节。

---

## 四、日常更新数据（核心工作流）

工具里的修改**不会自动写回仓库**（静态站点无法写文件）。
流程是「在仓库里改 JSON → push → 工具重载」。两种改法：

### A. Excel 校准流（推荐，对应你原来的 json2excel）
```
Gemini 出 JSON
 → json2excel 转 Excel
 → 手动补遗漏镜头、校准 shot_number / cam_id
 → Excel 转回 JSON
 → 覆盖 projects/<项目>/shots.json
 → git commit + push
 → 打开工具点右上角「↻ 重载」即更新
```

### B. 工具内编辑流（少量微调）
```
工具里点开镜头改字段（右上角变 ● 有未导出修改）
 → 「导入/导出」Tab → 选「导出项目」→ 点「导出 shots.json」
 → 用下载的文件覆盖 projects/<项目>/shots.json
 → git commit + push → 「↻ 重载」
```

> 「导入/导出」Tab 还有 `changes.json`、`production.json`、`manifest.json` 的导出按钮，改了哪个就导哪个。

### 提交命令（命令行用户）
```bash
git add .
git commit -m "update 伊丽川预告 shots"
git push
```
推完等 Pages 部署完（约 1 分钟），在工具里点「↻ 重载」。

---

## 五、新增一个项目（例：绝区零已演示）

1. 新建文件夹和镜头文件：`projects/绝区零/shots.json`（内容是镜头数组）
2. 编辑 `projects/manifest.json`，加一行：
   ```json
   {
     "projects": [
       { "name": "伊丽川预告", "dir": "伊丽川预告" },
       { "name": "绝区零",     "dir": "绝区零" }
     ]
   }
   ```
   - `name` = 工具里显示的项目名
   - `dir`  = 文件夹名（可不同；中文文件夹没问题，工具会自动 URL 编码）
3. commit + push → 重载。

> **进阶（可选）**：若担心中文文件夹在某些环境出问题，可把 `dir` 设成英文，例如
> `{ "name": "绝区零", "dir": "zzz" }`，文件夹叫 `projects/zzz/`，显示名仍是中文。

---

## 六、JSON 字段说明

`shots.json` 兼容 Gemini 模板 8 的下划线字段（`shot_number`、`cam_id`、`emotion_intensity`、`core_intent`、`creation_type` 等），直接把 Gemini 输出贴进文件即可，工具自动映射。

`changes.json` 字段：`camIdDraft` / `camIdFinal` / `changeTypes`（数组）/ `draftDesc` / `finalDesc` / `reason` / `rhythmEffect` / `project`。

`production.json` 字段：`summary` + `dims`（每项含 `dim` / `score` / `highlights` / `issues` / `outstanding`）。

完整字段表见项目上下文文档 `storyboard_project_context.md` 的「JSON 字段参考」。

---

## 七、本地预览（不部署也能测）

直接双击 `index.html` 会因 `file://` 无法 fetch 而报错。要用本地服务器：

```bash
cd storyboard-analyzer
python -m http.server 8000
# 浏览器打开 http://localhost:8000
```

---

## 八、常见问题

- **打开是空的 / 红色报错条**：多半是没通过 http(s) 访问（直接双击了文件），或 `manifest.json` 路径不对、JSON 有语法错。报错条会显示具体原因。
- **改了 JSON 但工具没变**：Pages 部署有几十秒延迟；部署完点「↻ 重载」；仍不行就强刷（Ctrl/Cmd+Shift+R）清缓存。
- **中文文件夹打不开**：极少见；按第五节「进阶」把 `dir` 改成英文即可。
- **想加构建版本（Vite）**：本方案故意不用构建，最省心。数据改动永远不需要重新构建；只有改 `index.html` 里的代码才需要。若以后想要更快加载，可迁到 Vite + GitHub Actions，但日常 JSON 工作流完全一致。
