KinectMinimal2D
===============
Minimal example project for 2D Kinect games. Use this project as a starting point for new games which will only need basic 2D graphics.


Usage
===============
After cloning, open the solution in Visual Studio. The project should be runnable immediately with Visual Studio's launch options.

If the project is working correctly, the program should wait a few seconds while it sets up the Kinect, then display the color video feed.


Requirements
===============
- Visual Studio 2012+
- Kinect SDK 1.8 (Kinect v1) or 2.+ (Kinect v2)
- A working and connected Kinect


Modifying the Project
===============
## Responding to Data From Kinect
The ```Sensor[XXX]FrameReady``` functions are registered as event handlers in the ```WindowLoaded``` function.
Any time the Kinect has a frame of the proper type ready, it will call the event handler registered to that frame type.
You can then use the data from the ```EventArgs``` to update variables that will be used for gameplay, drawing, etc.

## Drawing On Screen
Inside of any of the [XXX]FrameReady event handlers, acquire a ```DrawingContext``` thus:
```C#
using (DrawingContext dc = this.drawingGroup.Open())
{
  // ... call methods on dc
}
```
inside the using block, you can put logic to draw something to the screen every time a particular type of frame is ready.

Be aware that having draw steps in multiple FrameReady handlers can have undesirable effects with conflicting or overlapping output!

## Drawing With Colors, Brushes, and Pens
Microsoft provides [good documentation](http://msdn.microsoft.com/en-us/library/aa983677(v=vs.71).aspx) on the differences between these. When using these, it is recommended that you keep them as member variables on classes using them. Given a reference to a Color, Brush, or Pen, draw calls can be used:
```C#
  DrawingContext drawingContext = getDrawingContext();
  drawingContext.rect(fillbrush, outlinePen, new Rectangle(x, y, w, h), irrelevantAnimationsObj);
```

Also, these objects can be constructed from one another thusly:
```C#
Color.fromRgb(r, g, b)
new Brush(myColor)
new Brush(Colors.SOME_COLOR_CONST)
new Pen(myBrush)
new Pen(new Brush(Colors.SOME_COLOR_CONST))
new Pen(new Brush(Color.fromRgb(r, g, b)))
```

Check the signatures and documentation for ```DrawingContext``` methods to see available drawing options. Take note that such calls are drawn on top of each other, such that later draw calls are visible on top of older ones.

## Accessing Raw Video Data
The sensor object has ```ColorStream``` and ```DepthStream``` which allow access to the raw video data from the Kinect.
The frame objects which come off these streams allow access to 4-byte (BGRA) wide pixels in row-major format.
The simplest use for the raw streams is to pass their data through to the screen using a ```WriteableBitmap```:
```C#
this.colorBitmap.WritePixels(
  new Int32Rect(0, 0, this.colorBitmap.PixelWidth, this.colorBitmap.PixelHeight),
  this.pixelBytes,
  this.colorBitmap.PixelWidth * sizeof(int),
  0);
```
Using the intermediate bitmap for drawing is recommended as a performance optimization.
More complex processing of the raw data is possible, but beyond the scope of this README.

## Basic Skeleton Handling
The sensor object has a ```SkeletonStream``` which provides a series of frames of skeleton data, where one frame corresponds to a set of skeletons at one point in time. Before using the skeleton data, the array of ```Skeleton```s should be copied out of the opened ```SkeletonFrame```:
```C#
Skeleton[] skeletons;
using (SkeletonFrame skeletonFrame = e.OpenSkeletonFrame())
    {
        if (skeletonFrame != null)
        {
            skeletons = new Skeleton[skeletonFrame.SkeletonArrayLength];
            skeletonFrame.CopySkeletonDataTo(skeletons);
        }
    }
```
With skeleton data in hand, it is possible to draw them:
```C#
for (Skeleton skelly in skeletons) 
{
  draw(skelly); // Draws each joint and bone of the skeleton using brushes/pens
}
```
or update some game state:
```C#
for (Skeleton skelly in skeletons)
{
  saveSkellyData(skelly); // Save the skelly to e.g., disk, memory, a server, etc.
```
The skeleton data might even be used to affect the execution of the other frame event handlers:
```C#
private void SensorSkeletonFrameReady(object s, SkeletonFrameEventArgs e)
{
  // ... iterating over each skeleton
    saveJointData(skelly); // Save the location of each joint in screen space
  // ... end iteration
}
private void SensorColorFrameReady(object s, ColorFrameEventArgs e)
{
  // ... iterating over saved joint information
    drawPixelsAroundJoint(joint); // Draw an N-pixel radius circle around each joint, filling it with color frame data
  // ... end iteration
}
```
Conceptually, this would result in circular cutouts of the the user's joints using actual video data.
