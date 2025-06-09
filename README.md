# 📕 小红书MCP工具包

[![许可证](https://img.shields.io/github/license/aki66938/xhs-toolkit)](LICENSE)
[![微信公众号](https://img.shields.io/badge/凯隐的无人化生产矩阵-公众号-bule?style=flat-square&logo=wechat)](src/static/qrcode_for_gh_19088e185f66_258.jpg)

一个强大的小红书自动化工具包，支持通过MCP协议与AI客户端（如Claude Desktop等）集成，实现智能内容创作和发布。

## ✨ 主要特性

- 🍪 **Cookie管理**: 安全获取、验证和管理小红书登录凭证
- 🤖 **MCP协议支持**: 与Claude Desktop等AI客户端无缝集成
- 📝 **自动发布**: 支持图文和视频笔记的自动化发布
- 👤 **用户信息**: 获取用户档案
- 🎯 **统一接口**: 一个工具解决llm操作小红书自动化需求

## 📋 功能清单

- [x] **图文发布** - 支持发布图文笔记
- [x] **视频发布** - 支持发布视频笔记 
- [ ] **内容搜索** - 支持指定搜索（开发计划中）

## 📋 环境要求

- **浏览器**: Google Chrome 浏览器
- **驱动**: ChromeDriver (`brew install chromedriver`)

## 🚀 快速开始

### 一键安装（推荐）

```bash
# 下载并运行安装脚本
curl -sSL https://raw.githubusercontent.com/aki66938/xiaohongshu-mcp-toolkit/main/install.sh | bash

# 或者手动运行
git clone https://github.com/aki66938/xiaohongshu-mcp-toolkit.git
cd xiaohongshu-mcp-toolkit
bash install.sh
```

### 下载使用

1. 从 [Releases页面](https://github.com/aki66938/xhs-toolkit/releases/latest) 下载适合你操作系统的版本
2. 解压并运行：
   ```bash
   # macOS/Linux
   chmod +x xhs-toolkit
   ./xhs-toolkit status
   
   # Windows
   xhs-toolkit.exe status
   ```

### 从源码运行

```bash
git clone https://github.com/aki66938/xiaohongshu-mcp-toolkit.git
cd xiaohongshu-mcp-toolkit
pip install -r requirements.txt
python xhs_toolkit.py status
```



## 🛠️ 使用指南

### 1. 创建配置文件

复制并编辑配置文件：

```bash
cp env_example .env
vim .env  # 编辑配置
```

**必需配置**：
```bash
# Chrome浏览器路径
CHROME_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"

# ChromeDriver路径  
WEBDRIVER_CHROME_DRIVER="/opt/homebrew/bin/chromedriver"

# Cookies存储路径
json_path="./xhs/cookies"
```

### 2. 获取登录凭证

```bash
./xhs-toolkit cookie save
```


在弹出的浏览器中：
1. 登录小红书创作者中心
2. 确保能正常访问创作者中心功能
3. 建议点击进入【发布笔记】页面，确认权限完整
4. 完成后按回车键保存

### 3. 启动MCP服务器

```bash
./xhs-toolkit server start
```

### 4. 客户端配置
**Claude Desktop**

在 `~/.claude_desktop_config.json` 中添加：

```json
{
  "mcpServers": {
    "xiaohongshu": {
      "command": "curl",
      "args": [
        "-N",
        "-H", "Accept: text/event-stream",
        "http://localhost:8000/sse"
      ]
    }
  }
}
```

**cherry studio**

在MCP配置中添加

![Cherry Studio配置](src/static/cherrystudio.png)


**n8n**

在n8n的AI agent节点的tool中添加配置配置

![n8n的AI agent配置](src/static/n8n_mcp.png)


## 🔧 主要功能

### MCP工具列表

| 工具名称 | 功能说明 | 参数 | 备注 |
|---------|----------|------|------|
| `test_connection` | 测试连接 | 无 | |
| `start_publish_task` | 启动异步发布任务 ⚡ | title, content, tags, images, videos |  |
| `check_task_status` | 检查任务状态 | task_id | 配合异步任务使用 |
| `get_task_result` | 获取任务结果 | task_id | 获取最终发布结果 |
| `publish_xiaohongshu_note` | 发布笔记（同步） | title, content, tags, images, videos | 视频发布可能超时 |
| `search_xiaohongshu_notes` | 搜索笔记 | keyword, limit | |
| `get_xiaohongshu_user_info` | 获取用户信息 | user_id | |

### 📱 异步发布模式 ⚡ (v1.1.2新增)

解决Cherry Studio等MCP客户端的超时问题：

**使用方法**：
1. **启动任务**：`start_publish_task("标题", "内容", videos="视频路径")`
2. **检查进度**：`check_task_status("任务ID")`
3. **获取结果**：`get_task_result("任务ID")`


### 快速发布

```bash
# 图文发布
python xhs_toolkit.py publish "今日分享" "内容文本" --tags "生活,美食" --images "/path/to/image1.jpg,/path/to/image2.jpg"

# 视频发布 🆕
python xhs_toolkit.py publish "视频分享" "视频内容描述" --tags "生活,vlog" --videos "/path/to/video.mp4"

# 命令行发布
./xhs-toolkit publish "今日分享" "内容文本" --tags "生活,美食" --images "/path/to/image1.jpg"

# 通过Claude发布（推荐）
# 图文：请发布一篇小红书笔记，标题："今日分享"，内容："..."，图片路径："/User/me/xhs/poster.png"
# 视频：请发布一篇小红书视频，标题："今日vlog"，内容："..."，视频路径："/User/me/xhs/video.mp4"
```

发布原理：
手工上传过程中，浏览器会弹窗让用户选中文件路径
告诉ai路径位置，ai会把路径参数对应丢给mcp的参数中，完成上传动作

**智能等待机制** ：
- **图片上传**：快速上传，无需等待
- **视频上传**：轮询检测上传进度，等待"上传成功"标识出现
- **超时保护**：最长等待2分钟，避免MCP调用超时 
- **状态监控**：DEBUG模式显示视频文件大小和时长信息
- **高效轮询**：每2秒检查一次，精确文本匹配 





## 🛠️ 常用命令

```bash
# 检查状态
./xhs-toolkit status

# Cookie管理
./xhs-toolkit cookie save      # 获取cookies
./xhs-toolkit cookie validate  # 验证cookies

# 服务器管理
./xhs-toolkit server start     # 启动服务器
./xhs-toolkit server start --port 8080  # 自定义端口
./xhs-toolkit server stop      # 停止服务器
./xhs-toolkit server status    # 检查服务器状态
```
## 🔐 安全承诺

- ✅ **本地存储**: 所有数据仅保存在本地
- ✅ **开源透明**: 代码完全开源，可审计
- ✅ **用户控制**: 您完全控制自己的数据


## Future

**关于后续版本开发计划**

目前工具箱阶段性任务算是告一段落，可用工具为图文发布和视频发布
因未来半个月内，作者可能会面临栖居状态的模态转换或者存在坐标的拓扑重构（搬家。。。）
故，作者处于---"行动者网络的「非行动」拓扑 "状态（暂时停工。。。）

源代码中埋了不少坑，自动发布的两个重点功能已经完成，如果不出大bug，功能上应该不会再更新，可能会优化
后续的开发重点方向以采集作者的页面数据为主，后续将考虑引入llm对作者的数据面进行分析，提供优化建议诸如此类。

当然，虽然源码方面暂时停更，但有空余时间会补齐文档：从基础项目的搭建开始到熟练使用本工具再到利用本工具完成自动化任务等。

文档将在微信公众号上更新，欢迎大家关注点击文档顶部的胶囊按钮扫码关注我的公众号，你的支持是我继续开发完善工具的动力！

## 📄 许可证

本项目基于 [MIT许可证](LICENSE) 开源。

---

<div align="center">
Made with ❤️ for content creators
</div> 
