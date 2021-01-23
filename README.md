# GuSprites for Boriel Basic.

This is a sprite library for Boriel Basic. Its goal is to have many animated sprites and
render them beating the scan line to avoid flickering.

It supports 1x1, 1x2 and 2x2 chars sprites and it can be configured to set how many of
each are available and can be rendered on screen to reduce as much as possible the
library footprint.

It also incorporates a tile system which can be enabled in order to create maps that are
preserved and autoredrawn. This can be also configure in replace mode or merge mode
(sprites replace tiles or sprites are merged with tiles)

In order to balance performance and memory consumption the library uses preshifted sprites but only once.
This allows the sprites to move in cells of 4 pixels resulting in an effective screen of 64x48 cells.

Sprites can be replaced resetting the sprite system and tile sets can be changed on the fly, so you can
load/unload sprites from the library as needed, you aren't restricted to a single set of sprites.

The library is completely free for use but a mention would be appreciated ;)

## UPDATE 2021-01-23

Added "SetTileChecked" function.

As the clear operations are created at the same time a sprite has been moved, if your game
must change that tile after the sprite leaves the tile then that cell will be rendered
incorrectly with the old tile.

To avoid this now you can use "SetTileChecked", it will check all find any clear op affecting
the tile and modify it setting the new tile as the source.

Added "CancelOps" function

If you have drawn a frame and want to preserve it as a background then you must get rid
of the clear operations or whenever you render a new frame these ops will clear the last
position of your old sprites.

This function will erase these operations.

Added "ClearScreen" function

This will clear the screen, tile map and pending operations all at once but will preserve
loaded sprites, not like ResetGFXLib,

## So, where the heck do I begin?

First, download the lib and give a look to the test program ;)
This repository contains a Visual Studio solution as I use SpectNetIDE to develop for ZX Spectrum.
The library itself is found in the "ZxBasicFiles" folder of the solution, the file called "GuSprites.zxbas".
If you are using a different IDE just copy the file to your project and add an include to it.

Second, configure the library as you need. At the begining of the library you will find some defines that you
can modify to set the required params. There are three sets of defines: defines for sprite quantity, defines
for on-screen sprites and defines to enable functions.

The first set configures how many sprites you will use. This allocates space on the shift buffer for the
specified sprites.

The second set configures the maximum number of sprites on screen at any moment. This allocates space on the
operation buffers.

The last set configures the tile system and allows you to enable/disable interrupts. The library needs to
disable the interrupts to avoid the ROM int to corrupt data so if not configured it will completely disable
the int. If you need to use it for keyboard scan or anything else you can enable the define so it will
enable/disable the int only on the required moments.

## Ok, it's configured, and now?

Of course, include the library in your program ;)

Now you must initialize the library. This is done with `InitGFXLib()`, this will initialize the buffers and
pointers of the library.

```basic
	InitGFXLib()
```
	

Next, if you want to use tiles set the tileset. A tileset is a set of 8 byte arrays that is used to render a
background on the scene. Each tileset must start with an empty char, this is used for clear purposes.

```basic
	Dim tileSet(3,8) as uByte => { _ 
		{0,0,0,0,0,0,0,0}, _
		{ 0,60,66,66,66,66,60,0 }, _
		{ 0,24,24,36,66,66,126,0 } _
	}

	SetTileset(@tileSet)
```

Then you can draw the background setting assigning tiles to the map using SetTile.
The first param indicates the sprite number, it is one-based excluding the empty one.
The second param indicates the attribute for the tile.
The third and fourth are the coordinates, zero based from the 32*24 screen chars.

```basic
	SetTile(1, 52, x, y)
	SetTile(2, 56, x, y)
```

Next you should create the sprites on the library. You can create as many you have
defined of each type (1x1, 1x2, 2x2).
Sprites are byte arrays representing chars sorted in columns, not in rows. This is very important or the sprites will not
render correctly.


	-------------
	|     |     |
	|  1  |  3  |
	|_____|_____|
	|     |     |
	|  2  |  4  |
	|     |     |
	-------------

To create a sprite call the corresponding function for the sprite type you're creating and it will return the
sprite number assigned to it.

```basic

	Dim sprite(8) as uByte => { 64,70,70,64,8,244,2,1 }
	Dim sprite2(16) as uByte => { 20,189,66,74,34,28,8,20, 50,34,70,101,58,18,19,24 }
	Dim sprite3(32) as uByte => { 73,73,73,73,73,73,73,73, 0,0,255,0,0,255,0,0, 73,73,73,73,73,73,73,73, 0,0,255,0,0,255,0,0 }

	Dim sNumber as uByte
	Dim sNumber2 as uByte
	Dim sNumber3 as uByte

	sNumber = Create1x1Sprite(@sprite)
	sNumber2 = Create1x2Sprite(@sprite2)
	sNumber3 = Create2x2Sprite(@sprite3)
```

Ok, now everything is ready, you can start your loop, draw as many sprites you like and render the final scene.

```basic

	do

	 Draw1x1Sprite(sNumber, 0, 0)
	 Draw1x2Sprite(sNumber2, 5, 5)
	 Draw2x2Sprite(sNumber3, 10, 10)

	 RenderFrame()

	loop

```

Each frame will be automatically cleaned in the next renderso you don't need to take care of anything.

## Hmmm, I want to change my sprites, what should I do?

Reset the library calling `ResetGFXLib()`, it will free all the buffers and clear all the maps.
Only the assigned tileset will be preserved.

After cleaning the lib a `clear` is recommended in order to erase the screen unless your tilemap covers
all the background.

Another option is to manually fill the tile map with the tile 0, that will clear the screen.



Well, I think for now that's all, if you need help feel free to open an issue ;)

Happy coding!