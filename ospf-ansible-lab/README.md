# OSPF Ansible Lab —  Summary

This lab automates OSPF configuration across a multi-router topology to demonstrate repeatable, auditable network changes using Ansible templates.

Key outcomes:
- Automated creation of OSPF areas and neighbors across 5 routers
- Idempotent playbooks safe for re-run in CI/CD
- Demonstrated validation using `show ip ospf neighbor` and ping tests

How to run:
1. ansible-playbook -i inventory site.yml
2. Verify neighbors: show ip ospf neighbor
