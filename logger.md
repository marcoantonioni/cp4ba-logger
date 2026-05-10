# CP4BA Logger Script Documentation

## Overview

The [`logger.sh`](scripts/logger.sh) script is a comprehensive Bash logging utility designed for CP4BA (Cloud Pak for Business Automation) projects. It provides enterprise-grade logging capabilities with multiple log levels, colored console output, file logging with automatic rotation, and flexible configuration options.

## Features

- **Enable/Disable Logging**: Master switch to turn logging on/off globally
- **Multiple Log Levels**: DEBUG, INFO, WARNING, ERROR, and ONLYMSG
- **Colored Console Output**: ANSI color-coded messages for better readability
- **File Logging**: Optional file output with configurable path
- **Automatic Log Rotation**: Size-based rotation with configurable backup count
- **ISO 8601 Timestamps**: Standard timestamp formatting
- **Caller Information Tracking**: Automatic tracking of source file, line number, and function
- **Level Filtering**: Only log messages at or above configured minimum level

## How It Works

### Architecture

The script is organized into four main sections:

1. **Configuration Section**: Environment variables for customizing logging behavior
2. **Helper Functions**: Utility functions for timestamps, colors, priorities, and rotation
3. **Main Logging Function**: Core [`log()`](scripts/logger.sh:162) function that processes and outputs messages
4. **Convenience Wrapper Functions**: Easy-to-use functions for each log level

### Logging Flow

1. Check if logging is enabled via `LOGGING_ENABLED`
2. Validate the log level
3. Compare message priority against minimum `LOG_LEVEL`
4. Build formatted log entry with timestamp and caller info
5. Output to console (with colors) if `LOG_TO_CONSOLE=true`
6. Check if log rotation is needed
7. Append to log file if `LOG_TO_FILE=true`

### Log Levels Priority

The script uses numeric priorities for level comparison:

- **DEBUG**: 0 (lowest priority)
- **INFO**: 1
- **WARNING**: 2
- **ERROR**: 3 (highest priority)
- **ONLYMSG**: Special level for message-only output

## Environment Variables

### Core Configuration

#### `LOGGING_ENABLED`
- **Type**: Boolean (`true`/`false`)
- **Default**: `false`
- **Description**: Master switch to enable/disable all logging functionality
- **Example**: `export LOGGING_ENABLED=true`

#### `LOG_LEVEL`
- **Type**: String
- **Options**: `DEBUG`, `INFO`, `WARNING`, `ERROR`
- **Default**: `INFO`
- **Description**: Minimum log level to display/record. Only messages at this level or higher will be logged
- **Example**: `export LOG_LEVEL="DEBUG"`

### Output Configuration

#### `LOG_TO_CONSOLE`
- **Type**: Boolean (`true`/`false`)
- **Default**: `true`
- **Description**: Enable/disable console output
- **Example**: `export LOG_TO_CONSOLE=true`

#### `LOG_TO_FILE`
- **Type**: Boolean (`true`/`false`)
- **Default**: `false`
- **Description**: Enable/disable file output
- **Example**: `export LOG_TO_FILE=true`

#### `LOG_FILE`
- **Type**: String (file path)
- **Default**: `""` (empty)
- **Description**: Path to the log file. Will be created if it doesn't exist, including parent directories
- **Example**: `export LOG_FILE="./logs/application.log"`

### Rotation Configuration

#### `LOG_MAX_SIZE`
- **Type**: Integer (bytes)
- **Default**: `10485760` (10MB)
- **Description**: Maximum log file size before rotation occurs
- **Example**: `export LOG_MAX_SIZE=$((50 * 1024 * 1024))  # 50MB`

#### `LOG_BACKUP_COUNT`
- **Type**: Integer
- **Default**: `5`
- **Description**: Number of rotated log files to keep (e.g., `application.log.1`, `application.log.2`, etc.)
- **Example**: `export LOG_BACKUP_COUNT=10`

### Color Configuration (Read-Only)

The following ANSI color codes are predefined and cannot be modified:

- `COLOR_RESET`: Reset to default
- `COLOR_DEBUG`: Gray (`\033[0;37m`)
- `COLOR_INFO`: Green (`\033[0;32m`)
- `COLOR_WARNING`: Yellow (`\033[0;33m`)
- `COLOR_ERROR`: Red (`\033[0;31m`)
- `COLOR_BOLD`: Bold text (`\033[1m`)

## Usage

### Basic Setup

1. **Source the script** in your bash script:
```bash
source ./cp4ba-logger/scripts/logger.sh
```

2. **Configure logging** (optional, uses defaults if not set):
```bash
export LOGGING_ENABLED=true
export LOG_LEVEL="INFO"
export LOG_TO_CONSOLE=true
export LOG_TO_FILE=true
export LOG_FILE="./logs/myapp.log"
```

3. **Use the logging functions**:
```bash
log_info "Application started"
log_debug "Processing user input: $user_input"
log_warning "Deprecated function called"
log_error "Failed to connect to database"
```

### Convenience Functions

#### `log_debug(message)`
Logs a DEBUG level message (gray color).
```bash
log_debug "Variable value: $my_var"
```

#### `log_info(message)`
Logs an INFO level message (green color).
```bash
log_info "Operation completed successfully"
```

#### `log_warning(message)`
Logs a WARNING level message (yellow color).
```bash
log_warning "Configuration file not found, using defaults"
```

#### `log_error(message)`
Logs an ERROR level message (red color).
```bash
log_error "Failed to write to file: $filename"
```

#### `log_msg(message)`
Logs a message without timestamp or level formatting (ONLYMSG level).
```bash
log_msg "Simple message output"
```

### Direct Log Function

You can also use the main [`log()`](scripts/logger.sh:162) function directly:
```bash
log "INFO" "This is an info message"
log "ERROR" "This is an error message"
```

## Complete Usage Example

```bash
#!/bin/bash

# Source the logger script
source ./cp4ba-logger/scripts/logger.sh

# Configure logging
export LOGGING_ENABLED=true
export LOG_LEVEL="DEBUG"
export LOG_TO_CONSOLE=true
export LOG_TO_FILE=true
export LOG_FILE="./logs/deployment.log"
export LOG_MAX_SIZE=$((5 * 1024 * 1024))  # 5MB
export LOG_BACKUP_COUNT=3

# Your application code
log_info "Starting CP4BA deployment"

log_debug "Checking prerequisites..."
if ! command -v oc &> /dev/null; then
    log_error "OpenShift CLI (oc) not found"
    exit 1
fi

log_info "Prerequisites check passed"

log_warning "Using default namespace: cp4ba"

# Simulate some operations
for i in {1..5}; do
    log_debug "Processing step $i"
    sleep 1
done

log_info "Deployment completed successfully"
```

### Output Format

**Console output** (with colors):
```
[2026-05-10T14:30:00+0200] [INFO] [deploy.sh:15:main] Starting CP4BA deployment
[2026-05-10T14:30:01+0200] [DEBUG] [deploy.sh:17:main] Checking prerequisites...
[2026-05-10T14:30:02+0200] [INFO] [deploy.sh:23:main] Prerequisites check passed
[2026-05-10T14:30:03+0200] [WARNING] [deploy.sh:25:main] Using default namespace: cp4ba
```

**File output** (without colors):
```
[2026-05-10T14:30:00+0200] [INFO] [deploy.sh:15:main] Starting CP4BA deployment
[2026-05-10T14:30:01+0200] [DEBUG] [deploy.sh:17:main] Checking prerequisites...
[2026-05-10T14:30:02+0200] [INFO] [deploy.sh:23:main] Prerequisites check passed
[2026-05-10T14:30:03+0200] [WARNING] [deploy.sh:25:main] Using default namespace: cp4ba
```

## Log Rotation

When the log file reaches `LOG_MAX_SIZE`, the script automatically:

1. Renames existing backup files (`.1` → `.2`, `.2` → `.3`, etc.)
2. Moves current log to `.1`
3. Creates a new empty log file
4. Deletes oldest backup if count exceeds `LOG_BACKUP_COUNT`

**Example rotation sequence**:
```
application.log       (current, 10MB)
application.log.1     (previous, 10MB)
application.log.2     (older, 10MB)
```

After rotation:
```
application.log       (new, empty)
application.log.1     (was application.log)
application.log.2     (was application.log.1)
application.log.3     (was application.log.2)
```

## Advanced Configuration Examples

### Production Environment
```bash
export LOGGING_ENABLED=true
export LOG_LEVEL="INFO"
export LOG_TO_CONSOLE=true
export LOG_TO_FILE=true
export LOG_FILE="/var/log/cp4ba/production.log"
export LOG_MAX_SIZE=$((100 * 1024 * 1024))  # 100MB
export LOG_BACKUP_COUNT=10
```

### Development Environment
```bash
export LOGGING_ENABLED=true
export LOG_LEVEL="DEBUG"
export LOG_TO_CONSOLE=true
export LOG_TO_FILE=false
```

### Silent Mode (File Only)
```bash
export LOGGING_ENABLED=true
export LOG_LEVEL="INFO"
export LOG_TO_CONSOLE=false
export LOG_TO_FILE=true
export LOG_FILE="./logs/silent.log"
```

### Disable Logging
```bash
export LOGGING_ENABLED=false
```

## Helper Functions

The script includes several internal helper functions:

- [`get_log_level_priority(level)`](scripts/logger.sh:75): Returns numeric priority for level comparison
- [`get_log_color(level)`](scripts/logger.sh:90): Returns ANSI color code for the specified level
- [`get_timestamp()`](scripts/logger.sh:104): Returns current timestamp in ISO 8601 format
- [`get_caller_info()`](scripts/logger.sh:111): Returns filename:line:function of the caller
- [`rotate_log()`](scripts/logger.sh:128): Performs log file rotation when size limit is exceeded

## Best Practices

1. **Always enable logging in production** to track issues and operations
2. **Use appropriate log levels**: DEBUG for development, INFO for production
3. **Configure file logging** for persistent records and troubleshooting
4. **Set reasonable rotation limits** to prevent disk space issues
5. **Use descriptive messages** that include context and variable values
6. **Avoid logging sensitive data** (passwords, tokens, personal information)
7. **Source the script early** in your main script before any operations

## Troubleshooting

### Logging not working
- Check that `LOGGING_ENABLED=true`
- Verify log level allows your messages (e.g., DEBUG messages won't show if `LOG_LEVEL="INFO"`)

### File not being created
- Ensure `LOG_TO_FILE=true`
- Verify `LOG_FILE` path is set and writable
- Check parent directory exists or can be created

### Colors not showing
- Verify terminal supports ANSI colors
- Check that `LOG_TO_CONSOLE=true`

### Rotation not working
- Ensure file system supports `stat` command
- Verify write permissions on log directory
- Check `LOG_MAX_SIZE` is set appropriately

## Version Information

- **Author**: Bob (AI Assistant)
- **Date**: 2026-04-29
- **Version**: 1.0
- **Script Location**: [`cp4ba-logger/scripts/logger.sh`](scripts/logger.sh)

## License

This script is part of the CP4BA projects installation toolkit.