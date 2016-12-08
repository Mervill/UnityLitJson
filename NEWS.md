LitJSON 1.1.1 (UnityLitJson)
=============

Merged [HyperHippo's](https://github.com/HyperHippo/litjson) v2 branch onto the main branch.
Added some Unity3D specific type bindings & platform compatibility patches.

*Note about building*

I've left the LitJson csproj intact so you can run the tests. When your 
ready to use the library in Unity3D I recommend you import the uncompleted 
source, there are some #if directives in UnityPlatform.cs that help make the
library work on WinRT (and possibly other platforms in the future).

LitJSON 1.1.0
=============

Implemented type-hinting.
To enable type-hinting set the JsonReader/JsonWriter's `TypeHinting` property to `true`.
Optionally, set the `HintTypeName` and `HintValueName` properties.
These properties are the JSON key names for the type and the data objects.
The defaults are "__type__" and "__value__" respectivly.

If enabled, the type-hinting system will only include type information for polymorphic fields/properties or collections which contains polymorphic elements.

```C#
// As an example, if we had the following schema:

// Contained in MyAssembly
namespace MyNamespace {

class Base {
	public int x = 1;
}

class Derived : Base {
	public int y = 2;
}

class Root {
	public Base field;
}

}

// And we had a root object which contained the following:
var root = new Root();
root.field = new Derived();

// Then serializing the object will include type hinting, since the field type does not match the given type exactly.
JsonWriter writer = new JsonWriter();
writer.TypeHinting = true;
string json = JsonMapper.ToJson(root, writer);

// To deserialize the resulting JSON, use a JsonReader with TypeHinting set to true.
JsonReader reader = new JsonReader(json);
reader.TypeHinting = true
Root deserializedRoot = JsonMapper.ToObject<Root>(reader);
```

The resultant JSON will be as follows:

```JSON
{
	"field": {
		"__type__": "MyNamespace.Derived, MyAssembly, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null",
		"__value__": {
			"x": 1,
			"y": 2
		}
	}
}
```

To property locate the correct type, the [Assembly Qualified Name](http://msdn.microsoft.com/en-us/library/system.type.assemblyqualifiedname.aspx) is serialized along with the object.
It is the most amount of information that can be given about a particular type.
Although verbose, it is needed to property determine the exact namespace and assembly the type is located in to find it.
In the future, it may be possibly to only specify a namespace and type name as the hint.
This would mean the library would have to search through each assembly for the correct type, which could significantly degrade performance.

Type-hinting also works for generic collections like `List<T>` and `IDictionary<K, V>`, and will only include hints for elements that do not match the generic argument exactly.
Type-hinting will also work for non-generic collections, in which case all elements are given serialized with a hint.

LitJSON 1.0.0
=============

JsonIgnore attribute added.
With this attribute, you can specify if an attribute read/written when serializaing or deserializing.
If you specify `[JsonIgnore(JsonIgnoreWhen.Serializing)]` the attribute will be deserialized to, but not serialized.
On the contrary, `[JsonIgnore(JsonIgnoreWhen.Deserializing)]` will serialize the attribute but will ignore it when deserializing.
The default, `[JsonIgnore]` will ignore the attribute completely.

LitJSON 0.9.0
=============

*To Be Announced*

Additions:

* Added `JsonData.Keys` property, with type `ICollection<string>`.


LitJSON 0.7.0
=============

2013-04-26

General changes and improvements:

* Simplified the building mechanism. Dropped the entire autotools
	infrastructure, and instead added a single Makefile to compile the project
	and run the tests with *GNU make*.

* Added `SkipNonMembers` property to `JsonReader`. When active, it allows to
	parse JSON data using `JsonMapper.ToObject<T>` and ignore any properties
	not available in the type `T`. Its default value is `true`.

* Started moving the documentation into Markdown format.

* Added a new section to the QuickStart Guide, regarding customisation of
	the library's behaviour.

Bug fixes:

* Convert `null` properties properly in `JsonData.ToJson`.

* Read nested arrays in `JsonMapper.ToObject` and `JsonMapper.ToObject<T>`
	correctly.

Contributors for this release:

	- whoo24
	- Christopher Dummy


LitJSON 0.5.0
=============

2007-10-04

New features and improvements:

* The JsonRader class now has two properties to control the reading of data
	from an input stream: EndOfInput and EndOfJson. The latter becomes
	true whenever a complete piece of JSON text has been read, while the
	former is a flag that becomes true when the stream itself reaches the end.
	This way, reading multiple JSON texts from the same input stream is
	straightforward.

* Added new base importers in JsonMapper for reading numeric values
	correctly into float and double members.

* Now Enum's can be imported/exported as numeric values.

* JsonData implements the IEquatable<T> interface now.


API changes:

	The following types are new:
		enum JsonType

	The following methods are new:
		IJsonWrapper.GetJsonType()
		IJsonWrapper.SetJsonType()

	The following properties are new:
		JsonReader.EndOfInput


Bug fixes:

* Correctly import/export properties that are read-only or write-only.

* Correctly convert null values when adding them as array elements or
	properties to a JsonData instance.

* Fixed conversion of empty JSON objects and arrays.


Thanks to all the contributors that reported problems and suggested fixes
for this release:
	Colin Alworth
	Ralf Callenberg
	andi


LitJSON 0.3.0
=============

2007-08-15

New features and improvements:

* Exporters and importers.
	Custom conversions using the JsonMapper class can be made through
	importers and exporters. These are delegates that tell the library how to
	perform conversions between non-basic types (i.e. not string, int, long,
	double or boolean) and JSON. There are base and custom
	exporters/importers. The base exporters and importers are built-in
	delegates that currently handle simple conversions between JSON and the
	following value types:

		byte
		char
		DateTime
		decimal
		sbyte
		short
		ushort
		uint
		ulong
	
	Custom exporters and importers can be defined through
	JsonMapper.RegisterExporter and JsonMapper.RegisterImporter, and they
	override the built-in conversions.

* Improved performance of JsonMapper.ToJson()
	A static JsonWriter is re-used to reduce the activity in the heap,
	improving performance for multiple conversions.

* Allowing extended grammar
	The lexer can now accept single-quoted strings, and comments in the
	following forms:

		// Single-line comment

		/*
		 * Multi-line comment
		 */

	This way, certain forms of input coming from JavaScript that don't
	necessarily conform to the strict JSON grammar are allowed and succesfully
	parsed.

	A JsonReader can be configured to accept only the strict grammar or the
	extended one. These extensions are allowed by default.

* API cleanups and additions.
	The following members are new:

		JsonData.Count
		JsonMapper.RegisterExporter()
		JsonMapper.RegisterImporter()
		JsonMapper.UnregisterExporters()
		JsonMapper.UnregisterImporters()
		JsonReader.AllowComments
		JsonReader.AllowSingleQuotedStrings
		JsonWriter.Reset()

	The following overloads have been added:

		JsonMapper.ToJson(object obj, JsonWriter writer)
		JsonMapper.ToObject(JsonReader reader)

	The following members have been renamed:

		JsonReader.HasReachedEnd is now JsonReader.EndOfJson


Bugs fixed:

* JsonMapper.ToJson() avoids entering an infinite recursion by defining a
	max nesting depth.
* The JsonData int indexer now behaves correctly both when it acts as an
	array and as an object.


LitJSON 0.1.0
=============

2007-08-09

First release.
