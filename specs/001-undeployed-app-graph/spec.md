# Feature Specification: Pre-Deployment App Graph Visualization

**Feature Branch**: `001-undeployed-app-graph`
**Created**: 2025-01-23
**Status**: Draft
**Input**: User description: "Radius currently stores the state of application deployments as an app graph within its data store. Today, the app graph does not get generated until the application is deployed. Build an app graph representation for applications that are defined (e.g. in an app.bicep file) but not yet deployed. Key Requirements: 1. Generate app graph from undeployed application definitions (e.g., app.bicep files) 2. Enrich the app graph representation with git changelog info (i.e., git commit data) so this data can be used to visualize how the app graph changes over time (across commits) 3. Enable visualization of the app graph and diff capabilities in GitHub on PRs, commit comparisons, etc."

## Clarifications

### Session 2025-01-23

- Q: What GitHub authentication method should be used for GitHub integration (GitHub App, OAuth token, or Personal Access Token)? → A: GitHub App (most secure and scalable; allows fine-grained permissions; better for organization-wide deployment)
- Q: What is the maximum supported graph size and rendering strategy for very large graphs? → A: Unlimited resources; automatically switch to list view above 100 resources (most flexible; provides fallback for very large graphs)

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Preview App Graph Before Deployment (Priority: P1)

As a Radius developer, I want to visualize the application graph structure from my Bicep definition files before deploying, so I can understand the relationships between resources and validate my application architecture early in the development cycle.

**Why this priority**: This is the foundational capability that enables all other scenarios. Developers need to see what they're building before committing resources to deployment, catching architectural issues early saves time and prevents costly mistakes in production.

**Independent Test**: Can be fully tested by providing a valid app.bicep file, generating the app graph representation, and verifying the graph contains all defined resources and their relationships. Delivers immediate value by allowing developers to validate architecture without deployment.

**Acceptance Scenarios**:

1. **Given** a developer has created an app.bicep file defining an application with multiple resources (containers, databases, networking), **When** they request to generate the app graph, **Then** the system produces a visual representation showing all resources and their connections
2. **Given** an app.bicep file with resource dependencies, **When** the app graph is generated, **Then** the graph accurately reflects dependency relationships between resources
3. **Given** an app.bicep file with syntax errors, **When** graph generation is attempted, **Then** the system provides clear error messages indicating which definitions cannot be parsed
4. **Given** an empty or minimal app.bicep file, **When** the app graph is generated, **Then** the system produces a valid graph with minimal nodes

---

### User Story 2 - Track App Graph Changes Across Git Commits (Priority: P2)

As a platform engineer, I want to see how the application architecture has evolved across git commits, so I can understand the history of architectural decisions and identify when specific changes were introduced.

**Why this priority**: Historical tracking adds significant value for teams managing complex applications over time. It enables architectural auditing, helps with troubleshooting when issues arise, and provides documentation of architectural evolution. This builds on P1 by adding temporal context.

**Independent Test**: Can be fully tested by committing multiple versions of an app.bicep file, generating app graphs for different commits, and verifying that each graph reflects the application state at that commit. Delivers value by providing historical architectural visibility.

**Acceptance Scenarios**:

1. **Given** an app.bicep file has been modified across multiple commits, **When** a user requests app graph history, **Then** the system displays graphs for each commit showing how the architecture evolved
2. **Given** two specific git commits, **When** a user requests to compare their app graphs, **Then** the system highlights additions, deletions, and modifications to resources and relationships
3. **Given** a commit that adds new resources, **When** viewing the app graph for that commit, **Then** the new resources are clearly identifiable with commit metadata (author, date, message)
4. **Given** a git repository with branches, **When** generating app graphs, **Then** the system correctly associates graphs with their respective branches

---

### User Story 3 - Visualize App Graph Diffs in GitHub Pull Requests (Priority: P2)

As a code reviewer, I want to see visual diffs of application architecture changes directly in GitHub pull requests, so I can quickly understand the impact of proposed changes without manually comparing Bicep files.

**Why this priority**: Integrating with the GitHub workflow where code reviews happen makes architectural review seamless and improves review quality. This is equally important as P2 for teams using PR-based workflows, as it brings architectural visibility into the existing review process.

**Independent Test**: Can be fully tested by creating a PR that modifies an app.bicep file, triggering the GitHub integration, and verifying that app graph diffs are displayed in the PR. Delivers value by making architectural changes visible during code review.

**Acceptance Scenarios**:

1. **Given** a pull request modifies an app.bicep file, **When** the PR is opened, **Then** the PR displays a visual diff showing before/after app graphs with highlighted changes
2. **Given** a pull request adds new resources, **When** viewing the app graph diff, **Then** new resources are highlighted in a distinct color with labels indicating they are additions
3. **Given** a pull request removes resources, **When** viewing the app graph diff, **Then** removed resources are shown with strikethrough or faded appearance
4. **Given** a pull request modifies resource relationships, **When** viewing the app graph diff, **Then** changed connections are clearly marked showing old and new relationships
5. **Given** multiple commits in a pull request, **When** viewing the app graph, **Then** the diff reflects the cumulative changes across all commits

---

### User Story 4 - Compare App Graphs Between Any Two Commits (Priority: P3)

As a Radius operator, I want to compare application architecture between any two commits or tags, so I can understand what changed between releases or investigate when specific architectural changes were introduced.

**Why this priority**: This extends the comparison capability beyond PRs to support operational and debugging scenarios. While valuable, it's less critical than PR integration since it addresses less frequent use cases.

**Independent Test**: Can be fully tested by selecting any two commits, generating app graphs for each, and displaying a comparison view. Delivers value for investigating historical changes and release planning.

**Acceptance Scenarios**:

1. **Given** a user selects two arbitrary commits, **When** requesting a comparison, **Then** the system displays side-by-side or overlaid app graphs with differences highlighted
2. **Given** comparison between a release tag and the current HEAD, **When** viewing the diff, **Then** all architectural changes since the release are visible
3. **Given** commits on different branches, **When** comparing their app graphs, **Then** the system clearly indicates branch context for each graph

---

### Edge Cases

- What happens when an app.bicep file references external modules or templates that aren't in the repository?
- How does the system handle extremely large app graphs with hundreds of resources that may be difficult to visualize?
- What happens when git history is rewritten (rebased, squashed) after app graphs have been generated?
- How does the system handle app.bicep files in subdirectories or monorepo structures with multiple applications?
- What happens when a commit is corrupted or the repository cannot be accessed?
- How does the system handle binary or non-text changes in git history?
- What happens when app.bicep files use parameterized values that affect graph structure?

## Requirements *(mandatory)*

### Functional Requirements

#### App Graph Generation

- **FR-001**: System MUST parse Bicep application definition files (app.bicep) without requiring deployment
- **FR-002**: System MUST extract all resource definitions from application files including containers, databases, volumes, gateways, and custom resources
- **FR-003**: System MUST identify and represent relationships between resources including dependencies, connections, and bindings
- **FR-004**: System MUST generate a structured app graph representation that includes nodes (resources) and edges (relationships)
- **FR-005**: System MUST handle Bicep syntax errors gracefully and provide meaningful error messages when parsing fails
- **FR-006**: System MUST support Bicep modules and correctly resolve module references to build complete app graphs
- **FR-007**: System MUST generate app graphs from definitions at any git commit or branch, not just the current working state

#### Git Integration

- **FR-008**: System MUST extract git commit metadata including commit SHA, author, timestamp, and commit message
- **FR-009**: System MUST associate generated app graphs with their corresponding git commits
- **FR-010**: System MUST support generating app graphs for historical commits without checking out the commit
- **FR-011**: System MUST detect when application definition files have changed between commits
- **FR-012**: System MUST support both single-application repositories and monorepos with multiple application definitions
- **FR-013**: System MUST handle git repository formats used by GitHub, GitLab, and other common git hosting platforms

#### Comparison and Diffing

- **FR-014**: System MUST compare app graphs from two different commits and identify additions, deletions, and modifications
- **FR-015**: System MUST detect added resources and connections in graph diffs
- **FR-016**: System MUST detect removed resources and connections in graph diffs
- **FR-017**: System MUST detect modified resource properties that affect graph structure (e.g., changed dependencies)
- **FR-018**: System MUST produce diff output that can be visualized in GitHub UI components

#### GitHub Integration

- **FR-019**: System MUST integrate with GitHub pull request workflows to automatically generate app graph diffs
- **FR-020**: System MUST post app graph visualizations as comments or status checks on pull requests
- **FR-021**: System MUST update app graph diffs when new commits are pushed to a pull request
- **FR-022**: System MUST provide a visual representation that renders correctly in GitHub's markdown and HTML rendering
- **FR-023**: System MUST support GitHub commit comparison views (comparing any two commits or branches)
- **FR-024**: System MUST authenticate with GitHub using GitHub App authentication to access repositories and post visualizations, providing fine-grained permissions and organization-wide deployment support

#### Visualization

- **FR-025**: System MUST render app graphs in a visual format showing resources as nodes and relationships as edges
- **FR-026**: System MUST distinguish between different resource types visually (e.g., using different shapes, colors, or icons)
- **FR-027**: System MUST highlight changes in diff views using visual indicators (colors, labels, or annotations)
- **FR-028**: System MUST support rendering graphs with unlimited resources, automatically switching from graph view to list view when resource count exceeds 100 to maintain usability and performance
- **FR-029**: System MUST provide interactive elements where supported (e.g., hovering for details, expanding/collapsing sections)

#### Performance and Reliability

- **FR-030**: System MUST generate app graphs within 30 seconds for typical application definitions (under 10 resources)
- **FR-031**: System MUST cache generated app graphs to avoid regenerating graphs for the same commit
- **FR-032**: System MUST handle repository access failures gracefully with appropriate error messages
- **FR-033**: System MUST process pull requests without blocking the PR workflow (run asynchronously)

### Key Entities

- **App Graph**: A structured representation of an application's architecture containing resources (nodes) and their relationships (edges). Includes metadata such as git commit information, generation timestamp, and source file paths. The graph is independent of deployment state.

- **Resource Node**: Represents a single resource in the application (e.g., container, database, gateway). Contains resource type, identifier, properties relevant to relationships, and position in the dependency hierarchy.

- **Relationship Edge**: Represents a connection between two resources. Includes edge type (dependency, binding, connection), directionality, and any properties that define the relationship.

- **Graph Diff**: A comparison between two app graphs showing additions (new nodes/edges), deletions (removed nodes/edges), and modifications (changed properties or relationships). Includes metadata about the source commits being compared.

- **Commit Context**: Git metadata associated with an app graph including commit SHA, branch name, author, timestamp, commit message, and file paths of the application definitions used to generate the graph.

- **Visualization Artifact**: A renderable representation of an app graph or graph diff in a format suitable for display (e.g., SVG, image, interactive HTML). Includes layout information, styling rules, and interactive elements where applicable.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Developers can generate and view an app graph from an undeployed Bicep definition within 30 seconds for typical applications
- **SC-002**: App graph visualization correctly represents 100% of resources and relationships defined in test application files
- **SC-003**: Graph diffs accurately identify all additions, deletions, and modifications when comparing commits (100% accuracy on test suite)
- **SC-004**: GitHub pull requests automatically display app graph diffs within 2 minutes of PR creation or update
- **SC-005**: 90% of developers can understand architectural changes from the graph diff without reading the raw Bicep code
- **SC-006**: System successfully handles repositories with up to 100 application definitions without performance degradation
- **SC-007**: App graph generation succeeds for 95% of valid Bicep application files in production repositories
- **SC-008**: Historical graph generation works for commits across at least 12 months of repository history
- **SC-009**: Graph diff visualizations render correctly in GitHub UI across desktop and mobile viewports
- **SC-010**: Architectural review time decreases by 40% for pull requests that include app graph diffs compared to manual Bicep file review

## Assumptions

- Application definitions are written in Bicep format as per current Radius standards
- Git is the version control system in use
- GitHub is the primary platform for code review and collaboration, though design should allow for extension to other platforms
- Users have appropriate repository access permissions to read application definitions
- Generated graphs do not need to include runtime-specific information (e.g., actual deployed instance counts, live connection strings)
- Graph generation can be performed on a server/service rather than requiring local execution
- Visual rendering will primarily target web browsers with modern CSS and SVG support
- Cached graphs can be persisted for a reasonable retention period (e.g., 90 days) without storage concerns
- Standard git operations (clone, fetch) are available to the system generating graphs
- GitHub API rate limits are sufficient for automated PR integration in typical team workflows
