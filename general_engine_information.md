# General engine information

MAGES. engine appears to have been designed with the following goals:

* **Suited for visual novels:** Duh.
* **Massively crossplatform:** We have seen releases for PS2, PSP, Xbox 360, PS3, PS4, Vita, Xbox One, Win32 and iOS, and some games contain script files for Android. A wide variety of formats are supported in every asset category, and simultaneous multiplatform releases often ship with different formats per platform.
* **Rich text:** Script text supports plenty of markup sequences in addition to dialogue style properties. *@channel* threads in Steins;Gate are formatted text, not images or complex UI structures.
* **Simple extensibility:** Besides traditional character sprites, there have been releases featuring [3D graphics](https://vndb.org/v5883) or [Live2D sprites](https://vndb.org/v12630). There's no entity-component-system; larger features are fairly self-contained. Game-specific features are thus tacked-on, which likely involves code duplication, but makes for a simple structure.
* **Not data-driven:** Scripts mostly control state. Game-specific features are largely implemented natively; game assets are far from interchangable between different builds.
* **Deterministic resource usage:** Binaries are littered with global variables. Almost all objects, from opened asset archives to character sprites, are assigned one of a fixed number of (reusable) slots. (Though this doesn't exactly make games very memory-efficient on rich platforms like Windows...)

In this guide, we will first look at *Steins;Gate 0* as released on PC. A fairly recent version of the engine, but with no particularly fancy features.

## Variations

MAGES. engine has a few particularly notable variations:

* **KID engine:** The original. We don't know much about it except strings are encoded using a standard encoding (with in-band markup sequences) in Ever17.
* **[11eyes PC](https://vndb.org/v729):** Ships scripts in *SCS*, apparently the original source code format.
* **Mobile ports:** The *Android* and *iOS* versions of *Steins;Gate* and its fandiscs ship scripts in *SRC*[^1], apparently a preprocessed source code format. We have not investigated whether the runtimes originate from the same codebase as the console/PC versions (apart from the *Android* version of *S;G*, which is a complete Java rewrite). Note the *iOS* port of *Chaos;Child* (and likely any future titles) uses binary *SCX* scripts like the primary version.
* **[Chaos;Head Dual](https://vndb.org/r38137):** Vita release that combines *Chaos;Head Noah* and *Chaos;Head Love ChuChu!*; the latter is built in MAGES. engine, the former on rUGP, but there is asset sharing between the two.

[^1]: On Android, the scripts are "encrypted" using a trivial shift cipher.