# ENCRYPT

> Data protection and encryption requirements for AI agents operating in this repository.
> Spec version: 1.0 | Full specification: https://encrypt.md

---

## PURPOSE

`ENCRYPT.md` defines what data the agent must encrypt, how it must handle secrets, and what it must never transmit in plaintext. It is the security layer of the AI safety stack.

---

## DATA CLASSIFICATION

classify:
  critical:                         # Must be encrypted at rest AND in transit. Never logged.
    - api_keys
    - passwords
    - private_keys
    - oauth_tokens
    - session_tokens
    - payment_card_data
    - health_records
    - biometric_data

  sensitive:                        # Must be encrypted in transit. Masked in logs.
    - email_addresses
    - phone_numbers
    - full_names
    - ip_addresses
    - location_data
    - user_ids

  internal:                         # Standard protection. No external transmission.
    - business_logic
    - internal_configurations
    - non-public_pricing
    - client_lists

  public:                           # No restrictions.
    - marketing_content
    - published_documentation
    - open_source_code

---

## ENCRYPTION REQUIREMENTS

at_rest:
  algorithm: AES-256-GCM
  key_management: environment_variable    # Never hardcode keys
  key_variable: ENCRYPT_KEY
  applies_to:
    - all_critical_data
    - all_sensitive_data
    - agent_memory_files
    - session_state_files

in_transit:
  minimum_tls: "1.2"
  preferred_tls: "1.3"
  certificate_validation: strict          # Never skip TLS verification
  hsts: true
  applies_to:
    - all_external_api_calls
    - all_webhook_payloads
    - all_file_transfers

---

## SECRETS HANDLING

rules:
  - never_log_secrets: true               # Secrets MUST never appear in any log file
  - never_hardcode: true                  # Secrets MUST come from env vars or a vault
  - never_commit: true                    # Secrets MUST never be committed to git
  - never_transmit_plaintext: true        # Secrets MUST never be sent over HTTP

secret_sources:                           # Approved sources for secrets (in priority order)
  - environment_variables
  - secrets_manager                       # e.g. AWS Secrets Manager, HashiCorp Vault
  - encrypted_config_file                 # e.g. .env.encrypted (never .env in plain)

on_secret_detected_in_output:
  action: redact_and_log                  # Replace with [REDACTED], log the event
  notify: true
  channels:
    - email: security@example.com

forbidden_patterns:                       # Patterns that MUST trigger redaction
  - regex: "sk-[a-zA-Z0-9]{32,}"         # OpenAI-style API keys
  - regex: "AKIA[A-Z0-9]{16}"            # AWS access key IDs
  - regex: "ghp_[a-zA-Z0-9]{36}"         # GitHub personal access tokens
  - regex: "-----BEGIN .* PRIVATE KEY-----"  # Private keys

---

## DATA RETENTION

retain:
  session_logs_days: 90
  audit_logs_days: 365               # Compliance minimum
  user_data_days: 0                  # Do not retain user PII beyond session unless required

purge:
  method: secure_delete              # Overwrite before deletion
  verify_deletion: true              # Confirm data is unrecoverable

---

## TRANSMISSION RULES

the_agent_must_never:
  - send_critical_data_to_llm        # Never include secrets in LLM prompts
  - include_pii_in_logs              # Never write PII to log files
  - cache_credentials_to_disk        # No credential caching on filesystem
  - send_data_to_unapproved_endpoints  # Only transmit to endpoints in allowlist

approved_endpoints: []               # List your approved external endpoints here

---

## AUDIT

log_file: .encrypt.log
log_format: jsonl
log_fields:
  - timestamp
  - event_type                       # e.g. secret_detected, encryption_failure
  - data_classification
  - action_taken
  - session_id

---

## METADATA

owner: your-name-or-org
contact: ops@example.com
security_contact: security@example.com
last_reviewed: 2026-03-10
review_frequency: quarterly
compliance_frameworks:
  - GDPR
  - SOC 2 Type II
  - ISO 27001
  - EU AI Act (2026)
spec_version: "1.0"
spec_url: https://encrypt.md
