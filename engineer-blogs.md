# Engineering Blogs

## Uber

**[CRISP: Critical Path Analysis for Microservice Architectures](https://www.uber.com/blog/crisp-critical-path-analysis-for-microservice-architectures/)**

Uber developed a tool, **CRISP** (named taking letters from critical and span), to pinpoint and quantify underlying services that impact the overall latency of a request in a microservice architecture. CRISP identifies the **critical path** in a complex service dependency graph. The longest path through the DAG is called the critical path in a parallel execution.

**Jaeger**, an open-source tool software, is used for tracing transactions between micro-services. It traces time visualized as a parallel computing DAG.

How to identify the critical path? - Computing the critical path from an individual trace involves replaying the trace in reverse order from the end of the trace to the beginning of the trace, looking for synchronization events.

Dealing with the Clock Skew - allowing a small overlap among the overlapping children & truncated to the finish time of the parent.
