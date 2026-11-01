# ADR-001: Use a Modular Monolith Initially

- Status: Accepted (initial design)
- Context: ParsePilot needs several related capabilities, but the initial team and workload do not justify microservice operations.
- Decision: Build one backend application with explicit modules and a separate worker process where asynchronous execution requires it.
- Consequences: Easier local development, transactions and debugging. Module boundaries must be respected; extract services only when scale or ownership requires it.
