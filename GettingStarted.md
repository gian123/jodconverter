# Prerequisites #

JODConverter 3.0 requires:

  1. Java 1.5 or later
  1. [OpenOffice.org](http://download.openoffice.org) 3.x or [LibreOffice](http://www.libreoffice.org/download/)

We recommend that you use the latest OpenOffice.org or LibreOffice version.

# Command Line #

The command line tool provides a good way to check that everything is working, i.e. that you have the right OOo/LibO version installed etc. To convert a document do e.g.:
```
java -jar lib/jodconverter-core-3.0-beta-4.jar test.odt test.pdf
```

Note: if all you want is a command line tool, you may also have a look at this [Python script](http://www.artofsolving.com/opensource/pyodconverter). It's smaller, simpler, and easier to customise.

**Important**: This tool is not intended to support multiple concurrent executions, e.g. being invoked at each web request from a PHP or Ruby app.

# Java Code #

Here's a basic example of Java code that converts a single document:
```
    OfficeManager officeManager = new DefaultOfficeManagerConfiguration().buildOfficeManager();
    officeManager.start();

    OfficeDocumentConverter converter = new OfficeDocumentConverter(officeManager);
    converter.convert(new File("test.odt"), new File("test.pdf");
        
    officeManager.stop();
```

Note that these few lines will actually do quite a few things: start an OOo process that listens on the port 2002, connect to it, tell OOo/LibO to load `test.odt`, then to save it as pdf, and finally terminate the OOo/LibO process.

Here's another example showing a few configuration options:
```
    OfficeManager officeManager = new DefaultOfficeManagerConfiguration()
      .setOfficeHome("/usr/lib/openoffice")
      .setConnectionProtocol(OfficeConnectionProtocol.PIPE)
      .setPipeNames("office1", "office2")
      .setTaskExecutionTimeout(30000L)
      .buildOfficeManager();
```
This `officeManager` will use the OOo/LibO installation in the specified directory, connecting via named pipes rather than TCP sockets, use a pool of two OOo/LibO processes identified by the given pipe names, and abort any conversion that takes longer than 30 seconds. (Note that using named pipes requires adding a native library to `java.library.path`; see [Configuration](Configuration.md) for more info.)

## Web Applications ##

A basic example of a web application is available in the Subversion repository, under `jodconverter-sample-webapp`.

The most important concept is that you should start a single `OfficeManager` instance when your webapp starts (using a servlet listener, Spring context configuration, or equivalent facility provided by your framework of choice), stop it when your webapp stops, and share it across all requests. The `OfficeManager` will take care of multi-threading.