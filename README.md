# arctic-hpc: an ARCTIC reference for AI assistants

This folder teaches an AI assistant about ARCTIC, Georgia State University's research computing system. With it, the assistant uses ARCTIC's real partition names, GPU types, storage paths, modules and policies when it answers questions or writes Slurm job scripts, instead of generic HPC guesses.

- `SKILL.md`: instructions for the assistant.
- `references/arctic-reference.md`: the ARCTIC facts, summarized from the [ARCTIC wiki](https://arcwiki.rs.gsu.edu), with links to the source pages.

The wiki is authoritative. If the assistant's answer and the wiki disagree, follow the wiki, and please [report the problem](#report-a-problem).

## Use it responsibly

This folder contains only public information. What you type into an AI tool, and the files you share with it, are covered by GSU policy:

- Follow [GSU's AI policy](https://technology.gsu.edu/ai/policy-and-guidance/).
- Before you use any AI tool with data that isn't public, check that the tool is approved for that data in GSU's [AI Tool Review Registry](https://technology.gsu.edu/ai/tools/) and the [data types guide](https://technology.gsu.edu/ai/using-ai-with-intention/).
- GSU prohibits DeepSeek and Qwen models, including running their open weights on ARCTIC.
- Check what the assistant produces before you run, publish or submit it.

## Install

### OpenCode with ARCTIC's AI models

ARCTIC's own models (the LiteLLM service) are approved by GSU's AI Committee for university data that isn't regulated. First set up OpenCode with them by following [Using OpenCode with ARCTIC's AI Models](https://arcwiki.rs.gsu.edu/en/LiteLLM/OpenCode). Then install this skill:

```bash
git clone https://github.com/arctic-gsu/arctic-hpc ~/.config/opencode/skills/arctic-hpc
```

To get updates:

```bash
git -C ~/.config/opencode/skills/arctic-hpc pull
```

OpenCode loads the skill when a question is about ARCTIC. You can also ask for it by name, for example: "Use the arctic-hpc skill to write a Slurm script for a V100 job."

### University-licensed assistants (Microsoft Copilot, Copilot Studio, Google Gemini, NotebookLM)

These assistants don't load skill folders, but you can give them the same information:

1. Download [`arctic-hpc.zip`](https://github.com/arctic-gsu/arctic-hpc/releases/latest/download/arctic-hpc.zip) and unzip it.
2. Create an agent, Gem or notebook in the assistant, following its own help pages.
3. Add `references/arctic-reference.md` as a knowledge source.
4. Paste the body of `SKILL.md` (everything after the second `---` line) as the agent's instructions, where the tool has a place for instructions.

Product menus change often, so this README doesn't list them. Check which GSU license you have and what data it's approved for in the [registry](https://technology.gsu.edu/ai/tools/).

### Other tools

This folder follows the open [Agent Skills](https://agentskills.io) format, so tools that support that format can load it from their skills folder. The release file [`arctic-hpc.zip`](https://github.com/arctic-gsu/arctic-hpc/releases/latest/download/arctic-hpc.zip) holds the `arctic-hpc` folder, for tools that install skills by upload. Before you use any AI tool with data that isn't public, check the [Tool Review Registry](https://technology.gsu.edu/ai/tools/).

## Report a problem

If the skill gives wrong or outdated ARCTIC information:

- Open an issue at https://github.com/arctic-gsu/arctic-hpc/issues, or
- Open a [Hydra ticket](https://hydra.gsu.edu) assigned to **arctic_developers**, or email arc@gsu.edu.

Say what you asked, what the assistant answered, and which wiki page shows the right answer.
