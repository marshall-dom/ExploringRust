# More About Cargo and Crates
In addition to the basic build, run and test features, Cargo also offers number of other
advanced features such as:
- Build customization through release profiles
- Publishing of crates on crates.io
- Organization of large projects with workspaces
- Installation of binaries from crates.io
- Extension of Cargo via custom commands

## Customizing Builds with Release Profiles
- Rust's release profiles are predefined and customizable profiles with different configurations
that allow us to have more control over compilation; each profile is configured independently
- Cargo has two main profiles:
    - dev
    - release
- The dev profile is defined with good defaults for development
- The release profile is defined with defaults suitable for release builds
- Default settings can be overriden by adding [profile.\*] sections for any profile you wish
to modify

## Publishing a Crate to Crates.io

### Making Useful Documentation Comments
- Rust has a particular kind of comment dedicated to writing documentation referred to as a
documentation comment; comments written in this syntax will automatically generate HTML
documentation
- Documentation comments use three slashes and support Markdown notation for formatting

### Exporting a Convenient Public API with pub use
- The structure of your public API should be a major consideration when publishing a crate.
Whoever will be using your crate will be unfamiliar with its structure; its hierarchy
should be as straightforwad as possible
- Cargo allows you to re-export items to make a public structure that is different from your
private structure via pub use. This takes a public item in one location and makes it public
somewhere else--as if it were definited in that location instead. This is incredibly helpful
for creating easy-to-use APIs even if your internal structure is a bit unorganized; you can
restructure your public API without making any changes to your internal project structure

### Setting Up a Crates.io Account
- You must create an account and get an API token before publishing any crates to crates.io
- Run the cargo login command to set up the command line tooling:

```bash
cargo login [API Key]
```

### Adding Metadata to a New Crate
- Before publishing, you must add metadata to your crate; this is done in the Cargo.toml
file in the [package] section
- The following metadata is needed:
    - name
    - version
    - author
    - description
    - license
- To publish the crate, run the cargo publish command

### Publishing a New Version of an Existing Crate
To publish an update to a crate, simply change the version number in the Cargo.toml file
and run the cargo publish command

### Removing Versions from Crates.io with cargo yank
- You can not remove previous versions of a crate from crates.io; however, you can
prevent any future projects from adding them as a new dependency
- Yanking a crate version allows existing projects to continue to depend on your crate,
but will prevent new projects from depending on the yanked version
- To yank a version of a crate, run the cargo yank command and use the --vers flag to
specify which version of the crate you would like to yank
- Yanks can also be done with the --undo flag
- **NOTE**: Yanking does not delete any code. If secrets are accidentally uploaded they must
be reset

## Cargo Workspaces
If you find that the library crate in your project is becoming too large, it is possible
to split your package into multiple library crates. Cargo has a built-in feature called
workspaces that helps to manage multiple related packages that are developed in tandem.
Structuring larger projects in this way allows you to maintain smaller, more managable
components and will help simplify coordination between crates if they are often changed
at the same time.

Note that if you wish to publish the crates in a workspace to crates.io, each crate in
the workspace must be published separately

### Creating a Workspace
- A workspace is a set of packages that share the same Cargo.lock file and output directory
- The Cargo.toml file in the root directory of the workspace contains a [workspace] section
that specifies the paths to each library crate in the workspace
- Each library crate in the workspace maintains its own Cargo.toml file; this file
will contain the dependencies for the crate
- **NOTE**: By default cargo does not assume that crates in a workspace will depend on each other,
so dependency relationships between crates must be explicitly defined

### Depending on an External Crate in a Workspace
There is only one Cargo.lock file for the workspace; this is located in the root directory of
the workspace. Having one .lock file for the entire workspace ensures that all of the crates in
the workspace are using the same version of all dependencies; this guarantees compatability among
all crates in the workspace and saves space as each dependency is only compiled once

### Adding Tests to a Workspace
- Running cargo test in a workspace will run test for all crates in the workspace
- We can run tests on a particular crate by using the -p flag with the cargo test command
to specify which crate we want to test

## Installing Binaries from Crates.io with cargo install
- The cargo install command allows you to install and use binary crates locally
- **NOTE**: You can only install packages that have binary targets. This means you
can only use it to install binaries that will run on their own; cargo install will not
work for crates that are only library targets
- Binaries installed via cargo install are stored in the installation root's bin folder

## Extending Cargo with Custom Commands
- Cargo can be extended to run custom subcommands; Any binary in your $PATH can be run as a
cargo subcommand
- To list all custom commands, run cargo --list
- Extensions can be installed via cargo install and then run as subcommands
    - Sweet!
