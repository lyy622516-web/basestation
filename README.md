# UHF Reader MultiFunction 基站样机硬件工程 (basestation)

本项目为 **UHF Reader MultiFunction（多功能超高频无源物联网读写器基站）** 的完整软硬件设计、PCB工程与技术文档资料库。

---

## 目录结构概览

```text
basestation/
├── 01_技术方案与设计报告/
│   ├── FTTx+RFID融合超高频无源物联网可重构端机技术方案与设计报告  龙彦宇 阳晟君 文光俊 20260930.docx
│   ├── 01_电源树架构与LDO载流分析报告.md
│   ├── 02_PCB各层走线状况分析报告.md
│   └── 设计思考/                                # 24 份相干功率合成与通信时序深度技术思考文档
├── 02_硬件架构与拓扑图/
│   ├── UHF_Reader_Power_Tree_Architecture.svg   # 硬件供电树架构与分配矢量拓扑图 (SVG)
│   ├── UHF_Reader_MultiFuntion_V5_Circuit_Architecture.svg # 系统硬件链路电路架构图 (SVG)
│   ├── 电源树_Visio可编辑版.vsdx                 # 供电树 Visio 可编辑原版工程
│   ├── 端机框架绘图.vsdx                        # 端机结构与整机框架 Visio
│   ├── PCB_All_Layers_Composite.png             # 全板 6 层走线高分辨率立体透视图 (300 DPI)
│   ├── L1_Top_Layer_Routing.png                 # L1 顶层射频微带线与表层器件走线图
│   ├── L2_GND_Plane_Routing.png                 # L2 完整接地参考与散热阵列图
│   ├── L3_Signal1_Power_Routing.png             # L3 内部电源主干母线分布图
│   ├── L4_Signal2_Digital_Routing.png           # L4 数字控制总线分布图
│   ├── L5_Signal3_Analog_Routing.png            # L5 敏感模拟基带 100Ω 差分走线图
│   └── L6_Bottom_Layer_Routing.png              # L6 底层接地去耦与 PA 散热开窗图
├── 03_BOM_物料清单/
│   ├── BOM_UHF_Reader_MultiFuntion_V5_LCSC.csv   # 嘉立创一键贴片物料清单 (CSV)
│   └── BOM_UHF_Reader_MultiFuntion_V5_LCSC.xlsx  # 格式化物料清单 (Excel)
├── 04_器件手册_Datasheet/                        # 23 份原厂芯片数据手册 (PDF)
│   ├── FM15F366 (复旦微 32位主控安全MCU)
│   ├── GD32E232 (兆易创新 Cortex-M23 从控MCU)
│   ├── FM13RD1616EL (超高频 RFID 专用 SOC)
│   ├── FW1103 (超低噪声前端放大器 LNA)
│   ├── HGC2005 (驱动放 DA & 功放 PA)
│   ├── SY8121BABC / SGM2036 系列低噪 LDO
│   └── ...
├── 05_嘉立创EDA工程/
│   ├── UHF_Reader_MultiFuntion_V5/              # 嘉立创专业版工程源码 (sch / pcb)
│   └── UHF_Reader_MultiFuntion_V5.eprj2         # 嘉立创专业版工程包
├── 06_Altium_Designer工程/
│   ├── UHF_Reader_MultiFunction_V4.PrjPcb       # Altium Designer 项目工程文件
│   ├── UHF_Reader_MultiFunction_V5.PcbDoc       # PCB 设计版图源文件 (6层高频板)
│   ├── *.SchDoc / *.Harness                     # 原理图图纸与线束定义
│   └── *.step / *.pdf                           # 3D 结构模型与生产预览
└── 07_物料映射与走线数据/
    ├── lcsc_map.json                            # 全板器件与立创商城物料编号映射表
    ├── pcb_pad_nets_truth.json                  # 基于 PCB 物理焊盘提取的网络基准真值表
    ├── pcb_routing_tracks_detail.csv            # 全板 2028 段走线起止坐标与线宽详表
    └── pcb_net_summary.csv                      # 全板各网络走线总长与分层统计汇总表
```

---

## 核心硬件技术指标

- **PCB 工艺**：标准 6 层高频高速板，板厚 1.23mm (FR-4)，外形尺寸 74.40mm × 63.40mm。
- **叠层规划**：
  - L1 (Top)：50Ω 射频微带线 (线宽 ~14.2 mil) 与主要元器件布局面。
  - L2 (GND)：完整实心参考地平面，PA 芯片底部 $4\times 4$ 散热孔阵列。
  - L3 (Signal 1)：`LDO_PWR` (3.9V) 与 `DC_5V` 内部加粗电源主干母线。
  - L4 (Signal 2)：数控衰减器并行总线 `ATTN1_D0~D6` 与数字控制总线。
  - L5 (Signal 3)：模拟基带高灵敏正交差分带状线 (`AMP_IP/IN`, `AMP_QP/QN`)。
  - L6 (Bottom)：底层去耦电容、底层接地屏蔽与 PA 散热开窗。
- **供电架构**：
  - 输入：DC 9V~12V (自恢复保险 1.3A + TVS 浪涌保护)。
  - 一级稳压：双路 SY8121BABC (2.0A Buck，效率 >92%) 产出 3.9V 与 5.0V。
  - 二级稳压：10 路独立 SGM2036 (300mA) 超低噪声 LDO，物理级隔离数字域、模拟基带、48MHz 晶振与大功率射频驱动。
- **射频链路**：
  - 工作频段：840 MHz ~ 960 MHz (支持 ISO18000-6C / EPC C1G2)。
  - 发射通道：读写器差分输出 $\to$ 平衡巴伦 $\to$ 驱动放大器 $\to$ 功率放大器 $\to$ 定向耦合器 $\to$ 多天线切换矩阵。
  - 接收通道：四天线选通 $\to$ 接收双级 FW1103 LNA $\to$ 带通滤波 $\to$ 正交下混频 $\to$ 模拟差分低通放大。
