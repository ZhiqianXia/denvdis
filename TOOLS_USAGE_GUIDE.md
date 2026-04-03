# denvdis 工具集使用指南

## 工具概述

denvdis项目包含了进行CUDA GPU代码分析和提取的多个核心工具。以下是6个主要工具的完整使用指南：

| 工具 | 功能 | 输入 | 输出 | CUDA版本 |
|------|------|------|------|----------|
| `denv` | 从nvdisasm提取SASS加密表 | nvdisasm二进制 | data/目录 | ≤10.1 |
| `denv11` | 从nvdisasm提取并解压LZ4编码的SASS表 | nvdisasm二进制 | data11/目录 | 11.x |
| `denv12` | 从nvdisasm提取并解压LZ4编码的SASS表 | nvdisasm二进制 | data12/目录 | 12.x+ |
| `cic12` | 从nvdisasm提取CICC中间代码 | nvdisasm二进制 | cicc12/目录 | 12.x |
| `deptx` | 从ptxas提取加密的PTX宏代码 | ptxas二进制 | macros/目录 | 10.1 |
| `nvb` | 从libnvrtc-builtins库提取LLVM BC代码 | libnvrtc-builtins.so | *.bc文件 | 任意 |

---

## 工具详解与使用方法

### 1. denv - SASS表提取工具 (CUDA ≤10.1)

**功能**: 从CUDA 10.1及更早版本的nvdisasm二进制文件中提取加密的SASS指令表。

**源代码**: `denv.cc`
**构建命令**: `gcc -o denv -I../../../../ELFIO denv.cc -lstdc++`
**依赖**: ELFIO库，自定义ELF解析

#### 工作原理

1. **加密方式**: XOR-Based加密
   - 64个32位种子 (seeds数组)
   - 盐表来自seed数据
   - LCG随机数生成器: `seed = 0x41C64E6D * wtf + 0x3039`

2. **数据定位**
   - 查找`.data`节
   - 在特定偏移位置定位机器描述表 (mds结构)

3. **解密流程**
   ```c
   struct one_md {
       size_t off, size;      // 偏移和大小
       const char *name;      // 输出文件名
   };

   // 对每个表项：
   // 1. 从.data节读取加密数据
   // 2. 使用XOR-Salt-LCG算法解密
   // 3. 写入到输出文件
   ```

#### 使用示例

```bash
# 前提条件：需要有CUDA 10.1的nvdisasm二进制
# 创建符号链接
ln -sf /path/to/nvdisasm/cuda-10.1 ./nvdisasm

# 运行提取
./denv ./nvdisasm
# 或指定路径
./denv /usr/local/cuda-10.1/bin/nvdisasm

# 检查输出
ls -la data/
# 输出: 各个SASS表文件
```

#### 输出文件结构

```
data/
├── sm30/       # Kepler架构数据
├── sm35/       # ...
├── sm50/       # Maxwell架构
├── sm52/
├── sm60/       # Pascal架构
├── sm61/
├── sm70/       # Volta架构
├── sm75/       # Turing架构
└── ...
```

#### 关键参数

- **加密种子数**: 64个 (覆盖64种标准操作)
- **盐表大小**: 256字节
- **LCG参数**: m=0x41C64E6D, c=0x3039

---

### 2. denv11 - SASS表提取工具 (CUDA 11.x)

**功能**: 从CUDA 11.x版本的nvdisasm二进制中提取LZ4压缩的SASS表。

**源代码**: `denv11.cc`
**构建命令**: `gcc -o denv11 -I../../../../ELFIO -I../lz4/lib denv11.cc ../lz4/lib/liblz4.a -lstdc++`
**依赖**: ELFIO库，LZ4压缩库

#### 与denv的区别

| 特性 | denv | denv11 |
|------|------|--------|
| 压缩 | 否 | LZ4 |
| 数据格式 | 原始SASS表 | 压缩后的SASS表 |
| 输出目录 | data/ | data11/ |
| CUDA版本 | ≤10.1 | 11.x |
| SM架构范围 | sm30-sm75 | sm50-sm86 |

#### 使用示例

```bash
# 前提：需要CUDA 11.x的nvdisasm
ln -sf /usr/local/cuda-11.x/bin/nvdisasm ./nvdisasm

# 运行提取和解压
./denv11 ./nvdisasm

# 验证输出
ls -la data11/
# 输出: LZ4解压后的SASS表
```

#### LZ4解压流程

```c
// denv11内部处理
1. 读取加密的LZ4压缩块
2. 使用XOR盐解密
3. 使用LZ4_decompress_safe()解压
4. 写入解压后的SASS表
```

#### 已支持架构

- **Maxwell**: sm50, sm52, sm53
- **Pascal**: sm60, sm61, sm62
- **Volta**: sm70
- **Turing**: sm75
- **Ampere**: sm80, sm86

---

### 3. denv12 - SASS表提取工具 (CUDA 12.x+)

**功能**: 从CUDA 12.x及更新版本的nvdisasm中提取LZ4压缩的SASS表，支持最新GPU架构。

**源代码**: `denv12.cc`
**构建命令**: `gcc -o denv12 -I../../../../ELFIO -I../lz4/lib denv12.cc ../lz4/lib/liblz4.a -lstdc++`
**依赖**: ELFIO库，LZ4压缩库

#### 扩展支持

相对于denv11，denv12支持更多新架构：

```
📊 架构时间线对比
┌─────────┬─────────┬─────────┐
│ 工具    │ 基础范围│ 扩展范围│
├─────────┼─────────┼─────────┤
│ denv    │ sm-75  │ Fermi   │
│ denv11  │ sm86   │ Ampere  │
│ denv12  │ sm120+ │ Hopper+ │
└─────────┴─────────┴─────────┘
```

新增架构：
- **Ada**: sm89
- **Hopper**: sm90, sm90a
- **Blackwell**: sm100, sm101, sm103, sm120

#### 使用示例

```bash
# 提取CUDA 12.x的数据
ln -sf /usr/local/cuda-12.x/bin/nvdisasm ./nvdisasm
./denv12 ./nvdisasm

# 输出到data12/
ls -la data12/sm90/  # Hopper架构数据
```

#### 种子差异

denv12可能使用了不同的加密种子，适配新SM版本的指令集变化。

---

### 4. cic12 - CICC中间代码提取工具

**功能**: 从CUDA 12.x的nvdisasm二进制中提取CICC中间表示代码。

**源代码**: `cic12.cc`
**构建命令**: `gcc -g -o cic12 -I../../../../ELFIO cic12.cc -lstdc++`
**依赖**: ELFIO库

#### CICC的作用

CICC（CUDA Intermediate Code)是CUDA编译工具链中的中间表示层：

```
源代码 → NVVM → CICC → SASS → 机器码
                ↑
           cic12提取这一层
```

#### 提取内容

```
cicc12/
├── c4CA7760      # 编译器符号表
├── c4CA7380      # 编译配置
├── llvm          # LLVM中间代码 (LZ4压缩)
├── ptx_out       # PTX输出形式 (LZ4压缩)
├── regs          # 寄存器分配信息 (LZ4压缩)
└── ...
```

#### 使用示例

```bash
# 前提条件：需要CUDA 12.x的nvdisasm和patch
ln -sf /usr/local/cuda-12.x/bin/nvdisasm ./nvdisasm

# 运行提取
./cic12 ./nvdisasm

# 查看输出
file cicc12/llvm   # 应该显示压缩格式
wc -c cicc12/*     # 查看各部分大小
```

#### 加密参数差异

cic12使用了与denv12不同的加密种子，这是因为CICC和SASS数据采用不同的加密密钥：

```c
// denv12使用的种子（SASS用）
const _DWORD seeds[64] = { ... }

// cic12使用的种子（CICC用）
const _DWORD seeds[64] = { ... }  // 完全不同的数组
```

---

### 5. deptx - PTX宏代码提取工具

**功能**: 从ptxas编译器工具链中提取加密的PTX宏定义和辅助代码。

**源代码**: `ptxas.cc` 编译而来
**构建命令**: `gcc -g -o deptx -I../../../../ELFIO ptxas.cc -lstdc++`
**依赖**: ELFIO库

#### 提取内容

PTX（Parallel Thread eXecution）是CUDA的低级汇编语言。deptx提取的内容包括：

```
macros/
├── 1      # Fermi时代的PTX宏
├── 2      #
├── 3      #
├── 4      #
├── 5      #
├── 6      # Maxwell时代
├── 7      #
└── ...    # 其他时代的宏定义
```

#### 加密数据定位

```c
struct one_md {
    size_t off, size;
    const char *name;
};

// 示例：Fermi时代的PTX宏
{ 0x987340, 0x9877E8 - 0x987340, "1" }  // 偏移，大小，名称
```

#### 使用示例

```bash
# 前提：需要有ptxas二进制（通常在CUDA工具库中）
# ptxas.cc内部硬编码了多个CUDA版本的ptxas路径

./deptx /path/to/ptxas

# 输出到macros/
ls -la macros/
# 输出: 1, 2, 3, ... (各时代的PTX宏)
```

#### ptxas与nvdisasm的关系

```
CUDA工具链：
┌─────────────────────────────────────────┐
│ CUDA Toolkit                            │
├─────────────────────┬───────────────────┤
│ nvdbg (编译器)      │ ptxas (汇编器)    │
├─────────────────────┼───────────────────┤
│ → nvdisasm (denv)   │ → ptxas (deptx)   │
│   包含SASS表        │   包含PTX宏       │
└─────────────────────┴───────────────────┘
```

---

### 6. nvb - LLVM Bitcode提取工具

**功能**: 从NVIDIA CUDA运行时库(`libnvrtc-builtins.so`)中动态提取LLVM Bitcode代码。

**源代码**: `nvb.cc`
**构建命令**: `gcc -o nvb nvb.cc -ldl`
**依赖**: 动态链接库 (dlopen/dlsym)

#### 工作原理

nvb使用动态链接和符号查找来提取库中的嵌入代码：

```c
// nvb使用的两个关键符号：
Hack getArchBuiltins(size_t *sz, int arch);  // 按架构获取BC
Hdr getBuiltinHeader(size_t *sz);            // 获取header

// 对每个架构进行暴力破解（1-128）
for (int arch = 1; arch <= 0x80; arch++) {
    size_t sz = 0;
    auto res = getArchBuiltins(&sz, arch);
    if (res) dump(arch, res, sz);  // 保存为 <arch>.bc
}
```

#### 使用示例

```bash
# 必须提供libnvrtc-builtins.so的路径
./nvb /path/to/libnvrtc-builtins.so.11.5

# 输出文件
ls -la
# 生成: 1.bc, 2.bc, 3.bc, ... (各架构的LLVM Bitcode)
# 以及: BuiltinHeader.h (编译器header文件)

# 使用llvm工具查看Bitcode
llvm-dis 1.bc  # → 1.ll (LLVM汇编)
```

#### 架构ID映射

nvb进行暴力扫描时会尝试架构ID 1-128。常见架构的ID：

```
1-5 (Fermi)
6-13 (Kepler, Maxwell)
14-30 (Maxwell续, Pascal-Volta)
31-50 (Volta, Turing)
51-70 (Turing续, Ampere)
71-100 (Ampere续, Ada)
101+ (Hopper, Blackwell)
```

#### 输出文件使用

生成的BC文件可用于：

1. **编译分析**
   ```bash
   llvm-dis *.bc      # 转换为可读的LLVM IR
   grep "define" *.ll # 搜索函数定义
   ```

2. **反汇编和模式识别**
   ```bash
   llvm-objdump -d 1.bc
   ```

3. **生成优化建议**
   ```bash
   opt -analyze -stats 1.bc
   ```

#### 错误处理

```bash
# 如果无法打开库
./nvb /path/to/lib
# 错误: cant open /path/to/lib: cannot open shared object file

# 如果找不到符号
./nvb ./libnvrtc-builtins.so
# 错误: cant hack: undefined symbol
```

---

## 完整工作流示例

### 场景：分析最新CUDA 12.x的GPU代码

```bash
#!/bin/bash
set -e

CUDA_VERSION="12.5"
CUDA_PATH="/usr/local/cuda-${CUDA_VERSION}"

# 步骤1：链接官方工具
ln -sf ${CUDA_PATH}/bin/nvdisasm ./nvdisasm

# 步骤2：提取SASS表 (支持新架构)
echo "[*] 提取SASS表..."
./denv12 ./nvdisasm
echo "✓ SASS表已存放到 data12/"

# 步骤3：提取CICC中间代码
echo "[*] 提取CICC代码..."
./cic12 ./nvdisasm
echo "✓ CICC数据已存放到 cicc12/"

# 步骤4：提取运行时库BC
echo "[*] 提取LLVM BC代码..."
./nvb ${CUDA_PATH}/lib64/libnvrtc-builtins.so.12.5
echo "✓ BC文件已保存"

# 步骤5：解析BC文件
echo "[*] 生成LLVM IR..."
for bc_file in *.bc; do
    llvm-dis "$bc_file"
done
echo "✓ LLVM IR文件已生成"

echo ""
echo "分析完成！"
echo "SASS数据: ls data12/sm90*/  # Hopper架构"
echo "CICC数据: ls cicc12/"
echo "LLVM IR: ls *.ll"
```

### 场景：比较不同CUDA版本的差异

```bash
#!/bin/bash

# 提取CUDA 10.1的数据
ln -sf /cuda_archive/10.1/nvdisasm nvdisasm_101
./denv ./nvdisasm_101

# 提取CUDA 11.5的数据
ln -sf /cuda_archive/11.5/nvdisasm nvdisasm_115
./denv11 ./nvdisasm_115

# 提取CUDA 12.5的数据
ln -sf /cuda_archive/12.5/nvdisasm nvdisasm_125
./denv12 ./nvdisasm_125

# 对比分析
echo "CUDA 10.1 支持的架构:"
ls data/ | sort

echo "CUDA 11.5 支持的架构:"
ls data11/ | sort

echo "CUDA 12.5 支持的架构:"
ls data12/ | sort

# 查看增量
echo "CUDA 11相对10的新增架构:"
diff <(ls data/) <(ls data11/) | grep ">"

echo "CUDA 12相对11的新增架构:"
diff <(ls data11/) <(ls data12/) | grep ">"
```

---

## 故障排除

### 常见错误与解决方案

#### 错误1：cannot find section .data

```
错误信息: cannot find section .data in ./nvdisasm
原因: 提供的二进制文件不是预期的CUDA工具
解决: 确保使用官方NVIDIA CUDA工具包中的nvdisasm
```

#### 错误2：cannot load ./nvdisasm

```
错误信息: cannot load ./nvdisasm
原因: nvdisasm符号链接损坏或不存在
解决:
  rm ./nvdisasm
  ln -sf /usr/local/cuda/bin/nvdisasm ./nvdisasm
```

#### 错误3：deptx segmentation fault

```
错误信息: Segmentation fault (core dumped)
原因: deptx没有参数或参数不正确
解决:
  # deptx需要ptxas路径但需要修改源代码中的硬编码路径
  # 或: 直接编辑ptxas.cc中的路径
```

#### 错误4：cannot create data/ (Permission denied)

```
原因: 当前目录没有写权限
解决:
  chmod u+w .
  或: 在有写权限的目录中运行
```

---

## 性能考虑

### 提取时间估计

| 工具 | 输入大小 | 提取时间 | 输出大小 |
|------|---------|---------|---------|
| denv | 50 MB | 2-3s | 200-300 MB |
| denv11 | 80 MB | 3-5s | 300-400 MB |
| denv12 | 100 MB | 5-8s | 400-500 MB |
| cic12 | 100 MB | 3-5s | 200-300 MB |
| nvb | N/A | 30-60s | 100-200 MB |

### 磁盘空间需求

```
总空间需求 = 所有data*/目录之和
≈ 1-2 GB（全CUDA版本完整提取）

每个架构数据：
sm30: ~5 MB
sm75: ~15 MB
sm90: ~20 MB
```

---

## 高级用法

### 自定义提取范围

修改源代码中的`one_md`数组可以选择性提取：

```c
// 示例：只提取sm75和sm90的数据
const one_md mds[] = {
    { 0x4CA7760, 0x860, "sm75", 0 },
    { 0x4CA7380, 0x3D8, "sm90", 0 },
    { 0, 0, nullptr, 0 }  // 终止符
};
```

### 批量处理多个CUDA版本

```bash
#!/bin/bash

CUDA_VERSIONS=(10.1 11.0 11.5 12.0 12.5)

for version in "${CUDA_VERSIONS[@]}"; do
    cuda_path="/usr/local/cuda-${version}"
    if [[ ! -d "$cuda_path" ]]; then continue; fi

    mkdir -p "extract_${version}"
    cd "extract_${version}"
    ln -sf "${cuda_path}/bin/nvdisasm" ./nvdisasm

    if [[ $version == 10* ]]; then
        ../denv ./nvdisasm
    elif [[ $version == 11* ]]; then
        ../denv11 ./nvdisasm
    else
        ../denv12 ./nvdisasm
    fi

    cd ..
done
```

### 与其他工具集成

denvdis工具的输出可用于：

1. **性能分析**：SASS表 → 指令调度分析
2. **安全审计**：LLVM BC → 中间代码审计
3. **反向工程**：CICC → 编译器优化分析
4. **定制编译**：PTX宏 → 自定义优化编译器

---

## 参考资源

### 源代码参考
- [denv.cc](../denv.cc) - SASS表提取基础版本
- [denv11.cc](../denv11.cc) - LZ4支持版本
- [denv12.cc](../denv12.cc) - 新架构支持版本
- [cic12.cc](../cic12.cc) - CICC提取工具
- [ptxas.cc](../ptxas.cc) - PTX宏提取（deptx源）
- [nvb.cc](../nvb.cc) - Bitcode提取工具

### CUDA官方文档
- [CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/)
- [PTX ISA Reference](https://docs.nvidia.com/cuda/parallel-thread-execution/)
- [nvdisasm Manual](https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html)

### 相关项目
- [ELFIO](https://github.com/serge1/ELFIO) - ELF解析库
- [LZ4](https://github.com/lz4/lz4) - 压缩库
- [LLVM](https://llvm.org/) - 编译基础设施

---

## 更新历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.2 | 2026-04-03 | 添加denv12和完整架构支持 |
| 1.1 | 2026-01-15 | 添加LZ4压缩支持（denv11） |
| 1.0 | 2025-09-01 | 初始版本（denv basic） |

---

## 许可证

denvdis项目的工具集采用自定义许可。详见[LICENSE](../LICENSE)文件。

**最后更新**: 2026-04-03
**维护者**: denvdis项目小组
