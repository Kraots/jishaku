.. currentmodule:: jishaku

What's new?
================

Version 2.3.0
-------------

Moved to disnake
Added the ability to choose whether ``jsk`` sends an embed or not, as well as having the possibility to set its colour.

Example:
```py
import os

# Setting the jsk command to return an embed
os.environ['JISHAKU_EMBEDDED_JSK'] = '1'

# Setting the embed's colour
# Acceptable formats:
    # 0x<hex>
    # #<hex>
    # 0x#<hex>
    # rgb(<number>, <number>, <number>)
    # Any of the classmethod in disnake.Colour
os.environ['JISHAKU_EMBEDDED_JSK_COLOUR'] = 0x2F3136
```

Version 2.2.0
-------------

The ``jsk sudo``, ``jsk su`` and ``jsk in`` commands have been removed and replaced with a single command that handles all three at once.

``jsk exec`` now automatically handles IDs or mentions for channels, users, or threads (only with discord v2.0a+).
Aliases with a postfix ``!`` bypass checks and cooldowns, like ``jsk sudo`` used to do.

Example of how the commands change with this release:

- ``jsk su @user command`` -> ``jsk exec @user command``
- ``jsk in #channel command`` -> ``jsk exec #channel command``
- ``jsk in #channel jsk su @user command`` -> ``jsk exec #channel @user command`` or ``jsk exec @user #channel command``
- ``jsk sudo command`` -> ``jsk exec! command``

This allows combinations that were previously not possible, for example,
``jsk exec! #channel @user command`` now executes a command as a user in another channel or thread, bypassing any checks or cooldowns that user or channel has against the command.

The flag system (i.e. the ``JISHAKU_FLAG=...`` system) has been rewritten to use various degrees of lazy evaluation.
This means setting flags like ``JISHAKU_HIDE`` and ``JISHAKU_RETAIN`` need only precede loading the Jishaku extension, as opposed to the entire module.

Flags that only evaluated at command runtime will now have their changes take effect immediately.
For example, executing ``os.environ['JISHAKU_NO_UNDERSCORE'] = '1'`` no longer requires a reload to take effect.

A programmatic interface for flags is available, however, its use is discouraged except in subclass initialization, due to the fact that the changes will **NOT** persist across reloads of the extension.

.. code:: python3

    jishaku.Flags.NO_UNDERSCORE = True

The ``jsk invite`` command has been added, which is a developer convenience command that supplies the invite link for the bot it is ran on.
This command is most useful for bots that predate the behavior change that merged bot and application IDs, saving the time of having to retrieve the application ID yourself.

Permissions can be supplied, e.g., ``jsk invite kick_members manage_messages`` will create an invite requesting those two permissions.

The invites produced request slash commands for convenience.

Some regressions have been fixed and other internal cleanup has been addressed in this release.

Version 2.1.0
-------------

A new implementation of PaginatorInterface has been created using Discord's interaction buttons system.
It is available when using discord.py 2.0.0 or greater (currently alpha).

Jishaku will now avoid uploading files either when detecting the author is on mobile or through an explicit ``JISHAKU_FORCE_PAGINATOR`` switch.
This is to better support mobile platforms that do not have inline file previews yet. (`PR #111 <https://github.com/Kraots/jishaku/pull/111>`_).

Humanize has been removed as a dependency. Selftest now uses Discord's own relative timestamp formatting markdown extension for timing,
and pretty printing of memory usage has been implemented within the Feature itself.

Version 2.0.0
--------------

Python version changes
~~~~~~~~~~~~~~~~~~~~~~~

Python version 3.7 has been dropped. Jishaku 2.0 requires Python 3.8 or greater.

New commands
~~~~~~~~~~~~~

- ``jsk rtt``
    Calculates the round-trip time between your bot and the Discord API.
    Reports exact values as well as an average and standard deviation.

- ``jsk dis``
    Disassembles a given piece of Python code in REPL context, returning the bytecode.
    This does not actually execute the code, but can be used to identify compiler behavior and optimizations.

- ``jsk permtrace``
    Calculates the effect of permissions and overwrites on a given member or set of roles in a channel.
    This can be used to identify why a member does, or does not, have a permission in a given place.

Command improvements
~~~~~~~~~~~~~~~~~~~~~
- ``jsk``
    Information on sharding status for both automatically and manually sharded bots is now displayed.

    The root 'jsk' command can now be sanely overridden, removed, or renamed using the Feature system.

- ``jsk py`` / ``jsk pyi``
    Exceptions now display the line from which they originate, instead of just the line number.

    Large results that fit within the Discord preview threshold are now uploaded as files,
    for better navigability.

- ``jsk sh``
    Timeout has been increased from 90 seconds from invocation, to 120 seconds from the last output.

    This should reduce the chance of termination from long-running installs or other processes.

- ``jsk source``
    Triple backticks inside of source files no longer cause the file content to spill outside of its codeblock.

    Large results that fit within the Discord preview threshold are now uploaded as files,
    for better navigability.

- ``jsk vc``
    Voice commands no longer appear if their relevant dependencies are not installed.

- ``jsk shutdown``
    Now uses ``bot.close`` to prevent deprecation warnings.

    Fixed a regression with braille J support.

API changes
~~~~~~~~~~~~

- Feature system
    The Feature system has been implemented as a means to solve subclassing problems with Jishaku.

    Certain functionality can now be disabled in subclasses, additional commands can be easily facilitated
    without affecting the native Jishaku cog, and overriding subcommands or groups is now possible without
    needing to reimplement commands that would otherwise become orphaned in the process.

- PaginatorInterface
    Updating mechanism has been entirely rewritten to better prevent cascades of message edits.

    This change has also made it possible to synchronously trigger interface updates, paving the way for
    possible native stream or TextIO support in the future.
