# pipeline

Pipeline is a set of reusable GitLab CI/CD components for golden-path apps: contract validation, security scanning, saved-plan OpenTofu plan and apply, policy checks, cost estimates, per-merge-request review environments, and a closing job that records every artifact. Each component writes its outputs under `gp-artifacts/`, and the last job hashes them into one `artifact-manifest.json` so a pipeline's artifacts can be audited. Include a component by pinning an exact version from the CI/CD catalog and passing the app, environment and role inputs it documents.
