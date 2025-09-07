# COP5615 Project 1 (Gleam Implementation)

## Team Members

- **VISHNU SAI PADYALA**  
  - UFID:32712860

## GitController Repository

[GitController Link]([https://github.com/your-repo-link](https://github.com/VISHNU07202003/COP5615-Project-1-Gleam-Implementation-))

# Project Description

This project demonstrates distributed task allocation using the **actor model in Gleam**. The program divides a given set of tasks among multiple worker processes. Each worker computes whether the sum of squares in its assigned range is a perfect square. The hub actor manages task distribution, collects results, and coordinates worker termination.

## Overview

The program uses three types of actors: `Controller`, `Task`, and `Main`.

- The **Controller** is responsible for managing workers, distributing work, and aggregating results.
- The **Tasks** compute whether the sum of squares of numbers in their assigned range is a perfect square.
- The **Main** function initializes the hub, creates the workers, and starts computation based on command-line arguments.

## Actors and Functionality

### Controller Actor
- **Purpose**: Central controller that distributes tasks and collects results.
- **Fields**:  
  - `next`: Next task index to assign.  
  - `last`: Upper bound of tasks.  
  - `assigned`: Number of tasks given so far.  
  - `completed`: Number of completed tasks.  
  - `hits`: List of successful starting positions that produced perfect square sums.  
  - `workers`: List of worker subjects.  

- **Messages**:  
  - `Begin(hub, upto, width, pool)`: Start computation with given parameters.  
  - `Noted(start)`: Report a successful result.  
  - `Ready(who)`: Task signals readiness for more work.  

- **Behavior**:  
  - Spawns workers and assigns them ranges.  
  - Collects results and prints them when all work is done.  
  - Shuts down workers gracefully.  

### Task Actor
- **Purpose**: Performs computation for a given range.  
- **Messages**:  
  - `Work(start, myself)`: Process range starting at `start`.  
  - `Halt`: Stop execution.  

- **Behavior**:  
  - Computes sum of squares for assigned range.  
  - Checks if result is a perfect square.  
  - Reports successful results back to hub.  
  - Requests more work when finished.  

### Helper Functions
- **`integer_sqrt(n)`**: Integer square root using iterative approximation.  
- **`is_square_renamed(n)`**: Checks if a number is a perfect square.  
- **`sumsq_range_renamed(a, b)`**: Computes sum of squares between `a` and `b`.  

### Main Function
- Reads command-line arguments:  
  - `n`: Upper bound of tasks.  
  - `k`: Width (size of each work unit).  
  - `pool`: Number of worker processes.  

- Creates hub actor and starts computation.  
- Waits for hub to finish or times out after 60 seconds.  

## Performance
- Parallelism is achieved through multiple workers.  
- Work distribution ensures efficient CPU utilization.  
- Results are aggregated at the hub.  

## Example Usage

Compile and run the program:

```bash
gleam run lukas.gleam 10000000 24 8
```

Here:  
- `10000000` → Total range of tasks  
- `24` → Work unit size  
- `8` → Number of workers  

## Example Output

Running with parameters (e.g. `gleam run 10000000 24 8`) produces:

```bash
1
9
20
25
44
76
121
197
304
353
540
856
1301
2053
3112
3597
5448
8576
12981
20425
30908
35709
54032
84996
128601
202289
306060
353585
534964
841476
1273121
2002557
3029784
3500233
5295700
8329856
```

If no results are found:  

```bash
none
```

If computation exceeds timeout:  

```bash
timeout
```


## System Diagram

The following diagram shows the interaction between the **Controller** and **Tasks** using the actor model:

```mermaid
graph TD
    A[Main] -->|Begin| B[Controller]
    B -->|Assign Work| C1[Task 1]
    B -->|Assign Work| C2[Task 2]
    B -->|Assign Work| C3[Task N]

    C1 -->|Ready/Noted| B
    C2 -->|Ready/Noted| B
    C3 -->|Ready/Noted| B

    B -->|Results| A
```
Performance Analysis
Benchmark Results (n = 10,000,000, k = 24)

The program was benchmarked with different pool sizes using PowerShell's Measure-Command.
Here are the observed results:

Pool=1 | Elapsed (ms): 444.87
Pool=2 | Elapsed (ms): 463.37
Pool=4 | Elapsed (ms): 445.24
Pool=8 | Elapsed (ms): 422.40

Throughput (ranges per second)

Pool=1 → ~22.5 million/sec

Pool=2 → ~21.6 million/sec

Pool=4 → ~22.5 million/sec

Pool=8 → ~23.7 million/sec

Parallel Efficiency

Pool=1 → 100% (baseline)

Pool=2 → ~96% (slight overhead)

Pool=4 → ~100% (no gain)

Pool=8 → ~106% (small improvement)

Summary

The program sustains ~22–24 million range checks per second.

Best elapsed time: ~422 ms with 8 workers.

Scaling efficiency is low, because actor communication overhead dominates when n=10,000,000 and k=24.
