# Git Branching and Commit Message Conventions

### Branch Naming Convention

#### Format:

```
change_type-JIRA-123-brief-summary
```

#### Example:

```
feat-JIRA-456-add-login-feature
fix-JIRA-789-correct-login-bug
```

### Commit Message Title Convention

#### Format:

```
[JIRA-123] brief summary goes here
```

#### Example:

```
[JIRA-456] add login feature
[JIRA-789] correct login bug
```

#### Descriptions and Titles:

-   **feature**: A new feature (Title: Features)
-   **fix**: A bug fix (Title: Bug Fixes)
-   **docs**: Documentation only changes (Title: Documentation)
-   **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc) (Title: Styles)
-   **refactor**: A code change that neither fixes a bug nor adds a feature (Title: Code Refactoring)
-   **perf**: A code change that improves performance (Title: Performance Improvements)
-   **test**: Adding missing tests or correcting existing tests (Title: Tests)
-   **build**: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm) (Title: Builds)
-   **ci**: Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs) (Title: Continuous Integrations)
-   **chore**: Other changes that don't modify src or test files (Title: Chores)
-   **revert**: Reverts a previous commit (Title: Reverts)
