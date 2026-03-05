# Setup: Session Logs (File Monitor)

The file monitor inputs require two things: an environment variable pointing to the user's home directory, and filesystem permissions so the Cribl Edge process can read `~/.claude/` and its contents.

## Step 1: Set the `CLAUDE_HOME` Environment Variable

All file monitor paths use `$CLAUDE_HOME/.claude/<subdir>` to resolve their target directories. Set `CLAUDE_HOME` to the **home directory** of the user that runs Claude Code. Restart the Cribl service after setting it.

> Note: replace `<user>` with the user that runs Claude Code.

### Linux

```bash
export CLAUDE_HOME=/home/<user>
```

### macOS

```bash
export CLAUDE_HOME=/Users/<user>
```

### Windows

Set the environment variable for the Cribl Edge service. You can do this during MSI installation or via the system environment:

```powershell
# PowerShell (run as Administrator)
[System.Environment]::SetEnvironmentVariable("CLAUDE_HOME", "C:\Users\<user>", "Machine")
```

Or add it via **System Properties > Environment Variables > System variables**.

> After setting the variable, restart the Cribl Edge service for it to take effect.

```
Platform                 CLAUDE_HOME Value
─────────────────────────────────────────────────────────
Linux                    /home/{user}
macOS                    /Users/{user}
Windows                  C:\Users\{user}
```

> **Note:** Some recent Claude Code versions on Linux may use `~/.config/claude/` instead of `~/.claude/`. Verify which directory contains your session files before configuring the input.

## Step 2: Grant Filesystem Permissions

Claude Code creates files with restrictive permissions. The approach to granting Cribl Edge read access differs by platform.

### Linux

Cribl Edge typically runs as the `cribl` user. Since Claude Code creates files with mode `0600` (owner-only), POSIX ACLs are required to grant read access without changing file ownership.

**Directory ACLs:**

```bash
# Grant cribl user access to traverse the .claude directory tree
setfacl -R -m u:cribl:r-x /home/<user>/.claude

# Set default ACLs so new subdirectories inherit permissions
setfacl -R -d -m u:cribl:r-x /home/<user>/.claude/
setfacl -R -d -m m::r-x /home/<user>/.claude/
```

**File ACL Mask Fix:**

Claude Code creates files with mode `0600`, which sets the POSIX ACL mask to `---` and nullifies all ACL grants. Fix existing files and directories:

```bash
# Fix mask on files (r-- is sufficient for regular files)
find /home/<user>/.claude/ -type f -exec setfacl -m m::r {} +

# Fix mask on directories (r-x needed for traversal)
find /home/<user>/.claude/ -type d -exec setfacl -m m::r-x {} +
```

**Cron Job for New Files:**

Since Claude Code will keep creating files with mode `0600`, add a cron job to periodically fix ACL masks. This cron entry must be added to the **`cribl` user's** crontab (not root), so the ACL changes are applied in the correct user context:

```bash
# Edit the cribl user's crontab
sudo crontab -u cribl -e

# Add this entry (runs every 5 minutes)
*/5 * * * * find /home/<user>/.claude/ -type f ! -perm /g+r -exec setfacl -m m::r {} + 2>/dev/null
```

**Verification:**

```bash
# Confirm cribl user can read a session file
sudo -u cribl cat /home/<user>/.claude/projects/<project>/<session-id>.jsonl | head -1

# Check that ACL mask is r-- (not ---) on .jsonl files
getfacl /home/<user>/.claude/projects/<project>/<session-id>.jsonl | grep mask
# Expected: mask::r--
```

### macOS

On macOS, Cribl Edge is typically installed under the current user account (not a separate `cribl` service user). If Edge is running as the same user that owns the Claude Code session files, no additional permission changes are needed.

If Edge runs as a different user, grant read access using macOS ACLs:

```bash
# Grant the cribl user read access to the .claude directory tree
chmod -R +a "cribl allow read,readattr,readextattr,readsecurity,list,search" /Users/<user>/.claude/
```

> **Note:** macOS does not support POSIX ACL masks the way Linux does. The `chmod +a` syntax is macOS-specific (uses the NFSv4 ACL model). A periodic `launchd` job or cron entry may still be needed if new files are created with restrictive permissions.

**Verification:**

```bash
# List ACLs on the .claude directory
ls -le /Users/<user>/.claude/

# Test read access (if running Edge as a different user)
sudo -u cribl cat /Users/<user>/.claude/projects/<project>/<session-id>.jsonl | head -1
```

### Windows

On Windows, Cribl Edge runs as the `LocalSystem` account by default. `LocalSystem` has broad read access to local files, so in most cases no additional permissions are required.

If Cribl Edge is configured to run under a custom service account, grant that account read access to the Claude Code projects directory:

```powershell
# PowerShell (run as Administrator)
$path = "C:\Users\<user>\.claude"
$acl = Get-Acl $path
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "<SERVICE_ACCOUNT>", "ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
)
$acl.AddAccessRule($rule)
Set-Acl $path $acl
```

Or use `icacls` from an elevated command prompt:

```cmd
icacls "C:\Users\<user>\.claude" /grant "<SERVICE_ACCOUNT>:(OI)(CI)RX" /T
```

**Verification:**

```powershell
# Check effective permissions
icacls "C:\Users\<user>\.claude"

# Test that the Cribl service can read a session file
Get-Content "C:\Users\<user>\.claude\projects\<project>\<session-id>.jsonl" -First 1
```
