# Architecture

## Component: configuration

The configuration component owns all configuration, such as which linters are enabled or general
behavior of super linter.

The configuration component follows the conventions of super linter and is assumed to recognize
those.

### API — method

`IsValidationEnabled(linter) Boolean`:

- `linter`, string: the linter's name
- returns `true` if the linter is enabled for validation

`ConfigurationFile(linter) string`:

- `linter`, string: the linter's name
- The path, on the container file system, of the linter's configuration file. Slim linter comes
  with default configuration for all supported linters that the user can override.

## Component: file

The file component provides to the linters the list of files and directories to process. It can
take change sets into account, to validate only files that have recently changed, extensions to
filter by file types, …

### API — method

TBD

## Interface: Result logger

A result logger logs information, warnings and error messages of a linter for a file at a position.
It also collects summary information.

A default implementation emits everything to Slim Linter's stdout. An alternate implementation
emits GitHub workflow commands.

## Interface: Stream listener

A stream listener receives the output of the linter, analyses it and emits info, warnings and
errors to the Result Logger.

A default implementation emits everything as information.

There will ultimately be linter-specific implementations.

## Component: Docker

The Docker component executes a Docker image with its environment and configuration against a mount
of the project submitted to slim linter and captures the result.

### API — Properties

- `Image`: string, name and label of the image to execute
- `Name`: string, name for the container, use the linter name
- `LocalDirectory`: file, local directory to mount on the container
- `RemoteMount`: string, mount point for the local directory on the container
- `OutListener`, `ErrListener`: listeners for the container's stdout, resp. stderr

### API — Method

`Run(arguments)`:

- `arguments`: string[], arguments for the container

## Component: linter plugin

A linter plugin encapsulates the configuration and execution of a version of a linter. All linter
plugins implement the same interface with method `validate` that, given a configuration, file and
Docker instance, lints a subset of the linter project and reports the result.

Slim linter will eventually provide as many implementations of this plugin interface as there are
linters supported by super-linter.

## Component: driver

The driver component orchestrates all other components: it discovers all implementations of the
plugin interface, executes them, and reports the results.

The driver runs the plugins concurrently.

### API — Properties

- `Logger`: ResultLogger, collects all linter's results

### API — Method

`Run`: executes all the linters, returns `false` if any linter fails

## Component: main

The main component launches the driver, as required by the targeted execution environment. At the
moment, there will be a main component to execute slim-linter as a GitHub Action, and one to
execute it as a Unix command-line tool.

### API — Method

`Run`: setup the application, then execute the driver

## Implementation choices

Slim-linter is implemented in TypeScript with minimal dependencies.
