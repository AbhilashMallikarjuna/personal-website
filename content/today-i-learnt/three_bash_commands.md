+++
title = '3 Bash Flags That Would Have Saved Me Hours of Debugging'
date = 2025-07-16T21:36:48+05:30
draft = false
hideSummary= true
+++

As a developer, terminal is my best friend. I use it extensively for my daily tasks like

-   Need to use git -> opens terminal
-   open folders in IDEs -> opens terminal
-   accessing remote servers -> opens terminal
-   and a lot of other things.

Last month, I spent 2 hours debugging why my deployment script was "successful" but half the services weren't actually deployed. Turns out, one command in the middle failed silently, but the script kept running and reported success. That's when I realized I needed to learn about bash's error handling.

Even after using terminal from my career start, I wasn't aware of these `flags` in bash terminal which would have saved me a lot of time when things went wrong.

## The Set Builtin Command

Before diving into specific flags, let me quickly explain what the `set` builtin is. It's a bash command that allows you to change the behavior of your shell or script. You can use it like this:

`set -o [option-name]`

Here are the three flags that I wish I knew earlier:

## 1. `errexit` - Stop on Errors

By default, bash will continue after errors. This is not what we expect most of the time. By setting this flag bash will stop the script execution on errors.

```bash
# Without set -e
rm /nonexistent/file.txt
echo "This will still run even if the above command failed"

# With set -e
set -e
rm /nonexistent/file.txt
echo "This will never execute if the above command fails"

```

Since this flag is frequently used, they have given a separate flag for ease of use. You can use `set -e` instead of `set -o errexit`.

## 2. `nounset` - Catch Undefined Variables

We depend heavily on variables while scripting. But if we forgot to assign value to variable and run it, bash will be like "You have not assigned anything to this variable, so I will consider it as empty string". This might not be our expectation most of the time.

```bash
# Without set -u
name=""  # forgot to assign actual value
echo "Hello $name"  # prints "Hello " - hard to debug

# With set -u
set -u
name=""
echo "Hello $name"  # script stops with error about unset variable

```

So if we set this flag, script will stop and return error. You can use `set -u` instead of `set -o nounset`.

## 3. `pipefail` - Fail Fast in Pipelines

Suppose you have piped multiple commands. While execution one of them fails, the default behavior is it doesn't fail the whole pipeline. Normally a pipeline returns the exit status of the last command, but with pipefail it returns the exit status of the first command that fails.

```bash
# Without pipefail
false | echo "success"  # pipeline succeeds because echo succeeds

# With pipefail
set -o pipefail
false | echo "success"  # pipeline fails because false command fails

```

This flag makes the entire pipeline fail if any command in it fails.

## Combining All Three - "Strict Mode"

You can combine all the above and keep it at the top of all your script files:

**`set -eou pipefail`**

This is the same as writing `set -e -o nounset -o pipefail`. Many developers call this "strict mode" or "safe mode" for bash scripts.

## When You Need to Bypass These Flags

Sometimes you might need to temporarily disable these flags when you intentionally want to handle errors differently:

```bash
set +e  # temporarily disable errexit
wget https://example.com/file.txt
if [ $? -ne 0 ]; then
    echo "Command failed, but that's okay"
fi
set -e  # re-enable errexit

```

## Common Gotchas

-   Commands that intentionally return non-zero exit codes (like `grep` when no matches found) will cause your script to exit with `set -e`
-   Be careful with conditional statements - they can sometimes mask errors even with these flags set

## Bonus: Debugging Flag

While we're talking about useful flags, here's one more that's great for debugging:

`set -x` - This prints each command before executing it. Super useful when you're trying to figure out what your script is actually doing.

## Before and After

Here's what a typical script looks like without these flags:

```bash
#!/bin/bash
# Risky script - can fail silently

backup_database
deploy_application
restart_services
echo "Deployment successful!"

```

And here's the same script with proper error handling:

```bash
#!/bin/bash
set -eou pipefail

backup_database
deploy_application  
restart_services
echo "Deployment successful!"

```

The second version will actually stop if any step fails, instead of pretending everything is fine.

## References

1.  https://wizardzines.com/comics/bash-errors/
2.  https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html
