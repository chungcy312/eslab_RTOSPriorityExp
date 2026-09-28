# FreeRTOS Priority Inversion Experiment

This project compares a FreeRTOS Mutex with a Binary Semaphore in a priority-inversion scenario.

## Project location

```text
/Users/ccy/STM32CubeIDE/workspace_2.2.0/FreeRTOS_Priority_Inversion/
```

The main application code is in:

```text
Core/Src/main.c
```

No button, LED, or Wi-Fi hardware is required for this experiment.

## Task design

The project contains four threads:

| Thread | Priority | Role |
|---|---:|---|
| `defaultTask` | Normal | Default FreeRTOS task |
| `LowPriorityTask` | Low | Owns the shared resource and performs CPU work for 1 second |
| `MediumPriorityTask` | Normal | Performs CPU work for 2 seconds without using the shared resource |
| `HighPriorityTask` | High | Waits for the shared resource and measures the waiting time |

The task timing is arranged as follows:

- Low task delay: 10 ms
- High task delay: 50 ms
- Medium task delay: 100 ms
- Low task work duration: 1,000 ms
- Medium task work duration: 2,000 ms

## Selecting the experiment

Open `Core/Src/main.c` and find this definition near the top of the file:

```c
#define RUN_SEMAPHORE_EXPERIMENT 0
```

### Run the Mutex experiment

Set the value to `0`:

```c
#define RUN_SEMAPHORE_EXPERIMENT 0
```

This starts the original three priority-inversion tasks using `resourceMutexHandle`.

### Run the Binary Semaphore experiment

Set the value to `1`:

```c
#define RUN_SEMAPHORE_EXPERIMENT 1
```

This starts the separate Semaphore tasks using `resourceSemaphoreHandle`:

- `StartSemaphoreLowTask`
- `StartSemaphoreMediumTask`
- `StartSemaphoreHighTask`

The Semaphore is created with an initial count of `1`.

After changing the value, build the project again and download/debug the new firmware to the board.

Do not run both experiment sets at the same time. The switch ensures that the measurements are not affected by the other set of tasks.

## Expressions to observe

Start a Debug session and add the following variables to the Expressions view. Pause the target before reading their values.

### Mutex experiment variables

```text
lowWorkCounter
mediumWorkCounter
highWaitTime
highAcquiredCount
lowHasMutex
```

Important result variable:

```text
highWaitTime
```

### Binary Semaphore experiment variables

```text
semaphoreLowWorkCounter
semaphoreMediumWorkCounter
semaphoreHighWaitTime
semaphoreHighAcquiredCount
semaphoreLowHasSemaphore
```

Important result variable:

```text
semaphoreHighWaitTime
```

## Meaning of the variables

| Variable | Meaning |
|---|---|
| `lowWorkCounter` / `semaphoreLowWorkCounter` | Number of CPU-work loop iterations completed by the Low task |
| `mediumWorkCounter` / `semaphoreMediumWorkCounter` | Number of CPU-work loop iterations completed by the Medium task |
| `highWaitTime` | High task waiting time when using the Mutex, in milliseconds |
| `semaphoreHighWaitTime` | High task waiting time when using the Binary Semaphore, in milliseconds |
| `highAcquiredCount` / `semaphoreHighAcquiredCount` | Number of successful resource acquisitions by the High task |
| `lowHasMutex` | Indicates whether the Mutex Low task currently owns the Mutex |
| `semaphoreLowHasSemaphore` | Indicates whether the Semaphore Low task currently owns the Binary Semaphore |

## Expected results

The expected values are approximately:

| Synchronization mechanism | High-task wait time |
|---|---:|
| Mutex | About 960 ms |
| Binary Semaphore | About 2,050 ms |

The Mutex should produce a shorter waiting time because FreeRTOS uses priority inheritance. The Binary Semaphore does not use priority inheritance, so the Medium task can delay the Low task and cause the High task to wait longer.

## Suggested comparison graph

For the report, create a bar chart with:

- X-axis: Synchronization mechanism
- Y-axis: High-task waiting time in milliseconds
- Bar 1: Mutex = 960 ms
- Bar 2: Binary Semaphore = 2,050 ms

The two experiments are run separately, so compare the recorded values from the two Debug sessions rather than expecting both result variables to be non-zero in the same run.
