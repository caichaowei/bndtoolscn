# Bndtools Repositories

Bndtools uses repositories to supply dependencies to be used at build-time and at runtime. A repository is essentially a collection of bundles, optionally with some kind of index. It may be located anywhere: in the workspace; somewhere else on the local file system; or on a remote server.

Bndtools uses repositories in the following ways:

To look up bundles specified on the Build Path by Bundle Symbolic Name (BSN) and version;
To resolve the Run Requirements list;
To look up bundles specified in the Run Bundles list by BSN and version.
Repositories are implemented as bnd plug-ins, and can be configured by editing the Plugins sections of the workspace configuration file (Bndtools menu » Open main bnd config).