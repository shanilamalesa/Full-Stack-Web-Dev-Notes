# Week 1 - Day 1 Assignment

## Title
Set Up Your Developer Workshop and Ship Your First Commit

## Overview
Today you install and verify the core tools every engineer in this programme will use for the next 30 weeks: Visual Studio Code, Git, Node.js, and a GitHub account. By the end of this assignment your workshop is functional, you have a GitHub repository with your name on it, and you have pushed your very first commit to the internet. This is the foundation everything else sits on.

## Learning Objectives Assessed
- Install and configure a professional development environment without AI-generated commands
- Verify each tool works from the terminal (not a GUI)
- Complete the full Git workflow end to end: init, add, commit, push
- Recognise and explain what each installed tool does and why it exists

## Prerequisites
- Completed the Day 1 reading: [Onboarding-Tool-Setup.md](./Onboarding-Tool-Setup.md)
- A working laptop (Windows, macOS, or Linux)
- A working internet connection
- An email address you can use to sign up for GitHub

## AI Usage Rules

**Ratio this week:** 90% manual / 10% AI
**Habit:** Ask AI to explain, not to do. See [../ai.md](../ai.md) for full rules.

- **ALLOWED FOR:** Asking AI to explain an error message you already read carefully, or to explain what a command you already ran did. Every AI interaction must be a question about something you have already tried.
- **NOT ALLOWED FOR:** Generating installation commands, writing your config, or producing any text you will paste into your terminal or commit. No copy-paste from AI to your machine this week.
- **AUDIT REQUIRED:** No. Week 1 does not require an AI Audit file. Starting Week 2 every assignment requires one.

## Tasks

### Task 1: Install and verify the four core tools

**What to do:**
1. Install Visual Studio Code from [code.visualstudio.com](https://code.visualstudio.com/). During install, tick "Add to PATH" if the installer offers it.
2. Open VS Code and install these four extensions from the Extensions panel (not from AI): Prettier, ESLint, Live Server, Tailwind CSS IntelliSense.
3. Install Git from [git-scm.com](https://git-scm.com/).
4. Install Node.js LTS from [nodejs.org](https://nodejs.org/).
5. Open a terminal (macOS: Terminal; Windows: Git Bash; Linux: your usual terminal) and run each of these commands, writing down the version number you see:
   - `code --version`
   - `git --version`
   - `node --version`
   - `npm --version`

**Expected output:**
A screenshot (or a plain text file named `versions.txt`) showing all four commands and their version numbers side by side in the terminal.

### Task 2: Configure Git with your identity

**What to do:**
Run these two commands in your terminal, substituting your own name and email. Use the same email you will use for GitHub in Task 3.

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Then verify the settings stuck by running:

```bash
git config --global user.name
git config --global user.email
```

**Expected output:**
A screenshot (or text note) showing the two verification commands printing your name and email back.

### Task 3: Create a GitHub account and your first repository

**What to do:**
1. Sign up for a free GitHub account at [github.com](https://github.com/) if you do not already have one. Use a professional username -- this will appear on your CV and your pull requests for years.
2. From your GitHub profile, create a new public repository named `my-first-project`. Tick "Add a README file".
3. On your computer, open the terminal, move to a folder where you keep code (create one if you have none -- `mkdir -p ~/Code && cd ~/Code`) and clone the repo:

```bash
git clone https://github.com/YOUR_USERNAME/my-first-project.git
cd my-first-project
```

**Expected output:**
You are inside the cloned folder and `ls` (or `dir` on Windows) shows the `README.md` file.

### Task 4: Create your first HTML page and push it

**What to do:**
1. Inside the cloned folder, create a new file called `index.html` by hand (no AI). The simplest valid HTML5 document is enough -- a doctype, an html element, a head with a title, and a body with one `<h1>` that says your name.
2. Open the file in VS Code. Right-click inside the file and pick "Open with Live Server" to confirm it renders in your browser.
3. Commit and push the new file with this Git workflow, each command typed by you:

```bash
git status
git add index.html
git status
git commit -m "feat: add index.html with my name"
git push
```

**Expected output:**
1. A screenshot of your `index.html` rendered in the browser via Live Server, showing your name.
2. A link to your repository on github.com where the new file is visible.

### Task 5: Explain what you installed

**What to do:**
Create a file called `notes.md` in the same repo. In four short paragraphs (2-3 sentences each), explain in your own words what VS Code, Git, Node.js, and the terminal are -- and why you had to install each one. No AI. No copy-paste. Your own words, even if they are imperfect.

Commit this file with the message `docs: add learning notes`.

**Expected output:**
A second commit on your repository with a `notes.md` file containing four short explanations.

## Stretch Goals (Optional - Extra Credit)

- Add your public SSH key to your GitHub account and re-clone the repository using the SSH URL instead of HTTPS. Document in `notes.md` how the experience changed.
- Install one extra VS Code extension of your choice (for example, "GitLens"). Open its page, read what it does, and write one paragraph in `notes.md` about why you picked it.
- Create a second repository called `sandbox` and clone it. You will use it later in the week for experiments.

## Submission Requirements

- **What to submit:**
  - Link to your `my-first-project` GitHub repository.
  - A screenshot of your terminal showing the four `--version` commands (Task 1).
  - A screenshot of your `index.html` rendered in the browser (Task 4).
- **Where to submit:** Post the repo link and screenshots in the Week 1 submissions channel on Discord.
- **Deadline:** End of Day 1, before your Day 2 session begins.
- **Format:** Screenshots as `.png` or `.jpg`. The repo must be public.

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Tool installation and verification | 20 | All four tools installed. All four `--version` commands run successfully and are captured in a screenshot. Partial marks if any tool is missing. |
| Git configuration | 10 | `git config --global user.name` and `user.email` both set and verified. |
| GitHub repository created | 15 | Repo is public, exists under your GitHub username, has a README, and is named `my-first-project`. |
| First HTML page written manually | 15 | `index.html` contains a valid HTML5 skeleton, renders in the browser via Live Server, and shows your name. Must be hand-typed -- facilitator will ask you to walk through one line. |
| Clean Git workflow | 15 | At least two meaningful commits on the repo with clear messages. `git push` succeeded and the files are visible on github.com. |
| Written explanations in notes.md | 15 | Four paragraphs, each in your own words, each explaining one tool accurately. No AI prose. |
| Code quality | 10 | File names match the instructions exactly. No random files committed by accident. No secrets or personal info in the repo. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Copy-pasting commands from AI.** If the command is wrong for your OS, you will spend an hour debugging something AI could not have known. Type every command yourself, read the official docs when unsure, and ask AI only to explain what you already tried.
- **Forgetting `git add` before `git commit`.** A common first-week bug: you commit and nothing happens because nothing was staged. Run `git status` between every step so you can see what Git thinks is going on.
- **Using your personal email privately and then committing from work.** Set `git config` once, carefully. Every commit you make from now on carries this email.

## Resources

- Day 1 reading notes: [Onboarding-Tool-Setup.md](./Onboarding-Tool-Setup.md)
- Week 1 AI boundaries: [../ai.md](../ai.md)
- Official VS Code downloads: https://code.visualstudio.com/
- Official Git downloads: https://git-scm.com/
- Official Node.js downloads: https://nodejs.org/
- GitHub signup: https://github.com/
