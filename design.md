# Audio Archive Technical Design

Our audio archive was designed to allow users to store and listen to sounds.
In addition to this, our audio archive also tracks metadata associated with each sound and allows users to apply audio effects when playing sounds back through the use of a command line interface.

## Overview

The business for our application mostly lives inside `src/commander.py`.
A `Commander` object gives you access to all of the core commands such as playing audio and listing available sounds.
Each available interface (which is currently only our command line interface located in `src/cli.py` can create a `Commander` object, and dispatch the correct function depending on the user's interaction.
When creating a `Commander` object, you must pass a `Storage` object, which handles all interactions with metadata such as the file paths associated with sounds and tags associated with sounds.
The `Storage` object represents this metadata as an `AudioMetadata` object.
So, when the `Commander` needs information on a sound, it can call a function to get `AudioMetadata` object(s) from `Storage`, and then handle either playing sounds or passing information back to the interface.

## Extension

With this architecture, we have attempted to maintain a separation of concerns between different classes.
It enables us to easily extend our archive.
For example, if we want to create another interface, it could maintain a `Commander` object and only worry about calling functions from that object.

The process for adding a new command involves jumping around more places, but should hopefully still be straightforward.

All commands spawn from `src/commander.py`, so it makes sense to begin by creating a new function in the `Commander` class (which will likely be fairly small).
If necessary, a new query can be added to the `Storage` class, which can then be implemented in the database class.
After this, you can extend whichever interface you choose to allow the user to call your new function.
For example, extending the CLI involves creating a new subparser, adding any arguments and flags, and then creating an _handle[Command] function.

If possible, a new test should be added in the `tests/` directory.

## Storage

Our storage design involves two layers.
Each `Storage` object is created with a `database` object.
This allows us to swap in parts - for example, we were initially not sure if we wanted to keep a `sqlite` database or a JSON file.
Writing our storage like this allows us to easily swap out parts, so we have the freedom to swap in different database implementations.
For example, if a user wanted to use cloud database, we could easily create a new class to interact with it satisfying our current database interface.

The `Storage` object handles all interactions unrelated to the database, such as file interactions, normalizing tags, and validating input.

All sounds are stored in a local directory.  We considered storing them in the database, but this would require storing them as binaries.  Note that we default to using `pathlib` for file operations over `os`.

We initially wanted to implement an LRU cache to reduce database queries.
However, we essentially decided that this would provide little benefit because we would need to update the database every time we played a sound to keep the last played record up to date.
While we technically could have only updated that information in the cache and had other checks to ensure that the cache and database stayed in sync, we were afraid of the bugs that having our cache and database out of sync might introduce.
