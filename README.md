appbundler
=============

A fork of the [Java Application Bundler](https://svn.java.net/svn/appbundler~svn) 
with the following changes:

- The native binary is created as universal ~(32/64)~ (x86_64/arm64)
- Fixes [icon not showing bug](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=7159381) in `JavaAppLauncher`
- Adds `LC_CTYPE` environment variable to the `Info.plist` file in order to fix an [issue with `File.exists()` in OpenJDK 7](http://java.net/jira/browse/MACOSX_PORT-165)  **(Contributed by Steve Hannah)**
- Allows to specify the name of the executable instead of using the default `"JavaAppLauncher"` **(contributed by Karl von Randow)**
- Adds `classpathref` support to the `bundleapp` task
- Adds support for `JVMArchs` and `LSArchitecturePriority` keys
- Allows to specify a custom value for `CFBundleVersion` 
- Allows specifying registered file extensions using `CFBundleDocumentTypes` and `UT[Ex|Im]portedTypeDeclarations`
- Passes to the Java application a set of system properties with the paths of
  the OSX special folders, whether the application is running in the
  sandbox (see below) and which modifier keys are held down while opening the app. With the latter, the Java application can mimic the behavior of e.g. iTunes or Photos to select another or create a new media library on startup with the option key.
- Allows overriding of passed JVM options by the bundled app itself via java.util.Preferences **(contributed by Hendrik Schreiber)**
- Allows writing arbitrary key-value pairs to `Info.plist` via `plistentry`
- Allows setting of environment variables via `Info.plist`
- Allows running the Application as `privileged` - e.g. for Setup **(Contributed by Gerry Weißbach)**
- Allows specifying a JNLP file (`jnlplaunchername`) as alternative to the `mainclassname` which can then be launched without hassle when the Application is signed. See [How to sign (dynamic) JNLP files for OSX 10.8.4 and Gatekeeper](http://stackoverflow.com/questions/16958130/how-to-sign-dynamic-jnlp-files-for-osx-10-8-4-and-gatekeeper) **(Contributed by Gerry Weißbach)**

These are the system properties passed to the JVM:

- `LibraryDirectory`
- `DocumentsDirectory`
- `CachesDirectory`
- `ApplicationSupportDirectory`
- `ApplicationDirectory`
- `AutosavedInformationDirectory`
- `DesktopDirectory`
- `DownloadsDirectory`
- `MoviesDirectory`
- `MusicDirectory`
- `PicturesDirectory`
- `SharedPublicDirectory`
- `SystemLibraryDirectory`
- `SystemApplicationSupportDirectory`
- `SystemCachesDirectory`
- `SystemApplicationDirectory`
- `SystemUserDirectory`
- `UserHome`  (the user's home directory, even if running within a sandbox)
- `SandboxEnabled` (the String `true` or `false`)
- `LaunchModifierFlagCapsLock` (the String `true` or `false`)
- `LaunchModifierFlagShift` (the String `true` or `false`)
- `LaunchModifierFlagControl` (the String `true` or `false`)
- `LaunchModifierFlagOption` (the String `true` or `false`)
- `LaunchModifierFlagCommand` (the String `true` or `false`)
- `LaunchModifierFlagNumericPad` (the String `true` or `false`)
- `LaunchModifierFlagHelp` (the String `true` or `false`)
- `LaunchModifierFlagFunction` (the String `true` or `false`)
- `LaunchModifierFlags` (an Integer)


For more details, please refer to the [task documentation](http://htmlpreview.github.io/?https://bitbucket.org/infinitekind/appbundler/raw/tip/appbundler/doc/appbundler.html).

Example 1:

    <target name="bundle">
      <taskdef name="bundleapp" 
        classpath="appbundler-1.0ea.jar"
        classname="com.oracle.appbundler.AppBundlerTask"/>

      <bundleapp 
          classpathref="runclasspathref"
          outputdirectory="${dist}"
          name="${bundle.name}"
          displayname="${bundle.displayname}"
          executableName="MyApp"
          identifier="com.company.product"
          shortversion="${version.public}"
          version="${version.internal}"
          icon="${icons.path}/${bundle.icns}"
          mainclassname="Main"
          copyright="2012 Your Company"
          applicationCategory="public.app-category.finance">
          
          <runtime dir="${runtime}/Contents/Home"/>

          <!-- the order matters here, and should be in order of preference -->
          <arch name="arm64"/>
          <arch name="x86_64"/>

          <bundledocument extensions="png,jpg"
            icon="${icons.path}/${image.icns}"
            name="Images"
            role="editor"
            handlerRank="owner">
          </bundledocument> 

          <bundledocument contentTypes="com.adobe.pdf"
            name="PDF files"
            role="viewer"
            handlerRank="alternate">
          </bundledocument>
          
          <bundledocument contentTypes="com.my.custom"
            name="Custom data"
            role="editor"
            exportableTypes="com.topografix.gpx">
          </bundledocument>
          
          <typedeclaration
            identifier="com.my.custom"
            description="Custom data"
            icon="${icons.path}/${data.icns}"
            conformsTo="com.apple.package"
            extensions="custom"
            mimeTypes="application/x-custom" />
        
          <typedeclaration
      	   imported = "true"
            identifier="com.topografix.gpx"
            referenceUrl="http://www.topografix.com/GPX/1/1/"
            description="GPS Exchange Format (GPX)"
            conformsTo="public.xml"
            extensions="gpx"
            mimeTypes="application/gpx+xml" />
          
          
          <!-- Define custom key-value pairs in Info.plist -->
          <plistentry key="ABCCustomKey" value="foobar"/>
          <plistentry key="ABCCustomBoolean" value="true" type="boolean"/>

          <!-- Workaround as com.apple.mrj.application.apple.menu.about.name property may no longer work -->
          <option value="-Xdock:name=${bundle.name}"/>

          <option value="-Dapple.laf.useScreenMenuBar=true"/>
          <option value="-Dcom.apple.macos.use-file-dialog-packages=true"/>
          <option value="-Dcom.apple.macos.useScreenMenuBar=true"/>
          <option value="-Dcom.apple.mrj.application.apple.menu.about.name=${bundle.name}"/>
          <option value="-Dcom.apple.smallTabs=true"/>
          <option value="-Dfile.encoding=UTF-8"/>

          <option value="-Xmx1024M" name="Xmx"/>
      </bundleapp>
    </target>

Example 2, use installed Java but require Java 8 (or later):

    <target name="bundle">
      <taskdef name="bundleapp" 
        classpath="appbundler-1.0ea.jar"
        classname="com.oracle.appbundler.AppBundlerTask"/>
      <bundleapp 
          jvmrequired="1.8"
          classpathref="runclasspathref"
          outputdirectory="${dist}"
          name="${bundle.name}"
          displayname="${bundle.displayname}"
          executableName="MyApp"
          identifier="com.company.product"
          shortversion="${version.public}"
          version="${version.internal}"
          icon="${icons.path}/${bundle.icns}"
          mainclassname="Main"
          copyright="2012 Your Company"
          applicationCategory="public.app-category.finance">
      </bundleapp>
    </target>

Example 2, use installed Java but require Java 8 (or later) JRE and not a JDK:

    <target name="bundle">
      <taskdef name="bundleapp" 
        classpath="appbundler-1.0ea.jar"
        classname="com.oracle.appbundler.AppBundlerTask"/>
      <bundleapp 
          jvmrequired="1.8"
          jrepreferred="true"
          classpathref="runclasspathref"
          outputdirectory="${dist}"
          name="${bundle.name}"
          displayname="${bundle.displayname}"
          executableName="MyApp"
          identifier="com.company.product"
          shortversion="${version.public}"
          version="${version.internal}"
          icon="${icons.path}/${bundle.icns}"
          mainclassname="Main"
          copyright="2012 Your Company"
          applicationCategory="public.app-category.finance">
      </bundleapp>
    </target>


