# AI: Skills

## Summary

Skills are definitions of how to perform certain actions which are commonly needed by AI
agents.
They don't comprise full information on a topic, but provide enough guidance to aid an AI
system to diagnose problems, follow procedures or perform certain tasks.

Within a project these skills are usually defined with the configuration directory for
the AI coding agent. For example:

* Qwen skills are held in: `.qwen/skills/`
* Claude skills are held in: `.claude/skills/`
* Gemini skills are held in: `.gemini/skills/`
* Codex skills are held in: `.agents/skills/`

Each skill is given a simple name for the directory, and contains a file `SKILL.md` which
describes the skill, with a short header. Other resources may also be stored in this directory
to help the skill with specific tasks.

More information on building these skill files can be found at [AgentSkills.io](https://agentskills.io/home).

## System-wide skills

The Build Environment has a number of skills built in, which are available when relevant
tasks are being performed. Usually there is no reason to explicitly invoke a skill, but
should it be necessary, asking for the skill to be applied will usually trigger the correct
behaviour.

The following skills are currently available:

* `using-bbcbasic`: Helps with the BASIC syntax, common usage and integration with RISC OS.
* `using-makefiles`: Helps with the build system's makefiles.
* `using-stronghelp`: Helps with the management of StrongHelp files.
* `using-cmhg`: Helps with the creation of CMHG files..
* `writing-cmodules`: Helps with the development, debugging and testing of modules in C.
* `writing-pymodules`: Helps with the development, debugging and testing of modules in Pyromaniac PyModules, particularly porting to and from C.
* `writing-prminxml`: Helps with creating documentation using PRM-in-XML.
* `riscos-re`: Helps with RISC OS reverse engineering.
* `riscos-commands`: Helps with using RISC OS commands.
* `riscos-output`: Helps with using RISC OS output (VDU, OS_Plot, Draw, Fonts, ColourTrans).

Within the environment you may use the command `ai skills list` to list the skills which
are known, both those that are built in and user skills.

The system-wide skills are stored in `/etc/agentskills` and the `ai` tools will create a
single directory which merges the system-wide and user skills into `.agents/skills`. This
is necessary because not all AI agents have a concept of separate system and user skills.

## User skills

User skills can be created in the `~/.agents/skills` directory in the host. This directory is
mapped into the relevant location for the AI agent, and will be made available for invocation.
If this directory has not been created, the skills will be isolated within the
build environment.

Qwen, Claude, Gemini and Codex support the `/skills` command to list known skills.

* To add a new skill, run `ai skills new <skillname>`.
* To delete a skill, run `ai skills delete <skillname>`.
* To edit a skill, run `ai skills edit <skillname>`.
* To display a skill, run `ai skills cat <skillname>` or `ai skills less <skillname>`.

The format of the skills is described on the [AgentSkills.io](https://agentskills.io/specification)
site.

### Example skill

A simple example skill would allow you to create a 'hello world' program with particular features.

* Run `ai skills new hello-world` to create a skeleton skill.
* Run `ai skills edit hello-world` to edit the skill definition.

Update the content to:

    ---
    name: hello-world
    description: Creates a simple demonstration program for RISC OS. Use when the user asks for a hello world program.
    ---
    # Hello world

    ## BASIC

    * Create a program that prints out "Hello world" in BBC BASIC.
    * Run the program to show that things work.
    * Create a README.md to explain what you did and how to run it.

* Start the AI agent of choice (eg `gemini`)
* Ask it "Write hello world".
* The AI agent should read the skill and follow the instructions.
