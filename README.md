# claude-skills

A collection of reusable Claude Code skills for Chinese academic paper workflows.

## Skills

### reduce-ai-rate

Reduces AI detection rate in Chinese academic papers (.docx). Features three rewrite styles (专业版/平衡版/口语化版) with different trade-offs between AI reduction and academic tone preservation.

See [skills/reduce-ai-rate/SKILL.md](skills/reduce-ai-rate/SKILL.md) for full documentation.

## Usage

```bash
# Clone this repo
git clone https://github.com/Xiao-Bai-233/claude-skills.git

# Copy the skill to your Claude Code skills directory
cp -r skills/reduce-ai-rate ~/.claude/skills/
```

Then in Claude Code, invoke:

```
/reduce-ai-rate
```
