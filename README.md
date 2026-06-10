<div align="center">

# github-actions-embedded

### Reusable CI/CD workflows for embedded systems and server projects

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Arduino](https://img.shields.io/badge/-Arduino-00979D?style=for-the-badge&logo=Arduino&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

</div>

Copy any workflow into your `.github/workflows/` folder and it works. Covers the full embedded + server CI/CD pipeline: sketch compilation → C++ build → Python test → Docker build → Jetson deploy.

---

## Workflows

| File | Trigger | What it does |
|------|---------|--------------|
| [`arduino-compile.yml`](./workflows/arduino-compile.yml) | `.ino` file push | Compiles every sketch against Uno / Mega / Leonardo with arduino-cli |
| [`cpp-build.yml`](./workflows/cpp-build.yml) | `.cpp` / `.h` push | CMake or direct g++ build + test runner |
| [`python-ci.yml`](./workflows/python-ci.yml) | `.py` push | Ruff lint → mypy type check → pytest with coverage report |
| [`docker-build-push.yml`](./workflows/docker-build-push.yml) | Push to `main` / tag | Multi-platform build (amd64 + **arm64**) → push to GHCR |
| [`deploy-jetson.yml`](./workflows/deploy-jetson.yml) | After Docker push | SSH into Jetson, pull image, restart container |

> **Why no live status badges here?** These files live in [`/workflows/`](./workflows/), not `.github/workflows/`, because this repo is a **library of reusable templates** — they are meant to be copied into other projects, not run inside this repo. You'll see their real green/red status badges in the repos that adopt them (e.g. [`fastapi-sensor-api`](https://github.com/kirilov07/fastapi-sensor-api/actions) and [`mqtt-iot-bridge`](https://github.com/kirilov07/mqtt-iot-bridge/actions), both running `python-ci.yml`).

---

## How to use

Copy the workflow file(s) you need into your repo:

```bash
cp workflows/python-ci.yml   your-repo/.github/workflows/
cp workflows/docker-build-push.yml your-repo/.github/workflows/
```

---

## Pipeline example — full IoT stack

```
Push .ino sketch          Push Python code            Push to main
        │                        │                          │
        ▼                        ▼                          ▼
arduino-compile.yml       python-ci.yml          docker-build-push.yml
  ├─ Uno                    ├─ ruff                  ├─ build amd64
  ├─ Mega                   ├─ mypy                  ├─ build arm64 (Jetson)
  └─ Leonardo               └─ pytest + cov          └─ push to GHCR
                                                              │
                                                              ▼
                                                      deploy-jetson.yml
                                                        └─ SSH → pull → restart
```

---

## Secrets required for deployment

Add these in **Settings → Secrets → Actions**:

| Secret | Value |
|--------|-------|
| `JETSON_HOST` | IP or hostname of your Jetson device |
| `JETSON_USER` | SSH username (e.g. `jetson`) |
| `JETSON_SSH_KEY` | Contents of `~/.ssh/id_rsa` on the deploy machine |

---

## Notes

- `docker-build-push.yml` builds for `linux/arm64` as well as `amd64` — the arm64 image runs natively on Jetson Orin Nano
- `arduino-compile.yml` uses `arduino-cli` — add more `arduino-cli lib install` lines for project-specific libraries
- `python-ci.yml` runs on Python 3.11 and 3.12 in parallel — remove one if you only target one version

---

## License

MIT — see [LICENSE](LICENSE)

---

<div align="center">

Built by [Kiril Kirilov Kirilov](https://github.com/kirilov07) — Embedded Systems & AI Engineer

</div>
