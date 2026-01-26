# ADR-0001: Repository Structure & Workflow

Decision:
- Use `infra/` for Terraform and Ansible, `services/` for service assets, `Docs/` for documentation
- Prefer trunk-based development with short-lived feature branches
- Documentation-first approach: runbooks, ADRs, and diagrams included from the start

Consequences:
- Reduced process overhead; faster iteration
- Clear separation of infra code vs. service configs vs. docs
