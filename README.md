# Squad Plugin Framework

**Any mods using Squad_Plugin_Framework must not edit the files, only reference the mod's classes, and set Squad_Plugin_Framework as a required mod on Steam Workshop**

This framework introduces mod cross-compatibility to Squad. Traditionally, Squad mods are compartmentalised into 'layers', restricting mods to new maps or total gameplay overhauls. To make mods compatible, a modder would have to provide source files to other modders, which would have to be hard referenced in other mods. This is an unsustainable approach and causes dependency hell: Mod B referencing an outdated version of Mod A would cause both Mod A and Mod B to crash.

This framework attempts to overcome these issues by standardising a mod asset structure as plugins. These plugins can then be loaded into levels at runtime, without any need for hard referencing other mods. Using UE4's asset registry, a plugin manager can fetch any plugins installed on a client/server and add these to the level.

This project is fully open source: https://github.com/Dan186D/Squad_Plugin_Framework

## Provided Plugin Classes

### SquadPlugin_C (ABSTRACT) inherits SQPrimaryData
The base plugin class. Includes an optional display name for UI purposes.

### SquadPlugin_Gamemode_C inherits SquadPlugin_C
Stores a soft class reference to a BP_GameMode, allowing any gamemode – including all of its custom classes e.g. player controller, hud, rulesets – to be loaded into a generic layer. Only one gamemode can be used at a time, so treat the gamemode as the base experience you’re looking for, and other plugins as optional extras.

### SquadPlugin_Additional_C (ABSTRACT) inherits SquadPlugin_C
A parent class for any plugins which can be loaded additionally to a gamemode.

### SquadPlugin_Ruleset_C inherits SquadPlugin_Additional_C
Stores a soft class reference to a SP_RulesetBP, allowing additional rulesets to be loaded into a generic layer. Rulesets are a proprietary OWI class which can be used as gameplay modifiers and mutators, including events for when players spawn and die etc.

### SquadPlugin_Replicated_C (ABSTRACT) inherits SquadPlugin_Additional_C
A parent class for any plugins which can be loaded on the client and/or the server. Stores a SP_PluginType enum stating how it should be loaded: only on clients, clients and the server, or just the server (which could then also replicate the spawned class to clients for networking purposes).

### SquadPlugin_Actor_C inherits SquadPlugin_Replicated_C
Stores a soft class reference to an Actor, allowing actors to be spawned into the world. This actor could be used as a manager which handles gameplay tasks and adds new features. For example, this actor could have a post process component with a night vision filter, which is toggled via a keyboard input. This would allow night vision to be loaded into any mod as a plugin.

## Rulesets

Squad does not currently support adding rulesets at runtime, so this framework provides a fake ruleset class, SP_RulesetBP, which is identical to a Squad ruleset. A SP_RulesetManager is a wrapper Squad ruleset that contains a list of the plugin SP_RulesetBPs. This SP_RulesetManager is then added as a map ruleset (in the level’s world settings), and will translate any Squad ruleset events to the plugin SP_RulesetBPs.

## Template Plugin Loader

This framework includes a template SquadPlugin Manager, SP_Manager. A modder can implement this manager however they want, loading the standardised SquadPlugin classes using the asset registry, but this is a useful guide.

This is an actor added to a level which handles all the plugin loading. On begin play, the manager attempts to load the saved gamemode plugin. Changing gamemode requires a level reload, so if the gamemode is changed this begin play flow will restart. The server’s manager then fetches any installed plugins, and replicates this to the client. The manager can hard reference required plugin assets, for plugins you don’t want users disabling, though this requires the modder to have the source code for the plugin (or a faked version of it). This template manager splits the available plugins into additional (optional) and gamemode plugins. The server’s manager then attempts to load the saved optional plugin selection, and replicates this to the client. The server manager then finally spawns all the required and active optional plugins into the level, with the client managers then also spawning any replicated plugins.

This template provides 2 extra callable functions, LoadGamemode(SquadPlugin_Gamemode) and SaveOptionalPluginSelection(SquadPlugin_Additional[]). These should be called by a UI of some kind, which this template does not provide.
