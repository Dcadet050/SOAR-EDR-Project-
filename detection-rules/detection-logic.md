# Detection Logic

The rule detects execution of the LaZagne credential dumping tool using multiple detection conditions.

Detection Conditions:

- FILE_PATH ends with "LaZagne.exe"
- COMMAND_LINE contains "lazagne"
- COMMAND_LINE ends with "all"
- SHA256 hash matches the known LaZagne executable
