#!/usr/bin/env node
/**
 * Author: @Enoughsdv 2025
 */

const fs = require('node:fs');
const path = require('node:path');
const strip = require('strip-comments');

const args = process.argv.slice(2);
let srcDir, distDir, verbose = false;
let excludePatterns = [];
let excludeMinified = false;

const MARKER_FILE = '.stripper-cli-marker';

function showHelp() {
    console.log(`
stripper-cli - Remove comments from JavaScript and TypeScript files

Usage:
  stripper-cli -s <source> -d <destination> [options]

Required Arguments:
  -s <path>              Source directory to copy and clean
  -d <path>              Destination directory for cleaned files

Options:
  --exclude <pattern>    Exclude files/folders matching pattern (can be used multiple times)
  --no-minified          Exclude all .min.js files from processing
  --verbose              Show detailed output of processed files
  --help                 Show this help message

Examples:
  stripper-cli -s ./src -d ./dist
  stripper-cli -s ./src -d ./dist --exclude "lib/" --verbose
  stripper-cli -s ./src -d ./dist --exclude "*.test.js" --no-minified

Exclude Patterns:
  - Use glob-like patterns: *, ?, folder/
  - Examples: "lib/", "*.min.js", "vendor/*"

Note: You can also use 'node stripper-cli' or 'node stripper-cli.js' if not installed globally.
`);
}

// Show help if no arguments provided
if (args.length === 0) {
    showHelp();
    process.exit(0);
}

for (let i = 0; i < args.length; i++) {
    const arg = args[i];
    if (arg === '-s' || arg === '--source') {
        srcDir = args[i + 1];
        if (!srcDir || srcDir.startsWith('-')) {
            console.error('Error: You must provide a valid source directory after -s');
            process.exit(1);
        }
        i++;
    } else if (arg === '-d' || arg === '--destination') {
        distDir = args[i + 1];
        if (!distDir || distDir.startsWith('-')) {
            console.error('Error: You must provide a valid destination directory after -d');
            process.exit(1);
        }
        i++;
    } else if (arg === '--exclude') {
        const pattern = args[i + 1];
        if (!pattern || pattern.startsWith('-')) {
            console.error('Error: You must provide a pattern after --exclude');
            process.exit(1);
        }
        excludePatterns.push(pattern);
        i++;
    } else if (arg === '--no-minified') {
        excludeMinified = true;
    } else if (arg === '--verbose' || arg === '-v') {
        verbose = true;
    } else if (arg === '--help' || arg === '-h') {
        showHelp();
        process.exit(0);
    } else {
        console.error(`Error: Unknown argument "${arg}". Use --help for usage information.`);
        process.exit(1);
    }
}

if (!srcDir || !distDir) {
    console.error('Error: You must provide both arguments -s <source_directory> and -d <destination_directory>');
    console.error('Use --help for usage information.');
    process.exit(1);
}

// Validate source directory
if (!fs.existsSync(srcDir)) {
    console.error(`Error: Source directory "${srcDir}" does not exist.`);
    process.exit(1);
}

if (!fs.statSync(srcDir).isDirectory()) {
    console.error(`Error: Source path "${srcDir}" is not a directory.`);
    process.exit(1);
}

// Convert to absolute paths
srcDir = path.resolve(srcDir);
distDir = path.resolve(distDir);

if (verbose) {
    console.log(`Source: ${srcDir}`);
    console.log(`Destination: ${distDir}`);
    if (excludePatterns.length > 0) {
        console.log(`Exclude patterns: ${excludePatterns.join(', ')}`);
    }
    if (excludeMinified) {
        console.log('Excluding .min.js files');
    }
    console.log('---');
}

function copyDir(src, dest) {
    fs.mkdirSync(dest, { recursive: true });
    for (const file of fs.readdirSync(src)) {
        const srcPath = path.join(src, file);
        const destPath = path.join(dest, file);
        const stat = fs.statSync(srcPath);

        if (stat.isDirectory()) {
            copyDir(srcPath, destPath);
        } else {
            fs.copyFileSync(srcPath, destPath);
        }
    }
}

function matchesExcludePatterns(filePath) {
    return excludePatterns.some(pattern => {
        if (pattern.endsWith('/') || pattern.endsWith('\\')) {
            const normPattern = pattern.replace(/\\/g, '/');
            const normPath = filePath.replace(/\\/g, '/');
            return normPath.startsWith(normPattern);
        }
        // Simple glob-like: supports * and ?
        const regex = new RegExp('^' + pattern.replace(/[.+^${}()|[\]\\]/g, '\\$&').replace(/\*/g, '.*').replace(/\?/g, '.') + '$');
        return regex.test(filePath);
    });
}

let filesProcessed = 0;
let filesSkipped = 0;

function cleanJsInDir(dir) {
    for (const file of fs.readdirSync(dir)) {
        if (file === MARKER_FILE) continue;
        const fullPath = path.join(dir, file);
        const stat = fs.statSync(fullPath);

        if (stat.isDirectory()) {
            cleanJsInDir(fullPath);
        } else if (file.endsWith('.js') || file.endsWith('.ts')) {
            const relPath = path.relative(distDir, fullPath).replace(/\\/g, '/');
            
            // Check exclusions
            if (excludeMinified && file.endsWith('.min.js')) {
                filesSkipped++;
                if (verbose) console.log(`Skipped (minified): ${relPath}`);
                continue;
            }
            
            if (matchesExcludePatterns(relPath)) {
                filesSkipped++;
                if (verbose) console.log(`Skipped (excluded): ${relPath}`);
                continue;
            }

            try {
                const code = fs.readFileSync(fullPath, 'utf8');
                const cleaned = strip(code, { preserveNewlines: false });
                fs.writeFileSync(fullPath, cleaned, 'utf8');
                filesProcessed++;
                if (verbose) console.log(`Cleaned: ${relPath}`);
            } catch (error) {
                console.error(`Error processing ${relPath}: ${error.message}`);
                filesSkipped++;
            }
        }
    }
}

if (fs.existsSync(distDir)) {
    const markerPath = path.join(distDir, MARKER_FILE);
    if (!fs.existsSync(markerPath)) {
        console.error(`Error: Destination directory "${distDir}" exists and was not created by this tool. Aborting to prevent data loss.`);
        process.exit(1);
    }
    fs.rmSync(distDir, { recursive: true, force: true });
}

fs.mkdirSync(distDir, { recursive: true });
fs.writeFileSync(path.join(distDir, MARKER_FILE), 'This directory was created by stripper-cli. Safe to replace.', 'utf8');
copyDir(srcDir, distDir);
cleanJsInDir(distDir);

// Final summary
console.log('\n--- Summary ---');
console.log(`Files processed: ${filesProcessed}`);
if (filesSkipped > 0) {
    console.log(`Files skipped: ${filesSkipped}`);
}
console.log(`✅ Operation completed successfully!`);
