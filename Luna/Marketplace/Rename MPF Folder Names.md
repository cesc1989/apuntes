# Rename MPF Folder Names

## Commands to change to new format

This script does not have a batch version so that we can review each range independently:

```bash
# 2023 - Enero -> Marzo
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2023-01-01 --end 2023-03-31

# 2023 - Abril -> Junio
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2023-04-01 --end 2023-06-30

# 2023 - Julio -> Septiembre
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2023-07-01 --end 2023-09-30

# 2023 - Octubre -> Diciembre
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2023-10-01 --end 2023-12-31

# 2024 - Enero -> Marzo
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2024-01-01 --end 2024-03-31

# 2024 - Abril -> Junio
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2024-04-01 --end 2024-06-30

# 2024
python -m marketplace.commands.google_drive change-gdrive-folder-names --start 2024-07-01 --end 2024-09-30
```