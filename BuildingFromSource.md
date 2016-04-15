# Building from source #

## Subversion Repository ##

The latest JODConverter source code is available from the Subversion repository. To check out the library do:
```
svn checkout http://jodconverter.googlecode.com/svn/trunk/jodconverter-core jodconverter-core
```

To check out the sample webapp:
```
svn checkout http://jodconverter.googlecode.com/svn/trunk/jodconverter-sample-webapp jodconverter-sample-webapp
```

## Maven Build ##

JODConverter uses Maven 3 as its build tool, so it can be built in the usual Maven way. A Sigar native library is now required for building (although it's optional at runtime): download the right one for your system from [svn.hyperic.org](http://svn.hyperic.org/projects/sigar_bin/dist/SIGAR_1_6_5/lib/) and save it to a local dir.
```
cd jodconverter-core
mvn -Djava.library.path=/path/to/sigar/lib install
```
and you should have `jodconverter-core-3.0-SNAPSHOT.jar` in the target dir.

The OOo installation path can be explicitly specified with e.g.
```
mvn -Doffice.home=/usr/lib64/libreoffice -Djava.library.path=/path/to/sigar/lib install
```

If there are test failure, try skipping the tests (not a good practice but because of OOo integration failures may be due to timeout values rather than real issues):
```
mvn -DskipTests install
```

You can build a ZIP distribution package containing all library dependencies with:
```
mvn assembly:single
```
which should result in a file called `jodconverter-core-3.0-SNAPSHOT-dist.zip` in the target dir.

Please see the [Maven](http://maven.apache.org) website for more information about Maven.