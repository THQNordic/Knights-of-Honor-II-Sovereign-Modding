===== How to create a basic mod

1. Open C:\Users\<your-user>\AppData\LocalLow\BlackSeaGames\Sovereign
	- Replace <your-user> with the proper user name.
	- Alternatively you can do this:
		- Start -> Run -> %AppData% to open ...\AppData\Roaming
		- backtrack to 'AppData' and then go to 'LocalLow'
		- navigate from there to BlackSeaGames\Sovereign

2. In that folder, create a folder called Mods (if it doesn't already exist)

3. In the Mods folder create a new folder with your mod's name, e.g. "My Mod"

4. In your mod folder create some or all of the following folders: Defs, Texts, Maps, Portraits
	- do NOT copy the entire folders from the game's original files, just make these folders
	- these folders are optional, e.g. if you only want to change .def files you don't need to create Maps, Texts, etc.

5. In these folders create one or more files containing only the changes you want to make (more on that below)
	- generally you can name the files any way you want, so give them some meaningful names
	- some files are searched by the game by name and they should be named the same way

6. In your mod folder create these text files (these are optional, but highly recommended):
	- mod_name.txt: 
		- single line of text, containing the "display name" of your mod.
		- the following characters should not be used in the mod name: '/' (slash), ';' (semicolon) and '"' (quote)
		- this text will be used to display the name of your mod in the preferences and other places in the game
		- it will also be used as the name of the mod in Steam Workshop if you upload your mod there
	- mod_description.txt:
		- free form text describing your mod (it can contain multiple lines)
		- it will appear as a tooltip for your mod in the preferences
		- it will also be used as the mod description in Steam Workshop if you upload your mod there

7. Test in-game by enabling your mod from the settings.

Here is an example folder structure of a basic mod:

Mods/
	MyMod/
		Defs/
			... some .def files (and/or some .csv files)
		Texts/
			... some .csv files (and/or some .wiki files)
		mod_name.txt
		mod_description.txt



===== General comments on making mods:

- You should NOT modify the original files in the game's install folder!
	- if you do, the game will detect that changes and will not let you play multiplayer
	- use the original files only as a reference, then add any changes you want in your mod folder

- You can mod A LOT of things in the defs, but not everything:
	- For example, creating your own map is not really possible, yet
	- Adding / changing models and other assets is also not supported, yet
	- We plan to add support for these, but it will require some significant effort which will take time



===== How to mod .def files in the Defs folder

The majority of the mod changes will probably happen by modding values in the game's .def files.
To do that you first create a .def file in your mod's Defs/ folder.
The name of the .def file (or multiple files) does not matter for the game.
It is best to choose file name(s) that describe the changes you're about to make.
Then you add the changes (and only the changes!) you want to make in your .def file.

For example, imagine you want to change the gold cost for hiring a royal court character to be 5 gold flat.
After some digging you find the value you need to change in the original game's files.
It is in Defs/Characters/statuses.def.
It looks like this (most of the content is skipped for shortness):

		def ForHireStatus : CharacterStatus
		{
			...
			cost
			{
				...
				gold = (200 + num_existing * 300)
			}
		}

You can mod this value by entering the following in your mod's def files:

		ForHireStatus
		{
			cost
			{
				gold = 5
			}
		}

Or, you can use this alternative (short) syntax instead:

		ForHireStatus.cost.gold = 5

You can mix short and verbose syntax, for example:

		ForHireStatus
		{
			cost.gold = 5
		}

or

		ForHireStatus.cost
		{
			gold = 5
		}

All of the above are equivalent.

A minimal mod content would consist of a single .def file (in the Defs folder) containing a single line of text:

		ForHireStatus.cost.gold = 5

The formal syntax for every "field" in the .def files is this:

		<type> <key> : <base> = <value>
		{
			// children, also in the form of <type> <key> : <base> = <value>
		}

Where <type>, <base>, <value> and children are optional.

When modding a field you can change not only its value, but also its type and base.
For example, imagine you have something like this in the original defs:
	
		some_type key : some_base = some_val

You write the following in your mod's .def files:

		modded_type key : modded_base = modded_val

And it will change the type, base and value of the field identified by "key".
Or, if you want to change only the type, for example, you leave the rest blank, like this:

		modded_type key

In this case the final modded field will look like this:

		modded_type key : some_base = some_val

If "key" does not currently exist then a new field will be created, with the given type, base and value.

This way you can change or add anything you want in the .def files.

The only "modding operation" left is to delete or completely overwrite a field.

To delete a field, use "delete" as a type.
For example, imagine this existed in the game's def files:

		def X : Something
		{
			... // children of X before Y
			Y
			{
				Z = 1
				... // children of X.Y after Z
			}
			... // children of X after Y
		}

If you write the following in your .def files:

		delete X

The final result would be as if the field X never existed in the original game files.

Alternatively, you can do this:

		delete X.Y

or this:

		X
		{
			delete Y
		}

Both of these are equivalent and the final modded result would be:

		def X : Something
		{
			... // children of X before Y
			... // children of X after Y
		}	

You can also "re-create" a field after deleting it, which is useful to completely overwrite a field.
One way to do it is like this:

		X
		{
			delete Y
			Y.Z = 2
		}

or this:

		delete X.Y
		X.Y.Z = 2

Both of these are equivalent and the final modded result would be:

		def X : Something
		{
			... // children of X before Y
			Y
			{
				Z = 2
			}
			... // children of X after Y
		}

Here is another example of a .def file in a mod, called InstantGoverning.def:

		// syntax 1
		GovernCityAction
		{
			prepare_duration = 0 // makes the action instant
			show_in_action_bar = true // makes the action appear in the character's action bar
		}

		// syntax 2
		ReassignGovernCityAction.prepare_duration = 0

		// example of "delete"
		AssignNewGovernCityAction
		{
			delete prepare_duration // if this key is missing it will default to 0
		}

		// syntax 2 can also be used for "delete", like this:
		// delete AssignNewGovernCityAction.prepare_duration




===== How to mod .csv files in the Defs folder

All .csv files in the game's original Defs folder are technically just alternative syntax for defs.
In other words, this table:

		*;A;B
		X;1;2
		Y;3;4

is equivalent to this .def:

		X
		{
			A = 1
			B = 2
		}
		Y
		{
			A = 3
			B = 4
		}

Note the '*' in the upper-left column of the table.
It is used as a wildcard, to be replaced with the values in its column.
So, for example, this table:

		extend *;cost_Workers
		Militia;1 Workers
		AdvancedMilitia;1 Workers

is equivalent to this .def:

		extend Militia
		{
			cost_Workers = "1 Workers"
		}
		extend AdvancedMilitia
		{
			cost_Workers = "1 Workers"
		}

Both of these are actually valid mod files.
They would mod the population cost of any Militia and Advanced Militia unit to 1.

There are only two .csv files in the game's original Defs folder: units.csv and buildings.csv
They are used to define numerous parameters for units and buildings, where table form is much more convenient.
Since the .csv format is just an alternative syntax to .def files, you can use any of them.
Note that the original units.csv contains many columns, but you don't have to use all of them in a mod.
It is OK to only use the columns you want to modify.



===== How to mod texts

You may notice there are a lot of localized texts inside the game's original .def files.
You should NOT try to modify these!
Even if you change them, your changes will not have any effect.
They are not used in the game (unless you manage to activate the "Developer" language).

Most of the "localized" texts used in the game are defined in .csv files.
The majority of these texts are in a big table, in Texts/languages.csv, which looks like this:

		"id";"en";"fr";...
		...
		"GovernCityAction.name";"Govern [{target}|City]";"Gouverner [{target:def}|Ville]";"...
		...

In this table the 1st column is the text "key" (unique identifier).
The other columns are the different translations for that text for each language supported in the game.

If you want to mod some texts in that table, you must create a file in your mod's text folder, called languages.csv.
The file name matters this time, as the game looks for that file by name.
Similar to .csv files in the defs, you don't have to include all columns, just the ones you want to modify.
And again, you only include the texts you want to modify, not all the 15K texts originally present there.

Here is an example of a languages.csv file in a mod:
		
		id;en
		"GovernCityAction.name";"Govern [{target}|City] (Instant)"

This changes the English text for "GovernCityAction.name" to "Govern [{target}|City] (Instant)".

In the original game files there are also per-language folders, like "en", "fr", "de", etc.
They contain additional texts, for each language, usually texts which require different forms and .wiki files.
For example, there is a file called en_teritories.csv in Texts/en/, which looks like this:

		*;base;ian;s;anian;ians
		default;;{base};{base}'s;a {ian};{base}
		...
		tn_Sredets;Sredets;Sredetsian;Sredets';a Sredetsian;Sredetsians
		...

It defines how the town name "Sredets" looks in different forms.
The "default" row defines a default value if a cell in the table is left empty. 
This is rarely used in the game's original files, but it works and can be used by modders.

"Sredets" is actually the old name of Sofia, the capital of Bulgaria in our time.
If you want to rename "Sredets" to "Sofia", you can do it like this:

		*;base;ian;s;anian;ians
		default;;{base};{base}'s;a {ian};{base}
		tn_Sredets;Sofia;Sofian;Sofia's;a Sofian;Sofians

Note that you must always keep the original text key (unique identifier) "tn_Sredets" and just change the translation.
You can do that in any .csv file in the Texts/en/ folder in your mod, the file name doesn't matter technically.
But it is a good idea to keep the original file name, for clarity.

Also note that this will only change the name of the town in English.
If you want to also do it for other languages, you need to also change the other language folders, e.g. Texts/fr/

There is another file format, .wiki, which is used for articles in the Royal Library.
In the original game files they are stored in subfolders called "wiki", per language.
For example, Texts/en/wiki/en_world.wiki looks like this:

		...
		#article World_Description
		#caption World
		The world of "Knights of Honor II: Sovereign" is a stylized depiction of the medieval "old world" ...
		{p}This territory is split into provinces, each ruled by a kingdom...
		{p}Each province, as well as each kingdom, has a religion and culture, and ...
		...

If you want to mod that text, you create a .wiki file anywhere in the Texts/en/ folder containing this:

		#article World_Description
		#caption World
		The world of "Knights of Honor II: Sovereign" (and the Example Mod) is a stylized depiction of ...
		{p}This territory is split into provinces, each ruled by a kingdom....
		{p}Each province, as well as each kingdom, has a religion and culture, and...

You can also mod multiple articles in one .wiki file.
The file name (and the folder) doesn't matter, as long as it is in Texts/<language_key>/ and has the .wiki extension.



===== How to add a new language

If you want to add another language to the game, you will need to mod Texts/languages.def.
The original file looks like this:

		Language //= "dev"
		{
			... 
			en = "English" 	  // English
			{
				enabled = true
				voiced = true
				voice_enabled = true
				complete = true
			}

			fr = "Français"   // French
			{
				enabled = true
				voiced = true
				voice_enabled = true
				complete = true
			}
			...
		}

You mod this file just like you mod any other .def file.
Note that this file is looked up by file name, so it must be called Texts/languages.def in your mod folder.
For example, let's add a new language - Gibberish, with language code "gbrsh" (similar to "en" for English).
Create a new file called Texts/languages.def in your mod folder with the following contents:

		Language.gbrsh = "Gibberish"
		{
			enabled = true
			voiced = false
			voice_enabled = false	
		}

Note that we didn't add "complete = true" for our language.
This means this language doesn't contain all text keys.
All missing text keys will fall-back to English (hard-coded).
If you intend to make a full language pack, add "complete = true".
This will make all missing texts to appear as non-localized texts instead of using English as a fall-back.

Now, let's add some translations to Gibbersih in our Text/languages.csv:

		ID;en;gbrsh
		"GovernCityAction.name";"Govern [{target}|City] (Instant)";"Gonevr [{target}|Ticy] (Istnatn)"
		"Diplomat.name";;"Pidlotam"

Here we added a "gbrsh" column, and translations for "GovernCityAction.name" and "Diplomat.name".
Note that Diplomat.name is empty for English, meaning we leave it as-is.
Also note how we don't change text variable names, like {target}, as they are used by name in the game.

Now let's translate "Sredets" (which is now Sofia in our mod) to Gibberish, e.g. "Sifoa".
Create a file called Texts/gbrsh/gbrsh_territories.csv containing this:

		*;base;ian;s;anian;ians
		default;;{base};{base}'s;a {ian};{base}
		tn_Sredets;Sifoa;Sifoan;Sifoa';a Sifoan;Sifoans

Now, let's translate the "New Campaign" button to Gibberish, e.g. "Wen Pamcaing".
Create a file called Texts/gbrsh/gbrsh_preload.csv containing this:

		*;base;
		Title.buttons.new_game;Wen Pamcaing;

Note that "gbrsh_preload.csv" is a special file name.
It is looked up by the game by name, before all other texts are loaded.
The file must be called Texts/xxx/xxx_preload.csv, where "xxx" is the language code for the current language.



===== How to mod maps

In the game's original files you will find a folder called "Maps/europe/".
There are 2 other folders in Maps/ which you should ignore for now.
Most of the files there are generated by developer-only tools and there is no easy way to mod them.
However, there are some .def and .csv files in Maps/europe/ which can be modded easily.
You mod these just like any other .def and .csv file.
Note that all files in the Maps/ folder are looked up by name, so you need to use the same filename in your mod.

To mod anything in Maps/europe/ you must create a folder called Maps/europe/ in your mod folder.
Then create a file called just like the original file and containing only the changes you want to make.
You can also copy coatOfArms*.png to your Maps/europe/ folder and edit them - they will override the originals.
This will also work for any binary file in that folder, but for the moment you don't have a good way of modding them.



===== How to mod portraits

In the original game's files there is a folder called "Portraits", which contains .zip files.
These contain many .png files with portraits in different sizes.
To mod them, you first need to create a folder called "Portraits" in your mod folder.
Then, you have two options
	- mod complete .zip files by copying them to your Portraits folder and then change their contents
	- mod individual .png files by simply putting your version of these .png files in your Portraits folder
Note that changes to portraits may require restart of the game to take effect.



===== How to mod icons, illustrations, etc.

Textures and sprites referenced in the game's def fields can be modded.
To achieve that you need to do the following:
	- create a folder in your mod, e.g. "Icons" (the name does not matter)
	- put your new image as a .png file there (e.g. market.png)
	- mod the defs you want to reference your new image

If the original reference was to a texture, it would look simple, like this:

		texture = "assets/<path>/<filename>.<ext>"

In this case you should replace it with:

		texture = "assets/<your_folder>/<your_filename>.png"

If the original reference was to a sprite, it would look a bit more complicated, like this:

		icon = "assets/<path>/<filename>.<ext>:<sprite_name>#<index>"

In this case you should replace it with:

		icon = "assets/<your_folder>/<your_filename>.png:<your_filename>#1"


For example, the icon for the Market Square building is originally defined like this:

		def MarketSquare : Building
		{		
			...
			icon = "Assets/UI/Elements/Buildings/Icons/UI_Structure_Icon_Market.tga:UI_Structure_Icon_Market#2"
			...
		}
		
And you can replace it like this:

		MarketSquare.icon = "assets/icons/market.png:market#1"



===== How to port an earlier game version mod (pre-patch)

Before the modding rework patch (1.2.0), mods were made by copying the entire Defs, Texts and Maps folders to the mod folder.
Then modders would change anything they want in their copy of these folders.
With an active mod, the game would completely ignore the original folders and use the (modded) copies from the mod.
In other words, a mod would completely override the game's original files. All of them.

This was problematic for many reasons:
	- There was no way to have more than one mod active at a time
	- Every patch would invalidate all mods, because of changes to the game's original files
	- Mods were quite big in size - around 280 MB

Still, some nice mods are already made using that system.
With the modding rework patch, these mods will be invalidated once more (and hopefully for the last time).

In order to "port" a mod using the old system to the new one, you need to:
	- Have a clean copy of the (pre-patch) original game files used as a "base" for the mod
		- we will provide such a copy for download in case you don't have it
	- Find all the differences between the original game files and the mod files
		- there are free tools for file compare, like WinMerge, to automate that process
	- Re-create the mod, this time only containing the differences from "base"

Technically, just copying the entire Defs, Texts, Maps folders and then changing few things will still work.
But this is a HORRIBLE idea. DO NOT DO this!
If you do it, you essentially overwrite all the original game files and get all the problems listed above.

This is the reason we changed the Mods folder location.

It used to be: "%AppData%\LocalLow\BlackSeaGames\Sovereign\Saves\Mods".
Now it is: "%AppData%\LocalLow\BlackSeaGames\Sovereign\Mods".

I.e., next to "Saves", instead of inside "Saves".



===== Mod conflicts

Since the mods only contain differences (deltas) from the original game files, you can have multiple active mods.
But there is still the possibility for different mods to modify the same value differently.
For example, if we have this in a Defs/CheaperMilitia.def file in one mod (called "Conflicting Mod"):
		
		Militia.cost_Workers = "2 Workers"

and this in Defs/CheapMilitia.csv in another mod (called "Example Mod"):

		extend *;cost_Workers
		Militia;1 Workers

we have two mods trying to mod the population cost with a different value!
This is called a "conflict".

The game detects these and will not let you activate conflicting mods together.
You can activate one of these mods, but when you try to activate the other you will get an error message.
The game will also save a detailed conflicts.log in the mod folder of the mod you tried to activate.
If you want to know exactly what the problem is, check the conflicts.log, it will look like this:

		******** Conflicts between mods:ConflictingMod (Conflicting Mod) and mods:ExampleMod (Example Mod):

		======== Def conflicts: 6

		-------- Militia.cost_Workers:
		C:\...\Mods\ConflictingMod\defs\CheaperMilitia.def(1): cost_Workers = "2 Workers"
		C:\...\Mods\ExampleMod\defs\CheapMilitia.csv(2): cost_Workers = 1 Workers

		...

There you will be able to find the exact file names, line numbers, and preview of the conflicting fields.
The game will also detect and log conflicts in texts and binary files.



===== Mod versions mismatch

Every time you save a game (singleplayer or multiplayer) the game also saves information about mods used for that game.
Similarly, when you create a multiplayer campaign the mod information is stored in the "lobby" data.
When you try to load a saved game or continue a multiplayer campaign or join a new multiplayer lobby, mods are checked.
If any differences in mods are found, there will be an icon (with a tooltip) letting you know what the differences are:
	- if there is a mod you don't have, the icon will be red.
	- if you have the needed mods, but they are not active (or they are active and should not be), the icon will be yellow.
	- if there are any mods used, but they are the same as your currently active mods, the icon will be green.	
For multiplayer, you cannot continue a saved campaign or join a new campaign if your mods don't match.
For loading a singleplayer game (and for creating a new campaign from save) it is just a warning.

It is important to know that the game does a checksum of the mod files and remembers it.
Even if you have a mod with the same name, in the same folder, it could still differ in content.
It is also possible to have two mods with the exact same content - the game will treat them as the same mod.

If a mod author updates his/her mod, the content will be different and the game will recognize that.
The game will treat the new version as a completely different mod.
As a result, you can get a tooltip listing two mods with the same name as different.
This is why the game also shows the checksum of each mod in the tooltip, so you can see if their content differs.

An unpleasant side-effect of this is that you may not be able to continue a multiplayer campaign after a mod update!
A possible work-around for this is to clone the campaign from the latest save and continue from there.

There is also a special handling for "text-only" mods.
If a mod only modifies the Texts/ folder it doesn't actually change the gameplay.
These mods are not saved in the saves and are considered compatible with any save / campaign.
If you have a text-only mod activated it will not prevent you from joining a multiplayer campaign without mods.
If the saved game or campaign was created with text-only mods, you can load / join them without mods.
For example, if you use a mod translating the game to a new language, you can use that mod in multiplayer safely.



===== How to upload a mod to Steam Workshop

We will provide a tool for download, allowing you to publish your mods to Steam Workshop.
The tool is very simple - essentially you just locate the folder of your mod and hit "Publish".
Optionally, you can also provide a screenshot image and some tags for your mod (we recommend you to do so).
For new mods, you will have to click a "Get ID" button once.
This will assign a Steam Workshop ID to your mod.
It will also save that ID in a file called "mod_workshop_id.txt" in your mod folder.
This ID will be used from then on to identify your mod, so you can publish updates to it.

Note that any mod in your Mods folder will take precedence (override) mods from Steam Workshop with the same ID.
Thus if you're developing a mod, you can continue to do so even if you have subscribed to your own mod.



===== More information

We will provide an example mod, containing most of the examples given here, plus some other modifications.
For further assistance you can always refer to the community forum / discord channel.

https://community.knightsofhonor.com/
https://discord.gg/knightsofhonor