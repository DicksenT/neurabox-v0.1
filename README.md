# NeuraBox

**An airlock for AI-generated code.**

AI coding agents (Cursor, Devin, Copilot) write code fast. Reviewing it safely is slow, and by the time you see the PR, the code is already in your repo. Scanners flag problems. Bots leave comments. Nothing actually stops non-compliant code from landing.

NeuraBox sits *before* the code touches your project. It runs the AI's output in an isolated Docker sandbox, enforces your policy, and only exports code that passes — with a full audit trail.


> **Not a scanner. Not a reviewer. A gate.**

---

## Demo

https://github.com/user-attachments/assets/582e8da2-51ed-4194-bd5e-7cdd850f2a7e

---

## Quickstart (Windows / CLI)

### 1. Download

Get the latest binary from [Releases] or run
```bash
npm install -g neurabox
```

### 2. Initialize a policy
```bash
neurabox --init
```
This creates `nb-policy.yaml` with safe defaults. Edit it to fit your project.

### 3. Add your AI provider

create `.env` and add the necessary variable:

```bash
API_KEY= ""
AI_BASE_URL= "https://api.deepseek.com"
AI_MODEL= "deepseek-chat"
```
NeuraBox supports any OpenAI-compatible API.

### 4. Run NeuraBox
```bash
neurabox "add a login route with basic validation"
```
NeuraBox will:

- Copy your project into an isolated shadow directory
- Spin up a Docker sandbox (network disabled)
- Send your prompt to the AI and apply the output inside the sandbox
- Run your policy checks (ESLint, custom commands, structure checks)
- Show a git diff and ask for approval
- Only then export the changes to your real project
- Generate an audit log automatically

## Default policy example

The default `nb-policy.yaml` looks like this — customize it for your stack:

```yaml
version: "0.1"
image: "node:20-alpine"       # Docker image for sandbox

mounts:
  - source: "."
    target: "/workspace"
    mode: "ro"
  - source: "./src"
    target: "/workspace/src"
    mode: "rw"

blocks:
  - ".env"
  - "node_modules"
  - ".git"
  - "secret.json"

checks:
  - cname: "structure"
    command: "[ -d 'src/controllers' ] && [ -d 'src/routes' ]"
  - cname: "no-internet-leak"
    command: "curl -m 2 google.com || echo 'Safe: No internet'"
  - cname: "performance-check"
    command: "timeout 5s yes > /dev/null || echo 'Safe: CPU limit hit'"
  - cname: "system-modification"
    command: "touch /usr/bin/virus || echo 'Safe: no files created'"
```

- `image` – the Docker image used for the sandbox
- `ai` – provider credentials (DeepSeek, OpenAI, or any compatible endpoint)
- `mounts` – folders exposed to the sandbox (`ro` = read-only, `rw` = writable)
- `blocks` – files that will never be exported (e.g., `.env`)
- `checks` – shell commands that must pass. If any check fails, the code is blocked.


## How is this different?

**Git worktree/branches** — No policy enforcement, no approval gate, 
no audit trail. Delete the worktree and the history is gone.

**Hooks/scripts** — The agent governing itself. NeuraBox is external 
— the agent has no say in whether its output passes.

**CodeRabbit/PR bots** — Advisory only, after code lands in repo. 
NeuraBox blocks before code exists in your codebase.

**Snyk/SonarQube** — Finds vulnerabilities in code that already exists. 
NeuraBox decides whether code is allowed to exist at all.

## Current limitations (early beta)

- Windows / CLI only
- Tested best with Node.js / JavaScript projects
- Uses Docker 
- No team features yet — single-user, local machine

I'm actively hardening the isolation model and plan to move to true hardware-level sandboxing.

## What next?

- harden up the sandbox, may change to firecracker
- Agents integration
- encrypt audit
- cloud-based

## Privacy

The binary sends no code, prompts or data to any server besides the AI provider you configure. 

## Feedback & early access

This is a private beta — it works, but it's scrappy.

I'd love your honest feedback, either here in GitHub issues or by email: `tandicksen@gmail.com`

If you’re using AI coding agents (Cursor, Copilot, Devin) and want to help shape the tool, reach out. I'm especially interested in:

- What policies would you actually enforce?
- Is network-disabled Docker enough, or do you need hardware-level isolation?
- What’s missing before you’d use this on a real project?
