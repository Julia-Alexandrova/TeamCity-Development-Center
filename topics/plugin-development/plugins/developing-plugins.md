Developing TeamCity Plugins
=========================


TeamCity functionality can be significantly extended by custom plugins. 
TeamCity plugins are written in Java 
(any JVM language with Java invulnerability like Kotlin or Groovy can be used), 
run within the TeamCity application and have access to internal entities 
of the TeamCity server or agent.

Besides this documentation, the following sources are available to you:

- [Open API Javadoc](http://javadoc.jetbrains.net/teamcity/openapi/current/)
- [the bundled sample plugin](https://confluence.jetbrains.com/display/TCD18/Bundled+Development+Package#BundledDevelopmentPackage-SamplePlugin)
- [the list of existing plugins and bundled open-source plugins](https://confluence.jetbrains.com/display/TW/Open-source+Bundled+Plugins)
- [TeamCity plugins forum](https://teamcity-support.jetbrains.com/hc/en-us/community/topics/200366719-TeamCity-Plugin-Development), 
where you can post questions. Please use the forum search before posting to avoid possible duplication.

Consider making your plugin public and submit it to the JetBrains [TeamCity plugin repository](https://plugins.jetbrains.com/teamcity) .