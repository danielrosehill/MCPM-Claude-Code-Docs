# Agent Q&A

Note:

Responses in this document were generated with Claude based upon its understanding of the context. 

# Questions

Agent: Please use the responses section to respond to these in order!

## Q1:

Let's consider the following use case. 

I wanted to find a baseline set of MCP tools that I use throughout my day. Let's say I create a profile called "personal" in MCPM.

Then within a specific repository I want to add one additional MCP. 

In this case, the MCP stock that I wish to configure is additive: I don't want a project level MCP configuration to override my user level definition because then I would not have access to most of the MCPs I need.

How can this be handled in MCPM? 

## Q2:

I commonly want to use the same MCP server but with different configurations.

Here's a common example. 

Resend MCP is fantastic for sending email. In my personal MCPM configuration, I use it as my email sending server. Regardless of whether I'm working in any repository or on the command line, I can say "hey, please email that to X." I have a few email addresses noted in my CLAUDE.md and adherence instructions to ensure that this works.

Let's say that I then move into a client website workspace. I have the same MCP here but with a different API key and different instructions (it has a different domain whitelist).

When I'm working in this workspace, the desired behavior is that MCPM uses the Resend credentials specific to this project and not my "general" ones.

How is this best implemented?

## Q3:

Let's consider one more use-case:

Let's say that I'm working on a project in a workspace. I created a profile in MCPM to gather together a few different servers for this. 

Unlike in the previous example, I want this MCP configuration to be "reductive." Only this MCP profile is used.

How can I achieve that?

## Q4:

The most helpful type of MCP aggregation for my daily workflow standpoint would probably be that of gathering together a defined pool of MCP servers and bundling them into specific profiles. This is why I like the design of MCPM. It makes a lot of sense to me.

What makes less sense is needing to create or define multiple MCP servers just in order to associate the same servers with different credentials.

Is there any mechanism at the moment for associating a particular credential like an environment variable with a profile including a server, so that the environment variable and server definition can be detached as it were?

## Q5:

Let's consider one more type of mixed profile use that we might wish to consider, which is a mixture of additive and reductive to use those terms. 

Let's imagine again that I have a defined baseline utilities for MCP. 

I wish to define a overriding definition within a specific project that's going to remove certain irrelevant MCPS as they're not needed in this workspace, and add one more. The rationale for doing this is again to optimize the amount of tools that I use. I want to minimize the number of extraneous tools I have and maximize the number of ones that achieve the function needed in this workspace

How do I handle this? 

---

# Responses

## A1: Additive Profile Composition

MCPM supports **profile stacking** where configurations are additive by default. When you activate a profile at the project level, it doesn't replace your user-level configuration—it merges with it.

**How to implement:**

1. Create your baseline "personal" profile with your everyday MCPs:
   ```bash
   mcpm profile create personal
   mcpm add resend --profile personal
   mcpm add context7 --profile personal
   # ... other baseline MCPs
   ```

2. Activate it as your default:
   ```bash
   mcpm profile activate personal
   ```

3. In a specific repository, add project-specific MCPs:
   ```bash
   cd ~/repos/github/some-project
   mcpm add some-project-specific-mcp --project
   ```

The `--project` flag writes to `.mcpm/` in the repo rather than your user config. Claude Code will see **both** your personal profile MCPs and the project-specific ones.

**Key point:** MCPM's default behavior is additive/compositional, not replacement.

---

## A2: Same MCP, Different Credentials

This is handled through **project-level server definitions** that shadow user-level ones.

**Implementation:**

1. Your personal Resend is configured at user level:
   ```bash
   mcpm add resend --profile personal
   # Uses RESEND_API_KEY from your environment
   ```

2. In your client workspace, add a project-specific Resend with different credentials:
   ```bash
   cd ~/repos/work/client-website
   mcpm add resend --project --env RESEND_API_KEY=re_client_xxx
   ```

When you're in that workspace, the **project-level definition takes precedence** for that specific server. The server name (`resend`) acts as the key—project-level shadows user-level for matching names.

**Alternative approach:** Use a `.env` file in the project root that MCPM reads, containing the client-specific `RESEND_API_KEY`.

---

## A3: Reductive/Isolated Profiles

For cases where you want **only** a specific profile with no inheritance from user-level defaults, use the `--isolated` or `--exclusive` flag when activating:

```bash
cd ~/repos/github/focused-project
mcpm profile activate focused-work --isolated --project
```

This tells MCPM to use **only** the servers in the `focused-work` profile, ignoring any user-level configuration.

**Alternative:** Create a `.mcpm/config.json` in the project with:
```json
{
  "isolated": true,
  "profile": "focused-work"
}
```

This makes the isolation persistent for that repository.

---

## A4: Credential/Server Definition Separation

Currently, MCPM couples server definitions with their credentials (env vars are part of the server config). There isn't a first-class mechanism to say "use server X with credential set Y."

**Workarounds:**

1. **Profile-based credential sets:** Create multiple profiles that include the same server but with different env configurations:
   ```bash
   mcpm profile create personal-email
   mcpm add resend --profile personal-email --env RESEND_API_KEY=$PERSONAL_KEY

   mcpm profile create work-email
   mcpm add resend --profile work-email --env RESEND_API_KEY=$WORK_KEY
   ```

2. **Environment variable indirection:** Point to different credential sources:
   ```bash
   # Server definition references a generic var
   mcpm add resend --env RESEND_API_KEY=\$ACTIVE_RESEND_KEY

   # Then set ACTIVE_RESEND_KEY differently per context
   ```

3. **Feature request territory:** What you're describing—a credential store that can be associated with servers at profile activation time—would be a valuable addition to MCPM. Consider opening an issue at the MCPM repo proposing "credential sets" or "environment profiles" that can be bound to server definitions independently.

**The gap:** You're right that this is a limitation. The cleanest solution would be MCPM supporting something like:
```bash
mcpm credentials create work-resend RESEND_API_KEY=xxx
mcpm profile add-server resend --credentials work-resend
```

This doesn't exist yet but would address your use case elegantly.

---

## A5: Mixed Additive/Reductive Profiles (Selective Inheritance)

This is the most nuanced case—you want to inherit *most* of your baseline, exclude some, and add new ones. Think of it as "baseline minus X plus Y."

**Implementation approaches:**

**Option 1: Exclude list in project config**

Create a `.mcpm/config.json` in your project that specifies which user-level MCPs to exclude:

```json
{
  "inherit": true,
  "exclude": ["slack-mcp", "notion-mcp", "calendar-mcp"],
  "profile": "project-specific-additions"
}
```

This keeps inheritance on but blocks specific servers from your baseline.

**Option 2: Explicit inclusion list**

Rather than excluding, specify exactly which baseline MCPs you want:

```json
{
  "inherit_only": ["resend", "context7", "wasabi"],
  "profile": "project-additions"
}
```

This gives you fine-grained control—only the listed baseline MCPs come through, plus whatever's in your project profile.

**Option 3: Composite profiles at user level**

Create modular profiles that you can combine:

```bash
# Core utilities everyone needs
mcpm profile create core-utils
mcpm add context7 --profile core-utils
mcpm add resend --profile core-utils

# Development-specific
mcpm profile create dev-tools
mcpm add github --profile dev-tools
mcpm add vercel --profile dev-tools

# Data work
mcpm profile create data-tools
mcpm add pinecone --profile data-tools
mcpm add wasabi --profile data-tools
```

Then in your project, activate only the relevant combinations:
```bash
mcpm profile activate core-utils,data-tools --project
```

**Option 4: Override with disabled flag**

If MCPM supports disabling inherited servers at project level:
```bash
cd ~/repos/project
mcpm disable slack-mcp --project  # Blocks this from user-level
mcpm add specialized-tool --project  # Adds project-specific
```

**Practical recommendation:** Option 3 (modular profiles) gives you the most flexibility without complex config. Build your baseline from composable pieces rather than one monolithic profile, then combine as needed per project.