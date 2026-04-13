# Skills

Skills defined in this repository:

- `python-fastapi-contributor`: reusable contributor workflow for Python and FastAPI repositories
- `repository-technical-analysis`: reusable investigation-first workflow for code repositories
- `gitlab-mr-comment-analysis`: reusable GitLab merge-request comment-analysis workflow
- `jira`: reusable Jira and Atlassian issue access, creation, and update workflow
- `splunk-jira`: Splunk-specific overlay on top of `jira` for reading, creating, and updating Splunk Jira tickets
- `guided-experience-service-contributor`: repo-specific contributor overlay for guided-experience-service
- `guided-experience-service-technical-analysis`: repo-specific technical analysis overlay for guided-experience-service
- `guided-experience-service-parallel-tests`: run guided-experience-service unit and integration tests with 10 workers
- `guided-experience-service-mr-comment-analysis`: guided-experience-service overlay that uses `gitlab-mr-comment-analysis` plus repo-specific technical analysis and proposed changes
- `multi-spawn-agent`: reusable template for spawning parallel worker agents with disjoint ownership

Skill locations:

- `python-fastapi-contributor/SKILL.md`
- `repository-technical-analysis/SKILL.md`
- `gitlab-mr-comment-analysis/SKILL.md`
- `jira/SKILL.md`
- `splunk-jira/SKILL.md`
- `guided-experience-service-contributor/SKILL.md`
- `guided-experience-service-technical-analysis/SKILL.md`
- `guided-experience-service-parallel-tests/SKILL.md`
- `guided-experience-service-mr-comment-analysis/SKILL.md`
- `multi-spawn-agent/SKILL.md`

Usage notes:

- The `guided-experience-service-*` skills are overlays intended to be used with the corresponding generic workflow skill.
- `splunk-jira` is an overlay intended to be used with `jira`.
- `guided-experience-service-mr-comment-analysis` specifically layers guided-experience-service analysis on top of `gitlab-mr-comment-analysis`.
- Codex discovers installed skills from `~/.codex/skills`, not directly from this repository.
- This repository installs skills via `git-hooks/post-commit`, which copies each top-level skill directory into `~/.codex/skills` after a commit.
- Restart Codex after installing or updating skills so the active session can pick them up.
