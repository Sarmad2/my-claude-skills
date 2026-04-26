---
name: do-it
description: A meta-skill that intelligently orchestrates all available Claude skills to complete any task — posters, websites, assignments, code, documents, simulations, and more. Trigger this skill whenever the user describes a task or deliverable they want completed and you need to figure out which skills to use and in what order. Use it for multi-step work, mixed-domain tasks, or any time the user says things like "do this for me", "make me a...", "help me with...", "I need a...", "create a...", "build a...", or "write a..." — especially when the task might benefit from combining multiple skills. This skill scans available skills, maps them to the task, shows a plan, gets confirmation, and then executes.
---

# do-it

A meta-skill that reads all available skills and orchestrates them intelligently to complete any task the user gives you.

## Your Job

When the user gives you a task, your job is to:
1. Scan available skills
2. Figure out which ones apply and in what order
3. Show the user a clear plan
4. Get a yes/no confirmation
5. Execute the plan, reading each relevant SKILL.md before acting on it

You are the conductor. The individual skills are the instruments.

---

## Step 1: Scan Available Skills

Run the following to see all available skills:

```bash
find /mnt/skills -name "SKILL.md" | sort
```

For each skill found, read its YAML frontmatter (`name` and `description` fields at the top of the file) to understand what it does. You do NOT need to read the full body of every skill at this stage — just the first ~20 lines (the frontmatter) is enough to decide relevance.

Organize what you find into three categories:
- **user skills** (`/mnt/skills/user/`) — custom user skills, highest priority
- **public skills** (`/mnt/skills/public/`) — general-purpose skills
- **example skills** (`/mnt/skills/examples/`) — reference only, use sparingly

---

## Step 2: Analyze the Task

Break the task down into what needs to happen. Ask yourself:

- What is the final deliverable? (a file? code? a design? an answer?)
- What intermediate steps are needed to produce it?
- Which skills map to each step?
- What order should the skills run in?

If multiple skills are relevant, chain them. For example:
- A poster → `frontend-design` (layout) → `pdf` (export)
- A research presentation → `scientific-brainstorming` (ideation) → `academic-pptx` (structure) → `pptx` (build)
- A physics sim → `qutip` (simulation) → `frontend-design` (visualization)

If a step has no matching skill, note it clearly. Do not pretend a skill covers something it doesn't. You will handle uncovered steps using Claude's general knowledge instead.

---

## Step 3: Show the Plan

Before doing anything, present a clear plan to the user. Use this format:

---

**📋 Task:** [restate the task in one sentence]

**🔧 Skill Plan:**

| Step | Skill | What it does for this task |
|------|-------|----------------------------|
| 1 | `skill-name` | [specific action] |
| 2 | `skill-name` | [specific action] |
| ... | ... | ... |

**⚠️ No skill found for:** [list any uncovered steps — or omit this row if everything is covered]
> These steps will be handled using Claude's general knowledge.

**Proceed? (yes/no)**

---

Keep it tight. One sentence per step. The user is technical — no need to over-explain.

---

## Step 4: Wait for Confirmation

Do not proceed until the user says yes (or equivalent: "go", "yep", "do it", etc.).

If the user says no, ask what they'd like to change.

---

## Step 5: Execute

Once confirmed, execute each step in order:

- **For each step that uses a skill:** Read that skill's full SKILL.md before acting. Follow its instructions precisely. It may tell you to install packages, use specific tools, or output to specific paths.
- **For each step with no skill:** Apply Claude's best general knowledge. Note to the user that no dedicated skill was available for this step.
- **Between steps:** Briefly tell the user which step you're on (e.g., "Step 2 of 3 — building the slides...").
- **At the end:** Present the final output clearly. If files were created, use `present_files` to share them.

---

## Edge Cases

**Task is completely outside all skills:**
Tell the user: "No skills are available for this task. I'll handle it using my general knowledge." Then proceed.

**Only one skill matches:**
No need for a table — just say "I'll use the `skill-name` skill for this. Proceed?"

**Task is ambiguous:**
Ask one clarifying question before building the plan. Keep it short.

**User skill conflicts with public skill:**
Always prefer the user skill (`/mnt/skills/user/`) — it's customized for the current user.

---

## Tone

- Be direct and efficient. Assume the user is technical unless context says otherwise.
- Show the plan, get the yes, execute. No fluff.
- Add a motivational line after the final output if it turned out great 🚀
