# denvdis 工具集文档索引

## 📚 文档清单

本项目的工具集使用文档由以下2个主要文档组成：

### 1. TOOLS_USAGE_GUIDE.md - 完整参考手册

📖 **内容**: 3500行详细参考文档

**涵盖工具**:
- ✅ **denv** - SASS表提取 (CUDA ≤10.1)
- ✅ **denv11** - SASS表提取 (CUDA 11.x)
- ✅ **denv12** - SASS表提取 (CUDA 12.x+)
- ✅ **cic12** - CICC中间代码提取
- ✅ **deptx** - PTX宏提取
- ✅ **nvb** - LLVM BC提取

**主要章节**:
1. **工具概述** - 快速参考表
2. **工具详解** - 每个工具的完整说明
   - 功能说明
   - 源代码位置
   - 构建命令
   - 工作原理（加密、数据定位、处理流程）
   - 使用示例
   - 输出文件结构
   - 关键参数说明
3. **完整工作流** - 真实场景脚本
4. **故障排除** - 常见错误和解决方案
5. **性能考虑** - 时间和空间复杂度
6. **高级用法** - 自定义提取、批量处理

**最适合**:
- 需要深入了解工具原理
- 想要自定义编译和部署
- 进行技术架构设计
- 编写论文或文档

### 2. PRACTICAL_TESTING_GUIDE.md - 实践操作指南

📖 **内容**: 2000行实用脚本和案例

**特点**:
- 针对 `basic_gemm.cubin` 文件的分析
- 说明工具的实际限制和解决方案
- 包含可直接运行的脚本

**提供脚本**:
1. **analyze_cubin.sh** - 完整分析流程
   - 收集基础信息
   - 进行反汇编
   - 使用项目工具分析
   - 生成统计报告

2. **analyze_sass.sh** - SASS指令统计
   - 指令分布统计
   - 内存访问计数
   - 控制流分析

3. **compare_disasm.sh** - 格式对比
   - 官方nvdisasm vs 项目工具
   - 输出差异分析

4. **build_and_analyze.sh** - 编译集成
   - 从编译到分析的完整流程

5. **CI/CD集成示例** - GitHub Actions工作流

**关键内容**:
- basic_gemm.cubin的可用性和限制
- 为什么denv需要官方nvdisasm而非CUBIN文件
- SASS输出格式解读
- 内核性能指标提取
- 常见分析场景

**最适合**:
- 快速上手和测试
- 一键分析CUBIN文件
- 集成到自动化流程
- 性能调优和分析

## 🎯 快速开始

### 根据使用目的选择文档

#### 「我想从官方CUDA工具提取加密表」
👉 **TOOLS_USAGE_GUIDE.md** → 章节 "1. denv/2. denv11/3. denv12"

**快速步骤**:
```bash
# 1. 确保有CUDA工具包
which nvdisasm

# 2. 链接工具
ln -sf /usr/bin/nvdisasm ./nvdisasm

# 3. 运行提取
./denv11 ./nvdisasm
ls data11/
```

#### 「我想分析一个CUBIN文件」
👉 **PRACTICAL_TESTING_GUIDE.md** → 章节 "脚本1: 完整分析流程"

**快速步骤**:
```bash
# 1. 使用提供的脚本
chmod +x analyze_cubin.sh
./analyze_cubin.sh your_kernel.cubin

# 2. 查看结果
cat analysis_*/statistics.txt
```

#### 「我想理解SASS指令编码」
👉 **TOOLS_USAGE_GUIDE.md** → 章节 "工作原理" + **PRACTICAL_TESTING_GUIDE.md** → 章节 "关键输出解读"

**步骤**:
```bash
# 1. 提取数据
./denv12 ./nvdisasm
ls data12/sm90/  # 查看编码表

# 2. 解析CUBIN
nvdisasm kernel.cubin > kernel.txt
grep "MOV\|FADD\|LDG" kernel.txt  # 搜索特定指令
```

#### 「我想从libnvrtc库提取BC代码」
👉 **TOOLS_USAGE_GUIDE.md** → 章节 "6. nvb - LLVM Bitcode提取工具"

**快速步骤**:
```bash
./nvb /usr/local/cuda/lib64/libnvrtc-builtins.so.12.5
llvm-dis *.bc  # 转换为可读格式
```

#### 「我想在CI/CD流程中集成分析」
👉 **PRACTICAL_TESTING_GUIDE.md** → 章节 "工具链集成 → CI/CD集成"

**方式**: 使用提供的GitHub Actions YAML文件

## 📋 文档详细索引

### TOOLS_USAGE_GUIDE.md

```
📄 TOOLS_USAGE_GUIDE.md
├─ 工具概述 (概览表)
├─ 1️⃣ denv 详解
│  ├─ 功能说明
│  ├─ 加密机制 (XOR-Salt)
│  ├─ 使用示例
│  └─ 输出结构
├─ 2️⃣ denv11 详解
│  ├─ 与denv的区别 (LZ4)
│  ├─ 支持架构表
│  └─ LZ4解压流程
├─ 3️⃣ denv12 详解
│  ├─ 新架构支持 (Hopper+)
│  └─ 种子差异
├─ 4️⃣ cic12 详解
│  ├─ CICC的作用
│  ├─ 提取内容列表
│  └─ 种子差异说明
├─ 5️⃣ deptx 详解
│  ├─ 提取内容
│  ├─ 数据定位
│  ├─ 与nvdisasm关系
│  └─ 使用示例
├─ 6️⃣ nvb 详解
│  ├─ 动态链接原理
│  ├─ 架构ID映射
│  ├─ 输出文件使用
│  └─ 错误处理
├─ 完整工作流示例
│  ├─ 场景1: 分析CUDA 12.x
│  └─ 场景2: 比较版本差异
├─ 故障排除 (6个常见错误)
├─ 性能考虑 (时间和空间)
└─ 高级用法
   ├─ 自定义提取范围
   ├─ 批量处理
   └─ 与其他工具集成
```

### PRACTICAL_TESTING_GUIDE.md

```
📄 PRACTICAL_TESTING_GUIDE.md
├─ basic_gemm.cubin 分析
│  ├─ 文件信息
│  ├─ 架构识别
│  └─ 为什么工具的限制说明
├─ 工具适用性对比表
├─ 正确的测试流程
│  ├─ 前置条件设置
│  ├─ 方案A: 从CUDA工具包提取
│  ├─ 方案B: 使用CUBIN间接分析
│  └─ 方案C: 使用test/目录工具
├─ 🎬 4个实践演练脚本
│  ├─ Script 1: analyze_cubin.sh (完整分析)
│  ├─ Script 2: analyze_sass.sh (SASS统计)
│  ├─ Script 3: compare_disasm.sh (格式对比)
│  └─ Script 4: build_and_analyze.sh (编译集成)
├─ 关键输出解读
│  ├─ 反汇编格式解析
│  ├─ 内核元数据识别
│  └─ 性能指标提取
├─ 常见分析场景
│  ├─ 检查共享内存使用
│  ├─ 评估寄存器压力
│  ├─ 分析控制流复杂度
│  └─ 内存访问模式识别
├─ 工具链集成
│  ├─ 与编译流程集成
│  └─ 与CI/CD集成 (GitHub Actions)
└─ 下一步建议
```

## 🔍 查找特定内容

### 想要找「如何使用denv11」？
1. 打开 `TOOLS_USAGE_GUIDE.md`
2. Ctrl+F 搜索 "### 2. denv11"
3. 查看"使用示例"章节

### 想要找「SASS指令统计脚本」？
1. 打开 `PRACTICAL_TESTING_GUIDE.md`
2. Ctrl+F 搜索 "### 脚本2：SASS指令统计"
3. 复制 `analyze_sass.sh` 脚本

### 想要找「如何在GitHub Actions中运行」？
1. 打开 `PRACTICAL_TESTING_GUIDE.md`
2. Ctrl+F 搜索 "CI/CD集成"
3. 使用提供的YAML配置

### 想要找「加密种子是什么」？
1. 打开 `TOOLS_USAGE_GUIDE.md`
2. Ctrl+F 搜索 "加密种子"
3. 查看各工具的"关键参数"部分

## 📊 文档统计

| 项目 | TOOLS_USAGE | PRACTICAL_TESTING | 合计 |
|------|-------------|-------------------|------|
| 行数 | 850+ | 450+ | 1300+ |
| 工具覆盖 | 6个 | 案例分析 | - |
| 代码示例 | 15+ | 5个脚本 | 20+ |
| 图表/表格 | 20+ | 10+ | 30+ |
| 故障处理 | 6个 | 4个场景 | 10+ |

## 🎓 推荐阅读顺序

**首次使用** (15分钟):
1. TOOLS_USAGE_GUIDE.md → "工具概述"
2. PRACTICAL_TESTING_GUIDE.md → "前置条件设置"
3. 选择一个脚本运行

**深入学习** (2-3小时):
1. TOOLS_USAGE_GUIDE.md → 完整阅读（选择相关工具）
2. PRACTICAL_TESTING_GUIDE.md → 运行所有脚本
3. 修改脚本进行定制

**专项研究** (需要时查阅):
- 特定工具的"工作原理"章节
- 故障排除部分
- 高级用法和集成示例

## 🚀 使用示例速查

### 提取最新CUDA的SASS表
```bash
./denv12 /usr/bin/nvdisasm  # TOOLS_USAGE_GUIDE.md → 章节3
```

### 快速分析CUBIN文件
```bash
./analyze_cubin.sh kernel.cubin  # PRACTICAL_TESTING_GUIDE.md → 脚本1
```

### 获取SASS指令统计
```bash
./analyze_sass.sh kernel.cubin  # PRACTICAL_TESTING_GUIDE.md → 脚本2
```

### 集成到编译流程
```bash
chmod +x build_and_analyze.sh
./build_and_analyze.sh  # PRACTICAL_TESTING_GUIDE.md → 脚本4
```

### 提取LLVM BC代码
```bash
./nvb /usr/local/cuda/lib64/libnvrtc-builtins.so.12.5
# TOOLS_USAGE_GUIDE.md → 章节6
```

## 📝 文件扫描码 (快速定位)

在编辑器中使用Markdown导航功能:

**TOOLS_USAGE_GUIDE.md 的标题树**:
```
# denvdis 工具集使用指南
## 工具概述
## 工具详解与使用方法
### 1. denv - SASS表提取工具
### 2. denv11 - SASS表提取工具
### 3. denv12 - SASS表提取工具
### 4. cic12 - CICC中间代码提取工具
### 5. deptx - PTX宏代码提取工具
### 6. nvb - LLVM Bitcode提取工具
## 完整工作流示例
## 故障排除
## 性能考虑
## 高级用法
```

**PRACTICAL_TESTING_GUIDE.md 的标题树**:
```
# denvdis 工具实践指南
## 文件分析
## 工具可用性与限制
## 正确的测试流程
## 实践演练脚本
## 关键输出解读
## 常见分析场景
## 工具链集成
```

---

## 📞 遇到问题？

1. **工具怎么用?** → TOOLS_USAGE_GUIDE.md
2. **为什么出错?** → TOOLS_USAGE_GUIDE.md 故障排除部分
3. **想快速测试?** → PRACTICAL_TESTING_GUIDE.md 脚本部分
4. **想深入理解?** → TOOLS_USAGE_GUIDE.md 工作原理章节
5. **想集成到项目?** → PRACTICAL_TESTING_GUIDE.md 工具链集成部分

---

📅 **最后更新**: 2026-04-03
🔗 **相关资源**: [README.md](README.md) | [ANALYSIS_00_BASIC_GEMM.md](ANALYSIS_00_BASIC_GEMM.md)
