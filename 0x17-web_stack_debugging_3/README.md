# 0x17. Web Stack Debugging #3

## Project Overview
This project focuses on debugging a web stack running a WordPress site on a LAMP (Linux, Apache, MySQL, and PHP) stack. The primary task involves identifying and fixing an issue that causes Apache to return a 500 Internal Server Error. The fix must be automated using a Puppet manifest.

## Requirements
- All scripts were tested on Ubuntu 14.04 LTS.
- Puppet manifests should:
  - End with a new line.
  - Pass `puppet-lint` version 2.1.1 without errors.
  - Run without errors.
  - Include a comment explaining the purpose.
  - Have a `.pp` extension.
- Tools used:
  - `strace` for debugging.
  - Puppet for automating the fix.

## Task: Strace is Your Friend

### Debugging Process

1. **Identifying the Issue with `strace`:**
   - The Apache server was returning a `500 Internal Server Error` when accessing the WordPress site. To investigate, the `strace` tool was used to trace system calls and signals. Here’s how it was done:
     - Attach `strace` to the Apache process:
       ```bash
       strace -p $(pgrep -f apache2) -o apache_strace.log
       ```
     - In a separate terminal, trigger the issue by making a request to the server:
       ```bash
       curl -sI http://127.0.0.1
       ```
     - Analyze the `apache_strace.log` file to identify the error.

2. **Finding the Root Cause:**
   - The log analysis revealed that the `wp-settings.php` file had a typo in the PHP file extension, written as `phpp` instead of `php`. This typo caused the server to fail when trying to load the required files, leading to the `500 Internal Server Error`.

3. **Automating the Fix with Puppet:**
   - A Puppet manifest was created to automatically fix the typo in the `wp-settings.php` file. The manifest uses the `sed` command to replace all occurrences of `phpp` with `php` in the specified file.

### Puppet Manifest

```puppet
# Fixes bad `phpp` extensions to `php` in the WordPress file `wp-settings.php`.

exec { 'fix-wordpress':
  command => 'sed -i s/phpp/php/g /var/www/html/wp-settings.php',
  path    => '/usr/local/bin/:/bin/'
}
