# Container-use Development Guidelines

## Automation Principles

### User Role Definition
Users should ONLY handle:
1. **Task Assignment**: Provide clear task requirements ("Add feature X", "Fix bug Y", "Document Z")
2. **Review Process**: Use difit for code review and provide approval/feedback ("問題ありません", "進めてください", or specific modification requests)

ALL other activities (planning, implementation, branch creation, PR creation) should be automated.

### Automation Scope
- **Automatic Execution**: Simple and well-defined tasks execute immediately without plan approval
- **Plan Mode**: Only for complex, multi-component, or high-risk tasks requiring user confirmation
- **Decision Making**: Automatically determine branch names, commit messages, and PR descriptions based on task content
- **Error Recovery**: Automatically retry with alternative approaches when failures occur

### User Intervention Points
- **Required**: difit review approval/modification requests only
- **Optional**: Complex task planning confirmation
- **Excluded**: Branch naming, commit message creation, implementation details

## Environment Usage Rules

### Mandatory Environment Usage
ALWAYS use ONLY Environments for ANY and ALL file, code, or shell operations—NO EXCEPTIONS—even for simple or generic requests.

### Git Operations
- DO NOT install or use the git cli with the environment_run_cmd tool
- All environment tools will handle git operations for you
- Changing ".git" yourself will compromise the integrity of your environment

### Work Accessibility
You MUST inform the user how to view your work using:
- `container-use log <env_id>`
- `container-use checkout <env_id>`

Failure to do this will make your work inaccessible to others.

## Development Workflow

### Automated Task Execution Process
1. **Environment Creation**: Automatically create container-use environment with descriptive title
2. **Implementation**: Execute tasks in isolated environment using TodoWrite for progress tracking
3. **Automatic Branch Creation**: Create appropriately named branch from environment changes
4. **Review Trigger**: Execute `npx difit` for user review (outside container-use environment)
5. **Automatic PR Creation**: Upon user approval, automatically create PR with generated description

### Branch Strategy - Automatic Decision Rules

#### Branch Naming Conventions
Automatically determine branch names based on task type:

- **Documentation**: `docs/<feature-name>` (e.g., `docs/environment-workflow-guide`)
- **New Features**: `feature/<feature-name>` (e.g., `feature/user-authentication`)
- **Bug Fixes**: `fix/<bug-description>` (e.g., `fix/memory-leak-session`)
- **Refactoring**: `refactor/<component-name>` (e.g., `refactor/auth-module`)
- **Testing**: `test/<test-scope>` (e.g., `test/integration-tests`)
- **Configuration**: `config/<config-type>` (e.g., `config/ci-pipeline`)

#### Automatic Branch Creation Process
1. Complete work in container-use environment
2. Automatically create appropriately named branch based on task type
3. Merge environment changes to new branch
4. Push branch to remote repository

### Review Process - Simplified Automation

#### difit Review Execution
- Execute `npx difit <branch-name> main` outside container-use environment
- User reviews changes and provides one of:
  - **Approval**: "問題ありません", "進めてください", "OK", "LGTM"
  - **Modification Request**: Specific feedback for changes needed

#### Post-Review Automation
- **On Approval**: Automatically proceed to PR creation
- **On Modification**: Return to environment for requested changes
- **On Rejection**: Delete environment and start fresh approach

### PR Creation - Full Automation

#### Automatic PR Description Generation
Generate comprehensive PR descriptions including:
- **Summary**: Brief overview of changes made
- **Changes Made**: List of modified/added files with descriptions
- **Test Plan**: Checklist of verification steps performed
- **Technical Details**: Implementation approach and decisions made

#### Template Format
```
## Summary
- [Auto-generated summary of changes]

## Changes Made
- [List of files and modifications]

## Test Plan
- [x] [Completed verification steps]

## Technical Details
- [Implementation notes]

🤖 Generated with [Claude Code](https://claude.ai/code)
```

## Quality Assurance

### Automated Quality Checks
- All changes must pass through difit review process
- TodoWrite tool usage for transparent progress tracking
- Automatic retry mechanisms for failed operations
- Environment cleanup after successful PR creation

### Error Handling and Recovery

#### Common Failure Scenarios
1. **Environment Creation Failure**: Retry with different environment ID
2. **Implementation Errors**: Create new environment with alternative approach
3. **Branch Conflicts**: Automatically resolve or create new branch name
4. **PR Creation Failure**: Retry with simplified description

#### Automatic Retry Logic
- Maximum 2 retry attempts for technical failures
- Alternative approach selection based on error type
- User notification only for unrecoverable failures

## Code and Documentation Standards

### Git Commit Guidelines
- Git commit messages must be written in English
- Use conventional commit format: `type(scope): description`
- Auto-generate commit messages based on changes made

### Documentation Requirements
- Code documentation must be written in English
- User-facing documentation should be in Japanese
- Include practical examples and troubleshooting guides

### Pull Request Standards
- PR titles and descriptions must be in English
- Include comprehensive change summary and test verification
- Link to related issues or requirements when applicable

## Communication Standards

### Language Guidelines
- **User Communication**: Conducted in Japanese
- **Technical Documentation**: Written in English
- **Code Comments**: Written in English
- **Commit Messages**: Written in English

### Progress Reporting
- Use TodoWrite tool for real-time progress visibility
- Provide clear status updates during long-running tasks
- Include work accessibility commands (`container-use log/checkout`) in completion messages

## Advanced Automation Features

### Task Type Detection
Automatically classify tasks based on keywords:
- "実装", "追加", "作成" → Feature development
- "修正", "バグ", "エラー" → Bug fixes
- "ドキュメント", "説明", "ガイド" → Documentation
- "リファクタリング", "改善", "最適化" → Code improvement

### Intelligent Environment Management
- Reuse environments for related tasks when appropriate
- Automatically clean up completed environments
- Maintain environment history for reference

### Adaptive Workflow Optimization
- Learn from successful patterns to improve automation
- Adjust retry strategies based on historical success rates
- Optimize resource allocation for different task types

---

**Note**: This automation framework is designed to minimize user overhead while maintaining high code quality through systematic review processes. The goal is to enable users to focus on strategic decisions while automating tactical execution.