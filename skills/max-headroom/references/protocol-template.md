# max-headroom.script
# schema: max-headroom.protocol/v1
# Load at the start of a Grok session to engage the installed Headroom package.
# Filled by the max-headroom skill. Do not invent paths — substitute from the live install.

[identity]
checkout = {{CHECKOUT}}
tag = {{TAG}}
version = {{VERSION}}
bin = {{HEADROOM_BIN}}
repo = https://github.com/headroomlabs-ai/headroom
pypi = headroom-ai

[runtime]
proxy_host = 127.0.0.1
proxy_port = 8787
proxy_url = http://127.0.0.1:8787
health_url = http://127.0.0.1:8787/health
dashboard_url = http://127.0.0.1:8787/dashboard
compress_url = http://127.0.0.1:8787/v1/compress
wrap_target = {{WRAP_TARGET}}
mcp_command = {{HEADROOM_BIN}}
mcp_args = mcp serve
grok_config = {{GROK_HOME}}/config.toml

[session]
# 1. If health_url is not 200/healthy, start: `{{HEADROOM_BIN}} proxy --port 8787` (background).
# 2. Use MCP tools headroom_compress, headroom_retrieve, headroom_stats when present.
# 3. Large tool/log/file payloads without MCP: POST compress_url with the real model name.
#    Multi-turn: set config.frozen_message_count to the already-forwarded prefix length;
#    resend previously forwarded messages, never the pristine originals.
# 4. On <<ccr:...>> markers, retrieve the original before answering from the stub.
# 5. Durable wrap (next Grok launch): `{{HEADROOM_BIN}} wrap {{WRAP_TARGET}}`
#    Undo: `{{HEADROOM_BIN}} unwrap {{WRAP_TARGET}}`
# 6. Do not reinstall, unwrap, or change HEADROOM_SAVINGS_PROFILE unless asked.
# 7. Diagnose with `{{HEADROOM_BIN}} doctor` and `{{HEADROOM_BIN}} perf`.

[verify]
headroom_version_cmd = {{HEADROOM_BIN}} --version
doctor_cmd = {{HEADROOM_BIN}} doctor
