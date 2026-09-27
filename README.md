# Assignment 2: Algorithmic Analysis, Correctness and Performance Trade-offs

## 1. Overview
This project implements three fundamental data structures from scratch in Java:
- **Dynamic Array** (resizing array implementation)
- **Doubly Linked List**
- **Min-Heap** (array-based binary heap)

The goal is to evaluate their algorithmic performance, prove correctness using loop invariants, execute controlled empirical workloads ($n = 100; 1,000; 10,000; 100,000$), and compare theoretical complexities against measured performance.

---

## 2. Complexity Analysis

| Data Structure | Operation | Best Case | Average Case | Worst Case | Auxiliary Space |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Dynamic Array** | `get(i)` | $\Theta(1)$ | $\Theta(1)$ | $\Theta(1)$ | $O(1)$ |
| | `add(x)` | $\Theta(1)$ | $\Theta(1)$ (amortized) | $\Theta(n)$ (resize) | $O(1)$ |
| | `add(i, x)` | $\Theta(1)$ (at end) | $\Theta(n)$ | $\Theta(n)$ (at index 0) | $O(1)$ |
| | `remove(i)` | $\Theta(1)$ (at end) | $\Theta(n)$ | $\Theta(n)$ (at index 0) | $O(1)$ |
| | `contains(x)` | $\Theta(1)$ | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |
| **Linked List** | `get(i)` | $\Theta(1)$ (ends) | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |
| | `add(x)` | $\Theta(1)$ | $\Theta(1)$ | $\Theta(1)$ | $O(1)$ |
| | `add(i, x)` | $\Theta(1)$ (ends) | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |
| | `remove(i)` | $\Theta(1)$ (ends) | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |
| | `contains(x)` | $\Theta(1)$ | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |
| **Min-Heap** | `insert(x)` | $\Theta(1)$ | $O(\log n)$ | $O(\log n)$ | $O(1)$ |
| | `peekMin()` | $\Theta(1)$ | $\Theta(1)$ | $\Theta(1)$ | $O(1)$ |
| | `extractMin()`| $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(1)$ |

### Justification
- **Dynamic Array `get(i)` vs Linked List `get(i)`:** Dynamic Array calculates memory offset directly in $O(1)$ time, whereas Linked List must traverse node pointers sequentially in $O(n)$ time.
- **Insertion at Index 0:** Linked List updates head pointers in $O(1)$ time, while Dynamic Array requires shifting all $n$ elements using `System.arraycopy()`, taking $O(n)$ time.

---

## 3. Algorithmic Correctness & Loop Invariants

### Proof 1: Linear Search in `DynamicArray.contains(x)`
- **Loop Invariant:** At the start of each iteration $i$ (where $0 \le i \le n$), the target value $x$ is not present in the subarray `data[0 ... i-1]`.
- **Initialization:** Prior to the first iteration ($i = 0$), the subarray `data[0 ... -1]` is empty. Thus, the statement holds vacuously.
- **Maintenance:** Assume the invariant holds for index $i$. During iteration $i$, the algorithm compares `data[i]` with $x$. If `data[i] == x`, the method immediately returns `true` (correct). If `data[i] != x`, $x$ is not present in `data[i]`. Incrementing $i$ to $i+1$ maintains the invariant for `data[0 ... i]`.
- **Termination:** The loop terminates when:
  1. $x$ is found at index $i$, returning `true` (correct).
  2. $i = n$. By the invariant, $x$ is not present in `data[0 ... n-1]`. The loop terminates and returns `false` (correct).

### Proof 2: Heap Sift-Down Restoration in `MinHeap.siftDown(i)`
- **Loop Invariant:** Prior to each iteration of `siftDown`, every node in the heap satisfies the Min-Heap property ($parent \le children$), except possibly at position $i$ relative to its immediate children.
- **Initialization:** Replacing the root node with the last element (`heap[size-1]`) disrupts the heap property exclusively at index $i = 0$. All subtrees beneath index $0$ strictly retain the Min-Heap property.
- **Maintenance:** At step $i$, the algorithm identifies the minimum element among `heap[i]`, `heap[left]`, and `heap[right]`. If `heap[i]` is not the minimum, it is swapped with `heap[smallest]`. The Min-Heap property is restored at node $i$, shifting any potential violation down to `smallest`.
- **Termination:** The loop terminates when node $i$ has no children ($2i + 1 \ge size$) or when `heap[i]` is smaller than or equal to both children. In both cases, the Min-Heap property holds across the entire structure.

---

## 4. Experimental Setup
- **Input Sizes ($n$):** $100, 1,000, 10,000, 100,000$
- **Workloads:** 4 fixed workloads covering random access, search, insertion/removal, and priority operations.
- **Repetitions:** 5 runs per experiment; reported values represent the average execution time.
- **Timing Mechanism:** `System.nanoTime()`
- **Randomization:** Deterministic seeding via `Random(42)`.

---

## 5. Experimental Results & Evidence

### Unit Tests Validation
![Unit Tests Result](results/plots/tests_execution.png)

### Execution Benchmarks Console Output
![Benchmark Console Output](results/plots/benchmark_console.png)

Raw output execution logs are saved in `results/tables/benchmark_data.txt`.

---

## 6. Performance and Design Analysis (Answers to Questions)

1. **How does increasing $n$ affect each workload?**
   - Workload 1 & 3 (Random Access & Middle Operations on Linked List) exhibit quadratic growth ($O(m \cdot n)$).
   - Workload 2 (Search) scales linearly ($O(m \cdot n)$ total) for both structures.
   - Workload 4 (Min-Heap) scales quasi-linearly ($O(n \log n)$), maintaining high throughput even at $n = 100,000$.

2. **Which experimental results agree with theoretical complexity?**
   - $O(1)$ access for Dynamic Array, $O(n)$ search for both lists, and $O(\log n)$ heap operations perfectly align with theoretical models.

3. **Where do experimental results differ from theoretical predictions?**
   - For small $n$ ($100$), Linked List operations appear faster than predicted due to JIT compilation overhead and small memory footprint fitting entirely in CPU cache.

4. **Why can two algorithms with the same Big-O complexity have different running times?**
   - Big-O hides constant factors, memory allocation overhead, cache locality differences, and CPU instruction-level optimizations.

5. **How do constant factors and implementation details affect performance?**
   - Dynamic Array utilizes contiguous memory memory blocks (`System.arraycopy`), benefiting from CPU L1/L2 cache prefetching, whereas Linked List suffers from pointer chasing and memory fragmentation.

6. **Why is a Dynamic Array preferable for some workloads?**
   - It provides $O(1)$ random access, low memory overhead per element, and superior CPU cache locality.

7. **When can a Linked List be useful?**
   - When frequent insertions and removals occur strictly at the head or tail, or when avoiding array reallocation/copying is critical.

8. **Why is a Heap appropriate for priority-based processing?**
   - It efficiently tracks minimum/maximum elements with $O(1)$ peek time and $O(\log n)$ insertion/extraction times without keeping the entire dataset fully sorted.

9. **How does the workload influence the choice of data structure?**
   - Choice depends on operation frequency: random access favors Dynamic Array, continuous head/tail edits favor Linked List, and priority filtering demands a Heap.

---

## 7. Conclusion
The experimental results confirm theoretical complexity expectations. Dynamic Array excels in access-heavy and append-heavy scenarios due to continuous cache locality. Linked List provides fast localized edits but suffers from traversal penalties. Min-Heap proves highly scalable for priority queues.
