# Configuration

For basic configuration instructions, see [this documentation](https://developers.openai.com/codex/config-basic).

For advanced configuration instructions, see [this documentation](https://developers.openai.com/codex/config-advanced).

For a full configuration reference, see [this documentation](https://developers.openai.com/codex/config-reference).

## Lifecycle hooks

Admins can set top-level `allow_managed_hooks_only = true` in
`requirements.toml` to ignore user, project, and session hook configs while
still allowing managed hooks from requirements and managed config layers. This
setting is only supported in `requirements.toml`; putting it in `config.toml`
does not enable managed-hooks-only mode.

### Hook trust

A hook from a managed layer (admin, MDM, or cloud-managed config) runs without
a trust step. A hook from your own user, project, or session config does not
run until it is trusted, and an enabled but untrusted hook is skipped. In a
headless run (`codex exec`) the skip is reported as a warning that names the
hook and its trust key; the interactive UI lists the same hooks in its hook
review.

Trust is recorded per hook under `[hooks.state]` in the same config file that
declares the hook:

```toml
[hooks.state."<key>"]
trusted_hash = "<hash>"
```

The `<key>` is `<config path>:<event>:<group index>:<handler index>`, where
`<event>` is the lower-snake-case event name (for example `session_start`),
`<group index>` is the position of the matcher group, and `<handler index>` is
the position of the handler within that group. The `<hash>` (prefixed
`sha256:`) is taken over the hook's normalized TOML identity, so it changes
whenever the hook definition is edited; a stale hash reads as modified since it
was last trusted, and the hook is skipped again until it is re-trusted.

The interactive hook review writes these `trusted_hash` values for you. There
is no supported non-interactive command to set them today, so an automated or
headless run has two options: trust the hooks once through the interactive
review (the values then persist in the config file and later headless runs
honor them), or pass `--dangerously-bypass-hook-trust`, which runs every
enabled hook without the trust check for that one invocation. That flag is
dangerous and is intended only for automation that already vets its hook
sources.
