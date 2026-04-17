# TVBox Tools 详细流程分析

## 整体流程概览

TVBox Tools 是一个用于自动化处理 TVBox 相关资源的工具，主要功能包括从 GitHub 克隆仓库、下载和处理线路文件、生成多仓和单仓 JSON 文件、处理镜像代理以及推送更新到 GitHub。

## 详细流程步骤

### 1. 初始化配置阶段

1. **环境变量读取**：
   - 从环境变量中读取 token、username、repo、signame、target、num、url、timeout、mirror、jar_suffix 等配置参数
   - 这些参数用于后续的 GitHub 操作和文件处理

2. **GetSrc 类初始化**：
   - 初始化各种参数，如 jar_suffix、mirror、registry、mirror_proxy 等
   - 配置请求头和会话，设置重试机制
   - 初始化 GitHub 镜像代理列表（gh1 和 gh2）

### 2. 仓库操作阶段

1. **设置 hosts**：
   - 调用 `set_hosts()` 方法，尝试获取并设置 GitHub 的加速 hosts

2. **镜像代理初始化**：
   - 调用 `mirror_init()` 方法，根据 mirror 参数选择合适的镜像代理
   - 配置对应的 slot 路径

3. **克隆 GitHub 仓库**：
   - 调用 `git_clone()` 方法，尝试克隆指定的 GitHub 仓库
   - 如果克隆失败，尝试使用镜像地址再次克隆
   - 确保创建必要的目录结构（如 jar 目录）

4. **获取本地仓库信息**：
   - 调用 `get_local_repo()` 方法，打开本地仓库并读取仓库信息
   - 配置用户信息和 http.postBuffer
   - 确定仓库的主分支（main 或 master）
   - 再次初始化镜像代理

### 3. 在线接口处理阶段

1. **批量处理在线接口**：
   - 调用 `batch_handle_online_interface()` 方法，处理多个在线接口
   - 分割 url 参数，处理每个在线接口

2. **仓库处理**：
   - 调用 `storeHouse()` 方法，处理不同类型的仓库：
     - **线路处理**：如果内容包含 'searchable'，则作为线路处理
     - **多仓处理**：如果内容包含 'storeHouse'，则作为多仓处理
     - **单仓处理**：其他情况作为单仓处理

3. **下载单仓**：
   - 调用 `down()` 方法，下载单仓中的所有线路
   - 对每个线路调用 `download()` 方法进行下载

4. **下载线路**：
   - 调用 `download()` 方法，下载单个线路
   - 处理 JavaScript 渲染和图片解析等特殊情况
   - 下载并处理 jar 文件

### 4. 文件生成和处理阶段

1. **生成 all.json 文件**：
   - 调用 `all()` 方法，整合所有文件到 all.json
   - 调用 `remove_duplicates()` 方法，移除重复文件
   - 生成包含所有线路的 JSON 文件

2. **处理镜像代理**：
   - 调用 `mirror_proxy2new()` 方法，替换文件中的镜像代理路径
   - 根据 mirror 参数选择不同的替换策略

### 5. 推送更新阶段

1. **推送更新到 GitHub**：
   - 调用 `git_push()` 方法，推送更新到 GitHub
   - 重置 commit 计数，优化仓库大小

2. **输出结果**：
   - 计算并输出总耗时
   - 输出影视仓 APP 配置接口地址

## 核心功能模块

### 1. 文件处理模块

- **文件去重**：`remove_duplicates()` 方法，根据文件大小和哈希值移除重复文件
- **jar 文件处理**：`rename_jar_suffix()`、`remove_all_except_jar()`、`remove_jar_file()` 方法，处理 jar 文件的后缀和清理
- **Emoji 移除**：`remove_emojis()` 方法，移除文本中的 Emoji 表情
- **JSON 兼容处理**：`json_compatible()` 方法，处理不规范的 JSON 格式

### 2. 网络请求模块

- **HTTP 请求**：使用 requests 库发送 HTTP 请求
- **JavaScript 渲染**：`js_render()` 方法，处理需要 JavaScript 渲染的页面
- **图片解析**：`picparse()` 方法，从图片中解析文本
- **镜像代理处理**：`ghproxy()` 方法，处理 GitHub 镜像代理

### 3. GitHub 操作模块

- **仓库克隆**：`git_clone()` 方法，克隆 GitHub 仓库
- **仓库配置**：`get_local_repo()` 方法，配置本地仓库信息
- **代码推送**：`git_push()` 方法，推送更新到 GitHub

### 4. 镜像代理模块

- **镜像代理初始化**：`mirror_init()` 方法，根据 mirror 参数选择合适的镜像代理
- **镜像代理替换**：`mirror_proxy2new()` 方法，替换文件中的镜像代理路径
- **URL 替换**：`replace_urls_gh1()` 和 `replace_urls_gh2()` 方法，替换不同类型的镜像代理 URL

## 技术特点

1. **多镜像支持**：支持多种 GitHub 镜像代理，提高下载速度和稳定性
2. **容错处理**：对各种异常情况进行了处理，提高工具的稳定性
3. **自动化处理**：从克隆仓库到推送更新，实现了全自动化处理
4. **灵活配置**：通过环境变量和参数，实现了灵活的配置
5. **代码优化**：通过重置 commit 计数，优化了仓库大小

## 执行流程总结

1. **配置初始化** → 2. **仓库克隆** → 3. **在线接口处理** → 4. **文件生成** → 5. **镜像代理处理** → 6. **推送更新** → 7. **结果输出**

该工具通过自动化的方式，简化了 TVBox 资源的管理和更新流程，提高了效率和稳定性。