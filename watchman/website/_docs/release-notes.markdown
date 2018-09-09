---
id: release-notes
title: Release Notes
layout: docs
section: Installation
permalink: docs/release-notes.html
---

We focus on the highlights only in these release notes.  For a full history
that includes all of the gory details, please see [the commit history on
GitHub](https://github.com/facebook/watchman/commits/master).

### Watchman 4.6.0 (2016-07-09)

* Improved I/O scheduling when processing recursive deletes and deep directory
  rename operations.
* Improved performance of the `ignore_dirs` configuration option on macOS and
  Windows systems.  We take advantage of an undocumented (but supported!)
  API to further accelerate this for the first 8 entries in the `ignore_dirs`
  on macOS.  Users that depend on this configuration to avoid recrawls will
  want to review and prioritize their most active build dirs to the front
  of the `ignore_dirs` specified in their `.watchmanconfig` file.
* Added an optional recrawl recovery strategy for macOS that will attempt to
  resync from the fseventsd journal rather than performing a full filesystem
  walk.  This is currently disabled by default but will likely be enabled
  by default in the next Watchman release.  You can enable this by setting
  `fsevents_try_resync: true` in either `/etc/watchman.json` or your
  `.watchmanconfig`.  This should reduce the frequency of recrawl warnings
  for some users/workloads, and also improves I/O for users with extremely
  large trees.
* Fixed accidental exponential time complexity issue with recursive deletes
  and deep directory rename operations on case-insensitive filesystems (such as
  macOS).  This manifested as high CPU utilization for extended periods of time.
* Added support for allowing non-owner access to a Watchman instance.  Only
  the owner is authorized to create or delete watches.  Non-owners can view
  information about existing watches.  Access control is based on unix domain
  socket permissions.  The new but not yet documented configuration options
  `sock_group` and `sock_access` can be used to control this new behavior.
* Added support for inetd-style socket activation of the watchman service.
  [this commit includes a sample configuration for systemd](https://github.com/facebook/watchman/commit/2985377eaf8c8538b28fae9add061b67991a87c2).
* Added the `symlink_target` field to the stored metadata for files.  This
  holds the text of the symbolic link for symlinks.  You can test whether it
  is supported by a watchman server using the capability name
  `field-symlink_target`.
* Fixed an issue where watchman may not reap child processes spawned by
  triggers.
* Fixed an issue where watchman may block forever during shutdown if there
  are other connected clients.
* Added `hint_num_dirs` configuration option.


### pywatchman 1.4.0 (????-??-??)

(These changes have not yet been released to pypi)

* Added immutable version of data results to bser.  This is cheaper to build
  from a serialized bser representation than the mutable version and is
  better suited to large result sets received from watchman.
* Fixed a number of misc. portability issues
* Added Python 3.x support

### Watchman 4.5.0 (2016-02-18)

* Fixed an inotify race condition for non-atomic directory replacements
  that was introduced in Watchman 4.4.

### Watchman 4.4.0 (2016-02-02)

* Added state-enter and state-leave commands can allow subscribers to more
  intelligently settle/coalesce events around hg update or builds.
* Fixed an issue where subscriptions could double-notify for the same events.
* Fixed an issue where subscriptions that never match any files add
  O(all-observed-files) CPU cost to every subscription dispatch

### Watchman 4.3.0 (2015-12-14)

* Improved handling of case insensitive renames; halved the memory usage
  and doubled crawl speed on OS X.

### Watchman 4.2.0 (2015-12-08)

* Increased strictness of checks for symlinks; rather than just checking
  whether the leaf of a directory tree is a symlink, we now check each
  component down from the root of the watch.  This improves detection
  and processing for directory-to-symlink (and vice versa) transitions.
* Increased priority of the watchman process on OS X.

### pywatchman 1.3.0 (2015-10-22)

* Added `watchman-make` and `watchman-wait` commands
* Added pure python implementation of BSER

### Watchman 4.1.0 (2015-10-20)

* Fixed an issue where symlink size was always reported as 0 on OS X
  using the new bulkstat functionality

### Watchman 4.0.0 (2015-10-19)

* Fixed an issue where a directory that was replaced by a symlink would
  cause a symlink traversal instead of correctly updating the type of the
  node and marking the children removed.
* Fixed a debugging log line that was emitted at the wrong log level on
  every directory traversal.

### Watchman 3.9.0 (2015-10-12)

* Fixed an issue where dir renames on OS X could cause us to lose track of
  the files inside the renamed dir
* Fixed an issue where dir deletes and replacements on Linux could cause us
  to lose track of the files inside the replaced dir (similar to the OS X issue
  above in manifestation, but a different root cause).
* Improved (re)crawl speed for dirs with more than a couple of entries on average
  (improvement can be up to 5x for dirs with up to 64 entries on average).
  You may now tune the `hint_num_files_per_dir` setting in your
  `.watchmanconfig` to better match your tree.  [More details](
  /watchman/docs/config.html#hint_num_files_per_dir)
* Improved (re)crawl speed on OS X 10.10 and later by using `getattrlistbulk`.
  This allows us to improve the data:syscall ratio during crawling and can
  improve throughput by up to 40% for larger trees.
* Add optional `sync_timeout` to the `clock` command
* Avoid accidentally passing descriptors other than the stdio streams
  when we spawn the watchman service.
* Fixed a race condition where we could start two sets of watcher threads
  for the same dir if two clients issue a `watch` or `watch-project` at
  the same time
* Added a helpful error for a tmux + launchd issue on OS X

### Watchman 3.8.0 (2015-09-14)

* Improved latency of processing kernel notifications. It should now be far
  less likely to run into an notification queue overflow.
* Improved idle behavior. There were a couple of places where watchman would
  wake up more often than was strictly needed and these have now been fixed.
  This is mostly of interest to laptop users on battery power.
* Improved inotify move tracking.  Some move operations could cause watchman
  to become confused and trigger a recrawl.  This has now been resolved.
* Hardened statedir and permissions. There was a possibility of a symlink
  attack and this has now been mitigated by re-structuring the statedir layout.
* Fixed a possible deadlock in the idle watch reaper
* Fixed an issue where the watchman -p log-level debug could drop log
  notifications in the CLI
* Disabled the IO-throttling-during-crawl that we added in 3.7. It proved to
  be more harmful than beneficial.
* `-j` CLI option now accepts either JSON or BSER encoded command on stdin
* Added [capabilities](/watchman/docs/capabilities.html) to the server,
  and added the [capabilityCheck](/watchman/docs/cmd/version.html#capabilityCheck)
  method to the python and node clients.

### pywatchman 1.2.0 (2015-08-15)

* Added the `capabilityCheck` method
* Added `SocketTimeout` exception to distinguish timeouts from protocol level
  exceptions

### fb-watchman 1.3.0 for node (2015-08-15)

* Added the [capabilityCheck](/watchman/docs/nodejs.html#checking-for-watchman-availability) method.

### pywatchman 1.0.0 (2015-08-06)

* First official pypi release, thanks to [@kwlzn](https://github.com/kwlzn)
  for setting up the release machinery for this.

### Watchman 3.7.0 (2015-08-05)

(Watchman 3.6.0 wasn't formally released)

* Fixed bug where `query match` on `foo*.java` with `wholename` scope
  would incorrectly match `foo/bar/baz.java`.
* Added `src/**/*.java` recursive glob pattern support to `query match`.
* Added options dictionary to `query`'s `match` operator.
* Added `includedotfiles` option to `query match` to include files
  whose names start with `.`.
* Added `noescape` option to `query match` to make `\` match literal `\`.
* We'll now automatically age out and stop watches. See [idle_reap_age_seconds](
/watchman/docs/config.html#idle_reap_age_seconds) for more information.
* `watch-project` will now try harder to re-use an existing watch and avoid
  creating an overlapping watch.
* Reduce I/O priority during crawling on systems that support this
* Fixed issue with the `long long` data type in the python BSER module

### fb-watchman 1.2.0 for node (2015-07-11)

* Updated the node client to more gracefully handle `undefined` values in
  objects when serializing them; we now omit keys whose values are `undefined`
  rather than throw an exception.

### Watchman 3.5.0 (2015-06-29)

* Fix the version number reported by watchman.

### Watchman 3.4.0 (2015-06-29)

* `trigger` now supports an optional `relative_root` argument. The trigger is
  evaluated with respect to this subdirectory. See
  [trigger](/watchman/docs/cmd/trigger.html#relative-roots) for more.

### fb-watchman 1.1.0 for node (2015-06-25)

* Updated the node client to handle 64-bit integer values using the
  [node-int64](https://www.npmjs.com/package/node-int64).  These are most
  likely to show up if your query fields include `size` and you have files
  larger than 2GB in your watched root.

### fb-watchman 1.0.0 for node (2015-06-23)

* Updated the node client to support [BSER](/watchman/docs/bser.html)
  encoding, fixing a quadratic performance issue in the JSON stream
  decoder that was used previously.

### Watchman 3.3.0 (2015-06-22)

* `query` and `subscribe` now support an optional `relative_root`
  argument. Inputs and outputs are evaluated with respect to this
  subdirectory. See
  [File Queries](/watchman/docs/file-query.html#relative-roots) for more.
