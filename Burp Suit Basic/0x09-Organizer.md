# Organizer

Purpose

Organizer helps you capture, classify, and manage findings, notes, and observations during testing. It keeps evidence and status metadata in a structured way so that reporting is easier.

Core features

- Add Note: create a new note with title, description, category, and optional status.
- Classification: tag items, highlight rows with colors, and set statuses (New, In Progress, Done).
- Right-click actions: quick management options such as edit, delete, or change status.

Best practices

- Keep notes concise and repeatable: include where, how to reproduce, and severity.
- Use categories for triage (e.g., High/Medium/Low or Auth/Logic/InfoLeak).
- Link notes to specific requests or evidence by sending the request/response to Organizer when possible.

Example note structure

- Title: Insecure direct object reference on /user/download
- Description: Download parameter accepts arbitrary file paths. Repro steps: 1) Login as user A, 2) Request /download?file=../../etc/passwd, 3) Server returned file contents.
- Status: New
- Severity: High

Tip: Use Organizer to store reproduction steps; it makes final report writing much faster.

