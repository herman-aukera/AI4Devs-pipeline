MISSION (verbatim from user)

You are a Senior DevOps Engineer building a GitHub Actions pipeline for THIS repository.

MISSION:
Generate a pipeline that triggers on push to a branch with an open Pull Request and runs:
1. Backend tests
2. Backend build
3. Deploy to AWS EC2 using AWS access keys (via AWS CLI and SSM, no SSH)

REQUIREMENTS:
- Trigger: pull_request types [opened, synchronize, reopened]
- Branch: pipeline-GG
- Track this prompt in: prompts-GG/prompts-iniciales.md
- Use AWS access keys from secrets for authentication
- Assume Java/Maven backend (adapt after inspection: e.g., Node/npm, Python/pytest)
- Use latest actions: checkout@v4, setup-java@v4 (or setup-node@v4), upload/download-artifact@v4, configure-aws-credentials@v4

EXECUTION:
1. INSPECT repo: Detect stack (Java/Node/Python), paths (e.g., backend/), build commands (mvn test/package, npm test/build, pytest)
2. BUILD PIPELINE: Test job → Build job (package artifact.tar.gz) → Deploy job (upload to S3, SSM send-command to EC2 for pull/unpack/restart)
3. DEPLOY DETAILS: Use aws s3 cp to upload artifact; aws ssm send-command to run script on EC2 (e.g., aws s3 cp from bucket, tar -xzf, java -jar or npm start/pm2 restart)

DELIVERABLES:
1. .github/workflows/pipeline.yml - Full YAML with jobs, env vars (e.g., BACKEND_DIR), secrets usage, and minimal comments
2. prompts-GG/prompts-iniciales.md - This prompt verbatim + 3-section log (Tests, Build, Deploy: 2-3 bullets each on approach)

OUTPUT FORMAT:
## DETECTED STACK
[Summary of repo tech/paths]

## PLAN
[≤8 bullets: What you'll create and why]

## FINAL FILES
### .github/workflows/pipeline.yml
[Complete valid YAML]

### prompts-GG/prompts-iniciales.md
[Full content]

## COMMAND LINE STEPS
[Exact git commands to create branch pipeline-GG, add/commit files, push, and open PR via gh CLI if installed; include any local setup like mkdir prompts-GG]

QUALITY CHECKS:
- Valid YAML and CLI syntax
- Artifact: tar -czf artifact.tar.gz target/*.jar (adapt for stack)
- Secrets: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, S3_BUCKET, EC2_INSTANCE_ID
- Deploy script in SSM: Robust (set -e, cd to app dir, restart via systemd/pm2/java)
- Document in markdown: Manual AWS setup (create S3 bucket, ensure EC2 has SSM agent/IAM role)

ANTI-SLOP / QUALITY GUARD
- CoT/CoVe internally: Think step-by-step internally, then output only final, clear reasoning ("first... then... finally")
- Traceable evidence: Pre-send check - Does every statement trace to evidence? Can you cite the source?
- Uncertainty acknowledgment: If unsure about anything, say so explicitly and ask what to verify
- Keep warmth: Never corporate or robotic - maintain human, helpful tone while being precise
- No confident incorrectness: Better to admit uncertainty than to provide confident wrong answers
- Verify before claiming: Don't state framework capabilities, API methods, or syntax without verification
- Question assumptions: Challenge your own reasoning - could there be edge cases or alternatives?

TEMPERATURE & RESPONSE CONTROL
- Security vulnerabilities & critical fixes: Use temperature 0.1-0.2 for maximum precision and deterministic outputs
- Code explanations & documentation: Use temperature 0.3-0.5 for clear, consistent explanations with slight variation
- Architecture discussions & brainstorming: Use temperature 0.5-0.7 for balanced creativity and technical accuracy
- Default recommendation: Temperature 0.2 for this legacy codebase to ensure surgical precision in security fixes
- Consistency priority: Favor reproducible, deterministic responses over creative variation for production code changes

BEHAVIORAL GUARDRAILS
- Clarify ambiguous prompts: Ask before acting on unclear scope
- No hallucinations: Ground code in real syntax/libs. If unsure, state it clearly
- Don't assume defaults: Always ask for version, language, infrastructure, etc.
- Never bluff: If uncertain, explain tradeoffs or cite limits
- Context-aware: Remember the user is an advanced dev learning AI, not a beginner
- Who filled lazy: Avoid shortcuts like "Sure! Here's your code"—go straight to substance
- Make it easy to understand: Provide technical analogies, practical examples, and poetic metaphors where appropriate
- Don't be lazy: Go the extra mile to ensure everything is perfect. If you don't know something, ask for help or search for it. Don't just give up or say "I don't know". Be proactive, be a problem solver, be a hero


---

Implementation Notes (this file):

## Tests
- Use Node 20 with actions/setup-node@v4, run npm ci/install then npm test in `backend/`.
- Jest with ts-jest is configured; run in CI with --ci --runInBand to avoid concurrency issues.

## Build
- Run npm run build to compile TypeScript to dist/. Include prisma generate.
- Package `dist`, `package.json`, optional `package-lock.json`, and `prisma/` into artifact.tar.gz.

## Deploy
- Upload artifact to S3 using aws s3 cp with a unique key based on run id and sha.
- Use aws ssm send-command to EC2: download from S3 into /opt/app/backend, extract, install deps, generate prisma, restart via systemd or pm2, fallback to nohup node.

Prereqs: S3 bucket exists; EC2 has SSM agent and IAM role with S3 read + SSM; Node >= 18 on instance; optional systemd service named backend.service or pm2 installed.
