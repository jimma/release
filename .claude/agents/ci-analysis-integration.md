---
name: ci-analysis-integration
description: Use this agent when you need to work together with other subagents.
model: sonnet
color: orange
---
# CI Analysis Integration Guide

## Agent Coordination Strategy

IMPORTANT: DON'T grep/find/search anything in folder "ci-operator/jobs"

The four CI analysis agents work together to provide comprehensive analysis of the Prow CI configuration system. Here's how they coordinate and interact:

## Analysis Hierarchy

```
Job Analyzer (Entry Point)
    ↓
Workflow Analyzer (Structure & Phases)
    ↓
Chain Analyzer (Composition & Order)
    ↓
Ref Analyzer (Implementation Details)
```

## Agent Interaction Patterns

### 1. Job-Centric Analysis Flow
When analyzing a job:
1. **Job Analyzer** parses the job configuration and identifies workflows
2. **Workflow Analyzer** analyzes each workflow's phases and components
3. **Chain Analyzer** breaks down chains into their constituent refs
4. **Ref Analyzer** provides detailed command implementation analysis

### 2. Component-Centric Analysis Flow
When analyzing a specific component:
1. **Ref Analyzer** provides the fundamental implementation details
2. **Chain Analyzer** shows how refs are composed into chains
3. **Workflow Analyzer** demonstrates chain integration in workflows
4. **Job Analyzer** shows workflow usage in actual jobs

### 3. Dependency Analysis Flow
When analyzing dependencies:
1. **Job Analyzer** identifies job-level dependencies
2. **Workflow Analyzer** maps workflow phase dependencies
3. **Chain Analyzer** analyzes chain execution dependencies
4. **Ref Analyzer** identifies resource and tool dependencies

## Cross-Agent Data Sharing

### Shared Context Information
- **File Locations**: Directory paths and file names
- **Naming Conventions**: Patterns for jobs, workflows, chains, refs
- **Parameter Mappings**: How parameters flow between levels
- **Resource References**: Shared resources and dependencies

### Analysis Results Flow
- **Job → Workflow**: Job parameters and execution context
- **Workflow → Chain**: Phase requirements and component composition
- **Chain → Ref**: Execution order and parameter passing
- **Ref → Chain**: Implementation capabilities and requirements

## Orchestration Patterns

### 1. Top-Down Analysis (Job First)
```
User provides job name
→ Job Analyzer finds workflows
→ Workflow Analyzer analyzes phases
→ Chain Analyzer breaks down chains
→ Ref Analyzer provides implementation details
```

### 2. Bottom-Up Analysis (Ref First)
```
User provides ref name
→ Ref Analyzer analyzes implementation
→ Chain Analyzer shows ref usage in chains
→ Workflow Analyzer maps chain integration
→ Job Analyzer shows end-to-end usage
```

### 3. Middle-Out Analysis (Workflow/Chain First)
```
User provides workflow or chain name
→ Workflow/Chain Analyzer analyzes structure
→ Job Analyzer provides context
→ Ref Analyzer provides implementation details
```

## Integration Scenarios

### 1. Complete Job Analysis
**Goal**: Understand what a job does end-to-end

**Coordination**:
1. Job Analyzer: Parse job config, identify workflows and parameters
2. Workflow Analyzer: Analyze workflow phases and execution flow
3. Chain Analyzer: Break down chains into execution sequences
4. Ref Analyzer: Provide detailed command analysis for critical steps

**Output**: Comprehensive job analysis with all levels of detail

### 2. Component Impact Analysis
**Goal**: Understand the impact of changing a component

**Coordination**:
1. Ref Analyzer: Analyze component implementation and dependencies
2. Chain Analyzer: Find all chains using this ref
3. Workflow Analyzer: Identify workflows using affected chains
4. Job Analyzer: List jobs that would be affected

**Output**: Impact assessment with affected jobs and workflows

### 3. Debugging Analysis
**Goal**: Debug a failing CI job

**Coordination**:
1. Job Analyzer: Parse job configuration and identify failure point
2. Workflow Analyzer: Map failure context within workflow phases
3. Chain Analyzer: Identify chain execution order and dependencies
4. Ref Analyzer: Analyze failed ref implementation and error handling

**Output**: Root cause analysis with debugging guidance

### 4. Optimization Analysis
**Goal**: Optimize CI execution performance

**Coordination**:
1. Job Analyzer: Identify job execution patterns and bottlenecks
2. Workflow Analyzer: Analyze phase execution timing
3. Chain Analyzer: Identify parallelization opportunities
4. Ref Analyzer: Analyze command efficiency and resource usage

**Output**: Optimization recommendations with specific suggestions

## Error Handling Coordination

### Error Propagation
- **Job Analyzer**: Handles job-level errors (missing jobs, malformed configs)
- **Workflow Analyzer**: Handles workflow errors (missing phases, invalid references)
- **Chain Analyzer**: Handles chain errors (missing refs, invalid composition)
- **Ref Analyzer**: Handles ref errors (missing commands, security issues)

### Error Recovery
- **Graceful Degradation**: Continue analysis with available information
- **Alternative Paths**: Suggest similar components or approaches
- **Clear Guidance**: Provide actionable error messages and fixes
- **Context Preservation**: Maintain analysis context across agent boundaries