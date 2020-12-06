---
title: "Skyrim Outfit System Revived"
draft: false
type: page
---
**Skyrim Outfit System Revived** is my attempt to resurrect an excellent Skyrim SE mod created by DavidJCobb and aers. The release package is available on the [Skyrim SE Nexus](https://www.nexusmods.com/skyrimspecialedition/mods/42162). The complete source code is on [GitLab](https://gitlab.com/metricexpansion/SkyrimOutfitSystemSE).

The basic concept is to allow the player to decouple the appearance of their armor from its stats. Here's a little video that shows how this mod works.

<video style="width: 100%" controls src="https://cdn.metricexpansion.com/file/avipublicfiles/The+Elder+Scrolls+V++Skyrim+Special+Edition+2020.10.29+-+21.44.43.01.mp4"></video>

DavidJCobb made the original version for Skyrim LE. aers then ported that mod to Skyrim SE 1.5.73. However, the port use hard-coded addresses while made it non-portable between different versions of Skyrim SE. Worse, it had a dependency on a custom fork of a library called CommonLibSSE (CLS). CLS is a support library that reconstructs the class definitions used by the game, among other things. It was also coded using fixed addresses. The mod also depended on SKSE as well, which also uses fixed addresses.

Some time later, a new version of CLS became available that leveraged the Address Library for SKSE. However, it also completely changed the interface and so upgrading to it became a non-trivial task.

For these reasons and other, it seems that aers decided to put the development on hold until he found time to work on upgrading the mod. That was in 2019.

Since it had been a while since development was suspended, I decided to dive into this and see what it would take to not only update the mod for the latest Skyrim runtime (currently 1.5.97), but to actually make it compatible with *any* version of the game!

After discovering the aforementioned situation, I got to work trying to refurbish the mod. I was able to first update the mod to the newest version of CLS. I combed through the mod looking for everywhere the new version of the CLS broke the interface and fixed it up.

Then I replayed aers's change to CLS. They amounted to a few additions to the reconstructed classes. Of course, I tried to use Address Library identifiers rather than hard-coded offsets in the new member definitions.

With CLS now fixed up, my next step was to fix the mod itself. I went through all the addresses defined by the mod and changed them to use Address Library offsets as well. Some addresses were actually in the middle of some functions (not the start). I used IDA Pro to find the starts of these functions and was able to then map them with the Address Library. I then add the difference manually. I did verify that this difference was the same across different versions of the game binary.

The final, and most difficult, task was to fix up the SKSE dependency. SKSE does not publish versions that use the Address Library. Luckily, most of the interesting address accesses happen through a few helper classes. By replacing the definition of these helper classes with something that can actually use the Address Library to translate 1.5.73 addresses into the addresses of whatever game version is running, I could make SKSE version independent. To support this, I also wrote a compile-time program that will dump the Address Library database for 1.5.73 into a reverse lookup table, which gets compiled into this helper class. At runtime, I simply load the database for the running version and address to identifier to address lookup.

I don't really need all the addresses to work, only the ones directly required by this mod, so I have it crash if some lookup was to fail. This is a dangerous technique, because I may not have caught all game address dereferences by SKSE, but I'm pretty confident that I have for the subset of functionality used by this mod.

Finally, I converted the entire build system to CMake, a build system I know well, so that I can coordinate all of this properly. For someone building the project from a fresh clone, all this complexity is hidden away.

Now that the mod is in good shape, I have started working on new features. They include a better quickselection menu, an option to automatically change the outfit based on location and weather, JSON export/import of outfits, and better integration with the vanilla equipping system.