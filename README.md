<h1>YouTube Auto Upload for Mac</h1>

<p>Automated YouTube uploads on macOS using a simple, folder-based workflow.</p>

<p>This project is designed for creators, editors, and travelers who want to:</p>
<ul>
  <li>export a video</li>
  <li>drop it into a folder</li>
  <li>walk away</li>
  <li>and let YouTube upload happen automatically</li>
</ul>

<hr />

<h2>What this project IS</h2>
<ul>
  <li>A macOS folder-based workflow</li>
  <li>A shell + Python wrapper around <code>tokland/youtube-upload</code></li>
  <li>Designed for real-world creator workflows (Final Cut Pro, Compressor, travel setups)</li>
  <li>Fully local, no cloud services involved</li>
</ul>

<hr />

<h2>What this project is NOT</h2>
<ul>
  <li>❌ Not a GUI application</li>
  <li>❌ Not a YouTube client replacement</li>
  <li>❌ Not a fork of <code>tokland/youtube-upload</code></li>
  <li>❌ Not cross-platform</li>
</ul>

<hr />

<h2>Requirements</h2>
<ul>
  <li>macOS</li>
  <li>Homebrew</li>
  <li>Python 3 (installed via Homebrew)</li>
  <li>A Google Cloud project with:
    <ul>
      <li>YouTube Data API v3 enabled</li>
      <li>OAuth <strong>Desktop App</strong></li>
      <li>Your Google account added as a <strong>test user</strong></li>
    </ul>
  </li>
</ul>

<hr />

<h2>Folder structure (this matters)</h2>

<p>This workflow relies on a <strong>fixed folder structure</strong> located in <code>Downloads</code>.</p>
<p>On your Mac, you will end up with:</p>

<pre><code>
~/Downloads/YouTube Auto Upload/
├── 0 - ADMIN
│   ├── auto_upload.sh
│   ├── client_secrets.json
│   └── .venv/
├── 1 - INBOX
├── 2 - DONE
├── 3 - FAILED
└── 4 - LOGS
</code></pre>

<h3>Folder roles</h3>
<ul>
  <li><strong>1 - INBOX</strong> → drop finished videos here</li>
  <li><strong>2 - DONE</strong> → successful uploads</li>
  <li><strong>3 - FAILED</strong> → failed uploads</li>
  <li><strong>4 - LOGS</strong> → upload logs</li>
  <li><strong>0 - ADMIN</strong> → scripts, Python environment, credentials</li>
</ul>

<hr />

<h2>Step-by-step installation</h2>

<h3>1. Install Homebrew (if needed)</h3>
<pre><code>
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
</code></pre>

<hr />

<h3>2. Install Python</h3>
<pre><code>
brew install python
</code></pre>

<hr />

<h3>3. Create the folder structure</h3>
<pre><code>
mkdir -p "$HOME/Downloads/YouTube Auto Upload"/{"0 - ADMIN","1 - INBOX","2 - DONE","3 - FAILED","4 - LOGS"}
</code></pre>
<p>This creates all required folders on your Mac.</p>

<hr />

<h3>4. Google Cloud configuration (manual)</h3>
<ol>
  <li>Go to <strong>Google Cloud Console</strong></li>
  <li>Create a new project</li>
  <li>Enable <strong>YouTube Data API v3</strong></li>
  <li>Create an <strong>OAuth Client ID</strong>
    <ul>
      <li>Application type: <strong>Desktop</strong></li>
    </ul>
  </li>
  <li>Download the file named <code>client_secrets.json</code></li>
  <li>Add your Google account as a <strong>test user</strong></li>
</ol>

<p>Place the downloaded file here:</p>
<pre><code>
~/Downloads/YouTube Auto Upload/0 - ADMIN/client_secrets.json
</code></pre>

<p>⚠️ This file is private. Never commit it.</p>

<hr />

<h3>5. Create the Python virtual environment</h3>
<pre><code>
cd "$HOME/Downloads/YouTube Auto Upload/0 - ADMIN"

/opt/homebrew/bin/python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install git+https://github.com/tokland/youtube-upload.git
</code></pre>

<p>This creates a local Python environment inside <code>0 - ADMIN/.venv</code>.</p>

<hr />

<h3>6. Create the upload script</h3>

<p>Create the file:</p>
<pre><code>
~/Downloads/YouTube Auto Upload/0 - ADMIN/auto_upload.sh
</code></pre>

<p>Paste the following content:</p>

<pre><code>
#!/bin/zsh
set -euo pipefail

ROOT="$HOME/Downloads/YouTube Auto Upload"
ADMIN="$ROOT/0 - ADMIN"
DONE="$ROOT/2 - DONE"
FAILED="$ROOT/3 - FAILED"
LOGS="$ROOT/4 - LOGS"

FILE="$1"
BASE="$(basename "$FILE")"
TITLE="${BASE%.*}"
LOG="$LOGS/${TITLE}.log"

mkdir -p "$DONE" "$FAILED" "$LOGS"

# Wait until the file size is stable (export finished)
prev=0
stable=0
while [ $stable -lt 3 ]; do
  size=$(stat -f%z "$FILE" 2>/dev/null || echo 0)
  if [ "$size" -gt 0 ] && [ "$size" -eq "$prev" ]; then
    stable=$((stable+1))
  else
    stable=0
  fi
  prev="$size"
  sleep 5
done

source "$ADMIN/.venv/bin/activate"

if python -m youtube_upload \
  --client-secrets="$ADMIN/client_secrets.json" \
  --privacy=private \
  --title="$TITLE" \
  "$FILE" >> "$LOG" 2>&1
then
  mv "$FILE" "$DONE/$BASE"
else
  mv "$FILE" "$FAILED/$BASE"
fi
</code></pre>

<p>Make it executable:</p>
<pre><code>
chmod +x "$HOME/Downloads/YouTube Auto Upload/0 - ADMIN/auto_upload.sh"
</code></pre>

<hr />

<h3>7. Automator configuration (Folder Action)</h3>
<ol>
  <li>Open <strong>Automator</strong></li>
  <li>Create a <strong>Folder Action</strong></li>
  <li>Target folder:
    <pre><code>
~/Downloads/YouTube Auto Upload/1 - INBOX
</code></pre>
  </li>
  <li>Add action: <strong>Run Shell Script</strong></li>
  <li>Shell: <code>/bin/zsh</code></li>
  <li>Pass input: <strong>as arguments</strong></li>
  <li>Script:
    <pre><code>
for f in "$@"; do
  "$HOME/Downloads/YouTube Auto Upload/0 - ADMIN/auto_upload.sh" "$f" &
done
</code></pre>
  </li>
  <li>Save the workflow</li>
</ol>

<hr />

<h2>First test</h2>
<ol>
  <li>Export a <strong>finished</strong> <code>.mp4</code> file</li>
  <li>Copy it into:
    <pre><code>
~/Downloads/YouTube Auto Upload/1 - INBOX
</code></pre>
  </li>
  <li>On first run:
    <ul>
      <li>A browser window opens</li>
      <li>Authorize YouTube access</li>
    </ul>
  </li>
  <li>Upload starts automatically</li>
  <li>The file is moved to:
    <ul>
      <li><code>2 - DONE</code> on success</li>
      <li><code>3 - FAILED</code> on error</li>
    </ul>
  </li>
  <li>Logs are written to <code>4 - LOGS</code></li>
</ol>

<hr />

<h2>Notes</h2>
<ul>
  <li>OAuth authorization happens <strong>once</strong></li>
  <li>Partial exports are ignored (file size stability check)</li>
  <li>No background daemon</li>
  <li>No launch agents</li>
  <li>Fully transparent and debuggable</li>
</ul>

<hr />

<h2>Credits</h2>
<p>This project relies on the excellent work from:</p>
<ul>
  <li>https://github.com/tokland/youtube-upload</li>
</ul>

<p>This repository only provides a macOS-oriented workflow around it.</p>

<hr />

<h2>How it works (and why)</h2>

<p>
This workflow is intentionally simple, transparent, and robust.
It relies on a few pragmatic design decisions to handle real-world
video export scenarios (Final Cut Pro, Compressor, long renders, travel setups).
</p>

<h3>1. No background daemon, no LaunchAgent</h3>

<p>
There is no background service running permanently on your Mac.
The workflow is triggered only when a file is added to the
<code>1 - INBOX</code> folder via an Automator Folder Action.
</p>

<p>
This makes the system:
</p>
<ul>
  <li>easy to debug</li>
  <li>easy to disable</li>
  <li>safe to use while traveling</li>
  <li>fully visible (no hidden processes)</li>
</ul>

<h3>2. Handling partial and temporary export files</h3>

<p>
Video export tools such as Final Cut Pro or Compressor often create
temporary or segmented files while exporting.
Triggering an upload too early would result in:
</p>

<ul>
  <li>uploading an incomplete file</li>
  <li>uploading multiple temporary versions</li>
  <li>failed or corrupted YouTube uploads</li>
</ul>

<p>
To avoid this, the script does <strong>not</strong> upload the file immediately.
Instead, it waits until the file size becomes stable.
</p>

<p>
Concretely:
</p>
<ul>
  <li>the file size is checked every few seconds</li>
  <li>the upload starts only after the size stops changing</li>
  <li>this guarantees that the export is fully finished</li>
</ul>

<p>
This approach is intentionally simple and avoids complex file locking,
filesystem watchers, or platform-specific APIs.
</p>

<h3>3. Why a dedicated INBOX folder</h3>

<p>
The <code>1 - INBOX</code> folder acts as a clear boundary between:
</p>

<ul>
  <li>export tools (Final Cut, Compressor, etc.)</li>
  <li>the upload system</li>
</ul>

<p>
Only <strong>finished files</strong> should be placed in this folder.
Everything inside it is considered eligible for upload.
</p>

<p>
This keeps the workflow predictable and avoids guessing user intent.
</p>

<h3>4. Automatic post-upload file handling</h3>

<p>
After the upload attempt:
</p>

<ul>
  <li>successful uploads are moved to <code>2 - DONE</code></li>
  <li>failed uploads are moved to <code>3 - FAILED</code></li>
  <li>detailed logs are written to <code>4 - LOGS</code></li>
</ul>

<p>
This makes it easy to:
</p>

<ul>
  <li>see what has already been uploaded</li>
  <li>retry failed uploads manually</li>
  <li>inspect errors without re-running the script</li>
</ul>

<h3>5. Isolated Python environment</h3>

<p>
All Python dependencies are installed inside a local virtual environment
(<code>.venv</code>) located in <code>0 - ADMIN</code>.
</p>

<p>
This ensures:
</p>

<ul>
  <li>no dependency conflicts with system Python</li>
  <li>no global package pollution</li>
  <li>reproducible behavior across machines</li>
</ul>

<h3>6. One-time OAuth authentication</h3>

<p>
The first upload triggers a browser-based OAuth authorization flow.
Once accepted:
</p>

<ul>
  <li>a local credential file is created automatically</li>
  <li>future uploads happen without user interaction</li>
</ul>

<p>
No credentials are sent anywhere except directly to Google.
</p>

<h3>7. Why this is not a fork</h3>

<p>
This repository does not modify or fork
<code>tokland/youtube-upload</code>.
</p>

<p>
Instead, it acts as a <strong>workflow wrapper</strong> around it,
focused on macOS usability, automation, and creator-oriented ergonomics.
</p>
