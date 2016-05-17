# Glyphr Studio data structures
## Spec Status: in progress...
Version 1 data structure evolved over time, but had some fairly major overhauls in the Beta timeframe to make sure the structure made sense.
Version 2 is a good opportunity to overhaul this structure.

## Project-level data

At a high level, the data structure must store the following things:

* Family metadata (was Typeface Metadata)
* Master metadata
* Instance metadata
* Glyph metadata
* Glyph-Layer data
* Features (was Ligatures)
* Positioning (was Kerning)
* ProjectSettings metadata
* ~~Components~~ were in v1 but removed, as it is better to treat them as just glyphs, that happen to have some properties (eg, unencoded, or flagged as not for export in binaries as standalone glyphs)

In v1 the overall structure for the global Glyphr Project (_GP) variable was equivalent to the content of the saved Glyphr Studio Project File (.txt)

## Top level collections

### Glyph

Object where the keys are strings of the name of that glyph (like `dollar` that is typically encoded as `$`)

Values are `GlyphMetadata` and `GlyphLayer` objects.

### Features

Object where the keys are GSUB feature codes (`ss01`)

Values are `String` objects.

https://www.adobe.com/devnet/opentype/afdko/topic_feature_file_syntax.html

### Positioning

Object where the keys are unique strings (like `kern1` `kern2`...)

Values are `HKern` or `VKern` objects.

### Family

Object with key/value pairs for various font metadata. 
These are a combination of what is required for generating SVG Fonts and OTF Fonts, like the family name, fsType, etc.

This is true metadata - any information needed to actually design glyphs (like key metrics) are stored in project settings.

### ProjectSettings

Object with key/value pairs for various information pertaining to designing the typeface. 
For example: UI preferences, project versioning information, etc.

## The GlyphLayer Object

The most important object in a font is a glyph drawing. 
Here is a high level representation of the `GlyphLayer`:

* ```GlyphLayer``` objects - contain:
	* ```ComponentInstance``` objects - these are basically links to other `GlyphLayer` objects that are treated like single shapes in this `GlyphLayer`.
	* ```Shape``` objects - contain:
		* A single ```Path``` object - which contains a list of:
			* ```PathPoint``` objects, each of which has three:
				* ```Coord``` objects:
					* `Point` - the on-curve path point
					* `In` - the off-curve handle going in to the Point (before it)
					* `Out` - the off-curve handle going out of the Point (after it)

#### ComponentsInstance, Shapes, and Glyphs

`ComponentsInstance`s allow for full Glyphs to be treated like a single shape in another `GlyphLayer`. 
Effectively this means `GlyphLayer`s must implement many of the same functions as `Shape`s do. 
`ComponentInstance`s then form a lightweight wrapper to implement these same functions, allowing `GlyphLayer`s to treat all children like ```Shape```s.

Why are ```Shape```s and ```Path```s separated?  Really, they could be combined.   In practice in V1, ```Path``` functionality is basically low level path stuff that didn't have to be shared. ```Shape```s existed to make it easy to think about coordinating functionality between ```ComponentInstance```s and ```GlyphLayer```s.

#### (OPEN QUESTION) Storing Path Points vs. Storing Bezier Curve Segments

**Path Points** in Glyphr Studio v1 are essentially 3 pieces of coordinate data (Path Point x/y, and two Control Point x/y).  This allows us to have things like a "Point Type" (flat / symmetric / corner).

Paths basically store an array of Path Points.

_**Pros of using Path Points**_ 
* "Point Type" is easy to implement
* Point actions (like add / remove) are easy to implement

**Bezier Curve Segments** is what is used by basically everyone else - a curve segment has an implied initial point x/y, then stores two control point x/y values and a finishing point x/y.

_**Pros of using Segments**_
* Bezier math is easier
* Other projects and technologies use it natively
	* SVG
	* Paper.js
	* OpenType.js

In the end we have to choose one, and do a little work to enable some functions that don't come 'for free'.
