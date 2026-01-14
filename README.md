# 题目刷题工具（Problem Drill Tool）

一个用于题目录入与刷题练习的轻量系统，包含统一的后端 API 与前端页面。项目支持客观题（选择/判断）与主观题的录入、导入与练习，提供将原始 txt 题目转换为 JSON 数据的脚本，方便在本地或局域网内部署使用。

主要特点
- 后端：Node.js + Express，提供 REST API（默认监听 8000 端口）
- 前端：React + Vite，提供题目录入（Entry）与练习（Practice）页面
- 数据：JSON 格式题库（data/objective_questions.json、data/subjective_questions.json）
- 支持从历史 txt 题库批量转换导入（scripts/convert_txt_to_json.py）
- 可在局域网内部署，方便多台设备访问练习页面

目录结构（简要）
- backend/ — 后端服务（Express）
- frontend/ — 前端页面（React + Vite）
- data/ — 运行时/生成的题库 JSON 文件
- scripts/ — 辅助脚本（如 txt -> json 的转换脚本）
- src/ — 原始 C++ 实现与题库 txt（历史/备用）
- README.md — 本文件

快速开始（开发环境）
先决条件：Node.js (>=14)、npm、Python3（用于脚本转换时）

1. 从 txt 生成 JSON（可选，如果已有 data 文件可跳过）
```bash
# 在仓库根目录运行
python3 scripts/convert_txt_to_json.py
# 会在 data/ 目录下生成 objective_questions.json 与 subjective_questions.json
```

2. 启动后端（默认端口 8000）
```bash
cd backend
npm install
npm run start
# 或：node server.js
```
后端默认监听 0.0.0.0:8000（可通过环境变量 PORT 修改）。

3. 启动前端（开发模式）
```bash
cd frontend
npm install
npm run dev -- --host 0.0.0.0 --port 5173
```
打开浏览器访问 http://localhost:5173 ，前端会默认请求 http://<当前主机>:8000 作为后端 API。

（生产部署）
- 可以使用 Vite 的构建产物（npm run build）并用静态服务器发布，或将前端打包后的静态文件部署到任何静态服务，并确保前端请求的 API 地址指向后端服务。

前端与后端通信说明
- 前端默认根据 window.location.hostname 构造 API 主机（在同一台机器部署时无需修改）。
- 若前后端分开部署到不同主机/机器，请修改前端中 API 常量：
  - frontend/src/pages/PracticePage.jsx
  - frontend/src/pages/EntryPage.jsx
  将 `API_BASE` 修改为 `http://<后端主机>:8000`，例如：
  ```js
  const API_BASE = "http://192.168.1.10:8000";
  ```

后端 API（主要）
- 客观题（objective）
  - GET /api/questions/objective
    - 获取全部客观题（JSON 数组）
  - GET /api/questions/objective/random?count=1&qtype=single
    - 随机抽题；参数：
      - count：题目数量（默认 1）
      - qtype：题型筛选（例如 single）
  - POST /api/questions/objective
    - 新增/导入客观题（请求体为题目 JSON 或特定文本导入格式，详见代码实现）
- 主观题（subjective）
  - GET /api/questions/subjective
    - 获取全部主观题
  - GET /api/questions/subjective/random?count=1
    - 随机抽取主观题
  - POST /api/questions/subjective
    - 新增/导入主观题

数据文件
- data/objective_questions.json — 客观题数据（数组）
- data/subjective_questions.json — 主观题数据（数组）
后端在写入这些文件之前会确保目录存在，文件缺失时读取会返回空数组（不会抛错）。

批量导入 / 从 txt 转换
- 脚本：scripts/convert_txt_to_json.py
  - 会读取仓库内 backup/（原客观题 txt）与 src/Subjective-Question/*.txt（主观题）并生成 data/*.json
  - 脚本对文件编码做了兼容处理（优先 utf-8，回退 gbk）
- 也可以使用前端的导入功能或后端提供的 POST 接口单个导入。

局域网部署（在同一内网访问）
1. 后端：确保后端绑定 0.0.0.0 并放行 8000 端口（示例命令见上）
2. 前端开发服务：使用 Vite dev 并允许外部访问（--host 0.0.0.0）并放行 5173 端口
3. 若前后端部署在不同机器，请在前端将 API_BASE 设置为后端所在机器 IP（如 http://192.168.1.10:8000）
4. 在局域网设备中访问前端地址：
   - http://192.168.1.10:5173

常见问题与提示
- 如果数据导入后前端没有显示，检查后端是否已正确写入 data/ 下的 JSON 文件，并确认前端请求的 API 地址与后端地址一致。
- 脚本编码兼容：convert_txt_to_json.py 会尝试 utf-8，再回退到 gbk（并 replace 错误字符），若遇到特殊编码请手动转换文件编码后重试。
- 后端错误会在控制台输出详细信息（backend/server.js 的错误处理中已做基本处理，返回 500 时会在控制台打印错误）。

开发建议
- 如果准备将服务暴露到公网，请在生产环境下使用反向代理（如 nginx）或进程管理（如 PM2），并启用 HTTPS。
- 若需在前端使用环境变量配置 API 地址，可改造代码以读取 env（例如 VITE_API_BASE），避免直接修改源码常量。

贡献
欢迎提交 issue 与 pull request。推荐：
- 在 issue 中描述复现步骤与预期行为
- 新增功能或修复请附带简短测试或复现示例

许可证
- 请参阅仓库根目录的 LICENSE 文件（如存在）。若无，请在提交前确定合适的开源许可证。

作者与联系方式
- 仓库：Shalomguan/Problem-Drill-Tool-2.0
- 如需进一步帮助可在仓库中打开 issue 或联系项目维护者。

---
说明：我已依据仓库后端与前端代码、脚本与子目录说明整理了上面的 README。如果你希望我直接在仓库中提交此 README（创建提交/分支并推送），我可以为你生成提交说明与变更补丁，或指导你如何手动替换并提交。请指示下一步你希望我如何操作。 
