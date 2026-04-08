# ArcGIS Pro Backup Manager

ArcGIS Pro Backup Manager is a Python toolbox built for ArcGIS Pro to make backups more usable, more controlled, and easier to maintain over time.

This tool gives users a way to set up a backup process properly, keep it organized, and come back later to adjust it without rebuilding everything from scratch. It supports both full project backups and map level backups, and it lets one profile hold more than one APRX so users are not forced into a one project at a time workflow.

The goal is simple. Set it up once, know it is there, and only come back when something actually needs to change.

## What This Tool Does

This tool allows users to:

Create named backup profiles

Store one or more APRX files in a single profile

Choose between full project backups using PPKX or map based backups using MPKX

Run backups manually when needed

Create scheduled backups

Load an existing schedule back into the form for review or revision

Enable or disable a schedule without deleting it

Update the maps tied to a profile

Remove schedules that are no longer needed

## Screenshots

### Main tool view
<img width="452" height="1107" alt="Screenshot 2026-04-08 at 9 17 28 AM" src="https://github.com/user-attachments/assets/f8503c77-7e0c-4ee4-ae95-aee29c04cce8" />

### First time profile setup
<img width="440" height="566" alt="Screenshot 2026-04-08 at 9 15 31 AM" src="https://github.com/user-attachments/assets/847af06f-8611-41d5-82ff-6da82460af86" />

### Schedule setup
<img width="442" height="345" alt="Screenshot 2026-04-08 at 9 19 14 AM" src="https://github.com/user-attachments/assets/6228d926-e3c2-4384-a856-04bef25ca8a9" />

Weekly Settings
<img width="448" height="553" alt="Screenshot 2026-04-08 at 9 19 59 AM" src="https://github.com/user-attachments/assets/e3aaad42-8af4-4d03-9107-51af013d23b2" />

### Run backup now
<img width="460" height="242" alt="Screenshot 2026-04-08 at 9 20 33 AM" src="https://github.com/user-attachments/assets/06a8f931-50c9-49ee-8aa1-6c4978f59403" />

### Load existing schedule into form
<img width="442" height="944" alt="Screenshot 2026-04-08 at 9 22 17 AM" src="https://github.com/user-attachments/assets/baf15748-17ae-4a3b-b2cc-4d617fd7c6a6" />

### Example output packages
![Example output packages](images/output-packages.png)

## Why It Exists

A lot of backup processes around ArcGIS Pro are still too manual. They depend on someone remembering to do them, or they are spread across scripts, task scheduler entries, and folders that are hard to interpret later.

This tool is meant to clean that up.

Instead of having one off backup steps scattered around, the user has a profile driven setup inside ArcGIS Pro. That gives them a clear place to define what gets backed up, where it goes, how it is packaged, and when it runs.

It is built for real use, not just for a one time script run.

## Package Type Options

### PPKX

PPKX is the stronger project backup option.

Use this when the goal is to preserve the broader ArcGIS Pro project context. If a profile contains multiple APRX files, the tool creates one PPKX per APRX.

This is usually the better option when the backup is meant to support recovery of actual project work.

### MPKX

MPKX is the map based option.

Use this when the goal is to capture selected maps as individual map packages. If a profile contains multiple APRX files, the tool can package maps across all APRXs in that profile.

This is useful when the focus is on map level backup, portability, or map specific snapshots.

### Package type comparison
![Package type comparison](images/package-type-comparison.png)

## Intended Workflow

For a first time user, the expected order is:

1. Save Or Update Profile  
2. Create Or Update Schedule  
3. Run Backup Now  
4. Load Existing Schedule Into Form later if changes are needed  

### Workflow overview
![Workflow overview](images/workflow-overview.png)

That gives users a clear path to get a backup process in place without trying to figure out the whole system at once.

## Tool Actions

### 1. First Time Setup  Save Or Update Profile

Creates or updates a backup profile.

A profile stores:

Profile name

One or more APRX files

Output folder

Settings folder

Selected maps

Tags

Package type

This is the foundation for everything else.

![Profile setup details](images/profile-setup-details.png)

### 2. First Time Setup  Create Or Update Schedule

Creates or updates a Windows scheduled task for a saved profile.

The user can define:

Task name

Schedule type

Start date and time

Days of week for weekly schedules

Optional run now behavior

![Create or update schedule](images/create-or-update-schedule.png)

### 3. Run Backup Now

Runs the selected profile immediately.

This is useful for testing, manual backup creation, or on demand backups outside the schedule.

![Run backup now details](images/run-backup-now-details.png)

### 4. Load Existing Schedule Into Form

Loads a saved schedule back into the tool so the user can review its values and work from the current setup instead of rebuilding it manually.

![Loaded schedule example](images/loaded-schedule-example.png)

### 5. Enable Or Disable Schedule

Turns an existing schedule on or off without deleting it.

![Enable or disable schedule](images/enable-disable-schedule.png)

### 6. Update Profile Maps

Adds, removes, or replaces the maps assigned to an existing profile.

![Update profile maps](images/update-profile-maps.png)

### 7. Remove Schedule

Deletes a scheduled task and removes it from the internal schedule registry.

![Remove schedule](images/remove-schedule.png)

## How It Works

### Requirements

- ArcGIS Pro
- ArcGIS Pro Python environment with `arcpy`
- Windows Task Scheduler
- Permission to create and modify scheduled tasks

### Architecture

The toolbox stores two lightweight registries in JSON format:

- `backup_profiles_registry.json`
- `backup_schedules_registry.json`

Each profile also writes a dedicated settings JSON file and a runner script that the Windows scheduled task executes.

### Backup Execution Model

When a profile is run:

- If the package type is `PPKX`, the tool calls `arcpy.management.PackageProject`
- If the package type is `MPKX`, the tool calls `arcpy.management.PackageMap`

For `PPKX`, one package is created per APRX in the profile.

For `MPKX`, one package is created per selected map, with APRX name prefixes added to reduce naming collisions across multiple projects.

### Schedule Execution Model

Schedules are created through PowerShell and registered in Windows Task Scheduler.

The tool writes a PowerShell script that:

- creates a scheduled task action
- assigns the selected trigger
- points the task to the ArcGIS Pro Python runner script
- passes the saved profile settings file as input

### File Outputs

A typical settings folder will contain:

- profile settings JSON
- runner Python script
- generated PowerShell schedule scripts
- logs folder

A typical output folder will contain:

- `.ppkx` files if using project backups
- `.mpkx` files if using map backups

### Multi APRX Handling

Profiles can contain more than one APRX.

For `PPKX`:
- one `.ppkx` is created for each APRX

For `MPKX`:
- maps are identified using a combined key format:  
  `ProjectName | MapName`

This allows the tool to distinguish maps from different APRX files in the same profile.

### Example folder structure
![Example folder structure](images/example-folder-structure.png)

## Installation

1. Save the `.pyt` file in a permanent location
2. Open ArcGIS Pro
3. In the Catalog pane, add the Python toolbox
4. Open the tool named **Backup Manager**

## Recommended Setup Process

1. Create a profile
2. Choose package type
3. Add one or more APRX files
4. Choose output and settings folders
5. Save the profile
6. Create the schedule
7. Run the backup once manually to confirm output

### Recommended setup example
![Recommended setup example](images/recommended-setup-example.png)

## Notes

This tool is designed around maintainability.

Users should be able to understand what is configured, where it is writing files, and how to adjust the setup later without starting over. It is meant to reduce the friction around backups while still keeping control where it belongs.
