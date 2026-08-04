Repo: https://github.com/YapiSamuel/VIGIL_2

# 🛡️ VIGIL — Pre-Execution Malware Triage CLI

VIGIL is a command-line tool that helps a security analyst answer one question fast: *is this suspicious script safe to hand off, or should I escalate it now?* It statically analyzes `.sh`, `.ps1`, and `.py` files (and archives of them) and returns a structured, explainable verdict in under 30 seconds — without ever running the file and without needing a sandbox.

## 🔍 Features
- Recursively unwraps layered obfuscation (base64, hex, ROT13, URL/`\x` encoding, gzip/zlib/bz2, and PowerShell `-EncodedCommand`) and analyzes **every** decoded layer, so a hidden C2 address is caught like plaintext
- Extracts indicators of compromise (IPs, domains, URLs) with defanging support and private-range filtering, each traced back to the exact layer it came from
- Detects malicious behavior with pattern matching and optional YARA rules, mapped to MITRE ATT&CK technique IDs
- Enriches findings with threat intelligence (VirusTotal, AbuseIPDB, URLhaus), hash-first so the sample is never uploaded without an explicit flag
- Produces a deterministic risk score plus a plain-English explanation where every claim cites the specific signal behind it
- Runs with zero configuration and no API keys, degrading gracefully to full local analysis instead of failing

## 🧰 Skills Demonstrated
- Secure software design: a strict "nothing is ever executed" model, network access confined to a single auditable module, and a written threat model
- Defensive parsing of hostile input — protection against zip-slip, decompression bombs, and recursive decode bombs
- Modular pipeline architecture with clear contracts between stages
- Working with external REST APIs, rate limiting, and graceful failure handling
- Test-driven development: 110 unit tests, including deliberately malformed and adversarial input
- Python (standard library first), SQLite caching, regular expressions, and CLI design

## 🎯 Outcome
The result is a focused, honest triage tool built the way it would need to be for a real Security Operations Center — safe by construction, explainable in every finding, and clear about its own limits. Building it deepened my understanding of malware obfuscation techniques, static analysis, secure input handling, and how to engineer a security tool that an analyst can actually trust. The project is open source and remains a work in progress.
