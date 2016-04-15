# New 3.0 Features #

Compared to JODConverter 2.x, version 3.0 will be a substantial rewrite of the code. The main focus will be on improving the reliability and scalability of working with an external OOo process. New features will include:

  * Manage (start/stop/restart) OOo instances from Java
  * Automatically restart an OOo instance if it crashes
  * Optionally use a pool of separate OOo instances
  * Abort conversions that take too long (according to a configurable timeout)
  * Automatically restart an OOo instance after n conversions (workaround for OOo memory leaks)

Additionally the new architecture will make it easier to use the core JODConverter classes as a generic framework for working with OOo - not just limited to document conversions.

As of beta-3, all of these features have been implemented.