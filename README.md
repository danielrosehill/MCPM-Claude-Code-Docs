# MCPM + Claude Code - Docs

Docs: Dec 09th 2025

I created this repository in order to document a few notes about using Claude Code in conjunction wtih the excellent [MCPM](https://mcpm.sh/).

While these notes were written for MCPM specifically, they might be relevant for other managers. 

The specific context in this repo:finding ways to ensure that Claude Code "plays nice" with external MCP managers in an Ubuntu workstation environment where neither Claude nor the external MCP manager can be managed via GUI.

I wanted to document the integration conceptually as well as in steps. 

## Source Documentation / References

References for MCP Manager (MCPM):

https://github.com/pathintegral-institute/mcpm.sh

Docs

https://github.com/pathintegral-institute/mcpm.sh/tree/main/docs

## The Core Idea / Value Proposition

There are a few frustrations that are quickly encountered when working with MCP and AI frontends. 

I use the term AI frontends deliberately to encompass the wide breadth of potential utilities that might leverage MCP: 

- Chat bot interfaces (run locally) 
- Agentic CLIs  

These challenges are:

- MCP tool definitions need to be read by AI agents. This means that they pose a context load. Context loads degrade inference. Therefore, the ideal implementation currently is to be selective about the use of MCP tooling. But this is very cumbersome from a user standpoint. Nobody wants to spend their day turning MCP servers and tools on and off just to suit their current context. 
- Another problem is the tendency of AI tools to insist on their own mechanism for managing MCP servers such as their own MCP management hubs and libraries.   
- This might be understandable, but gets very annoying very quickly. A developer or user who works across a number of IDEs and CLIs will quickly find it frustrating to have to manage a multiplicity of MCP server definitions. 
- The same user may commonly wish to use the same MCP, but with a number of different configurations. For example, in a work context, they might wish to have a Google workspace MCP with their work credentials. But when working on their own projects, use a separate set of credentials. 

A centralized MCP manager like MCPM solves a number of these pain points:

- Consolidate MCPs So that each tool can be just pointed to one MCP which can then handle the aggregation of tools. 
- Manage profile switching much more seamlessly. Users can define profiles, bundling MCPs which map roughly on to specific contexts in which they work. Those profiles can then be associated with specific workspaces. 

## Avoiding Turf Wars Via CLAUDE.md Instruction

If, like me, you use Claude Code as your daily AI agent, but you wish to use MCPM or some other tool for MCP server management.,  then I have found you need to be firm about your preference!

I added an instruction to my `CLAUDE.md` like:

"The user uses MCPM to manage MCP servers. Refer to its documentation here if required (docs link). If you are assisting the user with MCP server management, never add them to Claud Code. Always work with the user's preference for MCPM."

Without this instruction in place, I found that Claude was (understandably):

1) Assuming that if I wanted to add an MCP it was within its own framework (understandably) 
2) Getting mixed up about MCPM. Given that there are a large number of projects for MCP, this is why I'm providing the link to the repository docs to avoid any ambiguity. 

Adding these simple instructions have quickly eliminated these challenges. 

## How To Use Profilles 

At the time I'm writing this (Dec 09, 2025) Claude supports three levels of MCP tooling:

- User level MCPs which apply wherever the user uses the CLI.  
- Project level MCPS, which can be used to consolidate MCP resources for those working collaboratively on a project. These are usually defined within an MCP JSON file in the workspace.  
- Local scope (stored in local config)

See the official doc snippets in reference for the exact definitions.

---

## Profile To Project Mapping

There are a couple of ways in which MCPM and Claude Code can be used in conjunction:

- Create a couple of "meta" profiles (my own term). These reflect high level federation for MCP servers. You might create profiles called for example "personal," "work", "client-projects."
- Then map those profiles to specific projects using one of the methods below.

### Method 1: HTTP Endpoint + Project Scope (.mcp.json)

This is the cleanest approach for project-specific profiles. You run your MCPM profile as an HTTP endpoint, then reference it in the project's `.mcp.json`:

**Step 1: Run your MCPM profile as an HTTP endpoint**

```bash
# Run the data-analysis profile on port 8081
mcpm profile run data-analysis --http --port 8081
```

**Step 2: Create a `.mcp.json` in your project root**

```json
{
  "mcpServers": {
    "data-analysis": {
      "type": "http",
      "url": "http://localhost:8081"
    }
  }
}
```

### Method 2: Local Scope Override

For personal project-specific configurations that shouldn't be shared, use Claude Code's local scope. This stores the configuration in `~/.claude.json` under the project's path.

```bash
# Add an MCP server at local scope (only for this project, only for you)
claude mcp add --transport http data-tools --scope local http://localhost:8081
```

This is useful when:
- You have personal credentials or API keys in the profile
- The profile configuration is experimental
- You don't want to commit the MCP configuration to version control