================================================================================
Ender Terminal 完整静态反编译分析包 · 索引
================================================================================
分析对象: en.apk（Ender Terminal v2.1，包名 com.ender.terminal）
          + libTerminal.so v5.0.42（注入游戏的 native payload，29,705,024 B）
生成日期: 2026-09-13
生成方式: jadx 1.5.1 反编译 + Ghidra 12.1.3 深度分析 + 自研 CFF 解码管线
================================================================================

【文件夹结构】

01_APK反编译/
  en_decompiled.zip              完整反编译包（37.3 MB，jadx 输出）
  en_decompiled_sources/         解压后的全部源码（18,243 个文件）
    sources/                     18,058 个 Java 文件（含混淆，21 类未完全还原）
    resources/                   资源 + AndroidManifest.xml

02_libTerminal_so分析/
  libTerminal.so                 native payload 原文件（与远程 so_md5 一致）
  报告_算法.txt                  完整算法逆向报告（启动链/模块/数学内核/渲染）
  报告_调用图.txt                全库调用图报告（3,367 函数/189,601 case 块）
  报告_JNI_OnLoad.txt            JNI_OnLoad 破译（分发器/掩码/case 块）
  报告_Ghidra反编译.txt          Ghidra 12.1.3 深度反编译输出（CFF 证据）
  架构图_组件.svg                APK+so 整体组件架构图
  解码数据/
    decoded_blocks.txt           全库 189,601 个 case 块地址清单
    callgraph.txt                函数→调用 关系图（机器可读）

03_分析脚本/
  scan_dispatch.py               扫描 CFF 分发器
  analyze_tables.py              派发表定位
  find_rela_slots.py             RELATIVE 重定位槽解析
  decode_jni_table.py            JNI_OnLoad 表解码
  extract_dispatchers.py         提取全库 3,367 个 (表,掩码) 分发器
  decode_all.py                  全表解码 → 189,601 case 块
  build_callgraph.py             构建函数调用图
  decode_func.py                 单函数解码（表+掩码+case 块摘要）
  analyze_ctors.py               65 个构造器还原
  math_profile.py                浮点指令构成分析
  net_calls.py                   网络调用点定位
  disasm_range.py                地址区间反汇编
  （及配套 log 数据文件）

================================================================================
【关键结论速览】
================================================================================
1. 应用性质: Xposed + ptrace 双通道游戏外挂注入框架，目标《代号：超自然》
   （com.pi.czrxdfirst），功能含自瞄/绘制/内存功能/天空盒等。

2. 注入链路: hook com.je.game.JeMainActivity.onCreate →
   LocalSocket/TCP 27183/广播 START_HANDOFF/ContentProvider/文件兜底
   5 种途径把 libTerminal.so 送进游戏进程。

3. libTerminal.so 混淆形态: OLLVM 控制流平坦化，3,367 个混淆函数、
   189,601 个 case 块、逐函数编码掩码；已全部解码。

4. 核心算法:
   - F045 数学内核（6,168 块）: fmul+fmadd 平方和 + fsqrt = 3D 欧氏距离计算
   - F1172 渲染模块（878 块）: ANativeWindow_fromSurface×18 轮询绑定 Surface
   - 构造器链: C61 dlopen+dlsym×4 / C63/C64 环境指纹（HWCAP+系统属性）

5. 卡密系统: 服务端授权模型——本地无卡密校验；APK 仅解绑页(unbind_url)，
   so 内有加密 TCP 授权会话（F1806/F1807/0x173712c），协议内容需动态抓取。

================================================================================
【可复现说明】
================================================================================
- 所有脚本为 Python 3（依赖 elftools/capstone），输入文件为 libTerminal.so
- Ghidra 项目: C:\Users\XWT-WuYu\Downloads\ghidra\proj\LTProj3
  复跑脚本: C:\Users\XWT-WuYu\Downloads\ghidra_scripts\DecompileFuncs.java
- 静态分析到此为止的剩余项（需 Frida 动态）:
  * F045 距离公式的具体输入对象（游戏内存布局）
  * C61 dlopen 的库名与 4 个符号名
  * GL 间接调用完整清单（2,683 个 blr 的全局槽解析）
  * so 授权会话的域名与请求/响应协议
================================================================================
