# 计算机组成原理的学习笔记(9)-- CPU·其一 CPU的基本概念流水线技术数据通路

------

### 1. **组成**

- **定义**：计算机的核心部件，负责执行指令和处理数据。

- 组成部分：

  - **核心**：多个处理单元，提升多任务处理能力。
  - **控制单元**（CU）：解码指令并协调各组件运作。
  - **算术逻辑单元**（ALU）：执行算术和逻辑运算。
  - **寄存器**：临时存储器，用于存放数据和指令。
  - **缓存**（Cache）：提高数据访问速度，减少对主存的访问。包括L1、L2、L3缓存。
  - **总线接口单元**：负责与外部设备或内存交换数据。

------

### 2. **时钟周期**

- **定义**：计算机内部时钟信号完成一次周期所需的时间。

- **频率**：时钟频率以赫兹（Hz）为单位，表示每秒电路振荡的次数。

- **时钟周期与指令周期的关系**：时钟周期决定了CPU每个操作的速度，而指令周期是完成一条指令所需的时间。不同的指令可能需要不同数量的时钟周期来完成。

- 影响性能的因素：

  - 高频率通常意味着短的时钟周期，但也可能导致功耗和热量增加，影响稳定性。
- 时钟频率并不是唯一决定CPU性能的因素，还需要考虑指令集架构、缓存等因素。

------

### 3. **机器周期**

- **定义**：CPU完成一次基本操作所需的时间。

- **与时钟周期的关系**：一个机器周期通常由多个时钟周期组成，具体依赖于CPU的架构和指令复杂度。

- 机器周期的组成：

  - 一个机器周期通常包括**取指周期**、**间址周期**（地址计算）和**执行周期**。不同类型的操作可能需要不同的机器周期。

------

### 4. **指令周期**

- **定义**：执行一条指令所需的总时间，常由多个机器周期组成。

- 主要阶段：

  - **取指**（Fetch）：从内存中读取指令。
  - **解码**（Decode）：将指令解析为控制信号，准备执行。
  - **执行**（Execute）：根据指令执行相应的算术或逻辑运算，或进行数据传输。
  - **访存**（Memory Access）：如果指令涉及内存操作（如读取或写入数据），则进行访存。
  - **写回**（Write-back）：将运算结果写回寄存器或内存。

------

### 5. **流水线技术**

- **定义**：将指令的执行过程分解为多个阶段，允许同时处理多条指令。

- **优势**：提高指令吞吐量，优化资源利用，减少空闲时间。

- **流水线分段**：流水线通常包括多个阶段，如取指、解码、执行、访存、写回等。

- **流水线的深度**：流水线的阶段越多，每个阶段的任务越简单，但也可能带来更多的冒险问题。

- 冒险问题：

  - **数据冒险**：指令之间的数据依赖造成的冲突。可以通过**数据转发**或**流水线停顿**来解决。
  - **控制冒险**：由分支指令引起的指令流不确定性。通过**分支预测**和**延迟槽**来优化。
  - **结构冒险**：由于资源不足引起的冲突。通过增加硬件资源（如多个执行单元）来缓解。

------

### 6. **超标量技术**

- **定义**：允许在一个时钟周期内同时发射多条指令的处理器架构。

- 特点：

  - 多个执行单元并行处理指令，提高处理效率。
  - **动态调度**：处理器通过动态调度（如乱序执行）来提高指令吞吐量，尽可能利用空闲的执行单元。
  - **指令发射**：多个指令同时发射到不同的执行单元，减少了CPU的空闲时间。
  - **复杂的调度机制**：通过硬件或者编译器技术，处理器可以动态安排指令执行，优化指令间的空闲时间。

------

### 7. **超流水线技术**

- **定义**：将执行过程中的每个阶段进一步细分，以增加流水线的阶段数量。
- **优化目标**：通过细化每个阶段的操作，使得每个阶段执行的任务更简单，从而提高流水线的吞吐量。
- **挑战**：虽然细化阶段可以提高吞吐量，但也可能增加指令之间的延迟，尤其是在增加了很多**分支预测**和**指令重排**的情况下。

------

### 8. **超长指令技术**

- **定义**：将多个操作合并为一个长指令字，允许并行执行多个操作。

- **VLIW（超长指令字）**：通过将多个操作（如加法、乘法、数据传输等）打包成一个超长指令，在硬件上进行并行执行，指令调度通常由编译器完成，而非硬件。

- 特点：

  - 编译器在编译时负责指令调度。
  - 每条指令包含多个操作，可以在多个执行单元中并行执行。
  - 增加了编译器的复杂性，但可以有效提高处理器性能。

-----

### 9. **数据通路**

- **定义**：用于数据传输与处理的硬件部分，是执行指令的基础。
- 组成：
  - 寄存器、算术逻辑单元（ALU）、多路复用器（MUX）、数据总线。
- **工作流程**：在各个阶段中，负责从寄存器读取，执行操作，和存储结果。

-----

#### **专用数据通路**

##### 1. 取指周期（Instruction Fetch Cycle）

取指周期的任务是从内存中获取指令并加载到指令寄存器（IR）。在这一周期内，处理器执行以下微操作：

- **PC → MAR**：程序计数器（PC）的内容送到内存地址寄存器（MAR），以便确定当前要取指令的内存地址。
- **(MAR) → M**：打开MAR送到地址总线的输出门。
- **1 → R**：CU通过控制总线向主存发出读命令。
- **M(MAR) → MDR**：主存通过数据总线根据MAR中的地址所对应的存储单元中的内容(指令)送入MDR。
- **MDR → IR**：打开控制门，将内存数据寄存器（MDR）中的数据（即指令）送到指令寄存器（IR）中。
- **OP(IR) → CU**：打开指令操作码送往CU的输出门。
- **PC + 1 → PC**：程序计数器（PC）自增，准备下一条指令的地址。



##### 2. 间址周期（Address Calculation Cycle）

间址周期通常用于指令中的寻址操作。例如，对于`ADD R1, M`（将内存地址M中的值加到寄存器R1中）指令，间址周期的任务是计算指令中的操作数地址。在这一周期中，微操作如下：

- **Ad(MAR) → MAR**：打开MDR和MAR的控制门，将形式地址送到MAR。
- **(MAR) → AB → M**：打开MAR送到地址主线的输出门。
- **1 → R**：CU 通过控制总线向主存发出读命令。
- **M(MAR) → MDR**：主存通过数据总线，将MAR中地址所对应的存储单元的内容(有效地址)送到MDR。



##### 3. 执行周期（Execution Cycle）

执行周期是实际进行运算的过程。对于加法指令，它的任务是执行加法操作，并将结果存储回目的寄存器或内存。微操作如下：

- **MDR → MAR**：将有效地址送到MAR。
- **(MAR) → AB → M**：打开MAR送往地址总线的输出门。
- **1 → R**：CU通过控制总线向主存发出读命令。
- **M(MAR) → DB → MDR**：将操作数存入MDR。
- **C6,C7同时有效**：打开ACC和MDR链接ALU的控制门。
- **ACC + MDR → ACC**：打开ALU通往ACC的控制门，存入计算结果。



执行加法指令的微操作过程可总结为：

- **取指周期**：从内存中取出指令，并更新PC。
- **间址周期**：计算操作数的地址，并从内存中读取操作数。
- **执行周期**：执行加法操作，并将结果存储到目标寄存器。

-----
#### **内部总线结构**
##### 1. **取指周期（Instruction Fetch Cycle）**

在取指周期中，主要任务是从内存中获取当前要执行的指令，并更新程序计数器（PC）。通过内部总线将各个模块连接在一起以进行数据传输。

- **PC → MAR**：程序计数器（PC）的内容送到内存地址寄存器（MAR），确定当前要获取的指令的内存地址。
- **1 → R**：CU发出 "**R**"(读指令),
- **M(MAR) → MDR**：主存通过数据总线根据MAR中的地址所对应的存储单元中的内容(指令)送入MDR。
- **MDR → IR**：内存数据寄存器（MDR）中的数据（即当前的指令）通过总线传送到指令寄存器（IR）中。
- **PC + 1 → PC**：程序计数器（PC）自增，为下一条指令准备地址。

- 数据通过总线从PC传送到MAR，再从内存（通过MDR）传送到IR，最后更新PC指向下一条指令。总线在整个过程中充当着数据传输的媒介。

------

##### 2. **间址周期（Address Calculation Cycle）**

间址周期的任务是从指令中提取操作数的地址，并从内存中读取操作数。指令可能包含一个操作数地址，需要通过间址周期进行地址计算和数据获取。对于`ADD R1, M`，这里的`M`是内存地址。

- **Ad(MAR) → MAR**：将指令的形式地址经内部总线送至MAR。
- **1 → R**：C通过控制总线向主存发出读命令
- **M(MAR) → MDR**：主存通过数据总线，将MAR中地址所对应的存储单元的内容(有效地址)送到MDR。
- **MDR → A**：将内存数据寄存器（MDR）中的数据（即 `M` 的值）传送到寄存器A（临时寄存器，用于存放操作数）。



------

##### 3. **执行周期（Execution Cycle）**

执行周期是指令执行的核心部分，主要任务是通过ALU进行加法运算，并将结果存储到目标寄存器中。对于 `ADD R1, M`，目标是将内存 `M` 的值加到寄存器 `R1` 的值上。

- **(MDR)→ MAR**：将有效地址经内部总线送至 MAR。
- **1→R**：CU通过控制总线向主存发出读命令(R)。
- **M(MAR)→ MDR**：主存通过数据总线，将 MAR 中地址所对应存储单元的内容(数据)送至 MDR。
- **(MDR)→Y**：将操作数送至奇存器Y，直接作为 ALU 的一个输入。
- **ACC + Y → Z**：CU 向 ALU发出“ADD"加法指令的控制信号，完成 ACC和 MDR 内容的相加得到的结果直接送往寄存器 Z。
- **Z → ACC**：结果写入ACC。


---

#### 比较：

| **特性**         | **内部总线结构（Bus-Based Architecture）** | **专用数据通路（Dedicated Data Path Architecture）** |
| ---------------- | ------------------------------------------ | ---------------------------------------------------- |
| **带宽和延迟**   | 带宽较低，延迟较高，因为总线是共享资源     | 带宽较高，延迟较低，避免了总线瓶颈                   |
| **硬件资源消耗** | 硬件资源节约，减少了布线和硬件复杂性       | 需要大量的硬件资源，为每个操作设计专用通路           |
| **灵活性**       | 灵活性高，适合扩展和变更                   | 灵活性差，设计固定，扩展困难                         |
| **系统扩展性**   | 扩展性好，容易增加新组件                   | 扩展困难，增加新组件需要重新设计硬件                 |
| **并行性**       | 并行性较差，多个组件访问总线时会造成冲突   | 高并行性，可以同时处理多个数据流                     |
| **复杂性和维护** | 设计较简单，维护较容易                     | 设计和维护较复杂，需要更多的工程资源                 |
