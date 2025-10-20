# prowci Command

This is the subcommand to analyze the Prow CI configuration system. The Prow CI configuration is organized in a hierarchical structure:

- **Jobs**: Top-level CI jobs that execute workflows
- **Workflows**: Pre/test/post phase orchestration consisting of chains and refs
- **Chains**: Ordered sequences of refs and/or sub-chains
- **Refs**: Atomic operations with associated shell commands

Each element can be analyzed to understand what it actually does in the CI system.

## Usage
/prowci [argument]

The argument can be a job name, workflow name, chain name, or ref name. The command will automatically detect the type and route to the appropriate analyzer.

## Type Detection Logic

The command uses intelligent pattern matching to determine the argument type:

### Job Names
- Pattern: `*<org>-<repo>-<branch>-<arch>-<nightly>-<job-name>`
- Examples:
  - `periodic-ci-openshift-openshift-tests-private-release-4.19-amd64-nightly-aws-ipi-ovn-hypershift-mgmt-f14`
  - `pull-ci-openshift-hypershift-main-e2e-gcp-ovn-techpreview`
- Locations: `ci-operator/config/`

### Workflow Names
- Pattern: `<component>-<description>-workflow`
- File pattern: `*-workflow.yaml`
- Examples:
  - `cucushift-installer-rehearse-gcp-ipi`
  - `cucushift-installer-rehearse-aws-ipi-ovn-hypershift`
- Location: `ci-operator/step-registry/`

### Chain Names
- Pattern: `<component>-<description>-chain`
- File pattern: `*-chain.yaml`
- Examples:
  - `ipi-install`
  - `cucushift-installer-rehearse-gcp-ipi-provision`
  - `openshift-e2e-test-hypershift-qe-mgmt`
- Location: `ci-operator/step-registry/`

### Ref Names
- Pattern: `<component>-<description>-ref`
- File pattern: `*-ref.yaml`
- Examples:
  - `ipi-install-install`
  - `gather-gcp-console`
  - `cucushift-hypershift-extended-health-check`
- Location: `ci-operator/step-registry/`

## Conflict Resolution

When the input argument matches multiple types (e.g., both a workflow and a chain), the command will:

1. **List Matching Types**: Show all detected types with brief descriptions
2. **User Selection**: Allow the user to choose which type to analyze
3. **Recommendation**: Suggest the most likely intended type based on context

### Example Conflict Resolution
```
Input: "cucushift-installer-rehearse-gcp-ipi"

Detected types:
1. Workflow - cucushift-installer-rehearse-gcp-ipi-workflow.yaml
   Complete workflow with pre/test/post phases for GCP IPI testing

2. Chain - cucushift-installer-rehearse-gcp-ipi-provision-chain.yaml
   Provision chain for GCP IPI cluster setup

3. Chain - cucushift-installer-rehearse-gcp-ipi-deprovision-chain.yaml
   Deprovision chain for GCP IPI cluster cleanup

Which type would you like to analyze? (1-3), or 'all' for comprehensive analysis:
```

## Analysis Approaches

### 1. Single-Type Analysis
Important: When the argument clearly matches one type, the command routes directly to invoke appropriate subagent:
- **Job** → ci-job-analyzer
- **Workflow** → ci-workflow-analyzer
- **Chain** → ci-chain-analyzer
- **Ref** → ci-ref-analyzer

### 2. Multi-Type Analysis
When multiple types are detected, the user can choose:
- **Specific Type**: Analyze only the selected type
- **Comprehensive Analysis**: Use bottom-up approach from integration guide:
  - Start with the most specific type (usually ref)
  - Build up through chain → workflow → job
  - Show relationships and dependencies

### 3. Pattern-Based Search
When the argument is a partial match or pattern:
- **Search**: Find all matching components across all types
- **Categorization**: Group results by type
- **Selection**: Allow user to choose specific items to analyze

## Output Examples

### Job Analysis
```
/prowci periodic-ci-openshift-openshift-tests-private-release-4.19-amd64-nightly-aws-ipi-ovn-hypershift-mgmt-f14

[Job Analysis]
- Job Type: Periodic (monthly on 7th and 23rd at 21:30 UTC)
- Workflow: cucushift-installer-rehearse-aws-ipi-ovn-hypershift
- Test Chain: openshift-e2e-test-hypershift-qe-mgmt
- Environment: AWS, OVN, HyperShift, TechPreviewNoUpgrade
- Purpose: VolumeAttributesClass feature gate validation
```

### Workflow Analysis
```
/prowci cucushift-installer-rehearse-gcp-ipi

[Workflow Analysis]
- Type: Base workflow (empty test phase)
- Pre Phase: GCP IPI cluster provisioning
- Test Phase: Empty (for job customization)
- Post Phase: Cluster deprovisioning and artifacts
- Common Usage: Template for GCP IPI testing scenarios
```

### Chain Analysis
```
/prowci ipi-install

[Chain Analysis]
- Type: Installation chain
- Components: RBAC setup → Installation → Validation
- Purpose: Core OpenShift IPI cluster installation
- Used By: Multiple workflows across different platforms
- Key Features: Retry logic, artifact collection, health checks
```

### Ref Analysis
```
/prowci gather-gcp-console

[Ref Analysis]
- Command: gcloud compute instances get-serial-port-output
- Purpose: Collect GCP instance console logs for debugging
- Integration: Used in post-phase cleanup chains
- Security: Requires GCP project access
- Output: Console logs saved to artifacts directory
```

## Advanced Features

### 1. Relationship Mapping
Shows how components relate to each other:
- **Upstream Dependencies**: What this component depends on
- **Downstream Usage**: What uses this component
- **Alternatives**: Similar components that could be used instead

### 2. Impact Analysis
When analyzing a component, shows:
- **Affected Jobs**: Jobs that would be impacted by changes
- **Test Coverage**: What this component validates
- **Risk Assessment**: Potential failure points and impacts

### 3. Performance Insights
Provides performance-related information:
- **Execution Time**: Typical duration and bottlenecks
- **Resource Usage**: CPU, memory, and storage requirements
- **Parallelization**: Opportunities for parallel execution

### 4. Security Assessment
Analyzes security aspects:
- **Credential Requirements**: What credentials are needed
- **Permissions**: Required access levels and scopes
- **Data Sensitivity**: Handling of sensitive information

## Error Handling

The command provides helpful error messages for common issues:

### Component Not Found
```
Error: Component "xyz-abc" not found in CI configuration.

Did you mean:
- xyz-abc-ref (ref in ci-operator/step-registry/xyz/abc/)
- xyz-abc-chain (chain in ci-operator/step-registry/xyz/abc/)
- xyz-abc-workflow (workflow in ci-operator/step-registry/xyz/abc/)

Use /prowci search xyz-abc to find all matching components.
```

### Access Issues
```
Error: Cannot access configuration files for component "xyz-abc".

Possible causes:
- File permissions issue
- Component not available in this repository
- Network connectivity issues

Try checking if the component exists in the repository structure.
```

### Multiple Matches
```
Warning: "abc" matches multiple components:

Jobs (2):
- periodic-abc-test
- pull-ci-abc-build

Workflows (1):
- abc-workflow

Chains (3):
- abc-provision-chain
- abc-test-chain
- abc-cleanup-chain

Refs (5):
- abc-provision-ref
- abc-test-ref
- abc-validate-ref
- abc-gather-ref
- abc-cleanup-ref

Please be more specific or choose a number to analyze that component.
```

## Integration with CI System

This command provides direct access to the same analysis capabilities used by CI system maintainers:

- **Real-time Analysis**: Uses current configuration files
- **Complete Coverage**: Analyzes all aspects of the CI hierarchy
- **Maintainer Perspective**: Shows the same information CI maintainers use
- **Debugging Support**: Helps understand CI job failures and behavior

## Tips for Effective Usage

### 1. Be Specific
- Use full component names when possible
- Include context (platform, version, etc.) for better results

### 2. Start Broad, Then Narrow
- Use partial names to discover related components
- Select specific components for detailed analysis

### 3. Use Relationship Analysis
- Understand how components fit into the larger CI system
- Identify upstream and downstream dependencies

### 4. Leverage Conflict Resolution
- When multiple types match, explore each to understand the full picture
- Use comprehensive analysis to see relationships between types

### 5. Check Security and Performance
- Always review security implications, especially for production changes
- Consider performance impacts when modifying components

This command serves as your gateway to understanding the complete Prow CI configuration system, from high-level jobs down to individual shell commands.