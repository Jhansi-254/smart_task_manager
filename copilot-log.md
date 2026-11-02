# Project: Smart Task Manager
Stack: Python, Streamlit, SQLite, pytest.

## Architecture (do not mix layers)
- UI: app.py, dashboard_ui.py, collab_ui.py. Streamlit code only, no SQL.
- Pure logic: task_logic.py. Must NOT import streamlit or touch the database.
- Data: database.py (tasks) and collab_db.py (users, follows, plans, plan_members,
  plan_invitations, task_shares). All SQL lives here. collab_db.py also enforces
  the collaboration rules that need data, and handles authentication.
- dashboard_html.py only builds HTML/CSS strings.

## Database rules
- Values from users or runtime data must use ? placeholders, never concatenation.
- Dynamic SQL identifiers (e.g. a column name in a migration) must come from a fixed allowlist.
- Schema changes must be non-destructive (add nullable columns, never drop data).
- Due dates are ISO date strings (YYYY-MM-DD); past due dates are allowed (needed for overdue).
  created_at values are ISO timestamps with timezone.
- Keep the migration that assigns legacy unowned personal tasks to the first registered user.

## Business rules
- Priority: High, Medium or Low. Status: Pending, In Progress or Completed.
- Completed tasks are never overdue.
- Passwords: salted PBKDF2-SHA256 hashes only, never plain text, never logged.
- Logged-in user lives in st.session_state["user"]; task pages require login.
- Personal task views show the user's own tasks plus accepted shares. Recipients may
  change status of a shared task; only the owner may edit, delete or stop sharing.
- Plan tasks are visible only to plan members. Only members may add plan tasks or invite;
  assignments must be to members. Invite only people you follow; one pending invite per person per plan.
- Cannot follow yourself or follow twice.

## Security / output
- Escape user-controlled text for its output context: html.escape for HTML,
  md_safe for Markdown. Never put unchecked text into markup.

## Code style
- Small functions, meaningful names, type hints.
- Show expected errors (validation, collaboration, database) to the user with st.error.
  Catch specific exceptions only; never silently swallow unexpected ones.

## Testing
- Write pytest tests for every new logic or database function.
- Run `pytest -v` before every commit.
