# denvdis 工具实践指南 - 使用basic_gemm.cubin

## 文件分析

### basic_gemm.cubin 文件信息

```bash
$ file basic_gemm.cubin
basic_gemm.cubin: ELF 64-bit LSB executable, NVIDIA CUDA architecture

$ ls -lh basic_gemm.cubin
-rw-r--r-- 1 user user 8.2K basic_gemm.cubin

$ readelf -S basic_gemm.cubin
Section Headers:
  [Nr] Name              Type      Address           Offset  Size
  ...
  [ N] .text._Z...      PROGBITS  0x...            0x...    0x200  (SASS代码)
  [ N] .nv.info         LOPROC+0  ...              ...      ...    (元数据)
```

### CUDA架构识别

检查basic_gemm.cubin的目标架构：

```bash
$ strings basic_gemm.cubin | grep -i "target\|sm_"
# 输出: target sm_XX (其中XX是架构version)

# 或使用nvdisasm查看
$ ./nvdisasm basic_gemm.cubin 2>&1 | head -5
.target sm_XX
```

---

## 工具可用性与限制

由于basic_gemm.cubin的特殊性质（可能来自特定编译或格式），以下是各工具的适用情况：

| 工具 | 是否支持 | 原因 | 替代方案 |
|------|---------|------|---------|
| `denv/denv11/denv12` | ✓ 有条件 | 需要nvdisasm二进制，不是CUBIN文件 | 使用官方nvdisasm二进制 |
| `cic12` | ✓ 部分 | 同上，CICC数据可能需要特殊header | 使用官方库文件 |
| `deptx` | ✗ 否 | 需要ptxas工具链，不是CUBIN | 使用官方ptxas |
| `nvb` | ✓ 完全 | 不依赖特定CUBIN | 直接使用libnv库 |

### 为什么denv工具处理CUBIN会失败

denv/denv11/denv12工具的实际用途是**从CUDA编译器工具二进制中提取数据**，而不是处理CUBIN文件：

```
denv的真实工作流:
┌──────────────────────┐
│ /usr/bin/nvdisasm    │  ← CUDA官方工具的二进制文件
│ (包含加密的SASS表)   │
└──────────────┬───────┘
               │ denv处理
               ↓
        ┌──────────────┐
        │  data/目录   │ ← 解密后的SASS表
        │ (sm75/, ...) │
        └──────────────┘
```

而不是：

```
❌ 错误的理解：
basic_gemm.cubin → denv → 提取SASS
```

---

## 正确的测试流程

### 前置条件设置

```bash
# 1. 配置CUDA工具链访问
ln -sf /usr/bin/nvdisasm ./nvdisasm

# 2. 验证官方工具链
nvdisasm --version  # 应输出NVIDIA nvdisasm版本

# 3. 检查CUDA路径
which nvcc nvdisasm ptxas
echo $CUDA_HOME
```

### 测试方案A：从CUDA工具包中提取（推荐）

如果系统安装了CUDA，可以直接使用官方工具：

```bash
#!/bin/bash

# 对于CUDA 11.5及以上，使用denv11
./denv11 /usr/bin/nvdisasm 2>&1 | head -20

# 检查输出
ls -la data11/
```

### 测试方案B：使用任意CUBIN文件的间接分析

虽然denv不能直接处理CUBIN，但可以使用nvdisasm反汇编，然后分析输出：

```bash
#!/bin/bash

# 步骤1：反汇编CUBIN文件
nvdisasm basic_gemm.cubin > gemm_disasm.txt 2>&1

# 步骤2：解析反汇编输出
grep -A 100 "\.text\." gemm_disasm.txt | head -50

# 步骤3：使用项目工具分析
# 示例：使用test/nvd从反汇编输出提取信息
./test/nvd -c basic_gemm.cubin 2>&1 | head -30

# 步骤4：使用test/pa进行进一步分析
./test/pa basic_gemm.cubin  # 如果编译了pa工具
```

### 测试方案C：使用test/目录中的本地工具

test/目录包含了更灵活的分析工具：

```bash
#!/bin/bash

# 编译测试工具（如果还未编译）
cd test
make nvd

# 分析CUBIN
cd ..
./test/nvd -c basic_gemm.cubin

# 其他选项
./test/nvd -h basic_gemm.cubin      # hex dump
./test/nvd -e basic_gemm.cubin      # dump attributes
./test/nvd -O basic_gemm.cubin      # dump operands
./test/nvd -S basic_gemm.cubin      # dump sched info
./test/nvd -T basic_gemm.cubin      # track registers
```

---

## 实践演练脚本

### 脚本1：完整分析流程

```bash
#!/bin/bash
# analyze_cubin.sh

set -e

CUBIN="${1:-.basic_gemm.cubin}"
OUTDIR="analysis_$(date +%Y%m%d_%H%M%S)"

echo "[*] 创建分析目录..."
mkdir -p "$OUTDIR"

echo "[*] 收集基础信息..."
{
    echo "=== 文件信息 ==="
    file "$CUBIN"
    ls -lh "$CUBIN"

    echo ""
    echo "=== ELF节信息 ==="
    readelf -S "$CUBIN" | head -20

    echo ""
    echo "=== 符号表 ==="
    readelf -s "$CUBIN" | head -20
} > "$OUTDIR/basic_info.txt"

echo "[*] 进行反汇编..."
nvdisasm "$CUBIN" > "$OUTDIR/nvdisasm_output.txt" 2>&1 || true

echo "[*] 使用项目工具分析..."
{
    echo "=== nvd 兼容格式输出 ==="
    ./test/nvd -c "$CUBIN" 2>&1 || true
} > "$OUTDIR/nvd_analysis.txt"

echo "[*] 统计分析..."
{
    echo "SASS 指令总数: $(grep -c '^[[:space:]]*.*[0-9A-F]\\{4\\}' "$OUTDIR/nvdisasm_output.txt" || echo 'N/A')"
    echo "内核函数数: $(grep -c "^_Z" "$OUTDIR/nvdisasm_output.txt" || echo 'N/A')"
    echo "节总数: $(readelf -S "$CUBIN" | grep -c '^\s*\[[0-9]' || echo 'N/A')"
} > "$OUTDIR/statistics.txt"

echo "[*] 生成报告..."
cat > "$OUTDIR/README.txt" << 'EOF'
分析报告
========

包含文件：
- basic_info.txt: 文件基础信息和ELF结构
- nvdisasm_output.txt: nvdisasm反汇编完整输出
- nvd_analysis.txt: 项目工具分析结果
- statistics.txt: 统计数据

使用方法：
1. 查看basic_info.txt了解文件结构
2. 在nvdisasm_output.txt中搜索关键函数
3. 对比nvd_analysis.txt的输出格式
4. 使用statistics.txt中的数据进行对标

关键搜索示例：
- 函数定义: grep "^_Z" nvdisasm_output.txt
- SASS指令: grep "^[[:space:]]*\/\*" nvdisasm_output.txt
- 内存操作: grep -E "LDG|STG|LDS|STS" nvdisasm_output.txt
- 控制流: grep -E "BRA|JMP|EXIT" nvdisasm_output.txt
EOF

echo "[✓] 分析完成！"
echo "输出目录: $OUTDIR"
echo "执行: ls -la $OUTDIR"
```

**使用方法**：

```bash
#!/bin/bash
chmod +x analyze_cubin.sh
./analyze_cubin.sh basic_gemm.cubin
ls -la analysis_*/
cat analysis_*/statistics.txt
```

### 脚本2：SASS指令统计

```bash
#!/bin/bash
# analyze_sass.sh

CUBIN="${1:-.basic_gemm.cubin}"

echo "=== SASS 指令分布 ==="
nvdisasm "$CUBIN" 2>&1 | \
    grep -oE '^\s+[A-Z][A-Z0-9.]*' | \
    sed 's/^\s*//' | \
    sort | uniq -c | sort -rn | \
    awk '{printf "%5d  %-20s\n", $1, $2}' | \
    head -30

echo ""
echo "=== 內存访问统计 ==="
echo -n "LDG (Global Load): "
nvdisasm "$CUBIN" 2>&1 | grep -c "LDG" || echo 0

echo -n "STG (Global Store): "
nvdisasm "$CUBIN" 2>&1 | grep -c "STG" || echo 0

echo -n "LDS (Local Load): "
nvdisasm "$CUBIN" 2>&1 | grep -c "LDS" || echo 0

echo -n "STS (Local Store): "
nvdisasm "$CUBIN" 2>&1 | grep -c "STS" || echo 0

echo ""
echo "=== 控制流统计 ==="
echo -n "BRA (Branch): "
nvdisasm "$CUBIN" 2>&1 | grep -c "BRA" || echo 0

echo -n "EXIT (Thread Exit): "
nvdisasm "$CUBIN" 2>&1 | grep -c "EXIT" || echo 0

echo -n "JMP (Jump): "
nvdisasm "$CUBIN" 2>&1 | grep -c "JMP" || echo 0
```

**使用方法**：

```bash
chmod +x analyze_sass.sh
./analyze_sass.sh basic_gemm.cubin
```

预期输出示例：

```
=== SASS 指令分布 ===
   42  MOV
   35  IMAD
   28  FADD
   15  LDG
   12  STG
    8  IADD
    5  EXIT
    3  BRA

=== 內存访问统计 ===
LDG (Global Load): 15
STG (Global Store): 12
LDS (Local Load): 0
STS (Local Store): 0

=== 控制流统计 ===
BRA (Branch): 3
EXIT (Thread Exit): 1
JMP (Jump): 0
```

### 脚本3：与官方nvdisasm的格式对比

```bash
#!/bin/bash
# compare_disasm.sh

CUBIN="${1:-.basic_gemm.cubin}"
OUTDIR="comparison_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTDIR"

echo "[*] 获取官方nvdisasm输出..."
nvdisasm "$CUBIN" > "$OUTDIR/official.txt" 2>&1

echo "[*] 获取项目nvd工具输出..."
./test/nvd -c "$CUBIN" > "$OUTDIR/project_nvd.txt" 2>&1 || true

echo "[*] 提取.text节进行对比..."
{
    echo "=== Official nvdisasm .text section ==="
    grep -A 200 "^\.text\." "$OUTDIR/official.txt" | head -50

    echo ""
    echo "=== Project nvd .text section ==="
    grep -A 200 "^\.text\." "$OUTDIR/project_nvd.txt" | head -50
} > "$OUTDIR/comparison.txt"

echo "[*] 差异分析..."
diff -u \
    <(grep "^[[:space:]]*\/\*" "$OUTDIR/official.txt" | head -50) \
    <(grep "^[[:space:]]*\/\*" "$OUTDIR/project_nvd.txt" | head -50) \
    > "$OUTDIR/diff.txt" 2>&1 || echo "差异已保存"

echo "[✓] 对比完成"
echo "结果目录: $OUTDIR"
echo "要查看差异: diff $OUTDIR/official.txt $OUTDIR/project_nvd.txt"
```

---

## 关键输出解读

### 反汇编输出格式解析

```asm
/*0000*/                   MOV R1, c[0x0][0x28] ;
         ↑                 ↑  ↑  ↑    ↑   ↑     ↑
         偏移              指令 目标 源1 源2     终止符

/*00a0*/               @P0 EXIT ;
         ↑              ↑  ↑    ↑
         偏移 (16字节步长) 谓词 指令 (可选参数)
```

### 内核元数据识别

```
.nv.info 节包含：
├── EIATTR_REGCOUNT: 使用的寄存器数量
├── EIATTR_MAX_STACK_SIZE: 最大栈大小
├── EIATTR_MIN_STACK_SIZE: 最小栈大小
└── 其他编译器生成的属性

示例:
.byte   0x04, 0x2f           ; 寄存器计数属性
.word   index@(_Z6kernel...) ; 内核名称索引
.word   0x0000000c           ; 使用12个寄存器
```

### 性能指标提取

从反汇编输出可以推断：

```
总指令数 = (最后偏移 - 首个偏移) / 16 + 1

延迟估计 = ∑(指令延迟) + 内存访问延迟

示例计算：
- 14条SASS指令，~100个时钟周期总延迟
- 其中内存访问占大部分（LDG ~30-100 clocks）
- 纯计算部分 ~20-30 clocks
```

---

## 常见分析场景

### 场景1：确定内核是否使用共享内存

```bash
# 检查LDS/STS指令
nvdisasm basic_gemm.cubin 2>&1 | grep -E "LDS|STS"

# 如果有输出，说明使用了共享内存
# 否则，只使用全局内存和寄存器
```

### 场景2：评估寄存器压力

```bash
# 查看使用的寄存器个数
nvdisasm basic_gemm.cubin 2>&1 | grep -i "regcount\|registers"

# 手动计算：查看所有指令中最大的目标寄存器编号
nvdisasm basic_gemm.cubin | \
    sed -n '/\.text\./,/\.L_/p' | \
    grep -oE 'R[0-9]+' | \
    sed 's/R//' | \
    sort -n | \
    tail -1  # 最大寄存器号 + 1 = 使用数量
```

### 场景3：分析控制流复杂度

```bash
# 计算分支数量
echo "分支指令:"
nvdisasm basic_gemm.cubin 2>&1 | grep -E "BRA|JMP|CALL|RET"

# 计算谓词使用
echo "谓词使用:"
nvdisasm basic_gemm.cubin 2>&1 | grep "@P\|@!P" | wc -l

# 评估：
# - 无分支 = 线性执行，最高效
# - 少量分支 = 可能有条件执行，但有分歧代价
# - 多分支 = 复杂控制流，可能影响性能
```

### 场景4：内存访问模式识别

```bash
# 全局内存访问
echo "全局内存访问模式:"
nvdisasm basic_gemm.cubin 2>&1 | grep -E "LDG|STG|ATOM" | head -20

# 分析模式：
# LDG [R4]          = 简单加载
# LDG.E.SYS         = 使用L2缓存绕过
# LDG.NC            = 非缓存加载
# STG [R6], R9      = 非缓存存储
```

---

## 工具链集成

将分析工具集成到自动化流程：

### 与编译流程集成

```bash
#!/bin/bash
# build_and_analyze.sh

# 编译CUDA代码
nvcc -arch=sm_70 --cubin -o kernel.cubin kernel.cu

# 自动分析
./analyze_cubin.sh kernel.cubin

# 提取关键数据
REGS=$(readelf -p .nv.info kernel.cubin | grep -A1 REGCOUNT | tail -1)
echo "使用寄存器: $REGS"

# 生成报告
cat > analysis_report.txt << EOF
内核编译分析报告
================
CU文件: kernel.cubin
编译目标: sm_70 (Volta)
生成时间: $(date)

寄存器使用: $REGS
...
EOF
```

### 与持续集成集成

```yaml
# .github/workflows/cuda_analysis.yml
name: CUDA Kernel Analysis

on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install CUDA
        run: |
          sudo apt-get install cuda-toolkit-11-5
      - name: Compile
        run: nvcc -arch=sm_70 --cubin -o kernel.cubin kernel.cu
      - name: Analyze
        run: |
          ./analyze_cubin.sh kernel.cubin
          ./analyze_sass.sh kernel.cubin > sass_analysis.txt
      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: analysis-results
          path: analysis_*
```

---

## 下一步建议

1. **深入学习SASS ISA**
   - 阅读PTX到SASS的映射关系
   - 学习各指令的延迟和吞吐量

2. **性能调优**
   - 使用分析数据优化内核
   - 减少寄存器使用
   - 改进内存访问模式

3. **工具链扩展**
   - 编写自定义分析脚本
   - 集成到编译工具链
   - 自动生成性能报告

4. **深入研究加密机制**
   - 理解denv工具的XOR-Salt-LCG加密
   - 验证在不同CUDA版本上的一致性
   - 可能用于安全研究和逆向工程

---

**最后更新**: 2026-04-03
**相关文档**: [TOOLS_USAGE_GUIDE.md](TOOLS_USAGE_GUIDE.md)
