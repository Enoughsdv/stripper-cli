# stripper-cli
A simple CLI tool that copies directories and removes comments from JavaScript and TypeScript files.

> This is my first npm package! Built it because I needed a quick way to clean up code comments for build processes :).

## Features
- Removes `//` and `/* */` comments from JS/TS files
- Supports glob patterns for excluding files (`*.test.js`, `lib/`, etc.)
- Safe operation with marker files (won't overwrite existing directories)
- Shows processing statistics

## Installation
```sh
# Install globally
npm install -g stripper-cli

# Or use directly without installing
npx stripper-cli -s ./src -d ./dist
```

## Usage
```sh
stripper-cli -s <source> -d <destination> [options]
```

### Options
| Option | Aliases | Description |
|--------|---------|-------------|
| `-s <path>` | `--source` | Source directory |
| `-d <path>` | `--destination` | Destination directory |
| `--exclude <pattern>` | | Exclude files/folders (supports glob patterns) |
| `--no-minified` | | Skip `.min.js` files |
| `--verbose` | `-v` | Show detailed output |
| `--help` | `-h` | Show help |

### Examples
```sh
# Basic usage
stripper-cli -s ./src -d ./dist

# Exclude specific folders
stripper-cli -s ./src -d ./dist --exclude "lib/" --exclude "*.test.js"

# Skip minified files and show verbose output
stripper-cli -s ./src -d ./dist --no-minified --verbose
```

## What it does
**Before:**
```javascript
function test() {
    console.log('Hello'); // This prints hello
    /* Multi-line
       comment here */
    return 42;
}
```

**After:**
```javascript
function test() {
    console.log('Hello'); 
    
    return 42;
}
```

## Common patterns
```sh
# Exclude libraries and tests
stripper-cli -s ./src -d ./dist --exclude "lib/" --exclude "*.test.js"

# Skip all minified files
stripper-cli -s ./src -d ./dist --no-minified

# See what's happening
stripper-cli -s ./src -d ./dist --verbose
```

## Safety
The tool creates a `.stripper-cli-marker` file in the destination. If the destination exists without this marker, it will abort to prevent data loss.

## License
MIT
