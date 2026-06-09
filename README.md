# L476-FRTOS2-Gen2Waveforms
# STM32 CMSIS-RTOS2 Multi-Task Example

## Overview

This project demonstrates basic task scheduling using **CMSIS-RTOS2** on an STM32 microcontroller.

Two independent tasks are created:

* **Task 1 (myTask01)**

  * Priority: Normal
  * Updates `cnt_1`
  * Executes every 100 ms

* **Task 2 (myTask02)**

  * Priority: Low
  * Updates `cnt_2`
  * Executes every 250 ms

The purpose of this example is to observe how the RTOS scheduler switches between tasks and how task priorities affect execution.

---

## Global Variables

```c
int cnt_1 = 9;
int cnt_2 = 9;
```

These variables are incremented independently by different tasks.

---

## Task Configuration

### Task 1

```c
myTask01Handle = osThreadNew(StartTask01, NULL, &myTask01_attributes);
```

Attributes:

```c
.priority = osPriorityNormal
```

Function:

```c
void StartTask01(void *argument)
{
    for(;;)
    {
        if(cnt_1 >= 9)
            cnt_1 = 0;
        else
            cnt_1++;

        osDelay(100);
    }
}
```

Behavior:

* Counts from 0 to 9 repeatedly.
* Updates every 100 ms.

Sequence:

```text
0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 0 ...
```

---

### Task 2

```c
myTask02Handle = osThreadNew(StartTask02, NULL, &myTask02_attributes);
```

Attributes:

```c
.priority = osPriorityLow
```

Function:

```c
void StartTask02(void *argument)
{
    for(;;)
    {
        if(cnt_2 >= 9)
            cnt_2 = 0;
        else
            cnt_2++;

        osDelay(250);
    }
}
```

Behavior:

* Counts from 0 to 9 repeatedly.
* Updates every 250 ms.

Sequence:

```text
0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 0 ...
```

---

## Scheduler Initialization

The RTOS kernel is initialized before task creation:

```c
osKernelInitialize();
```

Tasks are created:

```c
myTask01Handle = osThreadNew(StartTask01, NULL, &myTask01_attributes);
myTask02Handle = osThreadNew(StartTask02, NULL, &myTask02_attributes);
```

The scheduler is started:

```c
osKernelStart();
```

After this point, task execution is controlled entirely by the RTOS scheduler.

---

## Expected Debug Observation

When watching variables in the debugger:

### cnt_1

Changes every 100 ms:

```text
0
1
2
3
4
5
6
7
8
9
0
...
```

### cnt_2

Changes every 250 ms:

```text
0
1
2
3
4
5
6
7
8
9
0
...
```

Since Task 1 runs more frequently, `cnt_1` changes faster than `cnt_2`.

---

## Priority Analysis

| Task   | Priority | Delay  |
| ------ | -------- | ------ |
| Task 1 | Normal   | 100 ms |
| Task 2 | Low      | 250 ms |

Task 1 has a higher priority than Task 2.

However, both tasks call:

```c
osDelay(...)
```

which places the task into the Blocked state, allowing other ready tasks to run.

As a result:

* Both tasks execute correctly.
* No task starvation occurs.
* Scheduler switches tasks according to priority and delay timing.

---

## Debugging Tips

### Live Expressions

Add:

```text
cnt_1
cnt_2
```

to Live Expressions.

Then press:

```text
Resume (F8)
```

to observe real-time value changes.

### Breakpoints

Place breakpoints inside:

```c
StartTask01()
```

and

```c
StartTask02()
```

to observe task switching behavior.

### Common Issue

If variable values do not change:

1. Ensure `osKernelStart()` is reached.
2. Run the program instead of staying paused.
3. Verify FreeRTOS/CMSIS-RTOS2 middleware is enabled.
4. Enable Live Expressions or SWV debugging.
5. Compile in Debug configuration.

---

## Learning Objectives

This example demonstrates:

* CMSIS-RTOS2 thread creation
* Task scheduling
* Task priorities
* Periodic execution using `osDelay()`
* Real-time debugging of RTOS variables
* Context switching between tasks

---

## Conclusion

The project successfully creates two RTOS tasks running at different priorities and execution periods. The variables `cnt_1` and `cnt_2` provide a simple way to visualize scheduler behavior and verify that CMSIS-RTOS2 is operating correctly on the STM32 platform.
