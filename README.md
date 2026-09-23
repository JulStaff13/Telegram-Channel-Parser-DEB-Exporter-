# Telegram Channel Parser & DEB Exporter (`telegram-parser-deb`)

**telegram-parser-deb** is a tool designed for system administrators, analysts, and researchers to parse public Telegram channels on Debian-based Linux distributions (Debian, Ubuntu, Linux Mint). The utility extracts message histories for specified date ranges, downloads media files in maximum quality, and generates structured PDF reports.

It is structured for building and distribution as a native `.deb` package, allowing straightforward dependency management on Debian systems and CLI execution via terminal.

---

## 🛠 Core Logic and Architecture

1. **Authentication and Data Fetching:**
* Operates via an asynchronous **Telethon** client connecting to the Telegram API.


* Resolves target channels via handles or links (`@username`) and filters messages strictly within a user-specified date window (`Start Date` to `End Date`).




2. **Media Group Handling (Albums):**
* Messages sharing a `grouped_id` are processed as a single logical post. Post captions and text are extracted from the primary message in the group.




3. **Asynchronous Media Download:**
* Downloads are managed concurrently using an `asyncio.Semaphore(5)` limit.


* Implements chunked downloading with resume capabilities and automatic retry handling for expired media references (`FileReferenceExpiredError`).


* When PDF-only mode is active (`pdf_only = True`), images are cached temporarily for PDF generation and deleted immediately after.




4. **PDF Generation and Chunking:**
* Reports are built using `FPDF2`, containing the message ID, timestamp, channel title, direct post URL, and cleaned message text.


* Attached images are embedded into the document layout.


* **Automatic File Chunking:** To prevent single PDF files from growing excessively large, reports split into separate files (Part 1, Part 2, etc.) once the file size reaches approximately ~95 MB.




5. **Unicode and Emoji Sanitization:**
* Regular expressions strip non-standard Unicode glyphs and emojis to prevent rendering crashes or missing character warnings with standard TrueType fonts (`DejaVuSans`).





---

## 📂 Repository File Structure

```text
telegram-parser-deb/
├── DEBIAN/
│   └── control          # Package metadata (version, architecture, dependencies)
├── usr/
│   ├── bin/
│   │   └── telegram-parser   # CLI entry point script/symlink
│   └── share/telegram-parser/
│       └── main.py      # Core application logic
├── build/               # Output directory for compiled .deb packages
├── build.sh             # Setup script for workspace packaging
├── rebuild.py           # Automatic version bump and build script
├── rebuild.sh           # Shell wrapper script for rebuilding
├── ReadMe.png           # CLI usage demonstration screenshot
├── README.md            # Repository documentation
├── LICENSE              # MIT License
└── .gitignore           # Git ignore configuration

```

---

## 📄 Detailed File Descriptions

### 1. `rebuild.py` (Automated Build & Deployment Tool)

A utility script that streamlines the package release pipeline:

* **Reads Version:** Parses the current version field from `DEBIAN/control` (e.g., `Version: 2.0.7`).


* **Bumps Patch Version:** Increments the minor patch version automatically (e.g., `2.0.7` ➔ `2.0.8`).


* **Updates Control File:** Writes the updated version back to `DEBIAN/control`.


* **Compiles Package:** Triggers `dpkg-deb --build` to compile the tree into a installable package inside `./build/telegram-parser_X.X.X_all.deb`.


* **Reinstalls Package:** Runs `sudo dpkg -i` to update the local installation to the newly compiled version.



### 2. `main.py` (Core Application)

Contains the primary business logic, including `TelegramParser`, `BackgroundGenerator`, and `PDFReport` classes:

* **`get_version()`:** Queries `dpkg-query` or reads the control file to display the correct runtime package version.


* **`BackgroundGenerator`:** Procedurally builds image backgrounds (gradient, geometric, texture, wave) using `Pillow` for generated post images.


* **`process_channel()`:** Manages user prompt inputs, executes Telegram queries, handles concurrent task orchestration, and updates `Rich` progress bars.



### 3. `DEBIAN/control`

Debian package management specification file. Important fields include:

* `Package`: Package identifier (`telegram-parser`).


* `Version`: Active version identifier (e.g., `2.0.11`).


* `Depends`: System runtime dependencies (`python3`, `python3-pip`, `fonts-dejavu`, etc.).



### 4. `build.sh` / `rebuild.sh`

Shell wrappers that handle file permission setup (`chmod +x`), clean output build folders, and execute `rebuild.py` or `dpkg-deb` from the command line.

---

## 🚀 Usage Instructions

### Rebuilding and Updating Package:

```bash
python3 rebuild.py

```

Increments the patch version, builds a new `.deb` package in `./build/`, and installs it on the host system.

### Running the Application:

```bash
telegram-parser

```

Launches the interactive CLI tool to configure output directories, dates, and parsing modes.
