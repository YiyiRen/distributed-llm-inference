# Project Vision

Build a distributed LLM inference platform from first principles to develop a
practical understanding of the systems behind production LLM serving.

The project focuses on the infrastructure surrounding inference rather than on
training a model or wrapping an existing API. Important components will first
be implemented in simplified form so their behavior and tradeoffs can be
observed directly. Production tools can then be integrated and compared
against those implementations.

## Goals

- Understand the lifecycle of an LLM inference request, including tokenization,
  prefill, decoding, streaming, and completion.
- Build scheduling, admission control, backpressure, request routing, and
  worker-management mechanisms.
- Explore continuous batching and KV/prefix caching through working
  implementations and measurements.
- Study how GPU memory and compute constraints influence serving-system design.
- Add meaningful observability for latency, throughput, utilization, queueing,
  and failures.
- Produce reproducible benchmarks that support architectural conclusions.
- Document design decisions, alternatives, tradeoffs, and lessons learned.

## Learning Principles

For each major component, the project should answer:

1. What problem does it solve?
2. Why does the problem matter for LLM workloads?
3. What design alternatives were considered?
4. What tradeoffs does the chosen design make?
5. What do tests and benchmarks demonstrate?

Implementation should favor clarity and measurable behavior over premature
complexity. Existing production systems such as vLLM will be used as references
and comparison points, not as substitutes for understanding the core concepts.

## Initial Milestones

1. Establish a baseline streaming inference service and benchmark harness.
2. Separate the scheduler, request queue, and inference workers.
3. Add admission control, concurrency limits, and backpressure.
4. Implement and evaluate continuous batching.
5. Explore KV-cache and prefix-cache management.
6. Add multi-worker routing, health checking, retries, and failure recovery.
7. Build end-to-end observability and reproducible workload benchmarks.
8. Containerize and deploy the platform, then compare it with vLLM.

Milestone details may change as experiments reveal better questions or expose
new constraints. Architectural claims should be supported by implementation,
tests, or measurements.

## Non-Goals

- Training a foundation model from scratch.
- Building custom CUDA or Triton kernels during the initial milestones.
- Creating a general-purpose chatbot product.
- Treating a thin wrapper around a hosted model API as an inference platform.
- Claiming production readiness or scale without evidence.

## Definition of Success

The project succeeds when it provides:

- A working and tested inference platform whose major components are understood.
- Reproducible experiments covering latency, throughput, capacity, and failure
  behavior.
- Clear documentation that explains the architecture and its tradeoffs.
- Concrete evidence of hands-on knowledge of distributed LLM serving systems.

