# haxeui-blank
Blank backend from haxeui
This is a blank starting point for building a haxeui backend, there's not too much that's required to get
through doing this. First thing you're gonna need to figure out is if you're going to build a composite or 
native backend. There are pros and cons to both.

### Native
#### Pros

#### Cons

### Composite
#### Pros
- Easier to build the backend
- All components customisability is down to the power of your rendering "suruface"

#### Cons
- Often tied to a render loop which can be a performance drag (especially on battery powered devices)
- Won't use native components, so will have non standard interfaces for mac vs linux vs windows

## Haxeui API Map 
A simple, NON-exhaustive list of things to map for a haxeui backend
- ScreenImpl
	- addComponent() - Adds components to the display list
	- removeComponent() - Removes components from display list
	- handleSetComponentIndex() - Handles position in display list
	- supportsEvents() - Tells haxeui which input events are mapped 
	- mapEvent() - Map haxeui events to your backend events. These events operate in a global scope
	- unmapEvent() - Tell haxeui how to stop listening to active events
	- get_width() && get_height() - Map window dimensions, typically multilplied by Toolkit scale factors
	- get_actualWidth() && get_actualHeight() - A non modified version of the window dimensions
	- You should ideally setup some kind of event listener in here to listen for window changes to distribute to components
- AssetsImpl
	- imageFromFile() - Map loading an image from a filepath 
	- imageFromBytes() - Map loading an image from bytes 
	- getImageFromHaxeResource() - How to load an image from a haxe resource 
- ComponentSurface
Base type that will be used for all haxeui components. Nothing specific needs to be mapped here, if you go the typedef route or the surface route (see later on in the guide) how much you expose "api wise" is purely a convenience/qol factor here.
- ImageDisplayImpl
	- new() - Initialise your image object "type"
	- validateData() - Checks if there's image data and assigns it to the image surface, also set height and width
	- validatePosition() - Pass the position data to your image surface
	- dispose() - Map any disposal mechanisms for your image surface
- TextDisplayImpl
	- new() - Typically initialise your text object "type"
	- validateData() - Pass the text to your text object
	- validatePosition() - Manage positional information like x/y and text alignment
	- validateDisplay() - A place to handle expected width and height of the text "display space"
	- validateStyle() - Identify and apply text styles to your text object type and return true where a change has occured
	- measureText() - Map the text dimensions your framework measures to haxeui
	- measureTextWidth() - Allows haxeui to get an updated measurement of text 
	- dispose() - Map any disposal mechanisms for your image surface
- ComponentImpl
	- handlePosition() - Recieves position updates that should mapped to your surface
	- handleSize() 
		- Receive size updates, you will map it to your surface in here
		- Provides the entry point for style updates
	- applyStyle() - In here you will map things like borders, background colours and typical css attributes
	- handleClipRect() - Map clipping to your surface, this will enable things like scrollviews
	- handleAddComponent() - Adds components to the display list
	- handleAddComponentAt() - Adds component to the display list at a particular position
	- handleRemoveComponent() - Removes components from display list
	- handleRemoveComponentAt() - Removes components at a particular position in the display list
	- handleVisibility() - Maps whether your surface can be seen or not
	- mapEvent() - Map haxeui events to your backend events. These events operate in a local scope (to the visual)
	- unmapEvent() - Tell haxeui how to stop listening to active events
	- createTextDisplay() - Create your text visual and add it to the local component tree
	- createImageDisplay() - Create your image and add it to the local component tree
	- removeImageDisplay() - Remove your image from the component tree
## Building a composite backend
I'm going to keep this quite simple and just point out the flow that I used to help develop the backend and key things to get started.
Most backends are going to have different kinds of nuances so the guide can't be too specific. But before starting I highly recommend
you have a look through the existing backends:

- [Kha](https://github.com/haxeui/haxeui-kha/)
- [Heaps](https://github.com/haxeui/haxeui-heaps/)
- [Openfl](https://github.com/haxeui/haxeui-openfl/)
- [Flixel](https://github.com/haxeui/haxeui-flixel/)
- [Ceramic](https://github.com/haxeui/haxeui-ceramic/)
- [HTML5](https://github.com/haxeui/haxeui-html5/)

There are so many backends built that it is extremely likely that there will be one that has similar quirks to the framework you're going to use. However, don't focus on that one you find, keep them all around, they're extremely useful resources. 

### Initial setup
- Create a new folder, whereever you keep your projects
- Run `haxelib newrepo` in a terminal, this will create `.haxelib` directory. 
	*If you haven't used this before, it just installs haxelibs to this folder rather than globally. I do it this way as it keeps*
	*everything close by and easy to work on*
- Next ```
haxelib git haxeui-core https://github.com/haxeui/haxeui-core
haxelib git haxeui-blank https://github.com/haxeui/haxeui-blank
```
- Install your libraries (local haxelib won't pick them up globally)
- Get an init of your frameworks running and add `haxeui-core` and `haxeui-blank` to your hxml/build config
We're ready to start working on the backend


### Starting
Most frameworks and setups are going to have some kind of hierarchical setup, a sprite, visual, object, rendertarget. Whatever it may be, I'm going to refer to this "base" as `Visual` from here on out. Its recommended in general to have a configurable root object, so 
that's where we're going to start.

Haxeui requires a `module.xml` file in your project, this tells haxeui where your custom components, xml and styles are going to be and you can also do a lot of cool project specific theming through here. Here's a simple starting file:

```xml
<module>
	<resources>
		<resource path="assets" prefix="project" />
	</resources>

	<components>
		<class package="ui" />
	</components>
</module>
```

`assets` is a folder in my projects root and `ui` is a package in the `src` classpath. `prefix=projects` is a shorthand way to refer to the folder path in other areas of the xml file. Next up...Open up `ToolkitOptions` and add something like:
```hx
typedef ToolkitOptions = {
	var ?customRoot:Visual;
}
```
All your haxeui components will be rendered onto this root component. Next thing to do is go to your frameworks initiating, you want the stage where you can safely add your root visual to your frameworks render loop. 

```hx
static function main() {
	//Rest of framework has been setup and we're safe to initialise haxeui
	haxeuiTests();
}

static function haxeuiTests() {
	var root = new Visual();
	Simple.add(root);
	Toolkit.init({customRoot: root});
}
```

That's basically the end of the beginning, if you compile your app you should see the following warnings:
> haxe/ui/core/Screen.hx:316: WARNING: Screen event "mousemove" not supported
> haxe/ui/core/Screen.hx:316: WARNING: Screen event "keydown" not supported
> haxe/ui/core/Screen.hx:316: WARNING: Screen event "keyup" not supported
> haxe/ui/core/Screen.hx:316: WARNING: Screen event "keydown" not supported

If you see the above, great! It means haxeui is now running and the initial base is ready. If not, you need to have a look at your initiation order, maybe you're calling it after the update loop has started and your Toolkit.init never gets reached.

### The backend
This part is quite a lot of fun, your backend behaves like an intermediary between your framework and haxeui-core. What we're doing in the backend (haxeui-blank) is telling haxeui-core how to do things in your frameworks language. So, we don't actually need to reimplement a lot of things, we only map the behaviours to the frameworks equivilent and by the end of this, all of the components available to view over at the [haxeui explorer](https://haxeui.org/explorer/) will just magically become available to you by the end.

I don't want to make it sound too simple, as it can be a bit of work that goes into fine tuning, but really, you'd be surprised at how a little goes a long way here. Order of implementation doesn't completely matter, some parts do depend on others, I'm going to go over the starting point otherwise, implement in the order of your own priorities. The aim isn't to go into a ton of detail in how to make a full haxeui backend, I just want to give a little bit of an introduction to the process and provide some context and direction, the established backends are all really helpful to read through during the process of developing the backend

#### ScreenImpl
So we're just going to add a way to add something to the root visual so that we can actually test any changes that we make along the way. First thing to do is override `addComponent` mine is pretty simple, it looks like:
```hx
class ScreenImpl extends ScreenBase {
	public function new() {
	}

	override function addComponent(component:Component):Component {
		options.customRoot.add(component.visual);
		this.rootComponents.push(component);
		return super.addComponent(component);
	}
}
```
There's going to be more to do in this file, but this is just the starting point!

#### 1) ComponentSurface (ComponentSurface.hx)
This is your base components type. This can be a `Visual`, `FlxSprite` an `Object`. Whatever, you call it can be mapped in two ways
1) `typedef ComponentSurface = Visual;`
- What this does is it maps the underlying type of all `ComponentSurface`'s to your underlying type, this can work if your base type does not have any name collisions. If there are name collisions, you move to the next step:
2) ```hx
class ComponentSurface {
	public var visual:RoundedRect;

	public function new() {
		this.visual = new RoundedRect();
	}

	public function add(value:Visual) {
		visual.add(value);
	}
	//Map any additional functions to your underlying surface type in here
}
```
This is the second way, pretty simple right? map a way to resize your visual convienently like a `setSize` helper function, a `posX`, `posY` getter/setter for the underlying visual's x/y, it doesn't matter how you name your methods or how you choose to do it. Just as long as they don't conflict with haxeui's base types, its worth being clear with the namings because you'll be using them later on. Add some functions, recompile, rename if any name conflicts popup.

Minimal list of things you might want to map: `add/removeChild(visual)`, visibility toggle, x/y, width/height, clipRect (if it makes sense for your framework). As you can see the base type for my surface is going to be a `RoundedRect`. This visual type in my framework has a lot of the features we're gonna need in haxeui, things like clipping, borders and gradients. So rather than a simple `Visual`, I made a type that covers a lot of haxeui needs to make the backend writing process easier.

#### Component Implementation (ComponentImpl.hx)
Where the surface will be your haxeui components base type for its components, this file maps out behaviours. Notice this class extends a class called "ComponentBase", its worth checking out the source code and just having a little browse through the functions under any `Backend` header. It isn't necessary that you need to map all of these functions, just good to have an idea of what can be mapped. Now! Our aim is to render a simple square to the screen using a haxeui component. So you need to override:

- addComponent, addComponentAt
- removeComponent, removeComponentAt
- containsComponent
- handleSize
- handlePosition

The concept here is simple, using macro magic, this file automagically has access to your component surface type - test it out. For the backend I'm working on whilst writing this, my surface has to be mapped, so I'm using option 2 above. If I type `this.visual` I get access to my underlying type - pretty cool right? You don't even necessarily need to map out the surface, just expose your type. It just makes writing the backend cleaner.

Here's an extremely simple, non optimised, example of what we're going to be doing:
```hx
class ComponentImpl extends ComponentBase {
	override function handleSize(width:Null<Float>, height:Null<Float>, style:Style) {
		this.setSize(width, height);
		if (style != null) {
			this.applyStyle(style);
		}
	}

	override function applyStyle(style:Style) {
		if (style.backgroundColor != null) {
			var color = Color.fromRGBInt(style.backgroundColor);
		}
	}

	override function handlePosition(left:Null<Float>, top:Null<Float>, style:Style) {
		this.setPos(left, top);
	}

	override function handleAddComponent(child:Component):Component {
		this.add(child.visual);
		return child;
	}

	override function handleAddComponentAt(child:Component, index:Int):Component {
		child.visual.depth = index;
		return addComponent(child);
	}

	override function handleRemoveComponent(child:Component, dispose:Bool = true):Component {
		this.remove(child.visual);
		if (dispose) {
			child.visual.destroy();
			this.remove(child.visual);
		} else {
			this.remove(child.visual);
		}
		return child;
	}

	override function handleRemoveComponentAt(index:Int, dispose:Bool = true):Component {
		return this.handleRemoveComponent(this.childComponents[index], dispose);
	}
}
```

Theoretically, that's enough to draw a square. So lets test that out!

Go back to your project and create a file called `Box.hx` under the `ui` package in your `src` directory. Add:
```hx
package ui;

import haxe.ui.containers.VBox;

@:xml('<vbox width="100" height="100" style="background-color: #FF0000" />')
class Box extends VBox {
	public function new() {
		super();
	}
}
```

Now you want to go to where you previously added your `Toolkit.init()` stuff and add the following:
```hx
Screen.instance.addComponent(new ui.Box());
```

You should see a box on screen now, the dimensions you set in your vbox xml! (100x100). This is the starting point for all haxeui components done! That's the basics done, now things get pretty organic from here, just start building out things, want to be able to map opacity to your components? Map that up in `applyStyle`. Want to see some hover effects? Map up events. 

There's not a whole lot more to go over, simply figure out the feature you want to add and then find the place to put it. But most things are pretty self explanatory, and if not, the other backends make it very clear where to go. 

### Hints and Tips

#### Infinite loop
In some cases you may find that when you mark a component as `ready()` it causes your app to get stuck in a loop, the solution for this is to map your `CallLaterImpl` in your backend. It doesn't need to complicated.
```hx
class CallLaterImpl {
    public function new(fn:Void->Void) {
			App.app.once(Run, (event) -> {
				fn();
			});
    }
}
```
This is mine, and all it does is, it runs the callback at the end of the frame

#### Text Measurement (TextDisplayImpl.hx)
In `validateDisplay` you want to assign the `_width` and `_height` of the base class, and in `measureText` you pass your text dimensions to haxeui by setting `_textWidth` and `_textHeight`

#### Depth in HaxeUI
Haxeui will either insert an item at the beginning of the hierarchy or at the end of it and build it up that way. It won't insert inbetween. So, you need to be able to adjust your display list accordingly. Generally inserting at the bottom or at the `0` index means display at the bottom but one should keep this in mind whilst mapping.
