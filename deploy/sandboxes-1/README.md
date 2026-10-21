# DeepTutor on sandboxes-1

Deployment notes for this fork's `sandboxes-1` branch. `main` tracks upstream
(HKUDS/DeepTutor) untouched; this branch is the release tag plus this folder.

- Plan: https://plan.agent-native.com/plans/plan-b45540aaecac4ab8
- Units in `units/` are linked into `~/.config/systemd/user` with
  `systemctl --user link`, so the running app is defined by this branch.
- Rootless Podman pod `deeptutor`: host `127.0.0.1:8340` -> container `:3782`;
  the pod reaches host `127.0.0.1:8317` (LLM gateway) only (`pasta -T 8317`).
- State lives in `../../data` (git-ignored upstream). Nothing secret is ever
  committed here: keys are entered in the app's Settings UI by the Operator.

## Upgrade

    git fetch upstream --tags && git rebase --onto vX.Y.Z vOLD sandboxes-1
    # change the image digest in units/deeptutor.service, then
    systemctl --user daemon-reload && systemctl --user restart deeptutor.service

## Install notes (v1.6.15, 2026-10-21)

- `--userns=keep-id` must be set on the **pod**: podman 4.9 refuses
  `--userns` together with `--pod` on `podman run`.
- The image entrypoint sets `DEEPTUTOR_IGNORE_PROCESS_ENV_OVERRIDES=1` and
  re-exports settings from `data/user/settings/*.json` on every start, so
  `-e AUTH_*` / `-e DEEPTUTOR_SANDBOX_ALLOW_SUBPROCESS` are no-ops. Set them in
  `auth.json` (`enabled`, `private_login_hosts` as `host:port`) and
  `system.json` (`cors_origins`, `sandbox_allow_subprocess: false`).
  `DEEPTUTOR_SANDBOX_RUNNER_URL` is read straight from the environment.
- First account at `/register` becomes admin when `username` and
  `password_hash` are both empty; registration closes after that.
- There is no `deeptutor` binary on PATH in the image; use
  `podman exec deeptutor-app python -m deeptutor_cli doctor --format json`.
- The front door must preserve the browser's `Host` (login is gated on it).
- The sandbox runner binds 0.0.0.0:8900 with no authentication; it is only
  safe inside the pod with no published port.
