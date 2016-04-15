# Configuration #

When using `DefaultOfficeManagerConfiguration`, there are a number of settings that can be configured. Some of the default settings used by JODConverter have been chosen because they have a greater chance of working out of the box, but they are not necessarily the optimal ones.

## Template Profile Dir ##

The `OfficeManager` creates a temporary profile dir for its OOo process, to avoid interfering with e.g. another OOo instance being used by the user.

By default this temporary profile will be a new one created by OOo with its own defaults settings, and relies on the `-nofirststartwizard` command line option.

You may want to provide a `templateProfileDir` containing customised settings instead. The `OfficeManager` will copy such template dir to the temporary profile, so OOo will use the same settings while still keeping the OOo instances separate.

The profile can be customised in OOo by selecting the Tools > Options menu item. Settings that may be worth customising for automated conversions include e.g.

  * Load/Save > General: you may e.g. want to disable "Save URLs relative to internet" for security reasons
  * Load/Save > Microsoft Office: these options affect conversions of embedded documents, e.g. an Excel table contained in a Word document. If not enabled, the embedded table will likely be lost when converting the Word document to another format.

## Connection Mode ##

OOo inter-process communication can use either TCP sockets or named pipes. The default is to use a TCP socket, on port 2002.

Named pipes have the advantage of not taking up TCP ports (with their potential security implications), and they are marginally faster. However they require a native library to be loaded by the JVM, and this means having to set the `java.library.path` system property. That's why it's not the default.

The path that needs to be added to `java.library.path` is different depending on the platform, but it should be the directory in the OOo installation containing `libjpipe`. On Linux it's e.g.:
```
java -Djava.library.path=/opt/openoffice.org/ure/lib ...
```
while on Windows:
```
java "-Djava.library.path=C:\Program Files\OpenOffice.org 3\URE\bin" ...
```


## Sigar Process Implementation ##

[Hyperic SIGAR](http://support.hyperic.com/display/SIGAR/Home) (System Information Gatherer And Reporter), provides a portable and native way of gathering system information.

Its a much more robust way of handling processes, and we recommend users to use this.

If you want to use Sigar in JODConverter, you have to change the ProcessManager and use SigarProcessManager

```
OfficeManager officeManager = new DefaultOfficeManagerConfiguration()
.setProcessManager(new SigarProcessManager())
.buildOfficeManager();

```

SIGAR requires native libraries to be run.
For example, minimal requirements to use the SIGAR Java API on Windows would be `sigar.jar` and `sigar-x86-winnt.dll`

Here is a complete list of native libraries required for your environment:
```
libsigar-x86-linux.so                                Linux AMD/Intel 32-bit
libsigar-amd64-linux.so                              Linux AMD/Intel 64-bit  
libsigar-ppc-linux.so                                Linux PowerPC 32-bit    
libsigar-ppc64-linux.so                              Linux PowerPC 64-bit    
libsigar-ia64-linux.so                               Linux Itanium 64-bit    
libsigar-s390x-linux.so                              Linux zSeries 64-bit    
sigar-x86-winnt.dll                                  Windows AMD/Intel 32-bit        
sigar-amd64-winnt.dll                                Windows AMD/Intel 64-bit        
libsigar-ppc-aix-5.so                                AIX PowerPC 32-bit      
libsigar-ppc64-aix-5.so                              AIX PowerPC 64-bit      
libsigar-pa-hpux-11.sl                               HP-UX PA-RISC 32-bit    
libsigar-ia64-hpux-11.sl                             HP-UX Itanium 64-bt     
libsigar-sparc-solaris.so                            Solaris Sparc 32-bit    
libsigar-sparc64-solaris.so                          Solaris Sparc 64-bit    
libsigar-x86-solaris.so                              Solaris AMD/Intel 32-bit        
libsigar-amd64-solaris.so                            Solaris AMD/Intel 64-bit        
libsigar-universal-macosx.dylib                      Mac OS X PowerPC/Intel 32-bit   
libsigar-universal64-macosx.dylib                    Mac OS X PowerPC/Intel 64-bit   
libsigar-x86-freebsd-5.so                            FreeBSD 5.x AMD/Intel 32-bit    
libsigar-x86-freebsd-6.so                            FreeBSD 6.x AMD/Intel 64-bit    
libsigar-amd64-freebsd-6.so                          FreeBSD 6.x AMD/Intel 64-bit
```
For an updated list see [here.](http://support.hyperic.com/display/SIGAR/Home#Home-binaries)

### Including Sigar in your project ###
There are two ways to include Sigar in your project

**One way is to** download Sigar distribution manually from [here](https://repository.jboss.org/nexus/content/repositories/thirdparty-uploads/org/hyperic/sigar-dist/1.6.5.132/sigar-dist-1.6.5.132.zip)
then unzip the file, and include `sigar.jar` to the classpath and the appropriate native library to `java.library.path`

---

However, if you want a more automatic way of doing it, you can add the following to the pom.xml file, and let maven do it for you:
```
  <dependency>
    <groupId>org.hyperic</groupId>
    <artifactId>sigar</artifactId>
    <version>1.6.5.132</version>
  </dependency>
  <dependency>
    <groupId>org.hyperic</groupId>
    <artifactId>sigar-dist</artifactId>
    <version>1.6.5.132</version>
    <type>zip</type>
  </dependency>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>2.2</version>
        <executions>
          <execution>
            <id>unpack-dependencies</id>
            <phase>compile</phase>
            <goals>
              <goal>unpack-dependencies</goal>
            </goals>
            <configuration>
              <includes>**/sigar-bin/lib/*</includes>
              <excludes>**/sigar-bin/lib/*jar</excludes>
              <includeGroupIds>org.hyperic</includeGroupIds>
              <includeArtifactIds>sigar-dist</includeArtifactIds>
              <outputDirectory>
                ${project.build.directory}/depends
                <!-- or: ${project.basedir}/depends -->
              </outputDirectory>
              </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```
This will create a folder called `depends` where the native libraries are found. You will need to add this folder to `java.library.path`.