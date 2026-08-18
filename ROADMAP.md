# Learning and Implementation Roadmap

This roadmap turns the goals in [PROJECT.md](PROJECT.md) into an approximately
12-week course of study and implementation. It assumes 2–3 focused hours per
day, six days per week, with one lighter review or rest day.

The schedule is a guide rather than a deadline. A milestone is complete only
when its concepts can be explained and its behavior is supported by tests or
measurements.

## How We Work

Each major component follows the same learning loop:

1. Study the underlying concept using primary sources.
2. Explain the concept in our own words.
3. Write a short design and predict its behavior before coding.
4. Implement the smallest useful version.
5. Test correctness, edge cases, and failure behavior.
6. Benchmark the relevant performance characteristics.
7. Record results, limitations, tradeoffs, and lessons learned.

Assistance should progress from questions and hints to pseudocode and focused
examples. Complete implementations should not replace understanding. Before a
change is merged, its author should be able to explain its important lines,
design decisions, and observed behavior.

Every substantial pull request should answer:

- What problem does this change solve?
- Why does the problem matter for LLM inference?
- Which alternatives were considered?
- Why was this design chosen?
- How was it tested or measured?
- What was learned?

## Suggested Daily Session

- 25 minutes: theory and primary-source reading
- 20 minutes: explain the concept from memory
- 75 minutes: implementation
- 25 minutes: tests or benchmarks
- 15 minutes: learning notes and next questions

## Milestone 1: Transformer Inference Fundamentals

**Target:** Week 1

### Learn

- Tokens, tokenization, logits, and sampling
- Autoregressive generation
- Attention at a conceptual and tensor-shape level
- Prefill and decode
- How inference differs from training
- Compute, memory capacity, memory bandwidth, and synchronization

### Build

- Load and inspect a small open model and tokenizer.
- Inspect token IDs, attention masks, logits, and output shapes.
- Invoke the model's forward pass directly.
- Implement greedy, token-by-token generation without using `model.generate()`.
- Measure prefill and per-token decode latency separately.
- Compare CPU and Apple MPS execution where supported.

### Completion Gate

- Implement a basic autoregressive generation loop.
- Identify prefill and decode in the implementation.
- Explain the important tensor shapes.
- Measure latency without treating the high-level generation API as a black box.

## Milestone 2: Baseline Streaming Service

**Target:** Week 2

### Learn

- Request lifecycles and state transitions
- Synchronous and asynchronous execution
- HTTP streaming and server-sent events
- Cancellation, deadlines, and timeouts
- Time to first token and inter-token latency

### Build

- A minimal inference HTTP service
- A streaming generation endpoint
- Explicit request and response schemas
- Cancellation when a client disconnects
- A command-line client
- Unit and integration tests

### Measure

- Time to first token (TTFT)
- Inter-token latency (ITL)
- End-to-end latency
- Output tokens per second

### Completion Gate

- Multiple clients can stream responses without mixing request state.
- Cancellation and timeout behavior is tested.
- Baseline latency metrics can be reproduced.

## Milestone 3: Benchmark Harness

**Target:** Week 3

### Learn

- Latency percentiles and throughput
- Open-loop and closed-loop load generation
- Warm-up effects and coordinated omission
- Prompt-length and output-length distributions

### Build

- A configurable and repeatable workload generator
- Concurrency and arrival-rate controls
- Structured benchmark output
- p50, p95, and p99 reporting
- Reproducible result tables or graphs

### Experiments

- Short and long prompts
- Short and long outputs
- Concurrency levels of 1, 2, 4, and 8
- CPU and MPS execution
- Cold and warmed-up execution

### Completion Gate

- The methodology and environment are documented.
- Repeated runs produce explainable results.
- Optimizations can be evaluated against a trustworthy baseline.

## Milestone 4: Scheduler and Worker Architecture

**Target:** Week 4

### Learn

- Queueing latency and producer-consumer systems
- Worker lifecycles and async coordination
- FIFO, priority scheduling, fairness, and head-of-line blocking

### Build

```text
API -> Request Queue -> Scheduler -> Worker -> Token Stream
```

- Start with scheduler and worker boundaries in one process.
- Define a request state machine.
- Preserve request identity through token streaming.
- Support queued and active cancellation.
- Handle worker exceptions explicitly.

### Completion Gate

- Request ordering and state transitions are deterministic and tested.
- Queued cancellation, worker failure, and concurrent streams are covered.
- Component boundaries and their purpose can be explained.

## Milestone 5: Admission Control and Backpressure

**Target:** Week 5

### Learn

- Bounded queues, concurrency limits, and overload behavior
- Little's Law and system capacity
- Rejecting early versus timing out late
- Fairness and starvation

### Build

- Maximum queue depth and active-request limits
- Request deadlines and overload responses
- Queue and execution timeout handling
- A documented initial fairness policy

### Experiments

- Increase offered load until the system saturates.
- Locate the sustainable throughput boundary.
- Observe queue latency near and beyond saturation.
- Compare bounded and unbounded queue behavior.

### Completion Gate

- Overload behavior is intentional, bounded, and tested.
- Capacity limits are supported by measurements.
- The chosen rejection and fairness policies can be defended.

## Milestone 6: Static and Continuous Batching

**Target:** Week 6

### Learn

- Static batching and padding waste
- Iteration-level scheduling and dynamic batch membership
- Prefill/decode interference
- Throughput and TTFT tradeoffs

### Build

1. A simulator with varied prompt and output lengths.
2. A real batching loop around the model.

- Add and remove requests between decode iterations.
- Maintain independent per-request generation state.
- Route generated tokens to the correct client.
- Compare FIFO and prefill-first scheduling.

### Completion Gate

- Continuous batching is demonstrated with requests of different lengths.
- Correctness is tested as requests join and leave the batch.
- Benchmark results quantify latency and throughput tradeoffs.

## Milestone 7: KV Cache and Memory Management

**Target:** Week 7

### Learn

- The role and tensor shapes of attention keys and values
- KV-cache memory growth with model and sequence dimensions
- Static, dynamic, offloaded, and quantized caches
- Internal and external fragmentation
- Prefix caching and paged allocation

### Build

- Compare generation with caching disabled and enabled.
- Inspect cache structures and tensor shapes.
- Create a cache-capacity calculator.
- Implement a simplified block allocator separately from the model runtime.
- Test allocation, release, exhaustion, and eviction behavior.
- Connect cache capacity to scheduling if practical.

### Completion Gate

- Approximate KV-cache memory can be derived from model configuration.
- Cache benefits and costs are demonstrated by measurement.
- The simplified allocator's invariants are documented and tested.

## Milestone 8: Multi-Worker Routing and Failure Handling

**Target:** Week 8

### Learn

- Process boundaries, health checks, and heartbeats
- Load-balancing policies
- Idempotency and the limits of retrying streamed requests
- Partial failure and graceful shutdown

### Build

```text
                    +-> Worker 1
API -> Router ------+-> Worker 2
                    +-> Worker 3
```

- Run workers as separate local processes with small or mock runtimes.
- Track health and capacity.
- Route new requests using a documented policy.
- Drain workers during shutdown.
- Define behavior for failures before and after partial output.

### Completion Gate

- Tests cover unavailable, slow, stale, and crashing workers.
- Retry boundaries for streamed responses are explicit.
- Routing and failure policies are observable and explainable.

## Milestone 9: Observability

**Target:** Week 9

### Learn

- Counters, gauges, histograms, and metric cardinality
- Traces, correlation IDs, and service-level indicators
- Separating queueing, model, and streaming latency

### Instrument

- Requests, errors, and cancellations
- Queue depth and active sequences
- TTFT, ITL, and end-to-end latency
- Input and output tokens
- Batch sizes and worker utilization
- Cache capacity, allocation, and eviction

### Completion Gate

- A request can be followed across system boundaries.
- Load-test behavior can be explained using metrics rather than guesses.
- A reproducible report or dashboard communicates system behavior.

## Milestone 10: Containers and Deployment

**Target:** Week 10

### Learn

- Container and process boundaries
- Readiness, liveness, resource limits, and graceful termination
- Rolling deployment and GPU scheduling concepts

### Build

- Reproducible container images
- A local multi-worker deployment
- Health probes and graceful termination
- Basic Kubernetes manifests
- Documentation for CPU/MPS development and NVIDIA deployment

### Completion Gate

- The service can be built and started reproducibly.
- Deployment health and shutdown behavior are tested.
- GPU resource requests and device-plugin responsibilities can be explained.

## Milestone 11: NVIDIA GPU and vLLM Comparison

**Target:** Week 11

Local development uses an Apple M1 Pro with 16 GB unified memory. This is useful
for system design, small-model experiments, and MPS measurements, but it does
not reproduce NVIDIA CUDA behavior. Short, planned sessions on an NVIDIA GPU
will be used when CUDA-specific evidence is required.

### Evaluate

- Run the established workload on CUDA.
- Profile GPU memory and latency.
- Run an equivalent workload using vLLM.
- Compare throughput, TTFT, memory behavior, and operational complexity.
- Explain why the educational and production implementations differ.

### Completion Gate

- Comparisons use the same documented workload where possible.
- Hardware and methodology limitations are disclosed.
- Conclusions are supported by results rather than product claims.

## Milestone 12: Portfolio and Interview Packaging

**Target:** Week 12

### Produce

- A concise README and architecture diagram
- Reproducible setup and demonstration instructions
- Benchmark methodology and results
- Design-decision and failure-handling documentation
- A summary of lessons and remaining limitations
- Resume bullets supported by completed work and measurements

### Practice

- Present the system in 5, 15, and 30 minutes.
- Explain each major design decision and alternative.
- Describe expected changes at 10x and 100x load.
- Walk through saturation, cache exhaustion, and worker failure.
- Compare static batching, continuous batching, and paged caching.

### Completion Gate

- A new reader can understand, run, and evaluate the project.
- Public claims are proportional to verified results.
- The implementation and its tradeoffs can be explained without relying on
  framework names as substitutes for understanding.

## Progress

| Milestone | Status |
| --- | --- |
| 1. Transformer inference fundamentals | Not started |
| 2. Baseline streaming service | Not started |
| 3. Benchmark harness | Not started |
| 4. Scheduler and worker architecture | Not started |
| 5. Admission control and backpressure | Not started |
| 6. Static and continuous batching | Not started |
| 7. KV cache and memory management | Not started |
| 8. Multi-worker routing and failure handling | Not started |
| 9. Observability | Not started |
| 10. Containers and deployment | Not started |
| 11. NVIDIA GPU and vLLM comparison | Not started |
| 12. Portfolio and interview packaging | Not started |

