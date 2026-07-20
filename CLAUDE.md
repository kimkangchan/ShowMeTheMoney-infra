# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This repository is in its scaffolding stage: only the top-level folder structure exists (`frontend/`, `backend/`, `docs/`, each currently holding just a `.gitkeep`). There is no application code, build tooling, package manifests, Terraform, Kubernetes manifests, or CI/CD configuration yet — do not assume any of these exist, and do not invent commands for them. Check the actual contents of a directory before assuming its tech stack.

## Project background

This is the second iteration ("2차 프로젝트") of "Show Me The Money", a personal finance management platform. The first iteration was an MVP (signup/login, income-expense CRUD, budgeting, dashboard) running on a VMware Fusion-based on-prem Kubernetes cluster. This iteration's goal is to migrate that MVP to an operable, AWS-based cloud service rather than build new product features. Planned direction (not yet implemented):

- **Infra migration**: on-prem Kubernetes → AWS EKS
- **Database**: RDS MySQL, for stronger backup/recovery
- **IaC**: Terraform, for infra reproducibility
- **CI/CD**: GitHub Actions + ECR + Helm for image build/packaging, ArgoCD for GitOps deployment
- **Monitoring**: CloudWatch
- **Service features**: retain the 1st-iteration MVP scope, but fix the frontend/backend API-contract misalignment that caused integration issues last time — API specs should be agreed upon between frontend and backend before implementation

## Repository layout intent

- `frontend/`, `backend/` — application code to be added later (stack not yet decided in this repo)
- `docs/` — architecture docs and operational guides (to be written)

Root `.gitignore` already anticipates future tooling: it excludes `.env` files (including `backend/.env`), Terraform artifacts (`*.tfstate`, `.terraform/`, `terraform.tfvars`), and `*.pem` key files — keep these patterns in mind once that tooling is added.
