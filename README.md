# MCP-Security: Vulnerability → MITRE ATT&CK Mapping Prototype

A team intern project from Summer 2025. We prototyped an AI agent that takes vulnerability scan findings and maps them to the MITRE ATT&CK techniques an attacker could use. The goal was to help analysts prioritize patching by showing what a vulnerability lets an attacker do.

> **Status:** Unfinished prototype. The internship ended before the full pipeline was complete. The code is shared as-is to show the approach.

## My contribution

I wrote **`test2.py`**, the technique extraction script:

- Loads the MITRE ATT&CK Enterprise dataset (STIX 2.x JSON)
- Keeps only `attack-pattern` objects (techniques and sub-techniques)
- Saves each technique's **ATT&CK ID, name, and description** to `results.json`

Output: **679 techniques** (211 parent techniques + 468 sub-techniques), a clean reference list for matching vulnerability findings to attacker techniques.

The other components were built by teammates.

## How the prototype fits together

```
Vulnerability scan findings (JSON)
          │
          ▼
LLM (OpenAI, function calling)  ──calls──▶  MITRE ATT&CK MCP server
          │                                   (lookup tools over ATT&CK STIX data)
          ▼
Report agent (LangGraph)  →  techniques mapped to each finding
```

| File | Purpose | State |
|---|---|---|
| `Mitre Attack/test2.py` | Extracts all ATT&CK techniques into `results.json` | Working |
| `Mitre Attack/results.json` | 679 techniques: ID, name, description | Output |
| `Mitre Attack/mitre_mcp_server.py` | MCP server exposing ATT&CK lookups (techniques, tactics, groups, software, mitigations) for enterprise, mobile, and ICS | Working |
| `Mitre Attack/mcp_caller.py` | Sends a vulnerability description to an LLM, which picks an ATT&CK lookup to run on the MCP server | Partial |
| `Mitre Attack/agent.py` | LangGraph orchestrator/worker skeleton for building a report | Skeleton |
| `Mitre Attack/enriched_scan_results.json` | Two sample scan findings (port, CVSS, severity, CVE references, remediation) | Sample data |
| `Mitre Attack/mcp_server.py` | Minimal MCP demo used while learning the framework | Scaffold |
| `Mitre Attack/data/` | Public MITRE ATT&CK STIX datasets (v17.1) | Public data |

## Example mapping

| Scan finding | Severity | Related ATT&CK technique |
|---|---|---|
| FTP login with default credentials (21/tcp) | High · CVSS 7.5 | T1078.001 Valid Accounts: Default Accounts |
| phpinfo() page exposed (80/tcp) | Medium · CVSS 5.3 | Information disclosure that helps an attacker with reconnaissance |

## Tech used

Python · MITRE ATT&CK (STIX 2.x) · mitreattack-python · FastMCP (Model Context Protocol) · OpenAI API (function calling) · LangGraph

## Run the extraction script

```bash
cd "Mitre Attack"
python test2.py   # reads data/enterprise-attack.json, writes results.json
```

## Next steps if continued

- Add each technique's tactic (e.g. Initial Access, Credential Access) to `results.json`
- Skip revoked or deprecated techniques when using newer ATT&CK releases
- Return MCP results to the LLM so it can write the final mapping
- Finish the LangGraph agent to produce a report for each finding

## Note

This repo contains only public MITRE ATT&CK data and sample scan findings. It contains no internal or customer data.
