# CI System Design Considerations

Continuous integration (CI) systems should be supported by certain general
design considerations (or guiding principles) to maintain their value as the
use of the underlying integration system increases - both in terms of the
number of developers and the complexity of the build systems and software they
execute in CI.

Below are these considerations at a high level, with lower-level sub-items to
inform technical implementation. This ordering of the high-level considerations
does not imply relative emphasis for each principle (though any implementation
should rank the importance of these principles based on specific technical
goals corresponding to the organization’s current needs and deficiencies).

---

## Reliable
1. **Avoid** resolving failures by retrying.
2. **Prevent** different CI jobs in the same compute environment from affecting
   each other’s outcomes.
3. **Ensure** failures are only related to relevant code changes (not invalid
   auth credentials, disk space issues, time of day, etc.).

---

## Available
1. **Prevent** jobs from waiting unreasonable amounts of time before starting
   execution.
2. **Operate** relevant monitoring/telemetry (backend) services at different
   layers of the stack to provide insight into resource bottlenecks.

---

## Efficient
1. **Utilize** system resources (disk IOPS, disk space, CPU cycles, RSS
   allocations, network I/O, etc.) in proportion to the job’s requirements.
2. **Right-size** inefficient jobs (ideally via automation, but at least via
   human intervention supported by automatically-captured historical metrics).
3. **Avoid** keeping CI execution environments alive longer than is
   cost-effective.  *Note: long provisioning times for some environments may
   justify quasi-ephemeral instead of purely ephemeral execution.*

---

## Secure
1. **Scope** user permissions in CI to the minimum set required for the job (not `root` by default).
2. **Use** short-lived credentials for job execution (preferably provided by a third-party auth service and provisioned per-job for auditing) and **avoid** writing them to the filesystem.
3. **Restrict** network configuration in CI execution environments to disallow unnecessary egress/ingress.
4. **Sign** CI artifacts intended for production consumption with a trusted authority, and **register** them with the CI job that produced them.

---

## Clear
1. **Make** execution environment versions and definitions easy to reproduce/introspect from the CI job output alone (e.g., by indicating the OCI image, tag, and hash used, or the NixOS toplevel path).
2. **Limit** log lines to information that helps diagnose failures (discourage success-only messages) and **ensure** that artifacts produced as side effects are only provided in successful cases.
3. **Provide** a single reproduction command in case of failure, allowing developers to execute all job steps in the same environment outside of CI.

---

## Supportive
1. **Prune** unavoidably long-running or flaky jobs that do not serve the needs of high-velocity repositories/branches (flag automatically via telemetry).
2. **Egress** as much work to a cache as possible in the event of failure to prevent redundant effort on developers’ machines.

---

## Performant
1. **Avoid** allowing inter-job dependencies to unnecessarily block progress (use fan-ins sparingly).
2. **Avoid** defining sequential steps that could be run in parallel (e.g., fetch two items simultaneously instead of one after the other).

---

## Observable
1. **Publish** host metrics from CI execution environments, including:
   - version of the CI execution environment (OCI image name+tag+hash, NixOS `/run/current-system`, etc.)
   - `machine-id`
   - available memory
   - disk I/O metrics (available space, IOPS, etc.)
   - network ingress/egress
   - unused CPU
2. **Instrument** the PID tree for each job to emit per-PID resource usage metrics.
3. **Make** CI logs visible within the CI UI and **centrally ingest** them (to support finding all jobs that failed in a specific, identifiable way).

---

## Flexible
1. **Support** execution environments matching both developers’ host platforms and the target runtime platforms of the software (even as these platforms evolve over time).
2. **Provide** first-class support for rolling out execution environment updates (new OCI runtime versions or variants, new CI AMI releases, etc.).
3. **Remain** “premises-agnostic” wherever practical (able to run as easily locally, on-premises, or in various cloud environments).
4. **Wrap** abstractions over common CI operations in type-safe “modules” (ensuring typed parameters), consumable in a way that is independent of the specific CI system (no Actions, Orbs, or Components).

