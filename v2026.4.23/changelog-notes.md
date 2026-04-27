# OpenClaw v2026.4.23 Changelog Notes

## Version Alignment
- **Target version**: v2026.4.23 (from package.json)
- **Unreleased changes**: Taken from CHANGELOG.md Unreleased section (as of analysis date)
- **Included features**: Includes all changes up to and including v2026.4.22

## New Features
- None identified in the Unreleased section (new features were added in v2026.4.22, such as TUI Embedded Mode).

## Fixes (Unreleased)
The following fixes are included in the Unreleased section and will be part of v2026.4.23:

1. **QQBot/security**: require framework auth for `/bot-approve` so unauthorized QQ senders cannot change exec approval settings through the unauthenticated pre-dispatch slash-command path. (#70706)
   - Implementation: Changes in QQBot plugin to enforce framework authentication for approval commands.
   - Evidence: Changelog entry
   - Confidence: High (directly from changelog)

2. **MCP/tools**: stop the ACPX OpenClaw tools bridge from listing or invoking owner-only tools such as `cron`, closing a privilege-escalation path for non-owner MCP callers. (#70698)
   - Implementation: Restrictions added to the ACPX OpenClaw tools bridge in the MCP integration.
   - Evidence: Changelog entry
   - Confidence: High

3. **Feishu/onboarding**: load Feishu setup surfaces through a setup-only barrel so first-run setup no longer imports Feishu's Lark SDK before bundled runtime deps are staged. (#70339)
   - Implementation: Adjusted Feishu onboarding to defer Lark SDK import until after runtime dependencies are available.
   - Evidence: Changelog entry
   - Confidence: High

4. **Approvals/startup**: let native approval handlers report ready after gateway authentication while replaying pending approvals in the background, so slow or failing replay delivery no longer blocks handler startup or amplifies reconnect storms.
   - Implementation: Modified approval handler initialization to decouple from pending replay delivery.
   - Evidence: Changelog entry
   - Confidence: High

5. **WhatsApp/security**: keep contact/vCard/location structured-object free text out of the inline message body and render it through fenced untrusted metadata JSON, limiting hidden prompt-injection payloads in names, phone fields, and location labels/comments.
   - Implementation: Changed WhatsApp message rendering to isolate untrusted structured data.
   - Evidence: Changelog entry
   - Confidence: High

6. **Group-chat/security**: keep channel-sourced group names and participant labels out of inline group system prompts and render them through fenced untrusted metadata JSON.
   - Implementation: Applied similar untrusted metadata rendering to group chat system prompts.
   - Evidence: Changelog entry
   - Confidence: High

7. **Plugins/startup**: restore bundled plugin `openclaw/plugin-sdk/*` resolution from packaged installs and external runtime-deps stage roots, so Telegram/Discord no longer crash-loop with `Cannot find package 'openclaw'` after missing dependency repair.
   - Implementation: Fixed plugin resolution paths for bundled plugins in packaged installations.
   - Evidence: Changelog entry
   - Confidence: High

8. **CLI/Claude**: run the same prompt-build hooks and trigger/channel context on `claude-cli` turns as on direct embedded runs, keeping Claude Code sessions aligned with OpenClaw workspace identity, routing, and hook-driven prompt mutations. (#70625)
   - Implementation: Unified prompt context handling between direct embedded runs and Claude CLI turns.
   - Evidence: Changelog entry
   - Confidence: High

9. **Discord/plugin startup**: keep subagent hooks lazy behind Discord's channel entry so packaged entry imports stay narrow and report import failures with the channel id and entry path.
   - Implementation: Delayed subagent hook initialization in Discord plugin to reduce entry point overhead.
   - Evidence: Changelog entry
   - Confidence: High

10. **Memory/doctor**: keep root durable memory canonicalized on `MEMORY.md`, stop treating lowercase `memory.md` as a runtime fallback, and let `openclaw doctor --fix` merge true split-brain root files into `MEMORY.md` with a backup. (#70621)
    - Implementation: Standardized memory file handling and added doctor command to merge split-brain files.
    - Evidence: Changelog entry
    - Confidence: High

11. **Providers/Anthropic Vertex**: restore ADC-backed model discovery after the lightweight provider-discovery path by resolving emitted discovery entries, exposing synthetic auth on bootstrap discovery, and honoring copied env snapshots when probing the default GCP ADC path. Fixes #65715. (#65716)
    - Implementation: Restored Application Default Credentials (ADC) flow for Vertex AI model discovery.
    - Evidence: Changelog entry
    - Confidence: High

12. **Codex harness/status**: pin embedded harness selection per session, show active non-PI harness ids such as `codex` in `/status`, and keep legacy transcripts on PI until `/new` or `/reset` so config changes cannot hot-switch existing sessions.
    - Implementation: Fixed session-specific harness selection and transcript persistence for Codex integration.
    - Evidence: Changelog entry
    - Confidence: High

## Breaking Changes
- None identified in the Unreleased section or v2026.4.22 changelog entries.

## Security Notes
- The Unreleased section includes multiple security fixes (marked with `/security` in the description).
- TUI Embedded Mode (carried over from v2026.4.22) reduces attack surface by eliminating network dependency for local terminal chats, while maintaining plugin approval gate enforcement.
- The Plugins/startup fix prevents crash-loops in Telegram/Discord by ensuring plugin resolution works in packaged installs, reducing the attack surface from missing dependency repair attempts.