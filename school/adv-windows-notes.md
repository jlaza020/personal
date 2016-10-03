# COP 4226 - Advanced Windows Programming

## Table of Contents

+ [Module 5 - Fonts, Colors, and Files] (#module-5---fonts-colors-and-files)
	+ [Serialization] (#serialization)
	+ [Serialization Interface] (#serialization-interface)
	+ [Colors] (#colors)
	+ [Fonts] (#fonts)

## Module 4

### Chapter 2 - MDI

Dragging calculation:

+ What is the calculation that is done in order to drag something on the screen?
	+ Need to know distance in X and Y.
+ Events
	+ mouseDown
	+ mouseUp

Psuedo code to do dragging calculation (in .zip file for Chapter 2):

	MouseDown(MouseEventArgs e){
		bMouseDown = true;  // Used because Points are structures and can't be null.
		ptMouseDown = e.Location;
	}

	MouseMove(MouseEventArgs e){
		if(bMouseDown){
			Point newLoc = new Point(
				this.Left + e.X - ptMouseDown.X,
				this.Top + e.Y - ptMouseDown.Y);
			this.Location = newLoc;
		}
	}

	MouseUp(MouseEventArgs e){
		bMouseUp = false;
		/* Should probably do MouseMove() here again */
	}

Create icons with ToolStrips.

StatusStrips have text and other controls.

To create MDI applications:

+ Create parent form with property `IsMdiContainer` set to `true`
+ Create child form with property `MdiParent` set to `this`  // Parent should set this.
+ LayoutMDI: arrange, tile vertical & horizontal, cascade

MDI has menu merging controlled by 2 properties:

+ **MergeAction**: append, insert, match, remove
+ **MergeIndex**: all index of one number precede all index of next number

Visual inheritance is for visual interface in designer. Menus, toolstrips,
status bars CANNOT be edited by the extending class in the designer. Have to
do it in code.

Code for creating new child in MDI:

	void cmdFileNewChild_Click(object sender, EventArgs e){
		Form child = new ChildForm();
		child.Text = "MDI Child " + nextChild;
		child.Closed += new EventHandler(ChildClosed);
		++nextChild;
		child.MdiParent = this;
		child.Show();
	}

Parent form changes LayoutMdi/MdiLayout. MergeOrder dictates in which order the
menu items appear. For example, 1 will be inserted in between menu items with
MergeOrder 0 and 2. Can arrange child forms in LayoutMdi (ArrangeIcons, 
TileVertical, TileHorizontal, Cascade, etc.) Menu divider has a MergeOrder as well.

### Chapter 3 - Help

Use a MaskedTextBox to force user to enter data in a specific format. Use the
Mask property. "0" means any digit. Use literals for phone numbers, dates, ssn,
etc. Example: (000) - 000 - 0000

**Thorough Validation**: validate everything with Validating event.

**ToolTip** is a component, *NOT* control, to the form. Not always visible. Has
a property - **extender property**. Any new property added by one object to another object on a form is called an extender property, because the former object extends the latter with additional functionality via a property. ToolTips mostly have static text. ErrorProviders mostly have dynamic text. ToolTips == ErrorProviders.

Populate the error provider with the contents of the tool tip:

	foreach(Control control in this.Controls){
		String toolTip = toolTip.GetToolTip(control);
		this.infoProvider.SetError(control, tooltip);
	}

HelpProvider is like tooltip - provides extender property. Automatically tied
to the help button and the F1 key.

## Module 5 - Fonts, Colors, and Files

### Serialization

**serialization**: Used to store a class to memory. (Really, its object hierarchy is stored.)
**deserialization**: Pretty much serialization but backwards.

There are several types of serialization. Binary is a popular format, but it is easily changeable to SOAP,
which is really just XML. SOAP is a great way to be able to look and re-configure a serialized object.

By default, .NET comes with the necessary assembly to serialize to binary. To serialize to SOAP, an
external .dll must be referenced.

The simplest way to serialize a class is to mark it as Serialize, using an annotation. This will work
if all members in class are serializable. Primitives are serializable. It is possible to mark an object
as NonSerialized.

C# Serializable Example:

	[Serializable]  // annotation
	public class SomeClass
	{
		[NonSerialized] SomeNonSerializedType
	}

FileAccess is an enumeration for setting the access rights when opening a file:

	enum FileAccess
	{
		Read			// you have the right to deserialize
		Write			// you have the right to serialize
		ReadWrite		// you have the right to deserialize and serialize (this is complex, don't really
						// (need this)
	}


FileMode is an enumeration for setting constraints on creating the actual file:

	enum FileMode
	{
		CreateNew
		Create			// serialize
		Open			// deserialize
		OpenOrCreate
		Truncate
		Append
	}

IFormatter does serialization:


	/* 
		Using a stream with access mode and privilege set to Create and Write, we create a new 
		BinaryFormatter object to serialize some instance of some class to the FileName specified.
	*/

	using (Stream stream = new FileStream(dlg.FileName, FileMode.Create, FileAccess.Write))
	{
		IFormatter formatter = new BinaryFormatter();
		formatter.Serialize(stream, instanceSomeClass);
	}

Use a similar technique to deserialize:

	/* 
		Using a stream with access mode and privilege set to Open and Read, we create a new 
		BinaryFormatter object to open and deserialize an existing file (using the FileName specified) to 
		an object and cast it to the desired object type.
	*/

	using (Stream stream = new FileStream(dlg.FileName, FileMode.Open, FileAccess.Read))
	{
		IFormatter formatter = new BinaryFormatter();
		SomeClass instanceSomeClass = (SomeClass) formatter.Deserialize(stream);
	}

It is possible to serialize multiple objects to the same file and deserialize (in same order):

	formatter.Serialize(stream, obj1);
	formatter.Serialize(stream, obj2);
	...

Must deserialize a class outside of a class (or its members inside the class).

Bad things happen if the class is not annotated as [Serializable] and you try to serialize. **Fields that
are marked as NonSerialzied must be initialized after deserialization.** Implement IDeserializationCallback
to define a method that is called after deserialization. It is the place to initialize non-serialized
members.

For example, fullName and credits are calculated fields. It doesn't make sense to serialize them because
their values depend on other fields.

### Serialization Interface

This is the HARD way. The easy is everything in the previous section.

The Serializable attribute is used when most of a class should be serialized. **ISerializable** takes the
opposite approach. Each piece of data must be explicitly serialized. The default is that the data is
not serialized.

ISerializable defines one method for serialization:

	/*
		SerializationInfo info is where we get the information that is being read from the file.
	*/

	public void GetObjectData(SerializationInfo info, StreamingContext context)
	{
		// Serialize or Deserialize here? I have no idea :(
	}

The interface does not define deserialization, but it can be handled by a class constructor that has the
same parameters as GetObjectData:

	public SomeClass(SerializationInfo info, Streaming Context context)
	{
		// Deserialize data in this method.
	}

Serialize by adding data to the info parameter:

	info.AddValue("myUniqueName", someData);

Deserialize by extracting data from the info parameter:

	SomeType obj = (SomeType) info.GetValue("myUniqueName", typeof(SomeType));

Summary using ISerializable:

	[Serializable]
	public class SomeClass : ISerializeable
	{
		SomeType t;
		SomeNoneSerializedType non;

		/* Deserialize */

		public SomeClass(Serialization info, StreamingContext context)
		{
			t = (SomeType)info.GetValue("myType", typeof(SomeType));
			non = SomeCalculation();
		}

		/* Serialize */

		public void GetObjectData(SerializationInfo info, StreamingContext context)
		{
			info.AddValue("myType", t);
		}

	}

Still use the same methods to serialize to a file and deserialize from a file as were used before.

### Colors

**Color** is a structure with methods for creating colors. It has many predefined colors like Color.Red,
Color.Blue, etc. Colors are defined by A, R, G, B (A stands for alpha which is opacity / transparency).

KnownColor is an enumeration of predefined colors with names. SystemColors is a static class of logical
colors like ActiveBorder, Highlight, etc.

Different ways to create colors:

+ FromArgb(R, G, B)							// decimal values (0-255), hex values, etc.
+ FromKnownColor(KnownColor.ActiveBorder);  // enumeration
+ FromName("ActiveBorder");

ColorTranslator is a static class that can translate from one format to another: FromHtml, ToHtml

ColorDialog is the standard dialog for defining a color. Access the Color property for data exchange:

	using (ColorDialog dlg = new ColorDialog())
	{
		dlg.Color = Color.Red;

		if (DialogResult.OK == dlg.ShowDialog())
		{
			this.Fore = dlg.Color;
		}
	}

With FullOpen property set, a more advanced view of ColorDialog is opened up.

### Fonts

Fonts are usually measured in points (1/72 of an inch).

DPI: dots per inch.

float pixels = (points * dpi) / 72f;

Design units are used to scale a font to a point size.

Graphics context is just a screen.

Fonts are IDisposable:

	using(Font font = new Font("Arial", 12))
	{
		...
	}

If the font is not available then the default font MS Sans Serif is returned.

The default unit is point.

Style can be added when the font is created. Use the FontStyle enumeration: Regular, Bold, Italic
Underline, Strikeout. It is a bit field, so the values can be ORed together. A bit field is an enumeration
whose values are powers of two (Regular=0, Bold=1, Italic, Underline=4, Strikeout=8):

	new Font ("Arial", 12, FontStyle.Bold | FontStyle.Italic);

Not all fonts support all styles. There is a way to check if a style is supported by a given font:

	if(fontFamily.isStyleAvailable(FontStyle.Bold))
	{
		...
	}

Font Families contain all the fonts with the same typeface, but with different styles and sizes.

Each system has a specific typeface for Generic Fonts:

+ GenericMonospace  // equally-spaced
+ GenericSansSerif  // variable width
+ GenericSerif  // winged

SystemFonts correspond to logical parts of the window: CaptionFont, DefaultFont, DialogFont, etc.

SystemFonts must be disposed, unlike SystemPens, SystemBrushes, and SystemColors.

Test the style of a font with `font.Bold, font.Italic, etc.`. These are read-only Boolean values though -
they cannot be changed with these properties. Remember that fonts cannot be changed after created.

FontDialog is the standard dialog for choosing fonts. Access the font for data exchange:

	using (FontDialog dlg = new FontDialog())
	{
		dlg.Color = new Font("Arial", 12);
		if(DialogResult.OK == dlg.ShowDialog())
		{
			this.Font = dlg.Font;
		}
	}
