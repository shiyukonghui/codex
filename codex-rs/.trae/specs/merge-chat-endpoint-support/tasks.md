# Tasks

- [x] Task 1: 添加WireApi枚举到provider.rs
  - [x] SubTask 1.1: 在provider.rs中添加WireApi枚举定义
  - [x] SubTask 1.2: 在Provider结构体中添加wire字段
  - [x] SubTask 1.3: 更新Provider的is_azure_responses_endpoint方法

- [x] Task 2: 添加Prompt结构体到common.rs
  - [x] SubTask 2.1: 在common.rs中添加Prompt结构体定义

- [x] Task 3: 添加sse/chat.rs模块
  - [x] SubTask 3.1: 复制chat.rs文件到sse目录
  - [x] SubTask 3.2: 更新sse/mod.rs导出chat模块

- [x] Task 4: 添加endpoint/streaming.rs模块
  - [x] SubTask 4.1: 创建streaming.rs文件
  - [x] SubTask 4.2: 更新endpoint/mod.rs添加streaming模块

- [x] Task 5: 添加endpoint/chat_legacy.rs模块
  - [x] SubTask 5.1: 创建chat_legacy.rs文件
  - [x] SubTask 5.2: 更新endpoint/mod.rs添加chat_legacy模块

- [x] Task 6: 添加requests/chat.rs和chat_legacy.rs模块
  - [x] SubTask 6.1: 创建requests/chat.rs文件
  - [x] SubTask 6.2: 创建requests/chat_legacy.rs文件
  - [x] SubTask 6.3: 更新requests/mod.rs添加chat和chat_legacy模块

- [x] Task 7: 更新lib.rs导出
  - [x] SubTask 7.1: 添加ChatClient导出
  - [x] SubTask 7.2: 添加ChatRequest导出
  - [x] SubTask 7.3: 添加ChatRequestBuilder导出
  - [x] SubTask 7.4: 添加Prompt导出
  - [x] SubTask 7.5: 添加WireApi导出
  - [x] SubTask 7.6: 添加AggregateStreamExt导出

- [x] Task 8: 修复编译错误
  - [x] SubTask 8.1: 运行cargo build检查编译错误
  - [x] SubTask 8.2: 修复所有编译错误

- [x] Task 9: 运行测试验证
  - [x] SubTask 9.1: 运行cargo test -p codex-api
  - [x] SubTask 9.2: 修复任何测试失败

# Task Dependencies
- [Task 3] depends on [Task 1]
- [Task 4] depends on [Task 1]
- [Task 5] depends on [Task 1, Task 3, Task 4]
- [Task 6] depends on [Task 1]
- [Task 7] depends on [Task 2, Task 5, Task 6]
- [Task 8] depends on [Task 7]
- [Task 9] depends on [Task 8]
