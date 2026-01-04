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
