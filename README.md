# Harbor RL Submission – Vikash Raj Purohit (@dushyant-byte)

## 1. Loom Walkthrough Video
- **Video URL:** [Watch 5-Minute Walkthrough](https://www.loom.com/share/YOUR_VIDEO_ID)
*(Make sure link sharing is set to "Anyone with the link can view")*

## 2. Task & Calibration Summary

- **Bug 1 (Easy):** Missing CLI argument check → `IndexError` on `sys.argv[1]` when no file is provided. Also no usage message.
- **Bug 2 (Medium):** Status filter incorrectly treats `PENDING` as valid. Negative / non-numeric amounts are only partially guarded.
- **Bug 3 (Hard):** Deduplication keeps the *first* occurrence of a `tx_id` instead of the one with the latest (timezone-aware) timestamp. Timezone conversion is incomplete.

- **Calibration Scores:**
  - `nop` = `0.00`
  - `oracle` (`solve.sh`) = `1.00`
  - Partial oracle (Bug 1+2 fixed, Bug 3 left broken) ≈ `0.55–0.70` (depending on weights)

## 3. Ablation & RL Advantage Table (`terminus-2`, k=3 Trials)

| Run | Change Tested | Trial 1 (Binary / Dense) | Trial 2 (Binary / Dense) | Trial 3 (Binary / Dense) | Mean Dense | Notes |
|-----|---------------|---------------------------|---------------------------|---------------------------|------------|-------|
| **Run 1: Baseline** | Original 3-bug task | `0 / 0.XX` | `0 / 0.XX` | `0 / 0.XX` | `0.XX` | At least one failure required |
| **Run 2: Ablation** | Describe the single controlled change you made | `...` | `...` | `...` | `...` | |

*(Fill after you run the 3+3 trials on your machine)*

**Advantage calculation (example for one trial):**
```
Binary Advantage  = Trial Binary Reward − Group Mean Binary
Dense Advantage   = Trial Dense Reward  − Group Mean Dense
```

## 4. Trajectory Forensics (`trajectory.json`)

- **Where the Agent Failed in Run 1:**  
  (After inspecting a failed trajectory) The agent typically fixed the CLI check and the PENDING status filter, then either stopped or failed on the latest-timestamp deduplication + timezone logic.

- **Ablation Hypothesis & Result:**  
  (Example) Adding a sample input CSV inside `/app` + a short note in `instruction.md` that “latest timestamp wins after converting to UTC” improved pass rate from X/3 to Y/3 because the agent could now self-test the edge case.

## 5. Dense Reward & Anti-Cheat Design

- **My Dense Reward Weights (`tests/test.sh`):**
  - CLI usage check ........ 0.15
  - Test A (status filter) . 0.25
  - Test B (dedup) ......... 0.30
  - Test C (TZ + malformed)  0.30
  - Final score multiplied by anti-cheat factor (0 or 1)

- **How `solution/hack.sh` Cheated & How I Blocked It:**  
  The hack overwrites `reconcile.py` with a stub that always prints a fixed JSON and ignores the CSV.  
  The verifier blocks it by:
  1. Checking that the source still contains `open(`/`csv`/`DictReader`
  2. Rejecting any source that hard-codes the exact expected totals (125.5, 30.0, 80.0)
  3. Running three different CSVs that produce three different correct answers

- **Remaining Reward-Hacking Risk:**  
  A smarter agent could still implement a minimal parser that only handles the three exact test files by content hash, or could write a solution that looks at `/tests` if the verifier files were somehow visible. Current hardening makes the simple hard-code and the “always return 1.0” attacks fail.

## 6. AI Tool Workflow

- **AI Tool & Model Used:** e.g. Claude Code / Cursor / Gemini CLI + gemini-2.5-flash or groq/...
- **How I Avoided Prompt Leakage:**  
  I never pasted the full assignment document into the same chat that was editing `reconcile.py`.  
  The agent that wrote the environment and tests never saw the exact bug locations written in the instruction.  
  Instruction.md only describes the *desired behaviour*, never which lines are broken.

---

### Quick Local Sanity Commands (run on your machine)

```bash
# After Harbor is installed and Docker is running
cd ~/harbor-rl-canary-<your-github-username>

# Must score ~0.00
harbor run -p ./tasks/reconcile-bug -a nop

# Must score 1.00
harbor run -p ./tasks/reconcile-bug -a oracle

# Baseline (3 trials)
harbor run -p ./tasks/reconcile-bug \
  -a terminus-2 \
  --model gemini/gemini-2.5-flash \
  -k 3 -n 1
```

---

## Submission Checklist (for @dushyant-byte)

1. Rename this folder to: `harbor-rl-canary-dushyant-byte`
2. Run the local sanity checks (`nop` → 0.00, `oracle` → 1.00)
3. Run Baseline (3 trials) + Ablation (3 trials) and fill the table above
4. Record the 5-minute Loom video and paste the link
5. Export AI chat transcripts into `ai_transcripts/`
6. Copy `result.json` + `trajectory.json` files into `jobs_summary/`
7. Create GitHub repo **exactly** named: `harbor-rl-canary-dushyant-byte`
8. Push:

```bash
cd harbor-rl-canary-dushyant-byte
rm -rf jobs/ .env .venv   # never push these
git init
git add .
git commit -m "Submit 2-day Harbor RL assignment - Vikash Raj Purohit"
git branch -M main
git remote add origin https://github.com/dushyant-byte/harbor-rl-canary-dushyant-byte.git
git push -u origin main
```

9. Send the repository link.
