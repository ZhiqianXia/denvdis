# DENVDIS 50步探索计划 - 无源码修改版本

## 项目概述
**denvdis** 是一个专门用于反向工程NVIDIA CUDA GPU二进制文件的工具包。通过破解CUDA编译器加密，提供对GPU代码生成全链条的可见性。

---

## **Phase 1: 基础架构学习 (Steps 1-10)**

### Step 1: 阅读项目README
- 文件: `/home/xpeng/github/denvdis/README.md`
- 学习内容:
  - 4个主要工具: `ced`, `nvd`, `ina`, `pa`
  - 依赖: ELFIO, FP16
  - blog文档参考

### Step 2: 检查Makefile结构
- 文件: `/home/xpeng/github/denvdis/Makefile`
- 学习内容:
  - 构建命令: `denv`, `denv7`, `denv11`, `denv12`, `deptx`, `deptx12`, `cic12`, `nvb`
  - 编译器标志
  - 依赖库配置

### Step 3: 理解目录结构
- 主要目录:
  - `data/` - Fermi-Maxwell SASS编码表
  - `data11/` - Maxwell-Ada SASS编码表
  - `data12/` - CUDA 12+ SASS编码表
  - `scripts/` - Perl分析脚本
  - `test/` - SASS工具实现
  - `cudaso/` - 二进制分析库

### Step 4: 学习CUDA二进制格式
- 文件: `/home/xpeng/github/denvdis/sht.txt` (ELF节类型)
- 学习内容:
  - SHT_CUDA_INFO (0x70000000)
  - SHT_CUDA_METADATA (0x70000004)
  - SHT_CUDA_GLOBAL, SHT_CUDA_CONSTANT等

### Step 5: 映射SM架构代号
- 文件: `/home/xpeng/github/denvdis/sm_version.txt`
- 学习内容:
  - Fermi: sm10-13 (0xa-0xd)
  - Kepler: sm20-35 (0x14-0x1E)
  - Maxwell: sm50-53 (0x32-0x35)
  - Pascal: sm60-62 (0x3c-0x3e)
  - Volta: sm70 (0x46)
  - Turing: sm75 (0x4b)
  - Ampere: sm80-86 (0x50-0x56)
  - Ada: sm89-90 (0x59-0x5a)
  - Hopper: sm100-120 (0x64-0x78)

### Step 6: 研究加密机制
- 文件: `/home/xpeng/github/denvdis/denv.cc` (第13-40行)
- 学习内容:
  - 64个种子数组 (seed[])
  - 盐表 (salt = seeds指针)
  - XOR加密原理
  - 伪随机数生成 (v3 = 1103515245 * ctx->wtf + 0x3039)

### Step 7: 分析数据文件组织
- 文件: `/home/xpeng/github/denvdis/data/sm75_1.txt` 等
- 学习内容:
  - 指令编码表格式
  - 字段定义
  - 形式变体

### Step 8: 识别CUDA版本边界
- denv (≤10.1): Fermi-Maxwell
- denv11/deptx (11.x): Maxwell-Ada
- denv12/deptx12 (12.x): Hopper+

### Step 9: 记录依赖项
- ELFIO (ELF解析)
- FP16 (半精度浮点)
- lz4 (LZ4压缩)
- libudis86 (x86反汇编)
- readline (命令行编辑)

### Step 10: 创建架构参考表
- 跨引用 data/, data11/, data12/
- 记录各版本支持的SM
- 标记加密种子对应关系

---

## **Phase 2: 提取和密码本分析 (Steps 11-20)**

### Step 11: 分析denv.cc解密逻辑
- 重点代码:
  ```c
  struct decr_ctx {
    _DWORD wtf;     // 伪RNG状态
    _DWORD seed;    // 当前种子
    _DWORD res;     // 结果缓存
    _BYTE l;        // 延续字节
  };
  ```

### Step 12: 理解LZ4解压缩
- denv11/denv12使用LZ4压缩
- 研究何时应用: SASS表的压缩存储

### Step 13: 分析one_md结构
- 文件: `/home/xpeng/github/denvdis/denv.cc` (~第90行)
- 字段含义:
  - `off`: 库文件中偏移
  - `size`: 加密数据大小
  - `cont`: 延续位标志
  - `init`: 初始化值 (0x4816 或 0x1486等)
  - `name`: SM型号名称

### Step 14: 研究ELF节提取
- 哪些节被解析
- 如何找到加密的SASS表
- 节头处理

### Step 15: 创建种子表映射
- 种子[0..63]与指令类型的关联
- salt表的用途

### Step 16: 跟踪盐生成
- salt = (const _BYTE *)seeds
- salt[字节值] 用于XOR

### Step 17: 分析libnvptxcompiler检测
- 如何工具定位NVIDIA库
- 版本识别方法

### Step 18: 研究版本检测
- 从ELF头识别CUDA版本
- 库文件特征

### Step 19: 研究延续位
- 某些指令跨越多个128位字
- 编码如何处理指令链

### Step 20: 记录完整提取管道
- .so → ELF解析 → 种子选择 → 解密 → LZ4解压 → data/输出

---

## **Phase 3: PTX和中间表示 (Steps 21-25)**

### Step 21: 研究PTX语法
- 伪汇编语言
- 架构独立

### Step 22: 分析ptxas.cc
- PTX二进制格式解析
- 中间代码提取

### Step 23: 理解CICC中间格式
- 编译器内部表示
- cic12.cc提取器

### Step 24: CUDA 12 CICC变化
- sm100/101/103/120新型号
- 编码差异

### Step 25: 研究编译器传递
- 文件: `/home/xpeng/github/denvdis/passes.txt`
- 优化、调度、寄存器分配等

---

## **Phase 4: SASS指令编码 (Steps 26-35)**

### Step 26: SASS 128位格式
- 指令字布局
- 字段位宽

### Step 27: 指令变体
- LDG: 14种形式
- F2FP: 60种形式
- 原因分析

### Step 28: 编码表解析
- data/sm75_1.txt格式
- 字段定义表

### Step 29: ina过滤器
- `+f`: 浮点立即数
- `+i`: 整数立即数
- `+C`: 常数库
- `+m`: 内存指令
- `+d`: 描述符引用
- `+u`: 统一寄存器
- `!`: 重置所有过滤器

### Step 30: 谓词编码
- nvd -p 选项
- 条件执行

### Step 31: 延迟信息
- nvd -S 调度表
- 指令延迟和吞吐量

### Step 32: LUT操作
- LOP3三输入逻辑
- 16位查找表

### Step 33: 统一寄存器
- nvd -u 跟踪
- 特殊寄存器处理

### Step 34: 寄存器追踪
- nvd -T 生命周期分析
- 寄存器冲突检测

### Step 35: 指令分类
- 内存操作 (LDG, STG等)
- 计算操作 (FADD, FMUL等)
- 控制流 (BRA, EXIT等)
- 转换指令 (F2F, I2F等)

---

## **Phase 5: CUBIN格式和节解析 (Steps 36-40)**

### Step 36: ELF头字段
- e1_type (ET_EXEC)
- e1_machine (0xBE, 190 = CUDA)
- e1_version (CUDA版本)
- e1_flags

### Step 37: CUDA节类型
- sht.txt 完整列表
- 每个节的含义

### Step 38: .text节
- 实际SASS指令
- 对齐和大小

### Step 39: 元数据节
- INFO: 函数信息
- METADATA: 参数、内存
- CALLGRAPH: 函数调用

### Step 40: 重定位
- R_CUDA_* 重定位类型
- 符号解析

---

## **Phase 6: 工具使用方法 (Steps 41-45)**

### Step 41: 编译nvd
- 依赖: ELFIO, FP16, lz4
- 输出格式: SASS反汇编

### Step 42: 测试ina
- 交互式汇编
- 形式选择过滤

### Step 43: 学习ced补丁语法
- sed风格补丁
- 修改指令

### Step 44: 使用pa解析
- 解析nvdisasm输出
- 格式转换

### Step 45: ced_base.cc库
- 核心功能实现
- 被所有工具使用

---

## **Phase 7: 脚本和自动化 (Steps 46-50)**

### Step 46: ead.pl分析
- 文件: `/home/xpeng/github/denvdis/scripts/ead.pl`
- 生成C++编码表

### Step 47: pal.pl生成
- 延迟表创建
- 调度模型

### Step 48: dump.pl验证
- ELF解析脚本
- 节内容检查

### Step 49: dg.pl转换
- 元数据到代码生成

### Step 50: 完整工作流
- 整合所有工具链
- 自定义分析管道

---

## **CUDA到SASS反向技术链**

```
┌─────────────────────┐
│  CUDA C/C++源代码   │
│  (host + kernel)    │
└──────────┬──────────┘
           │ nvcc前端
┌──────────┴──────────┐
│   LLVM IR (中间表示) │
│ (代码生成、优化)     │
└──────────┬──────────┘
           │ LLVM→PTX代码生成
┌──────────┴──────────────┐
│  PTX (伪汇编)            │
│  (可移植、与架构无关)    │
└──────────┬──────────────┘
           │ ptxas (PTX汇编器)
┌──────────┴──────────────┐
│  CICC (编译器中间代码)   │
│  (机器特定IR)           │
│  (调度、寄存器分配)     │
└──────────┬──────────────┘
           │ CICC→SASS代码生成
┌──────────┴──────────────┐
│  SASS (机器代码)        │
│  (128位指令, GPU特定)   │
│  (硬件操作码)           │
└──────────┬──────────────┘
           │ 二进制打包
┌──────────┴──────────────┐
│  CUBIN (CUDA二进制)      │
│  (ELF格式容器)          │
│  (元数据、HOST代码、   │
│   SASS段)               │
└──────────────────────────┘
```

---

## **关键文件快速参考**

| 文件 | 用途 | 优先级 |
|------|------|--------|
| denv.cc | 加密解密逻辑 | 最高 |
| denv11.cc | LZ4支持版本 | 最高 |
| denv12.cc | Hopper支持 | 最高 |
| ptxas.cc | PTX解析 | 高 |
| cic12.cc | CICC提取 | 高 |
| test/nvd.cc | SASS反汇编 | 高 |
| test/ina.cc | SASS汇编 | 高 |
| test/ced.cc | CUBIN编辑 | 中 |
| scripts/ead.pl | 编码生成 | 中 |
| sm_version.txt | 架构映射 | 参考 |
| sht.txt | ELF节类型 | 参考 |
| passes.txt | 编译阶段 | 参考 |

---

## **探索方法论（无源码修改）**

1. **代码阅读** - 分析源文件逻辑
2. **文档学习** - README和blog参考
3. **数据分析** - 研究data/文件格式
4. **逆向推理** - 从输出推断内部逻辑
5. **笔记记录** - 文档化发现和假设
6. **验证实验** - 使用现有工具确认理论

---

## **进度跟踪**

- [ ] Phase 1完成 (基础理解)
- [ ] Phase 2完成 (加密/解密)
- [ ] Phase 3完成 (PTX/CICC)
- [ ] Phase 4完成 (SASS编码)
- [ ] Phase 5完成 (CUBIN格式)
- [ ] Phase 6完成 (工具使用)
- [ ] Phase 7完成 (自动化)

