## About
This project is an attempt to take all commands from the [Arma 3 wiki](https://community.bistudio.com/wiki/Category:Scripting_Commands_Arma_3),
and save them in a consistent XML format for use in syntax checking.

Converting all commands will take a while since each command is manually converted to XML, with help of a Java-written
tool (the tool is not publicly available).

## Progress
* About 97% commands (2049 total conversions) have been converted. Phew. <!-- 2102 total-->

## How-to Use
### Intro
The contents of each command xml doesn't need to be strictly followed, however, it is meant to be as rigorous and strict
as possible to maximize type matching.
#### XML File Name
To retrieve a syntax for a command, the related file name will be `commandname.xml`. If the file does not exist, there is
no syntax available for the command. Please report any instances where a command is not supported.

The name of the file will always the command name in lower case with `.xml` concatenated. In Java code, it is `commandName.toLowerCase() + ".xml"`.

* File name for command [`WFSideText`](https://community.bistudio.com/wiki/WFSideText): `wfsidetext.xml`
* File name for command [`createVehicle`](https://community.bistudio.com/wiki/createVehicle): `createvehicle.xml`

#### XML Format
All xml files will have something like `<?xml version='1.0' encoding='UTF-8'?>`. If you are using an XML parsing library,
this tag should be handled already (it is recommended to use a library for XML parsing for proper encoding handling).

The root tag name will always be `command`. The attributes for the root tag are as follows:
* `name`: Command name in how it is written in the wiki (i.e. createVehicle, WFSideText). Essentially, it is the xml file
name without .xml and not in all lowercase.
* `version`: The version of the game that it was introduced in. This value can be anything and will most likely be a number
like 1.00 or 1.56.
* `format`: **As of right now, you can completely disregard this value.** The format of the .xml file (currently "1").
This value isn't ever expected to change, however, if a time comes that the format
of the xml for command syntaxes becomes insufficient, the value will change from "1" to "2" or the like. It's really
there to provide backwards compatibility if the format changes in the future.
* `game`: The game that the command was **introduced in**. The command may be supported in more than just this game!
The text value of this attribute will be non-user-friendly. The values are the "Image Link Prefix" values described
[here](https://community.bistudio.com/wiki/Template:Name). Here is a table that correlates the values:

|User-friendly name | `game` attribute value|
|---|---|
|Arma 2: Army of the Czech Republic|arma2acr|
|Arma 2: British Armed Forces|arma2baf|
|Arma 2: Private Military Company|arma2pmc|
|Arma 2|arma2|
|Arma 2: Operation Arrowhead|arma2oa|
|Arma 3|arma3|
|Arma 3: Zeus|zeus|
|Arma 3 Development Branch|arma3dev|
|Take On Helicopters|TKOH|
|Operation Flashpoint|ofp|
|Australians in Vietnam|aiv|
|Take On Mars|TKOM|
|Bohemia Interactive|bi|
|Operation Flashpoint: Elite|ofpe|
|Armed Assault|arma|
|Virtual Battlespace 3|vbs3|
|Operation Flashpoint: Resistance|ofpr|
|Virtual Battlespace 1|vbs1|
|Virtual Battlespace 2|vbs2|

### Command deprecation
The `deprecated` tag has no attributes and is a child of the `command` tag. If this tag exists (not guaranteed to exist), the command
is deprecated (no longer supported by Bohemia).

**Example**:
```
<command name='fakeCommand' version='1.56' game='arma3' format='1'>
<deprecated/>
...syntax tags, return tags, other things...
</command>
```

### Command Uncertainty
The `uncertain` tag has no attributes and is a child of the `command` tag. If this tag exists (not guaranteed to exist), the return type or 
at least 1 syntax is uncertain to be correct and has been estimated (this usually happens with very poor documentation).

**Example**:
```
<command name='fakeCommand' version='1.56' game='arma3' format='1'>
<uncertain/>
...syntax tags, return tags, other things...
</command>
```

### Syntax Tag
The `syntax` tag is a nested tag in the root tag `command`. This tag can occur multiple times. If there is more than one
`syntax` tag, it means there is more than 1 syntax for the command. The `syntax` tag has no attributes.

### Command Parameters (Param tag)
The parameters for the command will be nested inside a `syntax` tag. All commands can have a prefix argument or a postfix
argument (`prefixArgument COMMAND postfixArgument`). To access a syntax's parameters, collect all child elements of `syntax`,
excluding `return`.

All child elements described will either be an `array` tag or a `param` tag. Both tags will have an `order` attribute.
The `order` attribute determines whether or not the parameter is a prefix argument or a postfix argument. The `order`
attribute will either be "0", denoting prefix argument, or "1", denoting postfix argument.

The `param` tag will have the following attributes: `type`, `name`, `optional`, and `order`. The `type` attribute will be
one of the Types described below. The `name` attribute will describe the parameter (e.g. vehicle, text, object) and can be
thought of as a legitimate SQF code parameter variable name (because it is). The `optional` tag denotes whether or not
the parameter is optional; this attribute will have a "t" (meaning true) or "f" (meaning false) value.

The text content of the `param` tag is a description about the parameter.

The `optional` attribute is especially useful when the `param` tag is nested in an `array` tag. All optional parameters
don't need to exist. It should **NEVER EVER EVER** be the case that a required parameter follows an optional one! It
should always be something like as follows:
```
[required, required]

[required, optional]

[optional, optional]
```

AND NOT LIKE THIS:
```
[optional, required]

[optional, required, optional]

[required, optional, required]
```

For more information on the `array` tag, see Array Tag below.

**Example**- `param` tag:  
`<param type='OBJECT' name='objectName' optional='f' order='0'>HI! I'm the description!</param>`
Here, it is clear that the parameter is a prefix parameter since order=0 or is the first element of an array.

### Return Tag
The `return` tag describes the return type (see Types below) of the command and the tag itself has no attributes.

The `return` tag has 2 possible child tags: `array` and `value`. The `array` tag will have 2 attributes: `unbounded`
and `order`. The `value` tag will have 2 attributes: `type` (see Types below), and `order`.

The `value` tag can be alone in the `return` tag or will be nested in an `array` tag. The text content of the tag will be
a description about the value. This is very useful for when the `return` tag is an `array` and you need descriptions
about each index.

Here are descriptions of the `value` tag's attributes:
* `type`: The type of the value.
* `order`: The order in which it occurs in an array if nested in an `array` tag. If the `value` tag is not nested in an
`array` tag, this attribute can be ignored.

**Example**- `value` tag:  
`<value type='NOTHING' order='0'>Description...</value>`

For more information on the `array` tag, see Array Tag below.

### Array Tag
The `array` tag can occur in both the `return` tag and command's parameters. Both cases of the tag will have the same
attributes, except the command's parameters case will have an additional tag called `optional`
(see Command Parameters for more information).

The `unbounded` tag will either be "t" or "f", which means true or false respectively. Unbounded arrays (`unbounded`="t")
have an **indeterminate** length (it can be 0,1,2,3,...,50 etc). A bounded array (`unbounded`="f") will have a fixed and expected
length. This length should match the number of `value` tags nested in the `array` tag.

This is where things begin to get tricky, because an indeterminate array may or may not conform to a possible command's
argument count (a command may require 5 prefix arguments and the array may only provide 2). This is not something to
worry about. It is recommend to just assume the argument count is matched when dealing with unbounded arrays.

It is often the case that an `array` tag is unbounded and has a child element. This means that the command is returning an
array of that type that is unbounded. If the `array` tag has more than one child, the last child is the unbounded element.

**Example 1**- `array1` tag nested in `return`:
```
<array unbounded='t' order='0'>
    <value type='NUMBER' order='0'></value>
</array>
```
Here, the return type is an unbounded array of numbers (may be `[]`, may be `[1, 2]`, may be `[3.6, 9.8,1455]`, who knows).

**Example 2**- different unbounded:
```
<array unbounded='t' order='0'>
    <value type='NUMBER' order='0'></value>
    <value type='STRING' order='1'></value>
</array>
```
Here, the return type is an unbounded array of numbers and strings. The number is guaranteed to exist, however the string isn't.

Possible values: `[1]`,`[1, "hello"]`, `[3.6,"hello","hello2"]`, etc.

**Wrong** values: `[]`, `["hi"]`,`[0,0,"string"]` ,etc.

**Example 3**- array tag with unbounded array of bounded arrays:
For this example, an unbounded array of bounded arrays is expected
```
<array unbounded='t' order='0'>
    <array unbounded='f' order='0'>
        <value type='NUMBER' order='0'></value>
        <value type='STRING' order='1'></value>
    </array>
</array>
```

Possible values: `[]`, `[[0,""]]`, `[[0,""],[1,"hi"]]`

**Example 4**- array tag nested in `syntax`:  
```
<array unbounded='f' optional='f' order='1'>
    <param type='STRING' name='name1' optional='f' order='0'></param>
    <param type='STRING' name='name2' optional='f' order='1'></param>
</array>
```
Here the array is fixed. It it is clear that this is a postfix argument (as an array) for a command with precisely 2
strings required.

#### The Array Tag vs Array Type
The `array` tag means the parameters/values are determined. The `ARRAY` type described in Types is equivalent to:
 `<array unbounded='t'><value type='ANYTHING'></array>` or the `param` tag equivalent. So, the `ARRAY` type is an
 unbounded array of ANYTHING. Any time you expect an array and no `array` tag is present, but a `value` or `param` tag
 type is `ARRAY`, just assume it's correct.

 **Example**- both `ARRAY` type and `array` tag in the same command:  
 The [apply]() command xml is as follows:
 ```
 <?xml version='1.0' encoding='UTF-8'?>
 <command name='apply' version='1.56' game='arma3' format='1'>
 	<syntax>
 		<return>
 			<value type='ARRAY' order='0'>resulting array</value>
 		</return>
 		<param type='ARRAY' name='array' optional='f' order='0'>array of Anything</param>
 		<param type='CODE' name='code' optional='f' order='1'>code to be executed on each element of the array. Current
 			element value is stored in variable _x
 		</param>
 	</syntax>
 </command>
 ```
 You can see that it returns a `value` tag, but not an `array` tag. The command also takes in `ARRAY` prefix parameter.
 So, any time a type check happens, you could match either an `array` tag from a different command or an `ARRAY` type.

### Alt-types Tag
The `alt-types` tag is meant to provide multiple return or parameter types. There are `t` tags inside
the `alt-types` tag, which the `type` attribute of each `t` tag describes the alternate type.

The `alt-types` tag is not guaranteed to exist. If it doesn't exist, there is only 1 type for the return value or parameter.

**Example 1**- multiple return types:
The return type below returns a Number, String, or Code.
```
<return>
    <value type='NUMBER' order='0'>
        returns a number, code, or string
        <alt-types>
            <t type='STRING'/>
            <t type='CODE'/>
        </alt-types>
    </value>
</return>
```
**Example 2**- multiple parameter types:
The parameter below returns a Number, String, or Code.
```
<param type='NUMBER' name='p' optional='f' order='0'>
    returns a number, code, or string
    <alt-types>
        <t type='STRING'/>
        <t type='CODE'/>
    </alt-types>
</param>
```


### Types
This list comes from primarily [here](https://community.bistudio.com/wiki/Data_Types).

Any input into these types is much appreciated!

Here are all the types possible:

| Type name| Presentable name | Notes|
|---|---|---|
|ANYTHING|Anything|Literally accept any type. Can accept Boolean, Code, anything. This is basically shorthand for an `alt-types` tag filled with all available types.|
|ARRAY|Array|Used for when the elements in the array can't be determined.|
|BOOLEAN|Boolean|
|CODE|Code|
|CONFIG|Config|
|CONTROL|Control|
|DIARY_RECORD|Diary Record|
|DISPLAY|Display|
|EDEN_ENTITY|Eden Entity|
|EXCEPTION_TYPE|Exception Type|
|GROUP|Group|
|LOCATION|Location|[Wiki article](https://community.bistudio.com/wiki/Location)|
|NAMESPACE|Namespace|
|NET_OBJECT|NetObject|
|NIL|nil|Differs in nothing as this is an assignable value, where nothing is no return type/value.|
|NUMBER|Number|
|NOTHING|Nothing|No type/value.|
|OBJECT|Object|
|OBJECT_RTD|ObjectRTD|
|ORIENT|Orient|
|ORIENTATION|Orientation|
|SCRIPT_HANDLE|Script (Handle)|
|SIDE|Side|
|STRING|String|
|STRUCTURED_TEXT|Structured Text|
|TARGET|Target|
|TASK|Task|
|TEAM|Team|
|TEAM_MEMBER|Team Member|
|TRANS|Trans|
|TRANSFORMATION|Transformation|
|VOID|Void|
|IF|If Type|Created from `if` command. Used with `do` command.|
|FOR|For Type|Created from `for` command. Used with `do` command.|
|SWITCH|Switch Type|Created from `switch` command. Used with `do` command.|
|WHILE|While Type|Created from `while` command. Used with `do` command.|
|WITH|With Type|Created from `with` command. Used with `do` command.|

#### Types that are just shorthand for an array:
See [this wiki article](https://community.bistudio.com/wiki/Position) for more information on positions.

| Type name| Presentable name | Array Format |Notes|
|---|---|---|---|
|COLOR|Color|[r:*Number*, g:*Number*, b:*Number*, a:*Number*]|[Wiki article](https://community.bistudio.com/wiki/Color)|
|COLOR_RGB|Color RGB|[r:*Number*, g:*Number*, b:*Number*]|Notice no alpha value|
|ARRAY_OF_EDEN_ENTITIES|Array of Eden Entities|See wiki article.|[Wiki article](https://community.bistudio.com/wiki/Array_of_Eden_Entities)|
|POSITION|Position|[x:*Number*, y:*Number*, z:*Number*?]|Unclear what this is, but for simplicity, assume that z is optional.|
|POSITION_2D|Position 2D|[x:*Number*, y:*Number*]|
|POSITION_3D|Position 3D|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_ASL|Position ASL|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_ASLW|Position ASLW|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_ATL|Position ATL|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_AGL|Position AGL|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_AGLS|Position AGLS|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_WORLD|Position World|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_RELATIVE|Position Relative|[x:*Number*, y:*Number*, z:*Number*]|
|POSITION_CONFIG|Position Config|[x:*Number*, z:*Number*, y:*Number*]|Notice that z and y are swapped|
|VECTOR_3D|Vector 3D|[x:*Number*, y:*Number*, z:*Number*]|This type exists to clarify that it is a vector rather than a static position. From a type checking standpoint, it makes no difference.|
|WAYPOINT|Waypoint|**TODO TODO TODO**|An array format for waypoints|

## Example Syntax 1 - Basic Command
This is the xml for [abs](https://community.bistudio.com/wiki/abs).
```
<?xml version='1.0' encoding='UTF-8'?>
<command name='abs' version='1.00' game='ofp' format='1'>
	<syntax>
		<return>
			<value type='NUMBER' order='0'></value>
		</return>
		<param type='NUMBER' name='n' optional='f' order='1'></param>
	</syntax>
</command>
```

## Example Syntax 2 - Array return type
This is the xml for [actionKeys](https://community.bistudio.com/wiki/actionKeys).
```
<?xml version='1.0' encoding='UTF-8'?>
<command name='actionKeys' version='1.00' game='arma' format='1'>
	<syntax>
		<return>
			<array unbounded='t' order='0'>
				<value type='NUMBER' order='0'></value>
			</array>
		</return>
		<param type='STRING' name='userAction' optional='f' order='1'></param>
	</syntax>
</command>
```

## Example Syntax 3 - Array Parameter
This is the xml for [animate](https://community.bistudio.com/wiki/animate). This command has both a prefix argument and
a postfix argument.
```
<?xml version='1.0' encoding='UTF-8'?>
<command name='animate' version='1.75' game='ofpr' format='1'>
	<syntax>
		<return>
			<value type='NOTHING' order='0'></value>
		</return>
		<param type='OBJECT' name='objectName' optional='f' order='0'></param>
		<array unbounded='f' optional='f' order='1'>
			<param type='STRING' name='animationName' optional='f' order='0'>name of the animation. This is the
				class-name of the animation defined in the config.
			</param>
			<param type='NUMBER' name='phase' optional='f' order='1'>range 0 (start point of the animation) to 1 (end
				point of the animation). The speed, in which the animation is processed, is defined in the addon's
				config.cpp and cannot be changed during running missions.
			</param>
			<param type='BOOLEAN' name='speed' optional='f' order='2'>When true animation is instant. Since Arma 3
				v1.65.138459 Number &gt; 0 is treated as config speed value multiplier
				<alt-types>
					<t type='NUMBER'/>
				</alt-types>
			</param>
		</array>
	</syntax>
</command>
```

## Example Syntax 4 - Generic Array Return Type
This is the xml for [actionIDs](https://community.bistudio.com/wiki/actionIDs). This command has an unbounded return.
```
<?xml version='1.0' encoding='UTF-8'?>
<command name='actionIDs' version='1.63' game='arma3' format='1'>
	<syntax>
		<return>
			<array unbounded='t' order='0'>
				<value type='NUMBER' order='0'></value>
			</array>
		</return>
		<param type='OBJECT' name='entity' optional='f' order='1'>entity with added user actions</param>
	</syntax>
</command>
```
