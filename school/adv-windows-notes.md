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

## Module 7 - Pens, Brushes, Shapes, and Binding

### Introduction

We write to a **graphics** class. 

Graphics encapsulates a GDI+ drawing surface.

+ Clip Bounds: What is changing?
+ Dots per inch: Resolution
+ Page Scale: Scaling of pages
+ Transformations: Rotations, Offsets
+ VisibleClipBounds: Specifies what you can print or display.

GDI has been enhanced by later versions of .NET. 

**Manual drawing is bad** because we miss Windows messages.

**ALWAYS use the paint handler**. Repaints the screen when we get paint events. Use the graphics passed 
in the GraphicEventArgs. 

Only use CreateGraphics when you need to get measurements form the graphics context, like the size of the
current font.

### Color Brush

**Do not call the paint handler directly**. Use **Invalidate** to make an indirect call to the paint 
handler. Invalidate is also more efficient. Painting is slow. If several invalidate messages are in the
message queue, they can be combined into one call to the paint handler. Invalidate(true) used to
invalidate children.

Depending on display settings and app settings, a resized window may not display properly. By default,
only the new area of a control is redrawn when the control is resized. The remainder of the control is not
resized. To change this for a form, set the ResizeRedraw style to true: 
`this.SetStyle(ControlStyles.ResizeRedraw, true);`

Brushes are used to fill areas with color. Brush is an interface. There are five implementations:
**Solid, Texture, Hatch, LinearGradient, and PathGradient Brushes**:

+ SolidBrush:
	+ Pass a color to the constructor.
+ TextureBrush:
	+ Pass an image to the constructor.
	+ Has a WrapMode: Tile, TileFlipX, TileFlipY, TileFlipXY, Clamp
+ HatchBrush:
	+ 3 params: Style, foreground, and background colors
	+ Styles: HatchStyle.Cross, HatchStyle.Wave, etc.
+ LinearGradientBrush:
	+ 4 params: Area, start color, end color, angle
	+ Angle: Number or value from LinearGradientMode enumeration: Horizontal, Verical, ForwardDiagonal,
	BackwardDiagonal.

Predefined blends can be used: `brush.SetBlendTrianglarShape(0.5);` or `brush.SetSigmaBellShape(0.5f);`

A custom blend can be created with ColorBlend, Colors, and Positions. ColorBlend contains the colors and
the position where that color is 100%. The position is a percentage of the total length of the brush:

	ColorBlend blend = new ColorBlend();
	blend.Colors = new Color[] {Color.White, Color.Red, Color.Black};
	blendPositions = new float[] {0.0f, 0.5f, 1.0f};					// Parallel array.
	brush.InterpolationColors = blend;

### Brushes

PathGradientBrush is constructed from an array of points, so it is the most flexible kind of brush.

Colors blend from the outside of the center. The center point of the brush can be changed. SurroundColors
can set the colors that will blend from the center to the edge. The colors will be distributed evenly.
Custom colors can be set just like the LinearGradientBrush.

Can use `Enum.GetNames(typeof(KnownColor))` to get names from an enumeration.

To create a brush: `Brush blackBrush = System.Drawing.Brushes.Black;`

Pens are used to draw the borders of objects rather than fill them in like brushes.

### Pens

Pens implement IDisposable so use using statements.

Can be constructed from color or brush.

Pens have:

+ Line caps: Define what end of a line looks like. Anchor or points in the middle of bar graph,
for example.
+ Dashes
+ Alignment: Where to draw.
+ Joins: Sharp angle at the edge where two things join together.

Constructing pens:

+ `new Pen (Color. Red)`
+ `new Pen (Color.Red, 2.0)` // Default is pixels.
+ `new Pen (brush)`
+ `new Pen (brush, 2.0)`

Lines can be dashed. There is an enumeration for standard dashing patters: DashDot, Dot, Dash, DashDotDot,
Solid, Custom.

Can create custom dash patters using the Custom type:

	pen.DashStyle = DashStyle.Custom;

	pen.DashPattern = new float[] {1f, 1f, 2f, 1f, 3f, 1f, 4f, 1f}; // Width alternates (on, off, on, ...)

It is possible to make a compound pen. We specify where colors start and end based on the percentage of
pen's width: `pen.CompoundArray = new float[] {0.0f, 0.25f, 0.45f, 0.55f, 0.75f, 1.0f}`

This is partitioning the width of the pen. Array must begin with 0.0f and end with 1.0f.

### Align Graphics Path 

The default alignment is centered. Half the pen is inside the control. half is outside. The only other
implemented alignment is inset. How can an outset pen be drawn? To be continued.

Joins occur at the corners. They control how pointed the corner can be:

+ Bevel: cut the corner
+ Round: round the corner
+ Miter: the corner can be as pointed as need
+ MiterClipped: the corner can be pointed up to a certain length, then it is cut off.

5.0f is 5 widths of the pen for MiterClipped.

Curves are drawn from an array of points. A tension controls what the curve does. Tension of 0 for 3
points looks like a triangle. More tension -> more curvature (it's counter-intuitive).

What do the four points control in a bezier curve? Like a backwards 'S'. The positions of the flexion 
points pull and push the curves.

Define anti-aliasing. It is using additional colors to blend from one color to another. The Graphics 
class has the property for setting anti-aliasing? The name of the property is Smoothing Mode.

### Data Binding

Changes in the data class change the values in the UI controls. Changes to the UI controls change the
values in the data class.

.NET has automated this process as much as possible to decouple data class from UI.

The data class fires a message every time that a piece of data is changed. The UI uses **reflection** to
call setters in the data class.

This allows the data class to be used with any UI. The UI does add statements to tie itself to the data
class, but the statements can be changed to be used with any data class.

The data class should implement the INotifyPropertyChanged interface and fire the event every time a
property changes:

	[Serializable]
	public class Data : INotifyPropertyChanged
	{

	/* Only member of interface. */
	
	public event PropertyChangedEventHandler PropertyChanged;

	/* Add helper method. */

	protected void OnChange(string propName)
	{
		if(PropertyChanged != null)
		{
			PropertyChanged(this, new PropertyChangedEventArgs(propName));
		}
	}

	}

In the UI class, add bindings for each property from the data class. Tie the data class property to a
property in the UI: `this.textBoxCredits.DataBindings.Add("Text", data, "Credits");` // Automatically 
updated in the data class.

Things can be simplified even more with drag-and-drop:

+ You can define the data class as a data source.
+ Changed the data class view to Details.
+ Drag the data source onto an empty form.
+ Delete the navigator (it is not needed now).

Input boxes and labels are added for each data class property automatically. 

To fill form with data: `this.dataBindingSource.DataSource = data;`.

When serializing a data object, the binding is erased. Must add new deserialized object as a binding
source to UI.

## Module 8 - Threads, Text, and Transforms

### Thread Introduction

Each Windows Forms app has a main thread that shows the UI. The UI thread should not do a lot of
processing.

Threads are **non-deterministic**: the order of execution is unknown.

Thread limitations for COP 4226 because of shared data:

+ A thread can only access local variables, but not parameters.
+ A thread should not make changes to member variables, parameters, static variables, or properties.
+ A thread will not read member variables (even though it technically could with no problem).

### Using Delegates to Start Threads

Behind the scenes, a delegate is used when creating a thread.

Example code for creating a thread:

	Thread thread = new Thread(ThreadStart);
	thread.Start(value); // Argument passed to ThreadStart.

	virtual protected void ThreadStart(object obj)
	{ 
		Work(obj); // Helper method.
	}

Windows prevents threads from updating a UI variable it did not create.

Events are special types of delegates.

Code for defining a delegate:

	protected delegate void MyDel(int i, float f, MyClass z);

A delegate can have many parameters.

Code for creating a delegate:

	MyDel myDel = new MyDel(MyDelHandler);
	myDel(10, 12.4, myClass);

The delegate calls the method indirectly.

Delegates are better than simple threads for these reasons:

+ Parameter to delegate can be any type.
+ More than one parameter can be passed.
+ Delegate threads are allocated from a thread pool, so threads can be reused.

When a delegate is fired this way though, it is synchronous (like calling `Invoke`).

`BeginInvoke` is used to make an asynchronous call to the delegate method.

`BeginInvoke` has two more parameters:

+ A method to call when the thread ends. // Callback method.
+ The delegate that is making the call.

How to call `BeginInvoke`:

	calcPi.BeginInvoke((int) declimalPlacesNumericUpDown, EndCalcPi, calcPi);

End Thread Callback Method:

	virtual protected void EndCalcPi(IAsyncResult result)
	{
		try
		{
			CalcPiDelegate calcPi = (CalcPiDelegate)result.AsyncState; // Retrieve delegate.
			calcPi.EndInvoke(result); // Cleans up thread resources.
		}
		catch(Exception ex)
		{
			ShowProgress(ex.Message, 0, 0);
		}
	}

But it STILL crashes. The new thread is trying to access the UI controls. We are back to square one!

The `InvokeRequired` property of a control is true if a thread that did not create the control is trying
to update the control.

### Simplified Threading

The Control class also has `Invoke` and `BeginInvoke`, but a new thread is NOT created. Instead they
redirect to the thread that created the control (i.e., find the thread that created this control and run
this code on that thread). 

`control.Invoke(delegate)`; // Runs the delegate synchronously on the thread that created the control.  
`control.BeginInvoke(delegate);` // Does the same thing asynchronously.

Control Invoke Steps:

+ Create another delegate for the ShowProgress method.
	+ `Show ProgressDelegate showProgress = new ShowProgressDelegate(ShowProgress);`
+ Call this delegate with the Invoke method of the main form.
	+ `this.Invoke(showProgress, new object[] {pi.ToString(), digits, 0});`

Simplified Threading is a feature added to .NET 2.0. The `BackgroundWorker` component creates delegates
in the background and encapsulates them in the simple method calls.

Start a thread asynchronously with 
`this.backGroundWorker.RunWorkerAsync((int)decimalPlacesNumericUpDown.Value);` // Drop component on  form.

The background worker has an event named DoWork. Add a handler for it and start the work the thread.

	void backgroundWorker_DoWork(object sender, DoWorkEventArgs e)
	{
		calcPi((int)e.Argument);
	}

To report progress:

+ Set the property WorkerReportsProgress to true in the BackgroundWorker.
+ Add a handler for the ProgressChanged event, in the BackgroundWorker.
+ Update the UI in ProgressChanged. The method will be run on the thread that created the
BackgroundWorker.

ReportProgress can be called in two ways:

+ `backgroundWorker.ReportProgress(percentageChanged);`
+ Pass a custom object to ReportProgress. The custom object can be modified by either thread.
	+ `backgroundWorker.ReportProgress(percentageChanged, obj);`
	+ Use custom class to send data to the UI.
		+ `CalcPiUserState state = new CalcPiUserState(pi.ToString(), digits, i + digitCount);`  
		`backgroundWorker.ReportProgress(0, state);`  
		The method can calculate the percentage of progress from the data in the custom class.

Have a ProgressChanged event handler in UI to unpack CalcPiUserState and to call ShowProgress.

To have a callback, register a handler with the RunWorkerCompleted event. This is run on the UI thread.

DoWorkEventArgs in DoWork EventHandler has a Result property that can be used to set the result of a
thread.

Accessing e.Result will cause an error if the thread crashes.

Code to use before accessing e.Result.

`if(e.Error == null){ // Access e.Result }`

### Canceling a Thread

ReportProgress is asynchronous, not synchronous. 

Safe solution is to use the control.Invoke method to call the progress handler. This call is synchronous.
Use it instead of ReportProgress to pass data to the UI, when both threads can change the data.
ReportProgress is safe, only if values are read-only like structures.

If the UI update method expects a custom class that contains the user state, like 
`void ShowProgress (UserState state);` Create a delegate that can register a method that updates the UI.
`delegate void ShowProgressDelegate(UserState state);`

Instantiate an instance of the delegate: 
`ShowProgressDelegate showProgressDelegate = new ShowProgressDelegate(ShowProgress);`

Call the delegate with: `this.Invoke(showProgressDelegate, userState);` // Created on UI.

Now, both threads can modify userState without shared access by the other thread.

Control.Invoke passes ownership and is synchronous. By passing a custom object as the parameter, both
threads can read an modify it, but only one thread can change it at a time, so there is no chance for
data corruption.

To make an inefficient pause, use `Thread.Sleep(1000);`. This is busy waiting.

A better pause is that the main will create an event and call one of two methods: Set and Reset. The
thread will call the WaitOne method to see if it should pause. If the main form calls Reset, then the next
time the thread reaches the WaitOne call, the thread will stop. It will wait until the main form calls Set
before it continues to run. The ManualResetEvent is used to fire an event when a condition is met.

pauseEvent is an instance variable but it is thread-safe. Typically, call WaitOne whenever progress is
reported.

If a form is closing, close the thread in FormClosing EventHandler.

Always pass data synchronously. Control.Invoke is perfect for doing this.

### Passing Data to a Thread

BackgroundWorker has a property to let it know that the main thread wants it to cancel. The variable is
read in the worker. The property WorkerSupportsCancellation must be set. Indicate the thread should cancel
with bw.CancelAsync(). CancellationPending is a property used to check the flag - whether a cancellation
is pending or not. Test the worker is running with the property bw.IsBusy.

CancelAsync does NOT cancel the thread. A thread must check frequently to see if cancellation has been
requested. The running example in the class will check after every 9 digits of PI is calculated.

An example of manually canceling the thread:

	if(bw.CancellationPending)
	{
		closeResources();	// Helper method.
		break;
	}

Remember that CancellationPending will remain true until DoWork completes. The worker should detect if the
thread has been cancelled and set the Cancel property in the event args to true.

In RunWorkerCompleted event handler in UI thread, check if thread is canceled (e.Canceled in
RunWorkerCompletedEventArgs).

How do we get the data from the UI to the thread? In ReportProgress(), the object parameter passed by
reference can be used to send data from the UI to the thread. Only pass a variable that is created in the
thread. The main thread will unload the data and then modify the data.

Once ReportProgress returns, the worker thread can use the new data from the main thread. Create variable
in thread and pass it to the UI.

## Week 11 - Printing and Cloning

### 11.1: Printing Basics

**Print classes**:

+ `PrintDocument` describes the characteristics of what to be printed.
+ `BeginPrint`, `EndPrint`, `PrintPage`, `QueryPageSettings` are the four events in a `PrintDocument`.
+ `PrintControllers` talk to the printer.
+ `PrintPreview` allows the user to see what the printed document will look like.

Standard dialogs for print and print preview.

The minimum that needs to be done is add a print document to the application and handle the `PrintPage` event.

Event `PrintPage` has a `PrintPageEventArgs`. It is similar to painting:

	private void printDocumentSimple_PrintPage(object sender, PrintPageEventArgs e)
	{
		Graphics g = e.Graphics;
		using(Font font = new Font("Lucida Console", 36))
		{
			g.DrawString("Hello, \nPrinter", font, Brushes.Black, 0, 0);  // (0,0) is the start point.
		}
	
	}

**How to use a StandardPrintController**:

+ All print documents use a print controller class to interface with the actual printer. The default print controller displays the progress of the printing. It's the one that sends the page to the printer.
+ A standard print controller does not display the print page dialog. To use it, set the print document's print 
controller property: `printDocument.PrintController = new StandardPrintController();` Now, the dialog will not appear.

**How to use preview controls and dialogs**:

+ The preview control and dialog are useful, since they show how the doc will appear on the printer without actually printing.
+ For both, drag the control or component onto the dialog and set the `PrintDocument` property. Use the control to 
embed a print preview in another control.

**Multiple pages don't print automatically**:

+ In the `PrintPage` handler, set the `HasMorePages` property of `PrintPageEventArgs` to true to have the method called again.
+ The programmer must determine if there are more pages.
+ THe programmer must determine what to print on each page.

**How to define to display the correct information for each page?**

Don't use a loop! Simply print content, keep a member variable for counting and set `HasMorePages` to `true`.

Use `BeginPrint` and `EndPrint` to create and dispose resources that are reused in the print page event.
`PrintEventArgs` has `Cancel` and `PrintAction`. `PrintAction` indicates if the current action is to file, printer, or preview:

	bool preview = false;
	private void printDocument_BeginPrint(object sender, PrintEventArgs e)
	{
		preview = (e.PrintAction == PrintAction.PrintToPreview);
		m_fontPrinter = new Font("Lucida Console", 72);
	}

	private void printDocument_EndPrint(object sender, PrintEventArgs e)
	{
		m_fontPrinter.Dispose();
		m_fontPrinter = null;
		m_nPage = 1;
	}

### 11.2: Printing Within Page Bounds

When painting, the client rectangle is often used as the boundary of the drawing area (page: edge of paper vs
within margins). Printing outside of margins = header, footer.

`PrintPageEventArgs` has properties for the boundaries of the page:

+ `PageBounds` is the entire page.
+ `MarginBounds` is the `PageBounds`, offset by the margins.

Each of these returns measurements in display units on printer, 1/100 inch.

Not all printers can print to the edge of the page, but page bounds always start at (0,0) and has the size of the
paper, not the size of the printable area.

Page bounds has the size of the paper, not the size of the printable area. (**THIS IS A PROBLEM!**)

**How to calculate physical bounds** (see zip file from Ch. 8):

+ Use `PageSettings` and `Graphics` context to get the actual constraints of the printer.
+ Use `GetRealPageBounds`, `GetRealMarginBounds` to be sure that the margins are correct for all printers.

**RealPageBounds** (see 11:33 of video):

+ In print preview mode, there is not a problem, since the page is virtual.
+ The Graphics object has a property that returns the rectangle that printer can actually print in: `VisibleClipBounds`.
+ Since this is part of the Graphics, it might have a Page Scale. Transform the points from Page to Device and
+ then from Device to Display.

**MarginBounds**:

+ The `VisibleClipBounds` always starts at (0, 0).
+ The margin bounds are offset from this.
+ Margins have to be modified in order to be accurately represented on the printer (see end of video).

### 11.3: Using Margins

`PrintPageEventArgs` has a `PageSettings` property. `PageSettings` has properties that specify the distance from the edge of the paper to the upper left corner of the clip bounds. These are known as `HardMarginX` and `HardMarginY`. Use these to offset the clip bounds.

**How to modify the margin bounds**:

	public static Rectange GetRealMarginBounds(PrintPageEventArgs e, bool prview)
	{
		if(preview)  // Set by BeginPrint.
			return e.MarginBounds;

		Rectangle marginBounds = e.MarginBounds;  // Display units (everything in display units).

		marginBounds.Offset((int)-e.PageSettings.HardMarginX, (int)-e.PageSettings.HardMarginY);

		return marginBounds;
	}

**How to print outside the margins:**

Headers and footers are printed outside the margins. `StringFormat` can be used to place text in the footer area in
the lower right corner of the page: far, far. Header in the center top of page: near, center.

	StringFormat format = new StringFormat();
	format.Alignment = StringAlignment.Far;
	format.LineAlignment = StringAlignment.Far;
	
	/* preview is the member variable from earlier. */
	g.DrawString("footer stuff", font, brush, PrintUtils.GetRealPageBounds(e, preview), format);

**Page settings**:

+ Use a `PageSetupDialog` to change the default page settings for the app.
+ Page settings can be changed dynamically for each page.
+ `PageSettings` is a property of the `QueryPageSettingsEventArgs`.
+ Use the `QueryPageSettings` handler. It is called before each page.
+ Change the page settings that are passed in the `QueryPageSettingsEventArgs`, not the `PageSetupDialog` for the
form.
+ The `QueryPageSettings` method is called before every `PrintPage` call. It can be used to dynamically change the
margins for every page.

**How to allow some pages:**

+ Set the `AllowSomePages` property to true in the printDialog class.
+ In the `PrintSettings`, set the `ToPage`, `FromPage`, `MinPage`, and `MaxPage` properties.
+ The `PrintSettings` are accessible from the printDocument class.
+ Print handler must be able to read these and print the appropriate pages.

All printing is passed to the default printer, unless a PrintDialog is used to change it:

+ Set up the min, max pages before the dialog opens.
+ Set up from page and to page...
+ Set `AllowSomePage` to true...

### 11.4: Printing Techniques

**How can you print each string in an array so they start on a new line and wraps them in the printable area?**
(see beginning of video)

**How can the number of pages to be printed known before the pages are printed?**

This is difficult. .NET can count the pages. This can be done using a `PreviewPrintController`. It knows the number of pages after printing. Create a custom `PreviewPrintController` and override the `OnStartPrint` and `OnStartPage`
event handlers.

**Write a method that will** (see 11:20 and 12:40):

+ Save the current print controller.
+ Set the print controller to the custom one.
+ Print the document (virtually).
+ Retrieve the page count.
+ Reset the print controller to the original one.
+ Show `PrintDialog`.

**How can a large text file be displayed over multiple pages, performing word wrap as expected in a word processor?** (see 16:30)

The `FormatFlags` property can be set to `StringFormatFlags.LineLimit` and be used in StringFormat to display only
a full line. MeasureString has an overload that has two out paramseters: `out int charactersFitted` and `out int linesFilled`.

Use the number of characters that fit in the page and the substring method to find the remaining string that must
be printed. `stringToPrint = stringToPrint.Substring(characters OnPage)`

### 11.5: Copying and Cloning

	Not in the textbook.

Each object has `MemberWiseClone`, but it only makes a shallow copy of an object. The method is protected:

+ Create a new object.
+ Copy the non-static fields.
+ Value fields are bit-by-bit copies.
+ Object fields point to the same object as the original.

The `ICloneable` interface has one member: `public object Clone()``.

Four ways to implement a deep copy:

+ Use `ICloneable`.
+ Use copy constructors.
+ Use serialization.
+ Use reflection.

**The** `ICloneable` **interface simply requires that your implementation of the** `Clone` **method return a copy of the current object instance**:

+ It does not specify whether the cloning operation performs a deep copy, a shallow copy, or something in between.
+ It does not require all property values of the original instance to be copied to the new instance.
+ Not recommended to be implemented in public APIs.
+ Call `this.MemerbwiseClone()` for each field where the field's class also implements `ICloneable`.

**Deep cloning with a constructor** (see 10:30):

+ Like `ICloneable`.

**Serialization  with memory streams**:

+ Serialize the object to be deep copied.
+ Restore to a different object.

To deserialize, the stream must be reset: `stream.Seek(0, SeekOrigin.Begin)`.

	MemoryStream stream = new MemoryStream();
	CopyClass serial = new CopyClass();
	Serialize(stream, serial);
	Object obj = Deserialize(stream);
	CopyClass serial2 = obj as CopyClass;

**Reflection is too damn hard!**

## Week 12 - Images

### Images Introduction

**Image Class**

+ The Image (interface) class contains Bitmap and WMF formats. EMF is a newer version of WMF (Windows Meta File).
+ Draw an image on the graphics context with g.DrawImage(file, point). // Starting point.
+ The image will be drawn to its full size.

**Scaling and Clipping**

+ Scale the image to given rectangle: `g.DrawImage(img, rectScaling);`
+ Clip to the rectangle (and maybe scale): `g.DrawImage(img, destRect, srcRect, GraphicsUnit.Pixel)`
+ If the src and dest are the same size, then no scaling is done.

**Panning**

+ To implement panning, change the source rectangle and Invalidate.
+ Implement buttons to change the size of the offset.

**Skewing**

+ Skewing will scale the image to a parallelogram, instead of a rectangle.
+ Send DrawImage an array of three points.
+ The fourth point will be calculated to make a parallelogram.
+ `g.DrawImage(img, points);`

**Rotating and Flipping**

+ Before drawing the image, rotate/flip it.
+ Calls to these methods are cumulative. FlipX followed by FlipX does not have a flip.
+ Use the RotateFlipType enumeration to define the oepration.
+ img.RotateFlip(RotateFlipType.Rotate90FlipNone);

**Enum Class - Get Names, Parse**

Use the enum class to retrieve the names of all the elements in an enum. Create a sorted list of names in the enum. In the loop, retrieve the enumerated value from the string:

	string[] rotateFlipNames = Enum.GetNames(typeof(RotateFlipType));
	Array.Sort(rotateFlipNames);
	
	foreach(string name in rotateFlipNames)
	{
		RotateFlipType rotateFlip = (RotateFlipType)Enum.Parse(typeof(RotateFlipType), rotateFlipName);
		bitmap.RotateFlip(rotateFlip);
	}
	
To get values: If a sorted list is not needed, then just loop through the values by calling GetValues:
	
	foreach (RotateFlipType rotateFlip in Enum.GetValues(typeof(RotateFlipType)))
	{
		...
	}
	
**Recoloring**

+ Define a ColorMap for each color to change.
	+ Set the OldColor property to the color to find.
	+ Set the NewColor property to the color that will replace the old color.
+ Define an ImageAttributes and set the remap table to the ColorMap.
+ Call DrawImage with the image attributes.

Example:

	ColorMap[] colorMap = new ColorMap[1]
	colorMap[0] = new ColorMap()'
	colorMap[0].OldColor = Color.Lime;
	colorMap[0].NewColor = Color.White;
	
	ImageAttributes attr = new ImageAttributes();
	attr.SetRemapTable(colorMap);
	
	g.DrawImage(bmp, rect, 0, 0, rect.Width, rect.Height, g.PageUnit, attr);

###

The call `bmp.MakeTransparent()` makes an image transparent.

**Drawing to Images**

3 Steps:

+ Create a bitmap with the desired width, height, and pixel depth (number of color for each pixel).
+ Create a graphics context from the bitmap.
+ Draw on the graphics context.
+ Save the image.

**Create Image Graphics**

Obtain the pixel depth for the existing screen by creating a graphics object and passing it to the constructor of the
bitmap:

	Graphics display = this.CreateGraphics();
	Image img = new Bitmap(width, height, display);
	Graphics imageGraphics = Graphics.FromImage(img);
	
After drawing on the graphics context, the image can be saved. The default format is PNG, regardless of the extension: `img.Save(@"c:\image.png");`. Change the format with a second parameter from ImageFormat: Bmp, Emf, Gif, Icon, Jpeg, 
Tiff, Wmf: `img.Save(@"c:\image.gif", ImageFormat.Gif);`.

**Icons**

Icons are different than regular images. Icons have several bitmaps. The diffrent bitmaps are for different sizes.

Convert a regular bitmap to an icon: 

	IntPrt hIcon = bmp.GetHicon();
	Icon icon = Icon.FromHandle(hICon);

Draw an icon on the screen with:

	g.DrawIcon(icon, rect);
	g.DrawIconUnstretched(icon, rect);
	Steal icons from Windows executables with Icon.ExtractAssociatedIcon("program.exe");

Use Visual Studio to create and edit icons. Icons have many different resolutions, so edit all of them.



