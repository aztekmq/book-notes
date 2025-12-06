## 1. What I’d Put in Place for You

For your setup, I’d do **three things**:

1. **`PROJECT_SPEC.md`** – human-readable, detailed project contract
2. **`project_manifest.yaml`** – machine-friendly summary of modules & invariants
3. **A small “AI task prompt” template** you reuse in ChatGPT & Copilot Chat

You don’t need a custom RAG service yet – this gives you 80% of the benefit with very little overhead.

---

## 2. `PROJECT_SPEC.md` – Drop-In Template

Put this at the root of your repo:

**`PROJECT_SPEC.md`**

```markdown
# Project Spec – java-review

> SINGLE SOURCE OF TRUTH for what this repo is, does, and must never break.

---

## 1. Overview

**Project name:** `java-review`  
**Purpose:** Hands-on JVM tuning & diagnostics lab with three difficulty levels (beginner, intermediate, advanced) plus a JVM health analyzer tool.

**Primary goals:**
- Provide runnable Java examples that illustrate:
  - GC behavior and tuning
  - Heap sizing and OOM scenarios
  - Thread states, contention, and thread dumps
  - Advanced GC (G1, ZGC), JFR, and container-aware JVM behavior
- Provide a JVM health analysis CLI that can summarize JFR + GC logs into a “JVM Health Report”.

**Non-goals (for now):**
- No production-grade web UI.
- No distributed / clustered orchestration.
- No multi-language support beyond Java and a bit of shell/Make/Maven.

---

## 2. Repository Structure (High-Level)

- `beginner/`
  - B1 – GC basics
  - B2 – Heap sizing & OOM + heap dumps
  - B3 – Thread states & thread dumps
- `intermediate/`
  - I1 – G1 GC tuning & GC logs
  - I2 – Memory leak lab (heap dump analysis)
  - I3 – Thread contention lab
- `advanced/`
  - A1 – Low-latency GC (ZGC vs G1)
  - A2 – JFR profiling
  - A3 – Container-aware JVM (Docker)
- `analyzer/`
  - Maven project with `JvmHealthAnalyzer`:
    - Input: JFR file (+ optional GC log)
    - Output: text-based JVM Health Report

---

## 3. Cross-Cutting Invariants (MUST ALWAYS HOLD)

These are the things that **AI tools must not break**:

1. **Labs are self-contained and runnable.**
   - Each lab folder (`B1_gc_basics`, `I2_memory_leak_lab`, etc.) must compile independently.
   - All Java examples must compile with `javac` on JDK 17+ without extra dependencies.

2. **Code and README must stay in sync.**
   - Every code sample mentioned in a README must exist and match filenames exactly.
   - If code changes substantially, the README for that lab must be updated in the same change.

3. **Analyzer module behavior contract:**
   - `JvmHealthAnalyzer`:
     - Takes at least one argument: `JFR` path.
     - Optional second argument: GC log path.
     - Always prints:
       - Total JFR events
       - GC pause stats (count, total, avg, max) if present
       - Allocation stats (if present)
       - CPU load summary (if present)
   - Future changes must not remove these basic outputs.

4. **No external services required to run labs.**
   - Labs must run offline.
   - Only dependencies: JDK, optional Maven, and Docker for advanced container labs.

5. **Simple build flow:**
   - Top-level `Makefile` must continue to support:
     - `make beginner`
     - `make intermediate`
     - `make advanced`
     - `make analyzer`

---

## 4. Module-Level Responsibilities

### 4.1 Beginner Labs

- **B1_gc_basics**
  - Show impact of heap size on GC frequency and pause times.
  - Demonstrate basic GC logging (`-Xlog:gc*`).
- **B2_heap_sizing**
  - Reproduce OutOfMemoryError with a small heap.
  - Generate heap dump via `-XX:+HeapDumpOnOutOfMemoryError`.
  - README explains how to open and inspect dump in Eclipse MAT.
- **B3_thread_states**
  - Demonstrate all major thread states: RUNNABLE, BLOCKED, WAITING, TIMED_WAITING.
  - README shows how to generate and read thread dumps with `jstack` / `jcmd`.

### 4.2 Intermediate Labs

- **I1_gc_tuning_g1**
  - Demonstrate G1 GC with GC logs.
  - Show how to interpret pauses and heap behavior using GC viewer.
- **I2_memory_leak_lab**
  - Simulate a memory leak via static map.
  - Force heap dump, analyze in MAT to identify leak source.
- **I3_thread_dump_lock_contention**
  - Simulate lock contention with synchronized method.
  - Show blocked threads in thread dumps.
  - Encourage refactor with `AtomicInteger` or other concurrency tools.

### 4.3 Advanced Labs

- **A1_low_latency_gc**
  - Compare ZGC vs G1 using the same workload.
  - Focus on pause-time differences, not micro-optimizing throughput.
- **A2_jfr_profiling**
  - Start JFR recording from JVM startup.
  - Generate CPU & allocation hotspots for analysis in Java Mission Control.
- **A3_container_aware_jvm**
  - Show how JVM reacts to container memory/CPU limits.
  - Use `-XshowSettings:vm` to inspect heap and VM ergonomics.

### 4.4 Analyzer

- **analyzer/JvmHealthAnalyzer**
  - Read JFR events and produce a textual summary:
    - GC pause stats
    - Allocation volume
    - CPU load stats
  - Optionally parse GC log for simple pause stats.
  - Output must be usable as a quick “JVM Health Report” in CLI environments.

---

## 5. How to Work With AI/LLMs on This Project

Whenever using ChatGPT / Codex / Copilot Chat:

1. **Always remind the AI of invariants (Section 3).**
2. **If changing any code in a lab folder, update the corresponding README.**
3. **Never remove core behaviors listed in Section 4 without explicitly stating why.**
4. For bigger refactors, ask:
   > “Before coding, restate the responsibilities from Section 4 that must be preserved.”

---

## 6. Open TODOs / Future Enhancements

- Add HTML or Markdown version of JVM Health Report.
- Add sample JFR + GC logs under `samples/` for testing the analyzer.
- Add GitHub Actions job that runs `java` analyzer against sample data as a regression check.
```

This gives you a **single, anchored contract** you and the AI can keep referring to.

---

## 3. `project_manifest.yaml` – A Small, AI-Friendly Summary

This is optional but very helpful when you start automating prompts later:

**`project_manifest.yaml`**

```yaml
name: java-review
summary: JVM tuning & diagnostics lab (beginner/intermediate/advanced) + analyzer.
modules:
  - id: beginner
    path: beginner/
    responsibilities:
      - teach GC basics
      - heap sizing and OOM
      - thread states and dumps
  - id: intermediate
    path: intermediate/
    responsibilities:
      - G1 GC tuning
      - memory leak diagnosis
      - lock contention analysis
  - id: advanced
    path: advanced/
    responsibilities:
      - low-latency GC (ZGC vs G1)
      - JFR profiling
      - container-aware JVM behavior
  - id: analyzer
    path: analyzer/
    responsibilities:
      - parse JFR + optional GC log
      - print JVM Health Report to stdout
invariants:
  - "Each lab must compile standalone with javac (JDK 17+)."
  - "README and code filenames must stay in sync."
  - "Analyzer must always output GC pause, allocation, and CPU summaries if present."
  - "No external services are required to run labs."
build:
  - "Top-level Makefile must support targets: beginner, intermediate, advanced, analyzer."
```

Later, if you ever build a tiny Python wrapper around the OpenAI API, this YAML is exactly what you’d feed in per request.

---

## 4. How to Use This in Your Actual Workflow (ChatGPT + VS Code + Copilot)

### A. In ChatGPT

When you open a new session about this repo:

1. Paste the **top parts** of `PROJECT_SPEC.md` (Overview + Invariants + Module section relevant to your task).
2. Then ask for the change.

Example:

> “Here is the project spec for my `java-review` repo (snipped). You must respect these invariants. I now want to:
>
> * Add a new advanced lab A4 for ‘Heap Fragmentation & Compaction’.
>   Please update the repo structure, advanced/README, and sketch the new Java file for A4 without breaking any existing labs.”

The model now has the **scope + invariants** right up front.

---

### B. In VS Code with Copilot/Copilot Chat

Concrete habits:

* **Keep `PROJECT_SPEC.md` open** in a tab.

* When using **Copilot Chat**, do:

  * “In file `advanced/README.md`, update the advanced labs list to include A4 as described in `PROJECT_SPEC.md` Section 4.3. Don’t remove any existing labs.”

* When editing a file:

  * Add spec-related comments at the top, e.g.:

    ```java
    // SPEC: This lab must demonstrate ZGC vs G1 pause times for the same workload.
    // SPEC: Code must run on JDK 17+ and not depend on external libraries.
    ```

  Copilot will then “see” those constraints whenever it edits the file.

---

### C. When You Ask for Bigger Changes

Use a little **task template** in prompts:

```text
TASK: <what you want>

CONTEXT:
- Project: java-review (see PROJECT_SPEC.md, invariants and module responsibilities).
- Affected modules: <list folders/files>
- Constraints:
  - Do not remove any labs.
  - Keep README and code consistent.
  - Maintain analyzer CLI contract.

OUTPUT:
- List invariants that could be affected.
- Explain your approach in 3–5 bullet points.
- Then show the updated files (fully or as unified diffs).
```

You don’t have to do this every time, but for bigger edits it keeps the AI much more “project-aware.”

---

## 5. Short Answer to Your Question

> **“I am using Codex with ChatGPT and VSC and against my github repo, so what would be your best recommended solution. PROJECT_SPEC.md ?”**

**Yes.**
My best recommended solution for your setup is:

1. **Create and maintain a strong `PROJECT_SPEC.md` at the root** as the human contract.
2. **Optionally add `project_manifest.yaml`** as a more structured, machine-friendly summary.
3. **Always reference these in your prompts** (ChatGPT & Copilot Chat), especially the invariants section.
4. Sprinkle **SPEC comments** in critical files so the AI sees constraints even when it doesn’t have the spec pasted.
