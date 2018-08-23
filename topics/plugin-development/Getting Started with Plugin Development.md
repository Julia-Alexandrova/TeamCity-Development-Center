*   [Introduction](#GettingStartedwithPluginDevelopment-Introduction)
*   [Step 1. Set up the environment](#GettingStartedwithPluginDevelopment-Step1.Setuptheenvironment)
*   [Step 2. Generate a Maven project](#GettingStartedwithPluginDevelopment-Step2.GenerateaMavenproject)
    *   [View the project structure](#GettingStartedwithPluginDevelopment-Viewtheprojectstructure)
*   [Step 3. Edit the plugin descriptor](#GettingStartedwithPluginDevelopment-Step3.Edittheplugindescriptor)
*   [Step 4. Create the plugin sources](#GettingStartedwithPluginDevelopment-Step4.Createthepluginsources)
    *   [A. Create the plugin web\-resources](#GettingStartedwithPluginDevelopment-A.Createthepluginweb-resources)
    *   [B. Create the controller and obtain the path to the JSP](#GettingStartedwithPluginDevelopment-B.CreatethecontrollerandobtainthepathtotheJSP)
    *   [C. Update the Spring bean definition](#GettingStartedwithPluginDevelopment-C.UpdatetheSpringbeandefinition)
*   [Step 5. Build your project with Maven](#GettingStartedwithPluginDevelopment-Step5.BuildyourprojectwithMaven)
*   [Step 6. Install the plugin to TeamCity](#GettingStartedwithPluginDevelopment-Step6.InstalltheplugintoTeamCity)
*   [Next Steps](#GettingStartedwithPluginDevelopment-NextSteps)

Introduction
------------

A _plugin_ in TeamCity is a `zip` archive containing a number of classes packed into a JAR file 
and [plugin descriptor](https://confluence.jetbrains.com/display/TCDL/Plugins+Packaging#PluginsPackaging-PluginDescriptor) file.  
The TeamCity Open API can be found in the JetBrains [Maven repository](http://repository.jetbrains.com/all). 
The Javadoc reference for the API is available [online](http://javadoc.jetbrains.net/teamcity/openapi/current/) 
and locally in <[TeamCity Home Directory](https://confluence.jetbrains.com/display/TCDL/TeamCity+Home+Directory)\>/devPackage/javadoc/openApi\-help.jar, 
after you install TeamCity.

Step 1. Set up the environment
------------------------------

To get started writing a plugin for TeamCity, set up the plugin development environment.

1.  Download and install [Oracle Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html). 
Set the [Java\_Home](http://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/index.html) 
environment variable on your system. Java 1.8 is required, 
the 32\-bit version is recommended, the 64\-bit version
 [can be used](https://confluence.jetbrains.com/display/TCDL/Installing+and+Configuring+the+TeamCity+Server#InstallingandConfiguringtheTeamCityServer-x64jvm).
2.  Download and install [TeamCity](https://www.jetbrains.com/teamcity/download/) on your development machine. 
Since you are going to use this machine to test your plugin, 
it is recommended that this TeamCity server is of the same version as your production server. 
We are using TeamCity 2018.1 installed on Windows in our setup.
3.  Download and install a Java IDE; we are using 
[Intellij IDEA Community Edition](https://www.jetbrains.com/idea/download/), which has a built-in Maven integration.
4.  Download and install [Apache Maven](http://maven.apache.org/download.cgi). Maven 3.2.x+ is recommended. 
Set the M2\_HOME environment variable. Run `mvn -version` to verify your setup. We are using Apache Maven 3.5.4. in our setup.

Step 2. Generate a Maven project
--------------------------------

We'll generate a Maven project
 [from an archetype](https://confluence.jetbrains.com/display/TCDL/Developing+Plugins+Using+Maven#DevelopingPluginsUsingMaven-MavenArchetypes) 
residing in the JetBrains Maven repository. 

Executing the command below will produce a project for a server-side-only plugin. 
You will be asked to enter the Maven `groudId`, `artifactId`, `version,` `package name and teamcityVersion` for your plugin.
```
mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -DarchetypeRepository=http://download.jetbrains.com/teamcity-repository -DarchetypeArtifactId=teamcity-server-plugin -DarchetypeGroupId=org.jetbrains.teamcity.archetypes -DarchetypeVersion=RELEASE
```

We used the following values:

| Property         | Value
|------------------|----------------------------------------|
|`groudId`         | `com.demoDomain.teamcity.demoPlugin`    |
|`artifactId`      | `demoPlugin`                            |
|`version`         | leave the default `1.0-SNAPSHOT`        |
|`packaging`       | leave the default package namе          |
|`teamcityVersion` | 2018.1. (See the released versions list of the [TeamCity server API](https://download.jetbrains.com/teamcity-repository/org/jetbrains/teamcity/server-api/).|



`demoPlugin` will be used as the internal name of our plugin.

When the build finishes, you'll see that the `demoPlugin` directory was created in the directory where Maven was called.

### View the project structure

The root of the `demoPlugin` directory contains the following:

*   the `readme.txt` file with minimal instructions to develop a server\-side plugin
*   the `pom.xml` file which is your Project Object Model
*   the `teamcity-plugin.xml` file which is your [plugin descriptor](https://confluence.jetbrains.com/display/TCDL/Plugins+Packaging#PluginsPackaging-PluginDescriptor) containing meta information about the plugin.
*   the `demoPlugin-server` directory contains the plugin sources:
    *   `\src\main\java\zip` contains the AppServer.java file
    *   `src\main\resources` includes resources controlling the plugin look and feel.
    *   `src\main\resources\META-INF` folder contains `build-server-plugin-demo-plugin.xml`, the bean definition file for our plugin. TeamCity plugins are initialized in their own Spring containers and every plugin needs a Spring bean definition file describing the main services of the plugin.
*   the `build` directory contains the xml files which define how the project output is aggregated into a single distributable archive.

Step 3. Edit the plugin descriptor
----------------------------------

Open the teamcity\-plugin.xml file in the project root folder with Intellij IDEA and add details, such as the plugin display name, description, vendor, and etc. by modifying [the corresponding attributes](https://confluence.jetbrains.com/display/TCDL/Plugins+Packaging#PluginsPackaging-PluginDescriptor) in the file.

Step 4. Create the plugin sources
---------------------------------

Open the `pom.xml` from the project root folder with Intellij IDEA.

We are going to make a controller class which will return `Hello.jsp` via a specific TeamCity URL.

### A. Create the plugin web\-resources

The plugin web resources (files that are accessed via hyperlinks and JSP pages) are to be placed into the `buildServerResources` subfolder of the plugin's resources.

1.  First we'll create the directory for our jsp: go to the `demoPlugin-server\src\main\resources` directory in IDEA and create the `buildServerResources` directory.
2.  In the newly created `demoPlugin-server\src\main\resources\buildServerResources` directory, create the `Hello.jsp` file, e.g.

```
<html>
<body>
Hello world
</body>
</html>
```

### B. Create the controller and obtain the path to the JSP

Go to the `\demoPlugin\demoPlugin-server\src\main\java\com\demoDomain\teamcity\demoPlugin` and open the `AppServer.java` file to create a custom controller:

1.  We'll create a simple controller which extends the TeamCity **[jetbrains.buildServer.controllers.BaseController](http://javadoc.jetbrains.net/teamcity/openapi/current/jetbrains/buildServer/controllers/BaseController.html)**  
    class and implements the `BaseController.doHandle(HttpServletRequest, HttpServletResponse)` method.
2.  The TeamCity open API provides the **[jetbrains.buildServer.web.openapi.WebControllerManager](http://javadoc.jetbrains.net/teamcity/openapi/current/jetbrains/buildServer/web/openapi/WebControllerManager.html)** which allows registering custom controllers using the path to them: the path is a part of URL starting with a slash `/` appended to the URL of the server root.
3.  Next we need to construct the path to our JSP file. When a plugin is unpacked on the TeamCity server, the paths to its resources change. 
To obtain valid paths to the files after the plugin is installed, use the **[jetbrains.buildServer.web.openapi.PluginDescriptor](http://javadoc.jetbrains.net/teamcity/openapi/current/jetbrains/buildServer/web/openapi/PluginDescriptor.html)** class 
which implements the `getPluginResourcesPath` method; otherwise TeamCity might have difficulties finding the plugin resources.
    
    ```package com.demoDomain.teamcity.demoPlugin;
      
     import jetbrains.buildServer.controllers.BaseController;
     import jetbrains.buildServer.web.openapi.PluginDescriptor;
     import jetbrains.buildServer.web.openapi.WebControllerManager;
     import org.jetbrains.annotations.Nullable;
     import org.springframework.web.servlet.ModelAndView;
      
     import javax.servlet.http.HttpServletRequest;
     import javax.servlet.http.HttpServletResponse;
      
     public class AppServer extends BaseController {
         private PluginDescriptor myDescriptor;
      
         public AppServer (WebControllerManager manager, PluginDescriptor descriptor) {
             manager.registerController("/demoPlugin.html",this);
             myDescriptor=descriptor;
         }
      
         @Nullable
         @Override
         protected ModelAndView doHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
             return new ModelAndView(myDescriptor.getPluginResourcesPath("Hello.jsp"));
         }
     }
     ```

### C. Update the Spring bean definition

Go to the `demoPlugin-server\src\main\resources\META-INF` directory and update `build-server-plugin-demo-plugin.xml` to include our AppServer class.

```<?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
   <beans default-autowire="constructor">
       <bean class="com.demoDomain.teamcity.demoPlugin.AppServer"></bean>
   </beans>
   ```


Step 5. Build your project with Maven
-------------------------------------

Go to the root directory of your project and run

```
mvn package
```

The `target` directory of the project root will contain the `<demoPlugin>.zip` file. It is our plugin package, ready to be installed.

Step 6. Install the plugin to TeamCity
--------------------------------------

1.  In the TeamCitz Web UI, go to the *Administration | Plugins List page*
 and upload a plugin zip from your local machine using the corresponding link. 
2. Once the plugin is added, the option to restart the server appears on the page. 
Click the link to restart the server and locate the TeamCity Demo Plugin in the *Administration|Plugins List* to verify the plugin was installed correctly.


The Hello World page is available via `<TeamCity server URL>/demoPlugin.html`.

Next Steps
----------

[Read more](https://confluence.jetbrains.com/display/TCDL/Web+UI+Extensions) if you want to extend the TeamCity pages with custom elements.  
The detailed information on TeamCity plugin development is available [here](https://confluence.jetbrains.com/display/TCDL/Developing+TeamCity+Plugins).

>You can also use the [TeamCity SDK Maven plugin](https://github.com/nskvortsov/teamcity-sdk-maven-plugin) allowing you to control a TeamCity instance from the command line and to install a new/updated plugin created from a Maven archetype.

