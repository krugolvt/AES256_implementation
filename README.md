# AES-256 Physical-Aware Synthesis and Timing Optimization

This project explores the synthesis, timing analysis, and optimization of an AES-256 hardware implementation using **Cadence Genus**. The focus of the project was not on developing the AES algorithm itself, but on taking an AES-256 RTL design through an industry-style frontend ASIC flow, identifying timing bottlenecks, modifying the RTL architecture to improve timing, and studying the effect of migrating the same design between different semiconductor technology nodes.

The AES-256 design implements the standard 128-bit AES datapath with a 256-bit encryption key and 14 encryption rounds. The design contains the standard AES transformations including SubBytes, ShiftRows, MixColumns, AddRoundKey, and AES-256 key expansion. Functional correctness was verified using **NIST AES-256 Known Answer Test vectors** before and after synthesis.

## 28nm Synthesis and Timing Analysis

The design was first synthesized using **Cadence Genus Physical-Aware Synthesis** targeting a **TSMC 28nm HPM** technology at a target clock frequency of **500 MHz**, corresponding to a 2 ns clock period.

The initial synthesis run revealed a severe timing problem. The critical path was traced to the AES-256 key-expansion logic. In the original implementation, the key schedule attempted to generate all fifteen AES round keys within a single clock cycle. This resulted in a combinational path approximately **128 logic levels deep**. At the 500 MHz timing constraint, the resulting design had a Worst Negative Slack of approximately **-15.23 ns**, indicating that the problem could not realistically be solved only through synthesis-level optimizations such as cell sizing, buffering, or higher optimization effort.

## RTL Timing Optimization

The timing issue was therefore treated as an architectural problem rather than purely a synthesis problem. The key-expansion RTL was restructured so that the round keys were generated sequentially instead of calculating the complete AES-256 key schedule within one clock cycle.

The modified implementation generates the key schedule across multiple clock cycles using sequential logic. This significantly reduced the amount of combinational logic between registers. After restructuring the RTL and repeating synthesis using the same 28nm technology and timing constraints, the maximum combinational depth was reduced from approximately **128 logic levels to 18 logic levels**. The reported Worst Negative Slack improved from approximately **-15.23 ns to -48 ps**, while Total Negative Slack was reduced to zero.

This experiment demonstrated that large timing violations caused by deep combinational logic often require changes to the RTL microarchitecture rather than increasingly aggressive synthesis optimization.

## Migration from 28nm to 3nm

After optimizing the design at 28nm, the same RTL architecture was migrated to **Samsung 3nm SF3** technology. The RTL itself was kept unchanged during the migration. The design was re-targeted by changing the technology libraries used by the synthesis flow and re-running elaboration, technology mapping, optimization, and physical-aware synthesis.

The first 3nm synthesis run intentionally retained the original **500 MHz** timing constraint so that the effect of technology scaling could be observed independently of RTL changes. At this frequency, the design exhibited a large amount of positive timing slack, with approximately **+1.192 ns WNS**, showing that the design could operate considerably faster at the newer process node.

The timing constraint was therefore tightened and the design was re-synthesized at **1.2 GHz**, corresponding to a clock period of approximately **0.833 ns**. At this operating point, the design achieved a reported **+36 ps WNS with zero failing timing endpoints**. Increasing the clock frequency from 500 MHz to 1.2 GHz increased the theoretical AES throughput from approximately **64 Gbps to 153.6 Gbps**.

The migration demonstrates how technology scaling affects the achievable performance of the same RTL architecture and how timing constraints must be reconsidered when a design is moved between process nodes rather than simply reusing constraints developed for an older technology.

## Verification

Functional verification was performed using **NIST AES-256 Known Answer Tests (KATs)**. The AES implementation was tested using known plaintext, key, and ciphertext combinations from the AES standard. The report records successful testing of the RTL implementation as well as the synthesized implementations, ensuring that the timing-oriented RTL modifications did not change the expected encryption functionality.

## Project Takeaway

The main objective of this project was to study the relationship between **RTL architecture, synthesis, timing constraints, and semiconductor technology scaling**. The project demonstrates a complete timing-debug workflow: synthesizing an existing RTL design, identifying the critical path using timing reports, determining that the violation was architectural, modifying the RTL to reduce combinational depth, re-synthesizing the optimized design, and finally migrating the same architecture to a more advanced technology node.

The project provided hands-on exposure to **Cadence Genus, physical-aware synthesis, static timing analysis, SDC/MMMC concepts, critical-path analysis, RTL pipelining, timing optimization, and cross-node design migration**.

## Results and Screenshots

The repository includes selected screenshots from the synthesis and timing-analysis flow showing the initial 28nm timing failure, the timing result after RTL optimization, the 3nm timing analysis, and views of the synthesized AES design in Cadence Genus.

> **Note:** Proprietary foundry libraries, technology files, PDK data, and internal Cadence design collateral are not included in this repository.