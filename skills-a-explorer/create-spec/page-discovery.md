# PAGE Discovery

## Step 1: Read the controller and all templates

Read the controller action source code. Then read every template file referenced by the controller. Recursively read any partial templates included by those templates.

## Step 2: Capture the page UI (subagent)

**Prerequisites**: Read `.env` to get login credentials. If `.env` does not exist or is not filled, stop and ask the user to provide credentials.

Launch a **sub-agent** to capture the UI (keeps browser overhead out of main context). Pass it credentials and the page URL.

The sub-agent must:
1. Login to the application:
   ```
   agent-browser open "path/to/login-url"
   agent-browser snapshot -i
   agent-browser fill @<username-field> "$USERNAME"
   agent-browser fill @<password-field> "$PASSWORD"
   agent-browser click @<login-button>
   agent-browser snapshot -i
   ```
2. Navigate to the page and capture:
   ```
   agent-browser open "path/to/base-url<page-url>"
   agent-browser snapshot > path/to/features/<feature>/browser/snapshot.txt
   agent-browser screenshot path/to/features/<feature>/browser/screenshot.png
   agent-browser screenshot --full path/to/features/<feature>/browser/screenshot-full.png
   ```
3. Return the list of generated files.

After the sub-agent returns, read the snapshot file.

## Step 3: Build the migration slice

From the templates, the snapshot, and the controller, extract **every** touchpoint reachable from the page:
- Form actions (POST endpoints)
- AJAX calls ($.ajax, $.post, $.get, fetch)
- File downloads / exports
- Modal content fetches
- Links (`<a href=...>`) — navigation only, not part of the slice
- Redirects in the controller — redirect targets are NOT part of this slice

For each touchpoint found, determine whether it is in scope or not: A touchpoint is **in scope** (✅) if it is **triggered** from this page and performs business logic or loads data (form submit, AJAX call, delete, update, data fetch) — regardless of whether it redirects elsewhere after. A touchpoint is **out of scope** (❌) if it is just a navigation link to another page with no processing (`<a href>`).