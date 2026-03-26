Hi please improve the design by keeping all original feature and adding new feature that user can paste 510(k) submission summary, review note or 510(k) review guidance (txt, markdown), then agent will transform doc into organized document (please keep all original information) with 5 table and 20 entities with context in markdown in 2000~3000 words. User can modify the doc in markdown or text view. Then user can paste template of 510(k) review guidance or description of 510(k) review guidance (text or markdown) or choose using the default review guidance template. Then agent will transform previous doc into 510(k) review guidance based on the template or description with  5 tables, 20 entites and review checklist in markdown in 3000~4000 words. User can modify the results in markdown  or text view or keep prompt on the results (user can choose models). Then please create a description of skill (skill.md) of the preview review guidance (that will be used by agent to review 510(k) submission) by using the skill creator skill. User can modify the description of skill in markdown. Description of skill creator:---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---

# Skill Creator

A skill for creating new skills and iteratively improving them.

At a high level, the process of creating a skill goes like this:

- Decide what you want the skill to do and roughly how it should do it
- Write a draft of the skill
- Create a few test prompts and run claude-with-access-to-the-skill on them
- Help the user evaluate the results both qualitatively and quantitatively
  - While the runs happen in the background, draft some quantitative evals if there aren't any (if there are some, you can either use as is or modify if you feel something needs to change about them). Then explain them to the user (or if they already existed, explain the ones that already exist)
  - Use the `eval-viewer/generate_review.py` script to show the user the results for them to look at, and also let them look at the quantitative metrics
- Rewrite the skill based on feedback from the user's evaluation of the results (and also if there are any glaring flaws that become apparent from the quantitative benchmarks)
- Repeat until you're satisfied
- Expand the test set and try again at larger scale

Your job when using this skill is to figure out where the user is in this process and then jump in and help them progress through these stages. So for instance, maybe they're like "I want to make a skill for X". You can help narrow down what they mean, write a draft, write the test cases, figure out how they want to evaluate, run all the prompts, and repeat.

On the other hand, maybe they already have a draft of the skill. In this case you can go straight to the eval/iterate part of the loop.

Of course, you should always be flexible and if the user is like "I don't need to run a bunch of evaluations, just vibe with me", you can do that instead.

Then after the skill is done (but again, the order is flexible), you can also run the skill description improver, which we have a whole separate script for, to optimize the triggering of the skill.

Cool? Cool.

## Communicating with the user

The skill creator is liable to be used by people across a wide range of familiarity with coding jargon. If you haven't heard (and how could you, it's only very recently that it started), there's a trend now where the power of Claude is inspiring plumbers to open up their terminals, parents and grandparents to google "how to install npm". On the other hand, the bulk of users are probably fairly computer-literate.

So please pay attention to context cues to understand how to phrase your communication! In the default case, just to give you some idea:

- "evaluation" and "benchmark" are borderline, but OK
- for "JSON" and "assertion" you want to see serious cues from the user that they know what those things are before using them without explaining them

It's OK to briefly explain terms if you're in doubt, and feel free to clarify terms with a short definition if you're unsure if the user will get it.

---

## Creating a skill

### Capture Intent

Start by understanding the user's intent. The current conversation might already contain a workflow the user wants to capture (e.g., they say "turn this into a skill"). If so, extract answers from the conversation history first — the tools used, the sequence of steps, corrections the user made, input/output formats observed. The user may need to fill the gaps, and should confirm before proceeding to the next step.

1. What should this skill enable Claude to do?
2. When should this skill trigger? (what user phrases/contexts)
3. What's the expected output format?
4. Should we set up test cases to verify the skill works? Skills with objectively verifiable outputs (file transforms, data extraction, code generation, fixed workflow steps) benefit from test cases. Skills with subjective outputs (writing style, art) often don't need them. Suggest the appropriate default based on the skill type, but let the user decide.

### Interview and Research

Proactively ask questions about edge cases, input/output formats, example files, success criteria, and dependencies. Wait to write test prompts until you've got this part ironed out.

Check available MCPs - if useful for research (searching docs, finding similar skills, looking up best practices), research in parallel via subagents if available, otherwise inline. Come prepared with context to reduce burden on the user.

### Write the SKILL.md

Based on the user interview, fill in these components:

- **name**: Skill identifier
- **description**: When to trigger, what it does. This is the primary triggering mechanism - include both what the skill does AND specific contexts for when to use it. All "when to use" info goes here, not in the body. Note: currently Claude has a tendency to "undertrigger" skills -- to not use them when they'd be useful. To combat this, please make the skill descriptions a little bit "pushy". So for instance, instead of "How to build a simple fast dashboard to display internal Anthropic data.", you might write "How to build a simple fast dashboard to display internal Anthropic data. Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"
- **compatibility**: Required tools, dependencies (optional, rarely needed)
- **the rest of the skill :)**

### Skill Writing Guide

#### Anatomy of a Skill

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for deterministic/repetitive tasks
    ├── references/ - Docs loaded into context as needed
    └── assets/     - Files used in output (templates, icons, fonts)
```

#### Progressive Disclosure

Skills use a three-level loading system:
1. **Metadata** (name + description) - Always in context (~100 words)
2. **SKILL.md body** - In context whenever skill triggers (<500 lines ideal)
3. **Bundled resources** - As needed (unlimited, scripts can execute without loading)

These word counts are approximate and you can feel free to go longer if needed.

**Key patterns:**
- Keep SKILL.md under 500 lines; if you're approaching this limit, add an additional layer of hierarchy along with clear pointers about where the model using the skill should go next to follow up.
- Reference files clearly from SKILL.md with guidance on when to read them
- For large reference files (>300 lines), include a table of contents

**Domain organization**: When a skill supports multiple domains/frameworks, organize by variant:
```
cloud-deploy/
├── SKILL.md (workflow + selection)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```
Claude reads only the relevant reference file.

#### Principle of Lack of Surprise

This goes without saying, but skills must not contain malware, exploit code, or any content that could compromise system security. A skill's contents should not surprise the user in their intent if described. Don't go along with requests to create misleading skills or skills designed to facilitate unauthorized access, data exfiltration, or other malicious activities. Things like a "roleplay as an XYZ" are OK though.

#### Writing Patterns

Prefer using the imperative form in instructions.

**Defining output formats** - You can do it like this:
```markdown
## Report structure
ALWAYS use this exact template:
# [Title]
## Executive summary
## Key findings
## Recommendations
```

**Examples pattern** - It's useful to include examples. You can format them like this (but if "Input" and "Output" are in the examples you might want to deviate a little):
```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### Writing Style

Try to explain to the model why things are important in lieu of heavy-handed musty MUSTs. Use theory of mind and try to make the skill general and not super-narrow to specific examples. Start by writing a draft and then look at it with fresh eyes and improve it.

### Test Cases

After writing the skill draft, come up with 2-3 realistic test prompts — the kind of thing a real user would actually say. Share them with the user: [you don't have to use this exact language] "Here are a few test cases I'd like to try. Do these look right, or do you want to add more?" Then run them.

Save test cases to `evals/evals.json`. Don't write assertions yet — just the prompts. You'll draft assertions in the next step while the runs are in progress.

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

See `references/schemas.md` for the full schema (including the `assertions` field, which you'll add later).

## Running and evaluating test cases

This section is one continuous sequence — don't stop partway through. Do NOT use `/skill-test` or any other testing skill.

Put results in `<skill-name>-workspace/` as a sibling to the skill directory. Within the workspace, organize results by iteration (`iteration-1/`, `iteration-2/`, etc.) and within that, each test case gets a directory (`eval-0/`, `eval-1/`, etc.). Don't create all of this upfront — just create directories as you go.

### Step 1: Spawn all runs (with-skill AND baseline) in the same turn

For each test case, spawn two subagents in the same turn — one with the skill, one without. This is important: don't spawn the with-skill runs first and then come back for baselines later. Launch everything at once so it all finishes around the same time.

**With-skill run:**

```
Execute this task:
- Skill path: <path-to-skill>
- Task: <eval prompt>
- Input files: <eval files if any, or "none">
- Save outputs to: <workspace>/iteration-<N>/eval-<ID>/with_skill/outputs/
- Outputs to save: <what the user cares about — e.g., "the .docx file", "the final CSV">
```

**Baseline run** (same prompt, but the baseline depends on context):
- **Creating a new skill**: no skill at all. Same prompt, no skill path, save to `without_skill/outputs/`.
- **Improving an existing skill**: the old version. Before editing, snapshot the skill (`cp -r <skill-path> <workspace>/skill-snapshot/`), then point the baseline subagent at the snapshot. Save to `old_skill/outputs/`.

Write an `eval_metadata.json` for each test case (assertions can be empty for now). Give each eval a descriptive name based on what it's testing — not just "eval-0". Use this name for the directory too. If this iteration uses new or modified eval prompts, create these files for each new eval directory — don't assume they carry over from previous iterations.

```json
{
  "eval_id": 0,
  "eval_name": "descriptive-name-here",
  "prompt": "The user's task prompt",
  "assertions": []
}
```

### Step 2: While runs are in progress, draft assertions

Don't just wait for the runs to finish — you can use this time productively. Draft quantitative assertions for each test case and explain them to the user. If assertions already exist in `evals/evals.json`, review them and explain what they check.

Good assertions are objectively verifiable and have descriptive names — they should read clearly in the benchmark viewer so someone glancing at the results immediately understands what each one checks. Subjective skills (writing style, design quality) are better evaluated qualitatively — don't force assertions onto things that need human judgment.

Update the `eval_metadata.json` files and `evals/evals.json` with the assertions once drafted. Also explain to the user what they'll see in the viewer — both the qualitative outputs and the quantitative benchmark.

### Step 3: As runs complete, capture timing data

When each subagent task completes, you receive a notification containing `total_tokens` and `duration_ms`. Save this data immediately to `timing.json` in the run directory:

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

This is the only opportunity to capture this data — it comes through the task notification and isn't persisted elsewhere. Process each notification as it arrives rather than trying to batch them.

### Step 4: Grade, aggregate, and launch the viewer

Once all runs are done:

1. **Grade each run** — spawn a grader subagent (or grade inline) that reads `agents/grader.md` and evaluates each assertion against the outputs. Save results to `grading.json` in each run directory. The grading.json expectations array must use the fields `text`, `passed`, and `evidence` (not `name`/`met`/`details` or other variants) — the viewer depends on these exact field names. For assertions that can be checked programmatically, write and run a script rather than eyeballing it — scripts are faster, more reliable, and can be reused across iterations.

2. **Aggregate into benchmark** — run the aggregation script from the skill-creator directory:
   ```bash
   python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
   ```
   This produces `benchmark.json` and `benchmark.md` with pass_rate, time, and tokens for each configuration, with mean ± stddev and the delta. If generating benchmark.json manually, see `references/schemas.md` for the exact schema the viewer expects.
Put each with_skill version before its baseline counterpart.

3. **Do an analyst pass** — read the benchmark data and surface patterns the aggregate stats might hide. See `agents/analyzer.md` (the "Analyzing Benchmark Results" section) for what to look for — things like assertions that always pass regardless of skill (non-discriminating), high-variance evals (possibly flaky), and time/token tradeoffs.

4. **Launch the viewer** with both qualitative outputs and quantitative data:
   ```bash
   nohup python <skill-creator-path>/eval-viewer/generate_review.py \
     <workspace>/iteration-N \
     --skill-name "my-skill" \
     --benchmark <workspace>/iteration-N/benchmark.json \
     > /dev/null 2>&1 &
   VIEWER_PID=$!
   ```
   For iteration 2+, also pass `--previous-workspace <workspace>/iteration-<N-1>`.

   **Cowork / headless environments:** If `webbrowser.open()` is not available or the environment has no display, use `--static <output_path>` to write a standalone HTML file instead of starting a server. Feedback will be downloaded as a `feedback.json` file when the user clicks "Submit All Reviews". After download, copy `feedback.json` into the workspace directory for the next iteration to pick up.

Note: please use generate_review.py to create the viewer; there's no need to write custom HTML.

5. **Tell the user** something like: "I've opened the results in your browser. There are two tabs — 'Outputs' lets you click through each test case and leave feedback, 'Benchmark' shows the quantitative comparison. When you're done, come back here and let me know."

### What the user sees in the viewer

The "Outputs" tab shows one test case at a time:
- **Prompt**: the task that was given
- **Output**: the files the skill produced, rendered inline where possible
- **Previous Output** (iteration 2+): collapsed section showing last iteration's output
- **Formal Grades** (if grading was run): collapsed section showing assertion pass/fail
- **Feedback**: a textbox that auto-saves as they type
- **Previous Feedback** (iteration 2+): their comments from last time, shown below the textbox

The "Benchmark" tab shows the stats summary: pass rates, timing, and token usage for each configuration, with per-eval breakdowns and analyst observations.

Navigation is via prev/next buttons or arrow keys. When done, they click "Submit All Reviews" which saves all feedback to `feedback.json`.

### Step 5: Read the feedback

When the user tells you they're done, read `feedback.json`:

```json
{
  "reviews": [
    {"run_id": "eval-0-with_skill", "feedback": "the chart is missing axis labels", "timestamp": "..."},
    {"run_id": "eval-1-with_skill", "feedback": "", "timestamp": "..."},
    {"run_id": "eval-2-with_skill", "feedback": "perfect, love this", "timestamp": "..."}
  ],
  "status": "complete"
}
```

Empty feedback means the user thought it was fine. Focus your improvements on the test cases where the user had specific complaints.

Kill the viewer server when you're done with it:

```bash
kill $VIEWER_PID 2>/dev/null
```

---

## Improving the skill

This is the heart of the loop. You've run the test cases, the user has reviewed the results, and now you need to make the skill better based on their feedback.

### How to think about improvements

1. **Generalize from the feedback.** The big picture thing that's happening here is that we're trying to create skills that can be used a million times (maybe literally, maybe even more who knows) across many different prompts. Here you and the user are iterating on only a few examples over and over again because it helps move faster. The user knows these examples in and out and it's quick for them to assess new outputs. But if the skill you and the user are codeveloping works only for those examples, it's useless. Rather than put in fiddly overfitty changes, or oppressively constrictive MUSTs, if there's some stubborn issue, you might try branching out and using different metaphors, or recommending different patterns of working. It's relatively cheap to try and maybe you'll land on something great.

2. **Keep the prompt lean.** Remove things that aren't pulling their weight. Make sure to read the transcripts, not just the final outputs — if it looks like the skill is making the model waste a bunch of time doing things that are unproductive, you can try getting rid of the parts of the skill that are making it do that and seeing what happens.

3. **Explain the why.** Try hard to explain the **why** behind everything you're asking the model to do. Today's LLMs are *smart*. They have good theory of mind and when given a good harness can go beyond rote instructions and really make things happen. Even if the feedback from the user is terse or frustrated, try to actually understand the task and why the user is writing what they wrote, and what they actually wrote, and then transmit this understanding into the instructions. If you find yourself writing ALWAYS or NEVER in all caps, or using super rigid structures, that's a yellow flag — if possible, reframe and explain the reasoning so that the model understands why the thing you're asking for is important. That's a more humane, powerful, and effective approach.

4. **Look for repeated work across test cases.** Read the transcripts from the test runs and notice if the subagents all independently wrote similar helper scripts or took the same multi-step approach to something. If all 3 test cases resulted in the subagent writing a `create_docx.py` or a `build_chart.py`, that's a strong signal the skill should bundle that script. Write it once, put it in `scripts/`, and tell the skill to use it. This saves every future invocation from reinventing the wheel.

This task is pretty important (we are trying to create billions a year in economic value here!) and your thinking time is not the blocker; take your time and really mull things over. I'd suggest writing a draft revision and then looking at it anew and making improvements. Really do your best to get into the head of the user and understand what they want and need.

### The iteration loop

After improving the skill:

1. Apply your improvements to the skill
2. Rerun all test cases into a new `iteration-<N+1>/` directory, including baseline runs. If you're creating a new skill, the baseline is always `without_skill` (no skill) — that stays the same across iterations. If you're improving an existing skill, use your judgment on what makes sense as the baseline: the original version the user came in with, or the previous iteration.
3. Launch the reviewer with `--previous-workspace` pointing at the previous iteration
4. Wait for the user to review and tell you they're done
5. Read the new feedback, improve again, repeat

Keep going until:
- The user says they're happy
- The feedback is all empty (everything looks good)
- You're not making meaningful progress

---

## Advanced: Blind comparison

For situations where you want a more rigorous comparison between two versions of a skill (e.g., the user asks "is the new version actually better?"), there's a blind comparison system. Read `agents/comparator.md` and `agents/analyzer.md` for the details. The basic idea is: give two outputs to an independent agent without telling it which is which, and let it judge quality. Then analyze why the winner won.

This is optional, requires subagents, and most users won't need it. The human review loop is usually sufficient.

---

## Description Optimization

The description field in SKILL.md frontmatter is the primary mechanism that determines whether Claude invokes a skill. After creating or improving a skill, offer to optimize the description for better triggering accuracy.

### Step 1: Generate trigger eval queries

Create 20 eval queries — a mix of should-trigger and should-not-trigger. Save as JSON:

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

The queries must be realistic and something a Claude Code or Claude.ai user would actually type. Not abstract requests, but requests that are concrete and specific and have a good amount of detail. For instance, file paths, personal context about the user's job or situation, column names and values, company names, URLs. A little bit of backstory. Some might be in lowercase or contain abbreviations or typos or casual speech. Use a mix of different lengths, and focus on edge cases rather than making them clear-cut (the user will get a chance to sign off on them).

Bad: `"Format this data"`, `"Extract text from PDF"`, `"Create a chart"`

Good: `"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"`

For the **should-trigger** queries (8-10), think about coverage. You want different phrasings of the same intent — some formal, some casual. Include cases where the user doesn't explicitly name the skill or file type but clearly needs it. Throw in some uncommon use cases and cases where this skill competes with another but should win.

For the **should-not-trigger** queries (8-10), the most valuable ones are the near-misses — queries that share keywords or concepts with the skill but actually need something different. Think adjacent domains, ambiguous phrasing where a naive keyword match would trigger but shouldn't, and cases where the query touches on something the skill does but in a context where another tool is more appropriate.

The key thing to avoid: don't make should-not-trigger queries obviously irrelevant. "Write a fibonacci function" as a negative test for a PDF skill is too easy — it doesn't test anything. The negative cases should be genuinely tricky.

### Step 2: Review with user

Present the eval set to the user for review using the HTML template:

1. Read the template from `assets/eval_review.html`
2. Replace the placeholders:
   - `__EVAL_DATA_PLACEHOLDER__` → the JSON array of eval items (no quotes around it — it's a JS variable assignment)
   - `__SKILL_NAME_PLACEHOLDER__` → the skill's name
   - `__SKILL_DESCRIPTION_PLACEHOLDER__` → the skill's current description
3. Write to a temp file (e.g., `/tmp/eval_review_<skill-name>.html`) and open it: `open /tmp/eval_review_<skill-name>.html`
4. The user can edit queries, toggle should-trigger, add/remove entries, then click "Export Eval Set"
5. The file downloads to `~/Downloads/eval_set.json` — check the Downloads folder for the most recent version in case there are multiple (e.g., `eval_set (1).json`)

This step matters — bad eval queries lead to bad descriptions.

### Step 3: Run the optimization loop

Tell the user: "This will take some time — I'll run the optimization loop in the background and check on it periodically."

Save the eval set to the workspace, then run in the background:

```bash
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model <model-id-powering-this-session> \
  --max-iterations 5 \
  --verbose
```

Use the model ID from your system prompt (the one powering the current session) so the triggering test matches what the user actually experiences.

While it runs, periodically tail the output to give the user updates on which iteration it's on and what the scores look like.

This handles the full optimization loop automatically. It splits the eval set into 60% train and 40% held-out test, evaluates the current description (running each query 3 times to get a reliable trigger rate), then calls Claude to propose improvements based on what failed. It re-evaluates each new description on both train and test, iterating up to 5 times. When it's done, it opens an HTML report in the browser showing the results per iteration and returns JSON with `best_description` — selected by test score rather than train score to avoid overfitting.

### How skill triggering works

Understanding the triggering mechanism helps design better eval queries. Skills appear in Claude's `available_skills` list with their name + description, and Claude decides whether to consult a skill based on that description. The important thing to know is that Claude only consults skills for tasks it can't easily handle on its own — simple, one-step queries like "read this PDF" may not trigger a skill even if the description matches perfectly, because Claude can handle them directly with basic tools. Complex, multi-step, or specialized queries reliably trigger skills when the description matches.

This means your eval queries should be substantive enough that Claude would actually benefit from consulting a skill. Simple queries like "read file X" are poor test cases — they won't trigger skills regardless of description quality.

### Step 4: Apply the result

Take `best_description` from the JSON output and update the skill's SKILL.md frontmatter. Show the user before/after and report the scores.

---

### Package and Present (only if `present_files` tool is available)

Check whether you have access to the `present_files` tool. If you don't, skip this step. If you do, package the skill and present the .skill file to the user:

```bash
python -m scripts.package_skill <path/to/skill-folder>
```

After packaging, direct the user to the resulting `.skill` file path so they can install it.

---

## Claude.ai-specific instructions

In Claude.ai, the core workflow is the same (draft → test → review → improve → repeat), but because Claude.ai doesn't have subagents, some mechanics change. Here's what to adapt:

**Running test cases**: No subagents means no parallel execution. For each test case, read the skill's SKILL.md, then follow its instructions to accomplish the test prompt yourself. Do them one at a time. This is less rigorous than independent subagents (you wrote the skill and you're also running it, so you have full context), but it's a useful sanity check — and the human review step compensates. Skip the baseline runs — just use the skill to complete the task as requested.

**Reviewing results**: If you can't open a browser (e.g., Claude.ai's VM has no display, or you're on a remote server), skip the browser reviewer entirely. Instead, present results directly in the conversation. For each test case, show the prompt and the output. If the output is a file the user needs to see (like a .docx or .xlsx), save it to the filesystem and tell them where it is so they can download and inspect it. Ask for feedback inline: "How does this look? Anything you'd change?"

**Benchmarking**: Skip the quantitative benchmarking — it relies on baseline comparisons which aren't meaningful without subagents. Focus on qualitative feedback from the user.

**The iteration loop**: Same as before — improve the skill, rerun the test cases, ask for feedback — just without the browser reviewer in the middle. You can still organize results into iteration directories on the filesystem if you have one.

**Description optimization**: This section requires the `claude` CLI tool (specifically `claude -p`) which is only available in Claude Code. Skip it if you're on Claude.ai.

**Blind comparison**: Requires subagents. Skip it.

**Packaging**: The `package_skill.py` script works anywhere with Python and a filesystem. On Claude.ai, you can run it and the user can download the resulting `.skill` file.

**Updating an existing skill**: The user might be asking you to update an existing skill, not create a new one. In this case:
- **Preserve the original name.** Note the skill's directory name and `name` frontmatter field -- use them unchanged. E.g., if the installed skill is `research-helper`, output `research-helper.skill` (not `research-helper-v2`).
- **Copy to a writeable location before editing.** The installed skill path may be read-only. Copy to `/tmp/skill-name/`, edit there, and package from the copy.
- **If packaging manually, stage in `/tmp/` first**, then copy to the output directory -- direct writes may fail due to permissions.

---

## Cowork-Specific Instructions

If you're in Cowork, the main things to know are:

- You have subagents, so the main workflow (spawn test cases in parallel, run baselines, grade, etc.) all works. (However, if you run into severe problems with timeouts, it's OK to run the test prompts in series rather than parallel.)
- You don't have a browser or display, so when generating the eval viewer, use `--static <output_path>` to write a standalone HTML file instead of starting a server. Then proffer a link that the user can click to open the HTML in their browser.
- For whatever reason, the Cowork setup seems to disincline Claude from generating the eval viewer after running the tests, so just to reiterate: whether you're in Cowork or in Claude Code, after running tests, you should always generate the eval viewer for the human to look at examples before revising the skill yourself and trying to make corrections, using `generate_review.py` (not writing your own boutique html code). Sorry in advance but I'm gonna go all caps here: GENERATE THE EVAL VIEWER *BEFORE* evaluating inputs yourself. You want to get them in front of the human ASAP!
- Feedback works differently: since there's no running server, the viewer's "Submit All Reviews" button will download `feedback.json` as a file. You can then read it from there (you may have to request access first).
- Packaging works — `package_skill.py` just needs Python and a filesystem.
- Description optimization (`run_loop.py` / `run_eval.py`) should work in Cowork just fine since it uses `claude -p` via subprocess, not a browser, but please save it until you've fully finished making the skill and the user agrees it's in good shape.
- **Updating an existing skill**: The user might be asking you to update an existing skill, not create a new one. Follow the update guidance in the claude.ai section above.

---

## Reference files

The agents/ directory contains instructions for specialized subagents. Read them when you need to spawn the relevant subagent.

- `agents/grader.md` — How to evaluate assertions against outputs
- `agents/comparator.md` — How to do blind A/B comparison between two outputs
- `agents/analyzer.md` — How to analyze why one version beat another

The references/ directory has additional documentation:
- `references/schemas.md` — JSON structures for evals.json, grading.json, etc.

---

Repeating one more time the core loop here for emphasis:

- Figure out what the skill is about
- Draft or edit the skill
- Run claude-with-access-to-the-skill on test prompts
- With the user, evaluate the outputs:
  - Create benchmark.json and run `eval-viewer/generate_review.py` to help the user review them
  - Run quantitative evals
- Repeat until you and the user are satisfied
- Package the final skill and return it to the user.

Please add steps to your TodoList, if you have such a thing, to make sure you don't forget. If you're in Cowork, please specifically put "Create evals JSON and run `eval-viewer/generate_review.py` so human can review test cases" in your TodoList to make sure it happens.

Good luck!  Default template of 510(k) review guidance:   經皮冠狀動脈擴張術 (PTCA) 藥物塗層球囊 (DCB) 導管審查指引
1. 範疇
本指引適用於用於經皮冠狀動脈擴張術 (PTCA) 的藥物塗層球囊 (DCB) 導管。這些裝置設計用於在球囊擴張過程中，將藥物直接傳遞至冠狀動脈壁，以改善管腔直徑並降低再狹窄的發生率。

1.1 分類與法規狀態
TFDA 分類：E.0005 (經皮冠狀動脈擴張術導管)

風險等級：第三級 (高風險)

排除產品：含有生物藥物成分的組合產品不在本指引範疇內

1.2 基本原則
製造商必須提供完整的驗證數據，涵蓋臨床前測試以確保安全性與有效性。所有測試必須在完成、滅菌的產品或經驗證的等效樣品上進行。若使用非滅菌或未完成的樣品，必須提供充分的科學理由。

2. 審查重點
2.1 技術文件與產品描述
審查人員必須確認中文使用說明書 (IFU/標籤) 包含：

導管與藥物塗層的詳細材料組成 (含賦形劑)

藥物含量與劑量密度 (µg/mm²)

尺寸規格：球囊直徑、長度、工作長度、相容導絲尺寸

物理性能指標：額定爆破壓力 (RBP)

儲存條件：溫度、光敏性、保存期限

2.2 藥物成分的化學、製造與管制 (CMC)
已核准藥物：若藥物來源、製程、規格與台灣已核准藥物相同，可免除 CMC 資料，僅需提供證明

藥品主檔案 (DMF)：若藥物有有效 DMF，需提供授權書 (LOA) 與相關摘要

新藥物質：需提交完整 CMC 資料，包括外觀、聚合物分析、鑑別、雜質、殘留溶劑、含量均勻性

2.3 製造流程
重點在「塗層製程」：

提交詳細製造流程圖，標示關鍵管制點

最終產品規格與分析證書 (COA)

驗證塗層均勻性 (縱向與環向)

3. WOW 1：美人魚流程圖 (審查流程)
生物成分？

標準 DCB？

是 → 否 → 開始：上市前申請

技術文件審查 → 藥物成分評估 → 臨床前工程測試 → 安全性與有效性 → 最終核准決策

4. 安全與性能數據要求
4.1 工程與功能測試
球囊額定爆破壓力 (RBP)

疲勞測試 (反覆充氣/放氣)

順應性 (壓力 vs. 直徑)

接合強度 (導管各部件連接處)

柔韌性與抗折性能

4.2 塗層特性
劑量密度

塗層完整性 (模擬血管追蹤前後)

顆粒評估 (脫落顆粒數量與大小)

體外藥物釋放曲線

4.3 藥理與毒理
新藥物質需完整非臨床測試 (GLP)

一般毒性 (兩種哺乳動物)

安全藥理 (心血管、呼吸、中樞神經)

基因毒性與生殖毒性

藥代/藥效學 (PK/PD)

5. WOW 2：國際法規比較 (表 1)
特徵	台灣 (TFDA)	美國 (FDA)	歐盟 (MDR)
分類	第三級 (高風險)	第三級 (PMA)	第三級 (規則 8/14)
審查途徑	臨床前指引 (105.01.14)	上市前核准 (PMA)	MDR 2017/745 附錄 IX/X
藥物成分	需 CMC；允許 DMF	組合產品 (CDRH/CDER)	與 EMA/國家機構協商
臨床數據	視情況；常需本地	必須 (IDE/PMA)	必須 (臨床評估)
生物相容性	ISO 10993 系列	ISO 10993 + FDA 指引	ISO 10993 系列
上市後監測	定期安全報告	年度報告/上市後研究	PSUR / PMCF

6. WOW 3：模擬拒件風險評估 (表 2)
缺失類型	風險等級	風險描述	預測/緩解措施
CMC 不足	高	缺乏藥物塗層穩定性數據或雜質分析	驗證藥物保存期限符合裝置儲存條件
顆粒數量	高	顆粒脫落超過 USP <788> 標準	優化塗層製程並進行模擬測試
等效性缺口	中	使用非滅菌樣品測試無科學理由	所有性能測試需在完成滅菌產品上進行
PK/PD 缺口	中	劑量密度高卻未評估全身暴露	提供動物 PK 數據比較局部與全身濃度
老化方案不足	中	加速老化未包含塗層完整性與藥物含量分析	將所有保存期限項目納入老化驗證

7. WOW 4：法規追溯熱圖 (表 3)
測試類別	測試項目	參考標準	TFDA 重點
生物相容性	細胞毒性/致敏性	ISO 10993-5 / ISO 10993-10	主要接觸材料
血液相容性	ISO 10993-4	與血液成分交互作用
滅菌/致熱源	EO 殘留/內毒素	ISO 10993-7 / USP <85>	滅菌保證水準 (SAL 10⁻⁶)
機械性能	額定爆破壓力 (RBP)	ISO 25539-1 / FDA PTCA 指引	統計信心 (95/95)
導管接合強度	ISO 10555-1 / TFDA 指引	接合部拉伸強度
塗層	總藥物含量	USP <905> / ICH Q6A	劑量均勻性
顆粒物	USP <788> / ASTM F2119	栓塞風險評估
藥理	一般毒性	ICH S2B / S7A	GLP 合規
包裝	完整性/運輸壓力	ISO 11607 / ASTM D4169	無菌屏障維持

8. 技術深度解析
8.1 藥代動力學與藥物釋放
測量球囊表面殘留藥物

測量血液# SmartMed Review 4.2 — Comprehensive Technical Specification (Streamlit on Hugging Face Spaces)

**Document Version:** 4.2.0  
**Date:** 2026-03-25  
**Status:** Draft Technical Specification (Design + Implementation Guidance; **no code included**)  
**System Name:** SmartMed Review 4.2 (智慧醫材審查指引與清單生成系統)  
**Deployment Target:** Hugging Face Spaces (Streamlit)  
**Core Stack:** Streamlit + `agents.yaml` + Multi-LLM routing (Gemini / OpenAI / Anthropic / Grok) + PDF/OCR toolchain

---

## 1. Executive Summary

SmartMed Review 4.2 is an AI-powered, agentic regulatory workspace for medical device Regulatory Affairs (RA) teams and reviewer workflows. It **retains all original capabilities** of SmartMed Review 3.x/4.0/4.1—document ingestion, PDF preview and page selection, OCR (Python OCR + Gemini Vision OCR), guidance transformation into structured Markdown, 510(k) intelligence utilities, checklists and report drafting, dynamic agent orchestration via `agents.yaml`, SKILL panel rendering, and AI Note Keeper with editable inputs/outputs.

Version **4.2** focuses on **WOW-level user experience upgrades** and **fine-grained human-in-the-loop control** across the full agent pipeline, while extending the AI Note Keeper with **three additional AI features** (in addition to the existing six “AI Magics”). It also formalizes **three enhanced WOW visualization features**: **WOW Interactive Indicator**, **Live Log**, and **WOW Interactive Dashboard** with deeper interactivity, drilldowns, and observability.

### 1.1 Key Additions in 4.2 (Compared to 4.1)

1. **WOW UI v2** (new “WOW UI” layer maintained across the app)
   - Theme toggle: **Light / Dark**
   - Language toggle: **English / Traditional Chinese (繁體中文)**
   - **20 painter-inspired styles** (based on famous painters; style selectable or randomized via “Jackpot”)
   - Unified design tokens (color, typography, spacing, motion) applied consistently to all modules

2. **WOW Visualization Enhancements (3 features)**
   - **WOW Interactive Indicator**: agent/module run states with interactive drilldowns and status storytelling
   - **Live Log**: streaming events + filter/search + structured run events + export
   - **WOW Interactive Dashboard**: cross-module analytics, run timelines, model mix, token/cost estimates, and “bottleneck” insights

3. **API Key Handling: Environment-first + UI fallback**
   - Users can input provider keys in the web UI **only if missing in environment**
   - If a key exists in environment, UI shows “Managed by environment” and **never displays the key**
   - Session-only storage for UI-entered keys, never logged, never persisted

4. **Agent-by-agent execution studio**
   - Before running each agent:
     - user can **select model** (from allowlist)
     - **modify the prompt**
     - adjust parameters (max tokens, temperature)
   - After each agent run:
     - output is editable in **Text or Markdown view**
     - the edited output becomes the input for the next agent
   - Supports sequential execution with explicit “Run Next” controls and clear run-state UI

5. **AI Note Keeper expansion**
   - Paste notes (TXT/Markdown), transform into organized Markdown with **keywords highlighted in coral**
   - Toggle editing in Markdown or TXT view
   - Keep a custom prompt for further transformations OR apply AI Magics
   - **Adds 3 new AI Magics** (bringing the total to **9**)

6. **Multi-provider model selection expansion (standardized lists)**
   - OpenAI: `gpt-4o-mini`, `gpt-4.1-mini`
   - Gemini: `gemini-2.5-flash`, `gemini-3-flash-preview`
   - Anthropic: configurable “claude-*” family (e.g., `claude-3.5-sonnet`, `claude-3.5-haiku`, subject to account availability)
   - Grok: `grok-4-fast-reasoning`, `grok-3-mini`
   - Model availability is validated by provider-key presence + allowlist rules per module

---

## 2. Users, Use Cases, and Success Criteria

### 2.1 Primary Users
- **Regulatory Affairs specialists**: interpret guidance, prepare submission strategy, generate checklists
- **Medical device reviewers**: triage submissions, identify gaps, draft review memos and requests
- **Quality/compliance professionals**: ensure traceability, completeness, and process consistency
- **Technical leads**: maintain `agents.yaml`, validate prompts/skills, deploy on Hugging Face Spaces

### 2.2 Key User Journeys
1. **Guidance ingestion → OCR → Structured guidance Markdown**  
   Upload PDF, select pages, OCR/extract, generate structured reviewer-friendly guidance doc (with constraint checks).

2. **Agent pipeline execution with editing between agents**  
   Execute agents sequentially; customize prompts and models step-by-step; edit outputs before passing onward.

3. **AI Note Keeper**  
   Paste messy notes → receive organized Markdown with coral keywords → refine → apply AI Magics (9) using chosen models.

4. **Operational monitoring & debugging (WOW Observability)**  
   Use Interactive Indicator + Live Log + Dashboard to understand what ran, what failed, time/cost, and bottlenecks.

### 2.3 Definition of “Excellent Results”
- Users can **control** every major generation step (prompt, model, parameters, input/output edits)
- Outputs are reproducible and explainable via pinned prompts + run metadata
- Errors are understandable and actionable (preflight checks; consistent statuses)
- WOW UI feels cohesive, modern, and delightful without obscuring transparency

---

## 3. Scope, Non-Goals, and Design Principles

### 3.1 In-Scope Modules (All Original Features Preserved)
- WOW Dashboard + status + metrics
- Guidance Workspace:
  - Paste/upload TXT/MD/PDF
  - PDF preview + page selection
  - OCR/extraction (Python extraction + optional OCR; Gemini Vision OCR)
  - Structured guidance Markdown generation (2000–3000 words, exactly 3 tables)
  - Edit + download
- 510(k) intelligence utilities (PDF → Markdown transformer, summary/entities, comparator)
- Checklist & report drafting tools
- AI Note Keeper (now with 9 Magics)
- FDA orchestration + dynamic agents from guidance
- `agents.yaml` based orchestration
- SKILL panel rendering (`SKILL.md`)

### 3.2 Non-Goals (Explicit)
- No multi-user authentication, RBAC, audit database, or enterprise tenancy requirements
- No legal/regulatory authority claims; outputs are drafting aids
- No guaranteed deterministic compliance; LLMs are probabilistic
- No long-term server-side storage requirement (session-based by default)

### 3.3 Design Principles
- **Human-in-the-loop as default**: no black-box batch pipelines
- **Environment-first security**: keys from environment secrets take priority; UI fallback is optional and session-only
- **Provider-agnostic orchestration**: one unified LLM interface across providers
- **Graceful degradation**: core ingestion works even if optional OCR dependencies missing
- **WOW UX with observability**: make status, logs, and run artifacts first-class

---

## 4. High-Level Architecture

### 4.1 Runtime Components
**A. Streamlit UI Layer**
- Sidebar: global settings (theme, language, painter style, jackpot, default model/params), API key manager, optional `agents.yaml` override upload
- Main area: module tabs and workflow steps
- Persistent top status bar: WOW indicator + quick actions (clear session, export logs, download artifacts)

**B. Session State Layer (Ephemeral)**
Stores runtime-only artifacts:
- Uploaded PDF bytes (per file)
- Page selections + rendered thumbnails (if enabled)
- Extracted raw text per page + merged guidance text
- Generated guidance Markdown + compliance check results
- Agent prompts, pinned prompts, outputs (editable)
- Run events (structured) for live log/dashboard
- Session-only provider keys if entered via UI

**C. Document Processing Layer**
- PDF text extraction via `pypdf` (best-effort)
- PDF page rendering to images via PyMuPDF/Pillow (optional)
- Python OCR via pytesseract (optional; if dependencies installed)
- Gemini Vision OCR option for scanned documents

**D. LLM Integration Layer**
Unified routing by model name:
- OpenAI (Chat Completions)
- Gemini (generate_content)
- Anthropic (messages)
- Grok/xAI REST

**E. Agent Orchestration Layer**
- Agents defined in `agents.yaml` (system prompt, templates, defaults)
- Agent Studio UI supports:
  - pre-run editing of prompt and model
  - post-run editing of output
  - chaining output to next agent
  - run metadata capture (time, model, estimated tokens)

### 4.2 Cross-Cutting Concerns
- **Preflight validation**: block runs before they start if keys/models/inputs missing
- **Observability**: each run emits structured events to Live Log + Dashboard
- **Consistency**: WOW UI v2 design tokens applied across all modules

---

## 5. WOW UI v2 (Theme, Language, Painter Styles, Jackpot)

### 5.1 Global Controls
Accessible from sidebar and mirrored in a compact header menu:
- **Theme**: Light / Dark
- **Language**: English / Traditional Chinese  
  - Applies to UI labels, help text, warnings, and system messages  
  - (Generated content language remains controlled by prompts and module toggles.)
- **Painter Style (20)**: aesthetic overlay controlling gradients, background textures, accent harmonies
- **Jackpot**: randomly selects a painter style (with optional “lock style for this session”)

### 5.2 Painter Style Set (20 Styles)
Painter-inspired style presets define:
- background gradient palette
- accent color harmonies
- panel border glow intensity
- button style (solid vs glass vs outline)
- motion profile (subtle vs energetic)

**Example list (names used as UI labels; no IP assets embedded):**
1. Monet (soft atmospheric)
2. Van Gogh (high contrast strokes)
3. Picasso (geometric blocks)
4. Klimt (golden ornament)
5. Hokusai (indigo waves)
6. Rothko (color fields)
7. Pollock (speckled energy)
8. Vermeer (calm light)
9. Matisse (bold cut-outs)
10. Rembrandt (chiaroscuro)
11. Turner (mist + glow)
12. Cézanne (structured depth)
13. Magritte (surreal clarity)
14. Dalí (dream distortion)
15. Kandinsky (abstract rhythm)
16. Hopper (quiet modern)
17. O’Keeffe (organic minimal)
18. Basquiat (street annotation)
19. Bruegel (dense detail)
20. Ukiyo-e (woodblock flatness)

### 5.3 Design Tokens and Accessibility
- Typography scale with clear hierarchy (H1/H2/H3, body, monospace for logs)
- Minimum contrast ratios maintained in both themes
- Keyboard navigability:
  - tab focus states
  - action buttons reachable in order
- Coral highlight color reserved for **keywords** and critical emphasis (configurable but coral is default)

---

## 6. WOW Visualization Enhancements (3 Features)

> These three features must exist across **all** modules and be consistent with the WOW UI theme/language/style.

### 6.1 WOW Interactive Indicator (Enhanced)
A persistent indicator component representing:
- overall session state: idle / running / blocked / error / success
- module statuses: Guidance Workspace, Guidance Generator (Step 3), Agent Studio, Note Keeper, 510(k) tools
- active provider/model selection and key readiness state

**Interactive behaviors**
- Clicking the indicator opens a **Run Inspector Drawer**:
  - current run step (e.g., “Agent 3/7: Deficiency Extractor”)
  - elapsed time, retries (if any), and throttling notes
  - “What is happening?” human-readable explanation localized to UI language
  - immediate actions:
    - cancel run (best-effort)
    - copy run prompt (masked if contains secrets; should not)
    - open Live Log filtered to this run

**Status semantics**
- **Blocked**: preflight failed (missing key, missing input pages, invalid model)
- **Running**: LLM call in progress
- **Done**: successful completion + artifact saved
- **Warning**: completed but constraint checks failed (e.g., table count mismatch)
- **Error**: exception occurred; shows friendly summary + technical detail accordion

### 6.2 Live Log (Enhanced, Streaming-First)
A dedicated panel/tab providing:
- streaming run events (structured), appended in real time
- filters and search
- export to `.jsonl` and `.txt` (user-initiated)

**Event schema (minimum fields)**
- timestamp (UTC + local)
- run_id (UUID-like)
- module (Guidance/OCR/Step3/AgentStudio/NoteKeeper/510k)
- severity (info/warn/error)
- event_type (preflight, request_sent, partial_received, completed, retry, blocked, exception)
- provider + model
- input size estimates (chars/pages)
- output size estimates (chars)
- token/cost estimates (if available)
- latency metrics (queue time, generation time)

**UX details**
- Two modes:
  - **Simple**: readable narrative log (“Step 3 started… received output…”)
  - **Technical**: shows structured JSON fields
- One-click actions:
  - “Filter to current run”
  - “Copy error summary”
  - “Open artifact created by this event” (if available)
- Privacy guarantees:
  - never log API keys
  - redact any detected secret-like patterns in prompts (best-effort)

### 6.3 WOW Interactive Dashboard (Enhanced)
A central observability and productivity hub.

**Core widgets**
1. **Run Timeline** (interactive)
   - shows each run as a segment with start/end and status
   - click a segment → open Run Inspector with prompt pin, model, artifacts, logs
2. **Model Mix & Provider Mix**
   - donut/bar charts showing usage by model/provider within session
3. **Token/Cost Estimator Panel (best-effort)**
   - displays rough token estimate from input/output size
   - shows user-facing disclaimers (“estimates only”)
4. **Bottleneck Analyzer**
   - highlights slowest steps and likely causes (OCR page count, huge prompts, model choice)
5. **Constraint Compliance Monitor**
   - tracks Step 3 constraints: word count estimate + table count estimate
   - shows how many runs passed/failed constraints
6. **Artifact Quick Access**
   - latest Guidance Markdown
   - latest Agent output
   - latest Note Keeper organized note
   - download shortcuts

**Interactivity requirements**
- cross-filtering: selecting a model updates the timeline to highlight runs with that model
- deep-link behavior: dashboard → opens relevant module at the correct step with artifacts loaded in view

---

## 7. API Key Management (Environment-First, UI Fallback)

### 7.1 Key Sources and Priority
1. **Environment secrets** (Hugging Face Space secrets)
2. **User-provided UI key** (session-only)

### 7.2 UI Requirements
- If environment key exists for a provider:
  - show status: “Managed by environment”
  - show a green readiness indicator
  - do **not** show the key, not even partially
  - optional: show last-4 hash fingerprint **only if derived safely** (no reversible display), otherwise omit
- If environment key is missing:
  - show a password input box for user entry
  - show a “store only in this session” label
  - show “clear key” action

### 7.3 Preflight Blocking (Consistency Rule)
Before any LLM call:
- determine provider from selected model
- confirm provider key exists (env or session)
- if missing: set status to **Blocked**, log a structured preflight event, and show a clear localized message:
  - what is missing (which provider key)
  - how to fix (enter key or set Space secret)
  - suggest a model/provider that is available

---

## 8. Multi-Provider Model Selection (Standardized)

### 8.1 Global Model Allowlist (UI)
The model dropdowns across modules must include, at minimum:

**OpenAI**
- `gpt-4o-mini`
- `gpt-4.1-mini`

**Gemini**
- `gemini-2.5-flash`
- `gemini-3-flash-preview`

**Anthropic**
- “claude-*” models (configured list; examples):
  - `claude-3.5-sonnet`
  - `claude-3.5-haiku`

**Grok/xAI**
- `grok-4-fast-reasoning`
- `grok-3-mini`

### 8.2 Module-Specific Allowlists
Some modules may restrict to subsets (performance/cost/format reliability). Example:
- Gemini Vision OCR: Gemini models only
- Step 3 Guidance Generator: allow OpenAI + Gemini by default, optionally include Anthropic/Grok if validated for table constraints

---

## 9. Guidance OCR & Generator Module (Retained + Strengthened)

### 9.1 Step 1 — Ingest Guidance
Inputs:
- paste TXT/Markdown
- upload `.txt`, `.md`, `.pdf`

Rules:
- if both paste and file exist, user selects which to use (explicit toggle recommended to avoid surprises)
- normalize line endings; preserve headings and bullets

### 9.2 Step 1b — PDF Preview & Page Selection
- inline PDF preview
- show page count
- multi-select pages
- guardrails:
  - default selection limited (e.g., first 3–5 pages)
  - warnings when selecting large page ranges
  - show “estimated OCR time/cost” if possible

### 9.3 Step 2 — Extraction / OCR
Modes:
1. **Python extraction** via `pypdf` (fast; best for digital PDFs)
2. **Python OCR** (optional): render pages + `pytesseract` (depends on runtime)
3. **LLM OCR (Gemini Vision)**: render pages to images and transcribe

Requirements:
- create merged raw guidance text with page delimiters
- generate OCR diagnostics per page (chars extracted, OCR method used, warnings)

Safety limits:
- max pages per run (configurable; default recommended)
- visible progress via WOW indicator + Live Log events

### 9.4 Step 3 — Generate Organized Guidance Markdown (2000–3000 words; exactly 3 tables)
**Purpose:** produce a reviewer-friendly structured guidance document.

Inputs:
- raw guidance text
- output language toggle (English / Traditional Chinese)
- editable prompt
- model selection (OpenAI/Gemini + others if enabled)
- parameters: max tokens, temperature

Hard constraints enforced by prompt contract:
- 2000–3000 words (note: heuristic for CJK; UI labels it as an estimate)
- exactly **3 Markdown tables** in the entire document
- no hallucinated requirements; if uncertain, explicitly state uncertainty and recommend verification
- structured headings for reviewer use (scope, key review points, evidence expectations, common deficiencies, etc.)

**Compliance checks (post-generation, heuristic)**
- word count estimate
- table count estimate (regex-based)
- show pass/warn status; warn triggers “Regenerate with constraints” option (design-level)

Outputs:
- `guidance_doc_md`
- pinned prompt snapshot used
- run metadata for dashboard/log

### 9.5 Step 4 — Edit & Download
- dual view: Markdown / plain text
- downloads: `.md` and `.txt`
- artifact versioning within session (keep last N versions)

---

## 10. Agent Studio (agents.yaml) — Editable, Stepwise Execution

### 10.1 Agent Definitions
Agents are loaded from `agents.yaml` with:
- name, description
- system prompt template
- user prompt template (parameterized)
- default model and parameter hints
- input/output artifact bindings

### 10.2 Pre-Run Controls (Per Agent)
Before running an agent, user can:
- select model (from allowlist)
- edit prompt content (system and/or user prompt depending on configuration)
- adjust max tokens and temperature
- choose input source artifact (e.g., guidance doc, previous agent output, pasted text)

Preflight checks:
- key availability for chosen model/provider
- input not empty
- length warnings (huge context → suggest trimming)

### 10.3 Post-Run Controls (Editable Output Chaining)
After execution:
- show output in:
  - Markdown view (rendered + source)
  - Text view
- user can edit output; edited output becomes the “effective output” artifact
- “Use as input to next agent” is explicit and visible
- store both:
  - raw model output (read-only)
  - user-edited effective output (used downstream)

### 10.4 Run Orchestration Modes
- **One-by-one (default)**: user manually runs each agent step
- Optional “guided run” mode (still stepwise): UI guides to next step but never auto-runs without confirmation

---

## 11. AI Note Keeper (Expanded) + AI Magics (Now 9)

### 11.1 Note Ingestion and Organization
User can paste:
- `.txt` content
- Markdown notes (messy or partial)

Primary “Organize Note” agent:
- transforms into structured Markdown:
  - title (if inferable)
  - sections with headings
  - bullet lists and action items
  - decision points and open questions
- extracts and highlights keywords in **coral color**
  - default: coral highlight via consistent Markdown/HTML styling approach supported by Streamlit rendering
  - user can customize keyword highlighting style (optional; coral remains default)

Editing:
- toggle Markdown view / TXT view
- edits update the effective note artifact

### 11.2 Prompt Pinning and Model Choice
- user can maintain a pinned “note transformation prompt”
- choose model per action (organize, refine, apply magic)
- provider-key preflight applies

### 11.3 AI Magics (Total 9)
The system retains the original 6 AI Magics and **adds 3 more**. Each magic:
- takes the effective note as input
- produces an editable Markdown output
- writes run metadata to dashboard/log
- is model-selectable

#### 11.3.1 Existing AI Magics (Retained, 6)
(These remain as originally designed; names may map to existing UI features.)
1. **Executive Summary Builder** (turn notes into brief + structured summary)
2. **Action Items Extractor** (tasks, owners placeholders, deadlines placeholders)
3. **Risk & Mitigation Draft** (identify risks; propose mitigations with uncertainty labels)
4. **Meeting Minutes Formatter** (attendees, agenda, decisions, next steps)
5. **Regulatory Checklist Generator** (convert notes into checklist rows)
6. **Keyword Highlighter/Refiner** (improve keyword selection; keep coral highlight)

#### 11.3.2 New AI Magics (Added in 4.2, 3)
7. **Traceability Matrix Builder (Notes → Evidence Map)**  
   Produces a Markdown table mapping:
   - note claim / requirement / question
   - suggested evidence type (test, document, standard, rationale)
   - status (unknown / needs confirmation / ready)
   - gaps and follow-ups  
   Includes explicit uncertainty tags and avoids fabricating evidence.

8. **Contradiction & Ambiguity Detector**  
   Scans notes to identify:
   - conflicting statements
   - ambiguous terms (e.g., “safe”, “validated”, “equivalent” without criteria)
   - missing definitions (device version, population, indications, endpoints)
   Outputs:
   - a list of issues
   - rewrite suggestions
   - “questions to ask” section for stakeholder follow-up

9. **Regulatory-Ready Rewrite (Localized Tone)**  
   Produces two parallel rewrites:
   - **Regulatory formal** (submission-ready tone)
   - **Internal actionable** (team-friendly tasks)  
   Respects the UI language toggle for labels and supports English/Traditional Chinese output selection. Adds a “Terminology normalization” section to standardize key device/regulatory terms.

---

## 12. 510(k) Intelligence, Comparator, and Report Tools (Preserved)

All existing 510(k) and report drafting tools remain in place. In 4.2 they inherit:
- WOW UI v2 theme/language/style
- consistent model selector and provider preflight
- Live Log events and Dashboard metrics
- optional Agent Studio chaining into/from 510(k) outputs

---

## 13. Reliability, Guardrails, and Error Handling

### 13.1 Common Failure Modes and Required Responses
- Missing key → block preflight; show fix instructions; log event
- Rate limit / quota → show provider-specific suggestion (retry later, switch model/provider)
- Oversized input → warn; suggest trimming pages or using summarization agent first
- OCR dependency missing → gracefully fallback (Python extraction or Gemini OCR) and explain in UI

### 13.2 Content Integrity Guardrails
- “Do not hallucinate” guidance included in system prompts for guidance generation and note transformations
- When source lacks details, model must:
  - state uncertainty
  - list what to verify

### 13.3 Constraint Compliance (Step 3)
- show compliance panel (word/table estimates)
- if failed:
  - status becomes Warning (not Error)
  - one-click “Regenerate with constraints” (design-level)
  - show targeted prompt hints (“avoid extra tables”)

---

## 14. Security, Privacy, and Data Handling

### 14.1 Data Handling Defaults
- session-state only storage
- no server-side persistence required
- user-initiated downloads only
- optional cache for performance (must not include keys)

### 14.2 Key Handling Guarantees
- never display environment keys
- never write keys to logs
- never persist UI-entered keys beyond session
- redact secret-like strings in Live Log (best-effort)

### 14.3 Hugging Face Spaces Considerations
- document clearly that Spaces runtime may restart, clearing session state
- recommend users download artifacts they care about

---

## 15. Deployment and Configuration

### 15.1 Required Files and Config
- Streamlit app entry
- `agents.yaml` shipped with default agents; optional user override upload
- `SKILL.md` for skill panel content
- environment secrets configured in Hugging Face Space:
  - `OPENAI_API_KEY`
  - `GEMINI_API_KEY` (or equivalent naming)
  - `ANTHROPIC_API_KEY`
  - `XAI_API_KEY` (or equivalent for Grok)

### 15.2 Operational Defaults
- conservative default max pages for OCR
- sensible default max tokens per module
- minimal telemetry: session-local only unless explicitly extended later

---

## 16. Acceptance Criteria (4.2)

1. Users can toggle **Light/Dark**, **English/Traditional Chinese**, and choose among **20 painter styles**; Jackpot random selection works.
2. WOW visualization features exist and are functional:
   - Interactive Indicator with drilldown
   - Live Log with filters/search/export
   - Interactive Dashboard with run timeline, model mix, token estimates, compliance monitor
3. API key UI:
   - shows input only when env key missing
   - never displays env key
   - UI key stored only in session
4. Agent Studio:
   - per-agent prompt/model/params editable before run
   - outputs editable after run and can be used as next input
5. AI Note Keeper:
   - organize note → Markdown with coral keywords
   - editable Markdown/TXT
   - 9 AI Magics available (6 original + 3 new)
6. System continues to run on Hugging Face Spaces using Streamlit, `agents.yaml`, and supports Gemini/OpenAI/Anthropic/Grok routing.

---

## 17. Follow-up Questions (20)

1. For the **20 painter styles**, do you want them to affect only colors/backgrounds, or also typography (e.g., serif vs sans), spacing density, and animation intensity?
2. Should the **Jackpot** selection re-roll on every page refresh, or remain stable for the whole session unless the user clicks Jackpot again?
3. Do you prefer that the **UI language toggle** changes only the interface, or should it also default the **generation language** for all modules unless overridden?
4. For Step 3’s “**2000–3000 words**” constraint, should Traditional Chinese be measured by estimated **word equivalents**, **character count**, or a dual metric displayed side-by-side?
5. Should the “**exactly 3 tables**” constraint apply strictly to *any* Markdown pipe table, or should HTML tables count too (and thus be disallowed)?
6. For **WOW Interactive Indicator**, do you want a single global indicator only, or separate mini-indicators embedded per module + a global summary?
7. Should the **Live Log** store only the current session, or also allow the user to download a “run bundle” containing logs + artifacts in a zip (user-initiated only)?
8. Do you want the **Dashboard** to estimate cost using configured per-model price tables, or show token counts only without any cost estimate?
9. For **token estimation**, should the app compute provider-specific tokenization estimates (more accurate) or use a simple chars→tokens heuristic (simpler, less accurate)?
10. In **Agent Studio**, should users be allowed to edit the **system prompt** for every agent, or only the **user prompt** by default with an “advanced” toggle?
11. Should agent chaining support **branching** (keeping multiple alternative outputs) or only a single linear “effective output” per agent?
12. Do you want an explicit **“diff viewer”** when a user edits an agent output (raw vs edited), to improve traceability?
13. For the **AI Note Keeper coral keyword highlight**, do you prefer inline highlighting (e.g., `<span style>`), or a separate “Keywords” section plus subtle inline emphasis for maximum Markdown portability?
14. Should the **Traceability Matrix Builder** magic (new) be allowed to create tables beyond Step 3 constraints (it’s a different module), or do you want a global “limit table creation” toggle?
15. For **Contradiction & Ambiguity Detector**, should it also output a list of **clarifying questions** formatted for email, meeting agenda, or Jira ticket creation?
16. For **Regulatory-Ready Rewrite (Localized Tone)**, should it generate bilingual output (EN + ZH) simultaneously, or only one selected language at a time?
17. Which **Anthropic models** should be explicitly listed in the UI (given account availability varies), and should the app allow a custom model name entry for Anthropic/Grok?
18. Do you want a **provider failover mode** (e.g., if OpenAI fails, retry once with Gemini) or should provider switching remain manual and explicit?
19. Should OCR support an additional “**auto-detect scanned pages**” heuristic (low extracted text triggers OCR suggestion), and how aggressive should it be?
20. Do you want the system to support **artifact persistence** across sessions (e.g., optional user download/upload bundles) while still avoiding any server-side storage?
