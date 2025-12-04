## Make It Work: Playing the Thief Series on Proton

*Nostalgia is a bitch, but I like her sometimes.*

![A portrait of Garrett from Thief Gold](/images/thief-gold-key-art-header.png)

The Thief series is still to this day one of my favorite game series. The immersive world and fantastic story are not commonly matched in today's gaming. Given that I now have a bunch of free time, I find myself returning to my childhood...because that's what you do when reality stinks. If you have not yet enjoyed any of the below games, please do. Play at night with the lights off and a good set of headphones.
+ Thief: The Dark Project
+ Thief II: The Metal Age
+ Thief: Deadly Shadows

Yes, Thief 4 was good, but out of scope.

### Install a Thingy

*"...but Trixie, you don't use Windows! You can't possibly play a video game!"*

Nonsense, poopy pants. With the introduction of Proton, Linux gaming has come a long way for the average gamer who doesn't argue over which text editor is best. However, sometimes a little manual work is still necessary to get a game going...especially a game from the 90's designed to run on Windows 98. Here's a little how-to so you can save yourself the headache.

Gamer setup:
+ Pop_OS! 22.04.
+ Steam version of the games.
+ GE-Proton-10-15.

You can also use Good Old Games or the CD, but you'll need to use Wine manually.

#### Thief: The Dark Project (and Thief Gold)
1. Install the base game.
2. Set the compatibility mode to run with Proton-GE 10-15.
3. Run the game once. It will get angy at you and the cutscenes won't work. Close the game.
4. Download [TFix](https://www.moddb.com/downloads/start/180264) and add it as a non-Steam game.
5. Set the compatibility mode to run with Proton-GE 10-15.
6. Run and install TFix. When it asks for the Thief installation location, you will need to manually type ```z:\home\USER\.steam\steam\steamapps\common\thief_gold\``` as the installer will not show hidden Linux folders.
7. Remove TFix from Steam.
8. Delete the files ```ir50_32.dll``` and ```ir41_32.dll``` from the installation directory as they conflict with Wine's versions. 
9. Run Thief.

All cutscenes should work. However, if you'd like to upgrade the models and cutscenes to a higher definition, then there are additional steps.

##### HD Mod
1. Download the [Thief HD Mod](https://www.moddb.com/mods/thief-gold-hd-texture-mod/downloads/thief-1-hd-mod-12-full-version-installer1).
2. Set the compatibility mode to run with Proton-GE 10-15.
3. Run and install the HD mod. When it asks for the Thief installation location, you will need to manually type ```z:\home\USER\.steam\steam\steamapps\common\thief_gold\``` as the installer will not show hidden Linux folders.
4. Ensure that LArge Address Awareness is enabled when prompted.
5. Remove the HD mod from Steam.
6. Run Thief.

##### HD Cutscenes
1. Download the [ESRGAN SD Cinematics Pack](https://www.moddb.com/mods/thief-gold-esrgan-pack/addons/thief-gold-esrgan-sd-cinematics-pack-v21)
2. Extract the contents of the 7-zip archive.
3. If you are running Thief Gold, then delete ```CREDITS.avi``` so that it does not conflict with the extended credits.
4. Rename all files so that the extension ```.avi``` is all upper case ```.AVI```. Windows may be sloppy with file paths, but Linux is not.
5. Replace the original cutscene files with the new ones.
