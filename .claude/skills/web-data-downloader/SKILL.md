---
name: web-data-downloader
description: Automate browser tasks to login to a website, navigate to data sections, apply filters, download Excel/XLS/CSV files, and optionally upload to cloud storage. Use when the user needs to automate repetitive data download workflows from web portals.
arguments: [url, username, password]
allowed-tools: mcp__playwright__browser_navigate mcp__playwright__browser_snapshot mcp__playwright__browser_click mcp__playwright__browser_type mcp__playwright__browser_fill_form mcp__playwright__browser_wait_for mcp__playwright__browser_take_screenshot mcp__playwright__browser_file_upload mcp__playwright__browser_select_option
disable-model-invocation: true
---

# Web Data Downloader

Automate the complete workflow of logging into a website, navigating to specific sections, setting data filters, downloading Excel/XLS/CSV files, and optionally uploading them to cloud storage (Google Drive, Dropbox, etc.).

## Arguments

- **$0 (url)**: The website URL to navigate to (e.g., https://portal.example.com)
- **$1 (username)**: Login username (optional - will prompt if not provided)
- **$2 (password)**: Login password (optional - will prompt if not provided)

## Workflow

### 1. Navigate to the Website

First, navigate to the target website:

```
Navigate to: $0
```

Use `mcp__playwright__browser_navigate` to go to the URL.

### 2. Take Initial Snapshot

Take a snapshot to understand the page structure:

```
Take a snapshot of the page to identify login elements
```

Use `mcp__playwright__browser_snapshot` to see the page structure with element targets.

### 3. Login Process

Based on the snapshot, identify the login form elements and fill them:

**If username and password were provided as arguments ($1 and $2):**
- Use them directly

**If not provided:**
- Ask the user for credentials before proceeding

Fill the login form using one of these approaches:

**Option A: Use fill_form for complete forms**
```
Fill the login form with username and password fields
```

**Option B: Use type for individual fields**
```
1. Type username into the username field
2. Type password into the password field
3. Click the login/submit button
```

After submitting, wait for the page to load:
```
Wait for successful login (check for dashboard, profile menu, or URL change)
```

### 4. Navigate to Data Section

After login, take another snapshot to see the authenticated interface:

```
Take a snapshot to identify navigation to data/reports section
```

Then navigate to the data download area:
- Click on the menu item that leads to data/reports
- Click on sub-menu items if needed
- Wait for the data section to load

### 5. Apply Data Filters

Once in the data section:

1. **Take a snapshot** to identify filter controls (dropdowns, date pickers, checkboxes)

2. **Ask the user** what filters they want to apply:
   - Date range
   - Categories/types
   - Status filters
   - Any other relevant filters

3. **Apply each filter** using appropriate actions:
   - Use `mcp__playwright__browser_click` for dropdowns and buttons
   - Use `mcp__playwright__browser_type` for text input and date fields
   - Use `mcp__playwright__browser_select_option` for dropdown selections
   - Use `mcp__playwright__browser_fill_form` for multi-field forms

4. **Wait for results** to update after applying filters

### 6. Download the Data

Locate and click the download/export button:

1. Take a snapshot if needed to find the download button
2. Click the download/export button (look for "Export", "Download", "Export to Excel", etc.)
3. If a download dialog appears with format options, select XLS/Excel format
4. Wait for the download to complete

The file will be saved to your Downloads folder by default.

### 7. Verify Success

After download:

1. Take a final screenshot as confirmation
2. Check the Downloads folder for the new file
3. Report the filename and location to the user

### 8. Upload to Cloud Storage (Optional)

If the user needs the file uploaded to Google Drive, Dropbox, or other cloud storage:

1. **Navigate to the cloud storage URL** provided by the user
2. **Sign in** if not already authenticated (may require passkey or SSO)
3. **Navigate to the target folder** if specified
4. **Click upload button** (usually "New" → "File upload" or similar)
5. **Use file upload tool** to select and upload the downloaded file
6. **Wait for upload to complete** and verify success
7. **Report the file location** to the user

## Error Handling

- If login fails: Take a screenshot and ask user to verify credentials
- If elements are not found: Take a snapshot and ask user to describe the layout
- If filters don't apply: Try alternative selectors or ask user for guidance
- If download doesn't start: Look for alternative download buttons or export options

## Tips for Success

1. **Always take snapshots** before trying to interact with elements - this ensures you use correct targets
2. **Use descriptive element descriptions** when clicking/typing to get proper permission
3. **Wait after each major action** (login, navigation, filter application) for the page to stabilize
4. **Take screenshots** at key points to document the process
5. **Ask the user for clarification** if the page structure is unclear
6. **Handle SSO and MFA gracefully** - if Microsoft/Google SSO or multi-factor authentication is required, wait for user to complete the authentication steps
7. **Location/site selection matters** - many enterprise portals require selecting a specific location or site before data becomes available
8. **Be flexible with file formats** - if user requests Excel/XLS but only CSV is available, download the CSV and inform the user

## Example Usage

### Basic Usage

```
/web-data-downloader https://portal.example.com john.doe@company.com mypassword
```

Or invoke without arguments and provide details interactively:

```
/web-data-downloader
```

Then answer questions about the website URL, credentials, and filters as prompted.

### Real-World Example: Envoy Deliveries Dashboard

This example demonstrates downloading pending deliveries from an Envoy dashboard:

**Task:** Download pending deliveries for a specific location and upload to Google Drive

**Steps Completed:**

1. **Navigate and Login**
   - URL: https://dashboard.envoy.com/login
   - Credentials: corporate email (howard.tsang@diageo.com)
   - Handled Microsoft SSO authentication with MFA

2. **Change Location**
   - Clicked location dropdown at top left
   - Selected "Diageo London, UK" from location list

3. **Navigate to Data Section**
   - Clicked "Deliveries" in left side menu
   - System showed 2,232 total deliveries

4. **Apply Filters**
   - Clicked dropdown showing "All deliveries"
   - Selected "Pending" to filter only undelivered items
   - Filtered results: 277 pending deliveries

5. **Export Data**
   - Clicked "Export" button at top right
   - Downloaded: export20260618-1-1orplme.csv (277 records)
   - File includes: Date Received, Creator, Recipient, Carrier, Tracking Number, Delivery Area

6. **Upload to Google Drive** (Optional Extension)
   - Navigated to Google Drive shared folder
   - Signed in with Google account
   - Clicked "New" → "File upload"
   - Uploaded the exported CSV file

**Key Learnings:**
- Handle multi-factor authentication by waiting for user to complete MFA steps
- Location/site selection is often required before accessing data
- CSV exports are common even when user requests Excel/XLS format
- Taking snapshots before each action ensures correct element targeting
- Export buttons are typically in top-right or action bar areas

## Security Notes

- Credentials are only used during the current session
- Consider using environment variables for sensitive credentials
- Browser session data is not persisted between runs
- Always verify you're on the correct domain before entering credentials
