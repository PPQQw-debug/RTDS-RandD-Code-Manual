# RTDS Timing Code Usage Summary by Hardware

## Overview

This document provides a comprehensive guide to timing code implementation across different RTDS hardware platforms. Based on analysis of source files in `C:\RTDS_SW_FX\BIN`, there are **two clock sources and two calculation methods** that vary between NovaCor 1.0, NovaCor 2.0, and PB5 platforms.

---

## 1. Two Clock Counter Functions

### **Option 1: getClockCount()**
- **Implementation**: `mftb` (time base) instruction
- **File**: `CMODEL_SOURCE\BLTIN_GCC\getClockCount.c`
- **Platform**: PB5, GPC, NovaCor2.0 (in some components)
- **Precision**: Lower, but simpler and more compatible

### **Option 2: getClockCountv2()** (Recommended)
- **Implementation**: `mfspr 776` (PMC6 register)
- **File**: `CMODEL_SOURCE\BLTIN_GCC\getClockCountv2.c`
- **Platform**: NovaCor 1.0, NovaCor 2.0 (most new components), FPGA models
- **Precision**: Higher (3.49125 GHz base frequency)
- **Note**: Introduced in multi-threaded scenarios to avoid `mftb` stalls (`CMODEL_SOURCE\SHARED_CODE\p8_vscPolling.c`)

---

## 2. Universal 32-bit Counter Overflow Handling

All components use this pattern to handle 32-bit counter wraparound:

```c
unsigned int maxCount = 0xFFFFFFFF;
if (start_count > final_count)
    ticksSoFar = maxCount - start_count + final_count;
else
    ticksSoFar = final_count - start_count;
```

**References**:
- `CMODEL_SOURCE\BLTIN_GCC\calculateExecutionTime.c`
- `CMODEL_SOURCE\BLTIN_GCC\calculateExecutionTimev2.c`
- Multiple components in DOTA, Relay, Substep modules

---

## 3. Hardware-Specific Implementation Table

| **Hardware** | **Clock Function** | **busSpeed Value** | **Unit** | **Evidence Files** |
|---|---|---|---|---|
| **PB5** | `getClockCount()` | 42.5 | ns/tick | `t2semaphore.c` |
| **GPC** | `getClockCount()` | 34.5 or 1/0.029 | ns/tick | `_hierarchy_cb.c`, `t2semaphore.c` |
| **NovaCor 1.0** | `getClockCountv2()` | 1/3.49125 | ns/tick | `twr_3ph.c`, `GTFPGA_Aurora_IFP.c`, `ss_UCM_*.c` |
| **NovaCor 2.0** | `getClockCount()` or `getClockCountv2()` | 1/3.49125 or 1000/512*2 | ns/tick | `rtds_DOTA_IFC.c` (note: limited v2 support) |

---

## 4. Ready-to-Use Code Templates

### **Template A: Basic Timing Framework (FPGA/NovaCor)**

```c
//#define _TIMING_     // Uncomment to enable timing measurements

OUTPUTS:
    double time_bg_t0_ns = createOutput(
        strcat2("time_bg_t0_section", Name), Name, -10, 110, "ns", "TRUE"
    );
    double time_t1_t2_ns = createOutput(
        strcat2("time_t1_t2_section", Name), Name, -10, 110, "ns", "TRUE"
    );

CODE:
    // ========== BEGIN TIMING BLOCK 1 (T0 Phase) ==========
    #ifdef _TIMING_
        int start_count = 0; 
        int final_count = 0;
        double busSpeed = 1/3.49125;          // For FPGA/NovaCor
        unsigned int maxCount = 0xFFFFFFFF;
        
        start_count = getClockCountv2();      // Record start time
    #endif
    
    // YOUR BUSINESS LOGIC HERE
    ts_loc_0 = (iIN_TS_LA + iIN_TS_LB + iIN_TS_LC) * one_over_three;
    oOT_TS_LA_AL = iIN_TS_LA - ts_loc_0;
    oOT_TS_LA_BE = (iIN_TS_LB - iIN_TS_LC) * one_over_rt3;
    
    #ifdef _TIMING_
        final_count = getClockCountv2();      // Record end time
        
        // Handle 32-bit counter overflow
        int ticksSoFar;
        if (start_count > final_count)
            ticksSoFar = maxCount - start_count + final_count;
        else
            ticksSoFar = final_count - start_count;
        
        // Convert to nanoseconds
        double nanoseconds = (double)ticksSoFar * busSpeed;
        time_bg_t0_ns = nanoseconds;
    #endif
```

### **Template B: Dual Timing Blocks (Two Phases)**

```c
//#define _TIMING_

OUTPUTS:
    double time_bg_t0_ns = createOutput(
        strcat2("time_bg_t0_section", Name), Name, -10, 110, "ns", "TRUE"
    );
    double time_t1_t2_ns = createOutput(
        strcat2("time_t1_t2_section", Name), Name, -10, 110, "ns", "TRUE"
    );

CODE:
    // ========== TIMING PHASE 1 (T0) ==========
    #ifdef _TIMING_
        int start_count = 0; 
        int final_count = 0;
        int start_count_2 = 0;
        int final_count_2 = 0;
        double busSpeed = 1/3.49125;
        unsigned int maxCount = 0xFFFFFFFF;
        
        start_count = getClockCountv2();
    #endif
    
    // T0 Phase Business Logic
    // ... your code here ...
    LOC_DIFF();
    REM_DIFF();
    
    #ifdef _TIMING_
        final_count = getClockCountv2();
        int ticksSoFar;
        if (start_count > final_count)
            ticksSoFar = maxCount - start_count + final_count;
        else
            ticksSoFar = final_count - start_count;
        
        double nanoseconds = (double)ticksSoFar * busSpeed;
        time_bg_t0_ns = nanoseconds;
    #endif
    
    // ========== TIMING PHASE 2 (T1-T2) ==========
    #ifdef _TIMING_
        start_count_2 = getClockCountv2();
    #endif
    
    // T1-T2 Phase Business Logic
    // ... your code here ...
    fabs_iw_diff_0 = fabs_iw_diff[0];
    diff_0_m = iw_diff_m[0];
    
    #ifdef _TIMING_
        final_count_2 = getClockCountv2();
        if (start_count_2 > final_count_2)
            ticksSoFar = maxCount - start_count_2 + final_count_2;
        else
            ticksSoFar = final_count_2 - start_count_2;
        
        nanoseconds = (double)ticksSoFar * busSpeed;
        time_t1_t2_ns = nanoseconds;
    #endif
```

### **Template C: PB5/GPC Platform (getClockCount version)**

```c
//#define _TIMING_

OUTPUTS:
    double time_execution_ns = createOutput(
        strcat2("time_execution_section", Name), Name, -10, 110, "ns", "TRUE"
    );

CODE:
    #ifdef _TIMING_
        unsigned int start_count = getClockCount();    // For PB5/GPC
        double busSpeed = 42.5;                        // PB5: 42.5 ns/tick
        // double busSpeed = 34.5;                     // GPC: 34.5 ns/tick
        unsigned int maxCount = 0xFFFFFFFF;
    #endif
    
    // YOUR TIMING-CRITICAL CODE HERE
    // ...
    
    #ifdef _TIMING_
        unsigned int final_count = getClockCount();
        
        // Handle overflow
        unsigned int ticksSoFar;
        if (start_count > final_count)
            ticksSoFar = maxCount - start_count + final_count;
        else
            ticksSoFar = final_count - start_count;
        
        // Convert to nanoseconds
        double nanoseconds = (double)ticksSoFar * busSpeed;
        time_execution_ns = nanoseconds;
    #endif
```

### **Template D: Platform Detection (Portable Code)**

```c
//#define _TIMING_

OUTPUTS:
    double time_execution_ns = createOutput(
        strcat2("time_exec_section", Name), Name, -10, 110, "ns", "TRUE"
    );

STATIC:
    // Select timing parameters based on platform
    #if defined(NOVACOR) || defined(FPGA)
        #define TIMING_FUNCTION getClockCountv2
        #define BUS_SPEED (1/3.49125)
    #elif defined(PB5)
        #define TIMING_FUNCTION getClockCount
        #define BUS_SPEED 42.5
    #elif defined(GPC)
        #define TIMING_FUNCTION getClockCount
        #define BUS_SPEED 34.5
    #else
        #define TIMING_FUNCTION getClockCountv2
        #define BUS_SPEED (1/3.49125)  // Default to NovaCor
    #endif

CODE:
    #ifdef _TIMING_
        unsigned int start_count = TIMING_FUNCTION();
        double busSpeed = BUS_SPEED;
        unsigned int maxCount = 0xFFFFFFFF;
    #endif
    
    // TIMING-CRITICAL CODE
    // ...
    
    #ifdef _TIMING_
        unsigned int final_count = TIMING_FUNCTION();
        
        unsigned int ticksSoFar;
        if (start_count > final_count)
            ticksSoFar = maxCount - start_count + final_count;
        else
            ticksSoFar = final_count - start_count;
        
        double nanoseconds = (double)ticksSoFar * busSpeed;
        time_execution_ns = nanoseconds;
    #endif
```

---

## 5. Why Timing Code Differs Across Platforms

Three compounding factors create the variation:

1. **Different Clock Sources**: `mftb` vs `mfspr 776 (PMC6)`
2. **Different Unit Definitions**: Some `busSpeed` is in MHz, others in ns/tick
3. **Component Migration History**: Different modules retain legacy code during version upgrades

---

## 6. Best Practices (Avoid Mixing Strategies)

✅ **DO:**
1. Choose ONE clock function per platform and stick with it
2. Use the matching busSpeed constant for your chosen function
3. Always handle 32-bit overflow (use the standard pattern above)
4. Enable/disable timing with `#ifdef _TIMING_` for compilation control
5. Test timing output range (typically -10 to 110 ns)

❌ **DON'T:**
1. Mix `getClockCount()` and `getClockCountv2()` in the same component
2. Reuse busSpeed constants across platforms (e.g., don't use `1/3.49125` with `getClockCount()`)
3. Hardcode platform assumptions—use conditional compilation or runtime detection
4. Forget overflow handling—use the standard unsigned comparison pattern

---

## 7. Key File Locations

```
C:\RTDS_SW_FX\BIN\CMODEL_SOURCE\
├── BLTIN_GCC/
│   ├── getClockCount.c              ← v1 timing function
│   ├── getClockCountv2.c            ← v2 timing function (recommended)
│   ├── calculateExecutionTime.c
│   ├── calculateExecutionTimev2.c
│   ├── builtin_gcc.h                ← Function declarations
│   ├── timestepPolling.c
│   └── t2semaphore.c
├── RELAY/
│   ├── twr_iwc.c                    ← Complete timing example
│   └── twr_3ph.c
├── Substep/
│   ├── ss_UCM_HALFBRIDGE.c          ← Complete timing example
│   ├── ss_UCM_Boost.c
│   └── ...other models
└── ...other directories
```

---

## 8. Quick Reference: Choosing Your Template

| **Scenario** | **Use Template** | **busSpeed** | **Function** |
|---|---|---|---|
| New FPGA or NovaCor model | Template A | `1/3.49125` | `getClockCountv2()` |
| Two distinct timing phases | Template B | `1/3.49125` | `getClockCountv2()` |
| Legacy PB5 code | Template C | `42.5` | `getClockCount()` |
| Multi-platform support needed | Template D | Platform-defined | Platform-defined |

---

## Summary

- **NovaCor / FPGA**: Use `getClockCountv2()` with `busSpeed = 1/3.49125`
- **PB5**: Use `getClockCount()` with `busSpeed = 42.5`
- **GPC**: Use `getClockCount()` with `busSpeed = 34.5`
- **Always handle overflow** with the standard 32-bit check
- **Use `#ifdef _TIMING_`** for conditional compilation
