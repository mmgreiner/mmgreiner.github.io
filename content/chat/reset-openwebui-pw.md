## Problem: reset local Open-WebUI password

There is a sql database here: `~/Library/Application Support/open-webui/data`.

## Reset it:

Let's do this carefully — the tricky part is that Open-WebUI doesn't store plain text passwords, so we need to generate a properly-hashed password and drop it into the right table.

**Step 1: Check the schema and confirm the email you're targeting**

In Terminal, in that same `data` folder:
```bash
sqlite3 webui.db ".schema auth"
sqlite3 webui.db "SELECT id, email, active FROM auth;"
```

This shows the `auth` table (separate from `user`), which is what actually holds passwords. Note the `email` of the account you want to reset.

**Step 2: Generate a new bcrypt password hash**

macOS's built-in Python usually doesn't have `bcrypt` installed, so install it first:
```bash
pip3 install bcrypt --user
```

Then generate a hash for your new password (replace `YourNewPassword123` with whatever you want):
```bash
python3 -c "import bcrypt; print(bcrypt.hashpw(b'YourNewPassword123', bcrypt.gensalt()).decode())"
```

This prints a hash that looks like `$2b$12$....` — copy that whole string.

**Step 3: Update the database**

```bash
sqlite3 webui.db "UPDATE auth SET password='PASTE_HASH_HERE' WHERE email='the-email-you-found@example.com';"
```

Make sure to keep the single quotes around the hash and email exactly as shown.

**Step 4: Verify and log in**
- Quit and relaunch the Open-WebUI app fully (not just close the window).
- Sign in with that email and `YourNewPassword123`.

One thing worth checking before you do this: what did the `SELECT id, name, email, role FROM user;` query show earlier? If that account's `role` isn't `admin`, resetting the password will get you logged in, but not as admin — worth confirming that first so we don't do this twice.
