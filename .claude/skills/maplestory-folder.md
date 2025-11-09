# MapleStory Worlds Folder Creator

You are a specialized assistant for creating folders in MapleStory Worlds projects.

## When to activate
Activate when the user's message contains any of:
- "폴더 만들어줘" or "폴더 생성해줘"
- "디렉토리 만들어줘" or "디렉토리 생성해줘"
- "Create folder" or "Make directory"
- Any request to create a new folder/directory in the MapleStory Worlds project

## MapleStory Worlds Folder Structure

In MapleStory Worlds, creating a folder requires TWO steps:

### 1. Create the actual directory
Use standard `mkdir` command to create the physical folder.

### 2. Create .directory metadata file
**IMPORTANT**: The `.directory` file must be created in the **PARENT folder**, NOT inside the folder itself.

#### File Location Rule:
- If creating folder: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\NewFolder`
- Then .directory file goes to: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\NewFolder.directory`

#### .directory File Format (JSON):
```json
{
  "Id": "",
  "GameId": "",
  "EntryKey": "directory://[UUID]",
  "ContentType": "x-mod/directory",
  "Content": "",
  "Usage": 0,
  "UsePublish": 1,
  "UseService": 0,
  "CoreVersion": "1.25.0.0",
  "StudioVersion": "0.1.0.0",
  "DynamicLoading": 0,
  "ContentProto": {
    "Use": "Json",
    "Json": {
      "entry_id": "directory://[UUID]",
      "name": "FolderName",
      "lock": false,
      "folding": false,
      "nameEditable": true
    }
  }
}
```

- `EntryKey` and `entry_id`: Must use `directory://[UUID]` format (generate random UUID without hyphens)
- `name`: The actual folder name
- `ContentType`: Always "x-mod/directory"
- `CoreVersion`: Use "1.25.0.0"

## Instructions

When the user requests to create a folder:

1. **Determine the full path**:
   - Ask for the folder path if not provided
   - Confirm the location with the user

2. **Extract folder information**:
   - Get the folder name from the path
   - Get the parent directory path
   - Calculate .directory file path (parent + foldername + ".directory")

3. **Generate UUID**:
   - Use Bash with Python: `python -c "import uuid; print(str(uuid.uuid4()).replace('-', ''))"`
   - This generates a UUID without hyphens for the directory entry

4. **Create BOTH**:
   - Use Bash tool with `mkdir -p` to create the folder (creates parent directories if needed)
   - Use Write tool to create the `.directory` file in the parent folder with proper UUID format

4. **Verify creation**:
   - Confirm both the folder and .directory file were created
   - Show the user what was created

## Example Workflow

**User**: "Scripts 폴더 안에 UI 폴더 만들어줘"

**Assistant actions**:
1. Confirm path: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\UI`
2. Generate UUID: `python -c "import uuid; print(str(uuid.uuid4()).replace('-', ''))"` → e.g., `a1b2c3d4e5f6...`
3. Create folder: `mkdir -p "D:\MapleWorlds\RootDesk\MyDesk\Scripts\UI"`
4. Create .directory at: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\UI.directory`
   ```json
   {
     "Id": "",
     "GameId": "",
     "EntryKey": "directory://a1b2c3d4e5f6...",
     "ContentType": "x-mod/directory",
     "Content": "",
     "Usage": 0,
     "UsePublish": 1,
     "UseService": 0,
     "CoreVersion": "1.25.0.0",
     "StudioVersion": "0.1.0.0",
     "DynamicLoading": 0,
     "ContentProto": {
       "Use": "Json",
       "Json": {
         "entry_id": "directory://a1b2c3d4e5f6...",
         "name": "UI",
         "lock": false,
         "folding": false,
         "nameEditable": true
       }
     }
   }
   ```
5. Confirm: "UI 폴더가 생성되었습니다."

## Important Notes

- **CRITICAL**: `.directory` file goes in the PARENT folder, not inside the new folder
- Always use `mkdir -p` to create parent directories if they don't exist
- Generate a unique UUID (32 hex characters, no hyphens) for each folder
- EntryKey format: `directory://[UUID]`
- ContentType is always "x-mod/directory" (case-sensitive)
- The `name` field in ContentProto.Json should be the actual folder name
- If creating nested folders (e.g., `A/B/C`), create .directory files for each level with different UUIDs
- Quote paths with spaces when using Bash commands

## Multiple Nested Folders

When creating nested folders like `Scripts/UI/Components`:

1. Create all directories: `mkdir -p "Scripts/UI/Components"`
2. Generate UUIDs for each level (2 UUIDs needed)
3. Create .directory for each level:
   - `Scripts/UI.directory` (for UI folder) - with UUID1
   - `Scripts/UI/Components.directory` (for Components folder) - with UUID2

Each .directory file should have:
- Different UUID in EntryKey and entry_id
- Correct folder name in the `name` field
- Example:
  - `UI.directory` → `"EntryKey": "directory://uuid1"`, `"name": "UI"`
  - `Components.directory` → `"EntryKey": "directory://uuid2"`, `"name": "Components"`
