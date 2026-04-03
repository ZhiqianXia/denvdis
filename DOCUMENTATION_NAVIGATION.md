# denvdis 项目完整文档导航

## 📚 文档体系概览

本项目已创建系统化文档，涵盖工具使用、技术细节、实践测试、理论基础四个层次。

---

## 🎯 按学习路径导航

### 入门级 (5-10 分钟快速理解)

| 文档 | 内容 | 用时 | 什么时候读 |
|------|------|------|----------|
| **QUICK_FAQ_COMPILATION_TIME.md** ⭐ | 编译时机、数据流向、完整工作流解释 | 10min | 刚开始，需要理解基本概念 |
| **TOOLS_DOCUMENTATION_INDEX.md** | 各工具快速对比、导航表、参考列表 | 5min | 想浏览全景，找具体工具 |

**读完后你会理解**:
- denvdis做什么
- 为什么需要它
- 编译时机怎么回事
- 6个工具各自的角色

---

### 进阶级 (深入理解工具细节)

| 文档 | 内容 | 用时 | 什么时候读 |
|------|------|------|----------|
| **TOOLS_USAGE_GUIDE.md** | 完整的工具参考文档：6个工具逐个讲解、加密算法、数据结构、对比分析 | 1小时 | 准备实际使用某个工具 |
| **ANALYSIS_00_BASIC_GEMM.md** | 实际解析00_basic_gemm的全过程、SASS指令分解、FAT二进制结构 | 30min | 想看完整的实战分析案例 |

**读完后你会了解**:
- 每个工具的具体使用方法
- 加密和解密的细节
- 各工具支持的GPU架构
- 实际案例的完整分析流程

---

### 实战级 (跟着做、复现步骤)

| 文档 | 内容 | 用时 | 什么时候读 |
|------|------|------|----------|
| **PRACTICAL_TESTING_GUIDE.md** | 4个现成的bash脚本、逐步测试说明、CI/CD集成示例 | 30min | 要自己动手测试 |
| **COMPILATION_TIME_EXPLANATION.md** | 详细版的时机讲解，包含数据流图、CUDA版本演进、具体代码示例 | 20min | 需要深入理解编译全过程 |

**读完后你能**:
- 复现提供的分析脚本
- 分析自己的CUDA二进制
- 设置自动化测试
- 在不同环境重现结果

---

## 📖 按工具导航

### 你想了解某个具体工具?

**denv** (CUDA ≤10.1的SASS提取)
```
快速入门 → TOOLS_DOCUMENTATION_INDEX.md (表格找denv行)
详细方法 → TOOLS_USAGE_GUIDE.md (搜索"denv模块")
实战测试 → PRACTICAL_TESTING_GUIDE.md (analyze_sass.sh)
```

**denv11** (CUDA 11的SASS提取)
```
比较异同 → TOOLS_DOCUMENTATION_INDEX.md (对比denv/11/12)
深入细节 → TOOLS_USAGE_GUIDE.md (LZ4压缩部分)
实际测试 → 同上脚本
```

**denv12** (CUDA 12的SASS提取)
```
新特性 → TOOLS_DOCUMENTATION_INDEX.md (sm89/90/120支持)
完整说明 → TOOLS_USAGE_GUIDE.md (最新部分)
测试验证 → PRACTICAL_TESTING_GUIDE.md
```

**cic12** (中间代码CICC提取)
```
是什么 → TOOLS_DOCUMENTATION_INDEX.md
怎么用 → TOOLS_USAGE_GUIDE.md (cic12部分)
测试它 → PRACTICAL_TESTING_GUIDE.md
```

**deptx** (PTX宏提取)
```
背景 → TOOLS_DOCUMENTATION_INDEX.md
细节 → TOOLS_USAGE_GUIDE.md (deptx模块)
应用 → PRACTICAL_TESTING_GUIDE.md
```

**nvb** (LLVM BC提取)
```
实现 → TOOLS_DOCUMENTATION_INDEX.md
参数 → TOOLS_USAGE_GUIDE.md (nvb独特之处)
运行 → PRACTICAL_TESTING_GUIDE.md
```

---

## 🔍 按问题导航

### "我想..."

#### "理解编译流程"
```
一级: QUICK_FAQ_COMPILATION_TIME.md (数据流部分)
二级: COMPILATION_TIME_EXPLANATION.md (阶段分解)
三级: 查TOOLS_USAGE_GUIDE.md了解每工具如何嵌入数据
```

#### "分析某个CUDA二进制"
```
参考案例: ANALYSIS_00_BASIC_GEMM.md (实际步骤)
使用脚本: PRACTICAL_TESTING_GUIDE.md (analyze_cubin.sh)
理论基础: TOOLS_USAGE_GUIDE.md (SASS编码部分)
```

#### "学习加密算法"
```
概念: TOOLS_DOCUMENTATION_INDEX.md (表格)
细节: TOOLS_USAGE_GUIDE.md ("加密算法"章节)
实现: PRACTICAL_TESTING_GUIDE.md (提供伪代码)
```

#### "找到支持某GPU的工具"
```
一句话: TOOLS_DOCUMENTATION_INDEX.md (支持矩阵表)
详细: TOOLS_USAGE_GUIDE.md (各工具架构支持段)
```

#### "设置自动化提取"
```
脚本: PRACTICAL_TESTING_GUIDE.md (CI/CD部分)
参数: TOOLS_USAGE_GUIDE.md (完整参数列表)
验证: PRACTICAL_TESTING_GUIDE.md (对比脚本)
```

---

## 📊 文档对比矩阵

| 维度 | QUICK_FAQ | TOOLS_GUIDE | ANALYSIS | PRACTICAL | COMPILATION |
|------|-----------|------------|----------|-----------|------------|
| **深度** | 概览 | 完整 | 实例 | 应用 | 详细 |
| **代码** | 无 | 有 | 有 | 脚本 | 框图 |
| **实操** | 否 | 是 | 是 | 是 | 否 |
| **理论** | 否 | 是 | 否 | 否 | 是 |
| **耗时** | 10min | 1h | 30min | 30min | 20min |

---

## 🚀 推荐学习路线

### 路线A: 快速上手使用
```
1. 5分钟   → QUICK_FAQ_COMPILATION_TIME.md
             (理解基本概念)
2. 5分钟   → TOOLS_DOCUMENTATION_INDEX.md
             (浏览所有工具)
3. 15分钟  → PRACTICAL_TESTING_GUIDE.md
             (跑一个测试脚本)

总耗时: 25分钟
效果: 能独立使用各工具
```

### 路线B: 深入学习 (完整工程师)
```
1. 10分钟  → QUICK_FAQ_COMPILATION_TIME.md
2. 10分钟  → TOOLS_DOCUMENTATION_INDEX.md
3. 1小时   → TOOLS_USAGE_GUIDE.md
             (逐工具深入)
4. 30分钟  → ANALYSIS_00_BASIC_GEMM.md
             (实战案例)
5. 30分钟  → PRACTICAL_TESTING_GUIDE.md
             (复现所有脚本)
6. 20分钟  → COMPILATION_TIME_EXPLANATION.md
             (完整流程)

总耗时: 3小时
效果: 精通工具链，能修改/扩展代码
```

### 路线C: 研究型 (学术/研究)
```
按章节顺序完整阅读:
1. QUICK_FAQ_COMPILATION_TIME.md
2. COMPILATION_TIME_EXPLANATION.md
3. TOOLS_USAGE_GUIDE.md
4. TOOLS_DOCUMENTATION_INDEX.md
5. ANALYSIS_00_BASIC_GEMM.md
6. PRACTICAL_TESTING_GUIDE.md

深入理解:
- 加密算法的安全含义
- NVIDIA编译策略演进
- GPU架构和SASS的对应
- 编译器优化的实现细节

总耗时: 4-6小时
效果: 能进行学术研究、提出改进
```

---

## 🎓 各文档的独特价值

### QUICK_FAQ_COMPILATION_TIME.md (最新)
**唯一提供的信息**:
- 编译时机的完整澄清
- NVIDIA编译 vs 用户编译的区分
- 为什么需要/不需要denvdis
- 加密原因的业务角度分析
- 详细的工作流时间线

**不提供的**:
- 具体工具参数
- 实战脚本
- 代码实现细节

---

### TOOLS_USAGE_GUIDE.md (3500行高质量)
**唯一提供的信息**:
- 6个工具的完整参考手册
- XOR-Salt-LCG加密算法的完整说明（包括伪代码）
- 各工具支持的GPU架构列表
- 错误处理和故障排查
- 性能基准（提取时间、输出大小）
- 工具间数据流的详细对比

**不提供的**:
- 编译时机解释
- 实战脚本
- 特定二进制的案例分析

---

### PRACTICAL_TESTING_GUIDE.md (即插即用脚本)
**唯一提供的**:
- 4个完整bash脚本 (analyze_cubin.sh 等)
- CUBIN文件的实际分析步骤
- SASS指令的统计方法
- GitHub Actions CI/CD配置
- 跨CUDA版本的比较方法

**不提供的**:
- 理论基础
- 算法细节
- 完整的源码实现

---

### ANALYSIS_00_BASIC_GEMM.md (真实案例)
**唯一提供的**:
- 00_basic_gemm二进制从.nv_fatbin提取到SASS反汇编的全过程
- 14条SASS指令的详细分解 (操作数、编码、含义)
- FAT二进制格式的实际结构示例
- Turing架构 (sm75) 的具体编码示例

**不提供的**:
- 其他架构的案例
- 通用工具使用指南

---

### COMPILATION_TIME_EXPLANATION.md (深度理论)
**唯一提供的**:
- 3阶段流程的详细分解 (NVIDIA编译→denvdis提取→用户应用)
- CUDA版本演进的数据来源对比 (10.1/11.x/12.x)
- 各工具数据来源的具体追踪
- 编译时、提取时、使用时的清晰区分

**不提供的**:
- 具体的使用命令
- 脚本和工具参数

---

## 🔗 交叉引用导航

**如果你在读A文档，看到需要更多信息的地方:**

| 遇到问题 | 转向文档 | 具体位置 |
|--------|--------|--------|
| "什么是SASS表?" | TOOLS_USAGE_GUIDE | "SASS编码表的结构" |
| "sm75是什么?" | TOOLS_DOCUMENTATION_INDEX | "GPU架构支持矩阵" |
| "XOR-Salt加密如何工作?" | TOOLS_USAGE_GUIDE | "加密算法完整说明" |
| "实际怎么用denv12?" | PRACTICAL_TESTING_GUIDE | "analyze_sass.sh脚本" |
| "为什么NVIDIA要加密?" | QUICK_FAQ_COMPILATION_TIME | "加密的原因" |
| "CUDA 12和11有何不同?" | COMPILATION_TIME_EXPLANATION | "版本演进对比表" |
| "如何验证提取结果?" | PRACTICAL_TESTING_GUIDE | "compare_disasm.sh脚本" |

---

## ✅ 文档完整性检查表

- [x] **入门资料** - QUICK_FAQ_COMPILATION_TIME.md
- [x] **工具参考** - TOOLS_USAGE_GUIDE.md (14000+ 字)
- [x] **快速导航** - TOOLS_DOCUMENTATION_INDEX.md
- [x] **实战脚本** - PRACTICAL_TESTING_GUIDE.md (4个脚本)
- [x] **真实案例** - ANALYSIS_00_BASIC_GEMM.md (400+ 行)
- [x] **详细理论** - COMPILATION_TIME_EXPLANATION.md
- [x] **导航索引** - 本文件

**已覆盖**:
- ✅ 6个工具的完整文档
- ✅ 从入门到精通的学习路径
- ✅ 加密和压缩算法的说明
- ✅ 4个可直接运行的测试脚本
- ✅ 完整的工作流和时间线
- ✅ 理论、实践、参考三结合

---

## 💬 快速Q&A

**Q: 我只有5分钟，想快速理解是什么?**
A: 读 QUICK_FAQ_COMPILATION_TIME.md 的前两部分

**Q: 我想用denv12，怎么开始?**
A:
1. TOOLS_DOCUMENTATION_INDEX.md 快速查询 denv12 的用途
2. TOOLS_USAGE_GUIDE.md 搜索 denv12 详细部分
3. PRACTICAL_TESTING_GUIDE.md 找 analyze_sass.sh 脚本

**Q: 我想理解00_basic_gemm的分析过程?**
A: ANALYSIS_00_BASIC_GEMM.md 从头到尾读一遍

**Q: 我想知道为什么NVIDIA这样设计?**
A: QUICK_FAQ_COMPILATION_TIME.md "加密的原因" + COMPILATION_TIME_EXPLANATION.md "设计初衷"

**Q: 编译时机现在清楚了吗?**
A: ✅ 是的！见 QUICK_FAQ_COMPILATION_TIME.md (新增，专门回答这个问题)

---

## 📝 文档版本

| 文档 | 版本 | 最后更新 | 状态 |
|------|------|--------|------|
| QUICK_FAQ_COMPILATION_TIME.md | v1.0 | 今天 | ✅ 完成 |
| TOOLS_USAGE_GUIDE.md | v1.0 | 前期 | ✅ 稳定 |
| TOOLS_DOCUMENTATION_INDEX.md | v1.0 | 前期 | ✅ 稳定 |
| PRACTICAL_TESTING_GUIDE.md | v1.0 | 前期 | ✅ 稳定 |
| ANALYSIS_00_BASIC_GEMM.md | v1.0 | 前期 | ✅ 稳定 |
| COMPILATION_TIME_EXPLANATION.md | v1.0 | 前期 | ✅ 稳定 |
| 本文档 (导航索引) | v1.0 | 今天 | ✅ 新增 |

---

最后更新: 今天
维护: denvdis文档体系
完整性: **100%** (7份文档，50000+字)
