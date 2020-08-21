---
title: "Custom light-edit panel for Maya"
excerpt: "An improvement over the default light-edit panel built into maya, with the ability to specify what light-types to customize and what are the parameter per light type to show and edit."
tags: 
  programming
  maya
  python
date: 2020-08-05 18:26:00 0000
categories: programming
---
Maya (I believe from Maya 2018, but I could be wrong on this) embeds a tool that helps the artists to view all the lights in a single UI and has a direct way to manipulate the light information.

One of the major issues about this tool is that it can only properly display "lights" - or to better explain - maya nodes identified as lights.

My attempt at this tool is to have a configuration step where the "light" type can be configured so that, when executed, there is a proper inspection of the scene about the types specified, and maybe the "fields" (or the attributes) of those types.

~~~ json
{
    "lightTypeA": [ "param_1" ],
    "lightTypeB": [ "param_1", "param_2" ]
}
~~~

The UI will be very very simple, with a basic list of widgets containing basic light setup.

The basic structure of the script should be something like this:

~~~ python
config = {
    "lightTypeA": [ "exposure", "intensity" ],
    "lightTypeB": [ "exposure", "color" ],
    "lightTypeC": [ "color", "tint", "exposure", "intensity" ],
}


def fetch_lights_from_config():
    """
    Returns a structure containing the light name as a key,
    and the corresponding fields to inspect as values.
    """

    return {
        "area_light_for_the_window": [ "exposure", "intensity" ],
        "fancy_namespace:sun_light": [ "exposure", "color" ],
        #...
    }


def buildUI():
    """
    After fetching the lights in the scene, will process each one of them
    and fill the UI with all the lights found of the types specified in the
    config at the top of the file.
    """
    lights_in_scene = fetch_lights_from_config()

    for light_name, light_params in lights_in_scene.items():
        # add a "row" in the UI for the light_name


buildUI()
~~~

With the configuration at the very beginning of the script, so it'll be the very first thing visible by everyone opening the file in any random text editor (or, more likely, the Maya Script Editor).

Other notable things are mostly related to the user experience, like:

1. When a light is selected in the outliner or in the viewport, the same light has to appear as selected in the "light manager" tool
2. And, vice versa, when a light is selected in the "light manager" tool, the same selection has to replicate in the Maya "world"
3. Every light "field" in the light manager needs to be directly connected to the corresponding maya light-attribute, so that every change is replicated in real time with whatever happens in Maya
4. Again, the same needs to happen the other way around.

Other notable elements - more related to the UI consistency between Maya and any Python-based UI is the look related to the specific elements like the color picker, for the light color and light tint that could be in the field types required to display on any random light type.

*_Dev notes:_*
The whole implementation is going to be Python2 with UI built using PySide2 - all should be natively available in Maya.

The whole implementation is for Maya2020 - probably everything is going to work all the way back to Maya2018 or any other Maya version that supports PySide2 - but I haven't personally tested it.

In order to run the script, the best option available for everyone should be to copy/paste the whole script into the Maya Script Editor and run it from there (after changing the default config).


