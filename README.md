# pipeline

GitLab CI/CD components for golden-path apps, plus artifact tracking. Every component writes its outputs under `gp-artifacts/`, and `artifact-record` writes one hashed manifest per pipeline.

| Component | Stage | What it does | Blocks the merge when |
| --- | --- | --- | --- |
| `contract-validate` | validate | `gold validate --strict` and `gold explain` on the app's contract | The contract is invalid |
| `security-scan` | validate | Runs `gold-scan static` from the platform scanner image | Secrets or critical/high findings without an exception |
| `tofu-plan` | plan | fmt, validate and a saved, encrypted plan per environment | The plan fails |
| `policy-check` | review | Conftest rules from `platform/policy/conftest` on the plan | Any rule is broken |
| `cost-estimate` | review | Infracost on the plan; warns over a threshold | Never |
| `tofu-apply` | deploy | Applies the saved plan in a GitLab environment, on the default branch only | — |
| `ephemeral-env` | deploy, cleanup | Per-MR review environment with a TTL, deleted on demand or on expiry | — |
| `artifact-record` | .post | Hashes every file under `gp-artifacts/` into `artifact-manifest.json` | — |

## Use

```yaml
stages: [validate, plan, review, deploy, cleanup]

include:
  - component: $CI_SERVER_FQDN/michaeljloria/golden-paths-devops/tofu-plan@1.0.0
    inputs:
      app: notifier
      env: dev
      root_dir: notifier/infra
      changes: "notifier/**/*"
      cloud: aws
      role: arn:aws:iam::<dev-account-id>:role/gp-plan-notifier
  - component: $CI_SERVER_FQDN/michaeljloria/golden-paths-devops/artifact-record@1.0.0
```

## Requirements

- The consuming project must be in the platform project's CI/CD job token allowlist, so jobs can download `gold` and clone `policy/` and `scripts/`.
- Cloud roles must trust the job's OIDC token, with audience `https://gitlab.com`.
- `cost-estimate` needs `INFRACOST_API_KEY` as a masked CI/CD variable.
