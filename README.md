[version_core]: https://img.shields.io/maven-metadata/v?color=informational&label=core%20(required,%20bundled)&metadataUrl=https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fdev%2Fmlnr%2FBotListHandler-core%2Fmaven-metadata.xml
[version_jda]: https://img.shields.io/maven-metadata/v?color=informational&label=jda&metadataUrl=https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fdev%2Fmlnr%2FBotListHandler-jda%2Fmaven-metadata.xml
[version_javacord]: https://img.shields.io/maven-metadata/v?color=informational&label=javacord&metadataUrl=https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fdev%2Fmlnr%2FBotListHandler-javacord%2Fmaven-metadata.xml

# BotListHandler

This handler can be used in 3 ways:
- standalone (updating the stats yourself)
- using JDA
- using Javacord

## Getting the handler

**Replace `%VERSION_xyz%` with the latest release tag:**
- ![version_core]
- ![version_jda]
- ![version_javacord]

### Modules
- core: the core module of the handler, required and bundled with every module (**however it's recommended to declare the dependency separately for independent updates**)
- jda: the jda module of the handler, use this if you intend to get the data from a JDA bot
- javacord: the javacord module of the handler, use this if you intend to get the data from a Javacord bot

### Gradle
```gradle
repositories {
  mavenCentral()
}

dependencies {
  // required (bundled with every module, see the Modules section)
  implementation group: 'dev.mlnr', name: 'BotListHandler-core', version: '%VERSION_core%'
  
  // optional
  implementation group: 'dev.mlnr', name: 'BotListHandler-jda', version: '%VERSION_jda%'
  implementation group: 'dev.mlnr', name: 'BotListHandler-javacord', version: '%VERSION_javacord%'
}
```

### Maven
```xml
<dependencies>
    <!--required (bundled with every module, see the Modules section)-->
    <dependency>
        <groupId>dev.mlnr</groupId>
        <artifactId>BotListHandler-core</artifactId>
        <version>%VERSION_core%</version>
    </dependency>
    <!--optional-->
    <dependency>
        <groupId>dev.mlnr</groupId>
        <artifactId>BotListHandler-jda</artifactId>
        <version>%VERSION_jda%</version>
    </dependency>
    <dependency>
        <groupId>dev.mlnr</groupId>
        <artifactId>BotListHandler-javacord</artifactId>
        <version>%VERSION_javacord%</version>
    </dependency>
</dependencies>
```

## Initialization

Using the `addBotList` method:
```java
BotListHandler botListHandler = new BLHBuilder().addBotList(BotList.TOP_GG, "top_gg_token")
  .addBotList(BotList.DSERVICES, "dservices_token")
  .build();
```
Using the constructor:
```java
Map<BotList, String> botLists = new EnumMap<>(BotList.class);
botLists.put(BotList.DBL, "dbl_token");
botLists.put(BotList.DEL, "del_token");

BotListHandler botListHandler = new BLHBuilder(botLists).build();
```
Using the `setBotLists` method:
```java
Map<BotList, String> botLists = new EnumMap<>(BotList.class);
botLists.put(BotList.BOTLIST_SPACE, "botlist_space_token");
botLists.put(BotList.DBOTS_GG, "dbots_gg_token");

BotListHandler botListHandler = new BLHBuilder().setBotLists(botLists).build();
```

**You can store the `BotListHandler` instance to add bot lists or hotswap invalid tokens at runtime.**

## Implementation

There are 3 ways to use BotListHandler:

### Standalone
```java
botListHandler.updateAllStats(botId, serverCount);
```

### Event based

```java
// JDA
BLHJDAListener jdaListener = new BLHJDAListener(botListHandler);
JDA jda = JDABuilder.create("token", intents)
  .addEventListeners(jdaListener)
  .build();
  
jda.awaitReady(); // optional, but if you want to update the stats after a ReadyEvent, it's required

// Javacord
BLHJavacordListener javacordListener = new BLHJavacordListener(botListHandler);
new DiscordApiBuilder().setToken(token)
        .addListener(javacordListener)
        .login();
```

### Automatic stats posting (recommended)
```java
// JDA - supports sharding
JDA jda = JDABuilder.create("token", intents)
  .build();
  
jda.awaitReady(); // optional

BLHJDAUpdater jdaUpdater = new BLHJDAUpdater(jda);
BotListHandler botListHandler = new BLHBuilder(jdaUpdater, botLists)
  .setAutoPostDelay(10, TimeUnit.MINUTES).build();

// Javacord - supports sharding. async approach - call join() after login() to block
new DiscordApiBuilder().setToken(token)
        .login()
        .thenAccept(discordApi -> {
            BLHJavacordUpdater javacordUpdater = new BLHJavacordUpdater(discordApi);
            new BLHBuilder(javacordUpdater, botLists)
                    .setAutoPostDelay(3, TimeUnit.MINUTES).build();
        });

// your own updater
MyUpdater myUpdater = new MyUpdater(); // this class has to implement IBLHUpdater
BotListHandler botListHandler = new BLHBuilder(myUpdater, botLists)
  .setAutoPostDelay(1, TimeUnit.HOURS).build();
```

### Your own updater for automatic stats posting

Creating your own updater for automatic stats posting is simple. All you have to do is to have your updater class implement `IBLHUpdater` and then provide values in the overriden methods.

## Currently supported bot lists

[botsondiscord.xyz](https://botsondiscord.xyz)

[discord.bots.gg](https://discord.bots.gg)

[discordbotlist.com](https://discordbotlist.com)

[discordbots.co](https://discordbots.co)

[discordextremelist.xyz](https://discordextremelist.xyz)

[discordlist.space](https://discordlist.space)

[discords.com](https://discords.com/bots)

[discordservices.net](https://discordservices.net)

[top.gg](https://top.gg)

## Troubleshooting

Please visit the [wiki](https://github.com/caneleex/BotListHandler/wiki/Troubleshooting) for troubleshooting steps. If the wiki doesn't contain the problem you're having, [open a new issue](https://github.com/caneleex/BotListHandler/issues/new).