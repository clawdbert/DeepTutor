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
