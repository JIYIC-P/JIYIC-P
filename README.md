## Hi there 👋
这是我的学习笔记
this is my study note

# 模拟电路
## Part 1  基础认知
### 

```mermaid
flowchart TB
    classDef outer stroke:indigo,fill:#eef2ff
    classDef header stroke:indigo,fill:#e0e7ff
    classDef cell stroke:teal,fill:#f0fdfa

    subgraph Table["电学元件与概念总结表"]
    direction TB

        subgraph HeaderRow[" "]
            direction LR
            H1["元件/概念"]:::header
            H2["关键结论"]:::header
            H3["原因"]:::header
        end

        subgraph Row2[" "]
            direction LR
            C1["电容"]:::cell
            C2["电压不能突变"]:::cell
            C3["突变需要无穷大电流"]:::cell
        end

        subgraph Row3[" "]
            direction LR
            D1["电感"]:::cell
            D2["电流不能突变"]:::cell
            D3["突变需要无穷大电压"]:::cell
        end

        subgraph Row4[" "]
            direction LR
            E1["地 GND"]:::cell
            E2["是参考点"]:::cell
            E3["电压本质是两点间的差值"]:::cell
        end

        subgraph Row5[" "]
            direction LR
            F1["示波器"]:::cell
            F2["显示电压随时间变化"]:::cell
            F3["能看到波形、失真、噪声、振荡"]:::cell
        end
    end
```
最核心的两句话是：电容怕“电压突变”，电感怕“电流突变”。

## part 2 数学基础
### 

```mermaid
flowchart TB
    subgraph 外部层[" "]
        A[上位机 / 第三方系统]
    end

    A <-->|串口下行指令| B1[串口接收]
    B2[串口发送] -->|串口上行反馈| A

    subgraph 通信层["通信层"]
        direction TB
        B1[串口接收<br/>帧头校验  长度校验  XOR校验<br/>指令解析  路由分发]
        B2[串口发送<br/>状态采集  帧组装  校验计算<br/>周期发送  姿态50ms  跟踪10ms  心跳1000ms]
        B3[心跳管理<br/>超时检测  连续3次超时告警<br/>链路断开保护]
        B4[错误处理<br/>帧错误计数  日志记录  异常丢弃]
    end

    B1 --> B3
    B1 --> B4
    B3 --> B2

    subgraph 逻辑层["逻辑层"]
        direction LR
        C[塔台控制模块<br/>俯仰轴转动  航向轴转动<br/>最大角速度限制  到位判定<br/>到位状态反馈]
        D[光电载荷模块<br/>IR/TV双光切换  变倍/聚焦控制<br/>黑热/白热切换  画中画开关<br/>2KM景深模拟  雾效渲染]
        E[目标管理模块<br/>行人/车辆/无人机生成<br/>运动模型驱动  路径规划<br/>激光干扰暂停5s  干扰冷却<br/>坐标实时更新]
        F[识别跟踪模块<br/>视锥体检测  遮挡判定<br/>自动锁定跟踪  目标选择<br/>方位/俯仰计算  丢失判定500ms]
        
        C <-->|自动跟踪角度输出| F
        D <-->|目标坐标读取| E
        C <--> D
        D <--> E
        E <--> F
    end

    B1 -->|指令分发| C
    B1 -->|指令分发| D
    B1 -->|指令分发| E
    B1 -->|指令分发| F

    C -->|状态上报| B2
    D -->|状态上报| B2
    E -->|状态上报| B2
    F -->|状态上报| B2

    G[场景管理模块<br/>沙漠/城镇/机场场景切换<br/>地面/空中视角切换<br/>预设航线加载JSON<br/>环境参数配置]
    H[激光功能模块<br/>激光测距  目标距离计算<br/>激光干扰  目标暂停5s加5s冷却<br/>干扰状态反馈]
    I[配置管理模块<br/>SimConfig.json  网络/显示/塔台/光学/激光/跟踪参数<br/>Routes.json  航线数据<br/>启动加载  热重载]

    C --> G
    D --> H
    E --> H
    I -.->|配置注入| C
    I -.->|配置注入| D
    I -.->|配置注入| E
    I -.->|配置注入| F
    I -.->|配置注入| G
    I -.->|配置注入| H

    subgraph 表现层["表现层 Unity渲染输出"]
        direction TB
        J[主显示窗口<br/>主画面渲染  TV/IR通道<br/>十字准星叠加  跟踪框叠加<br/>画面信息叠加  ZOOM/FOV/RANGE/AZ/EL<br/>画中画  右下角1/8 副通道]
        K[底部状态栏<br/>俯仰/航向/变倍  跟踪状态  激光状态<br/>串口状态  心跳延迟  场景/视角]
        L[调试信息层 F1切换<br/>FPS  串口帧统计RX/ERR  目标世界坐标<br/>摄像机FOV/Clip  航线航点索引]
    end

    H -->|测距结果  干扰状态| J
    G -->|场景渲染| J
    F -->|跟踪框位置| J
    C -->|塔台姿态| J
    D -->|通道画面| J

    J --> K
    J -.->|F1切换| L

    subgraph 数据层["数据层"]
        direction LR
        M[场景资源<br/>沙漠地形  城镇建筑  机场跑道<br/>行人模型  车辆模型  无人机模型<br/>材质/纹理/Shader]
        N[配置文件<br/>Config/SimConfig.json<br/>Config/Routes.json<br/>运行时自适应]
        O[日志文件<br/>Logs/运行日志  按日期分文件<br/>Logs/错误日志<br/>保留30天]
    end

    G -->|场景加载| M
    I -->|配置读取| N
    J -->|日志写入| O
    B4 -->|错误日志| O

    style A fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style B1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1.5px
    style B2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1.5px
    style B3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1.5px
    style B4 fill:#e8f5e9,stroke:#2e7d32,stroke-width:1.5px
    style C fill:#e3f2fd,stroke:#1565c0,stroke-width:1.5px
    style D fill:#e3f2fd,stroke:#1565c0,stroke-width:1.5px
    style E fill:#e3f2fd,stroke:#1565c0,stroke-width:1.5px
    style F fill:#e3f2fd,stroke:#1565c0,stroke-width:1.5px
    style G fill:#f3e5f5,stroke:#6a1b9a,stroke-width:1.5px
    style H fill:#fce4ec,stroke:#c62828,stroke-width:1.5px
    style I fill:#fff8e1,stroke:#f57f17,stroke-width:1.5px
    style J fill:#e0f7fa,stroke:#00838f,stroke-width:1.5px
    style K fill:#eceff1,stroke:#546e7a,stroke-width:1.5px
    style L fill:#fafafa,stroke:#9e9e9e,stroke-width:1.5px
    style M fill:#e8eaf6,stroke:#3949ab,stroke-width:1.5px
    style N fill:#e8eaf6,stroke:#3949ab,stroke-width:1.5px
    style O fill:#e8eaf6,stroke:#3949ab,stroke-width:1.5px
    ```
