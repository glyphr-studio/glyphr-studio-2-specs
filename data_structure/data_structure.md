# Glyphr Studio data structures
Version 1 data structure evolved over time, but had some fairly major overhauls in the Beta timeframe to make sure the structure made sense.  V2 would be another good opportunity to overhaul this structure, if need be.

## Project-level data

At a high level, the data structure must store the following things:
* Glyphs
* Ligatures
* Components
* Kerning information
* Typeface Metadata
* Glyphr Studio Project Settings

In v1 this was the overall structure for the global Glyphr Project (_GP) variable, which was equivalent to the content of the saved Glyphr Studio Project File (.txt)

## Top level collections

### glyphs
Object where the keys are string versions of the hexadecimal Unicode code point for that glyph (like '0x0041' for A) zero padded to 4 places.  Values are ```Glyph``` objects.


### ligatures
Object where the keys are concatenated string versions of the hexadecimal Unicode code points for the two or more glyphs in that ligature (like '0x00660x00660x0069' for ffi) each zero padded to 4 places.  Values are ```Glyph``` objects.


### components
Object where the keys are unique ID numbers with a 'com' prefix (like 'com1' 'com2'...).  Values are ```Glyph``` objects.


### kerning
Object where the keys are unique ID numbers with a 'kern' prefix (like 'kern1' 'kern2'...).  Values are ```HKern``` objects.


### metadata
Object with key/value pairs for various font metadata.  These are a combination of what is required for generating SVG Fonts and OTF Fonts.

This is true metadata - any information needed to actually design glyphs (like key metrics) are stored in project settings.

### projectsettings
Object with key/value pairs for various information pertaining to designing the typeface.  For example: key metrics, character range information, UI preferences, and project versioning information.


## The Glyph Object
Besides properties, there is a basic hierarchical organization to objects... the top of which is the Glyph object.  Here is a high level representation:

* ```Glyph``` objects - contain:
	* ```ComponentInstance``` objects - these are basically links to other Glyph objects that are treated like single shapes in this Glyph.
	* ```Shape``` objects - contain:
		* A single ```Path``` object - which contains a list of:
			* ```PathPoint``` objects, each of which has three:
				* ```Coord``` objects:
					* P - the path point
					* H1 - the handle before the path point
					* H2 - the handle after the path point

#### Components, Shapes, and Glyphs
Components allow for full Glyphs to be treated like a single shape in another Glyph.  Effectively this means ```Glyph```s must implement many of the same functions as ```Shape```s do.  ```ComponentInstance```s then form a lightweight wrapper to implement these same functions, allowing ```Glyph```s to treat all it's children like ```Shape```s.

Why are ```Shape```s and ```Path```s separated?  Really, they could be combined.   In practice in V1, ```Path``` functionality is basically low level path stuff that didn't have to be shared. ```Shape```s existed to make it easy to think about coordinating functionality between ```ComponentInstance```s and ```Glyph```s.

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
















