# The key to distinguishing Flutter from other technologies

* Flutter uses its own Native rendering engine to render views, which completes the closed loop of component rendering by itself; while frameworks such as RN and Weex only call system components through JavaScript virtual machine extensions, and finally are completed by Android or iOS systems The rendering of the component.
* Flutter's UI thread uses Dart to build view structure data. These data will be synthesized on the GPU thread, and then handed over to the Skia engine to be processed into GPU data, and these data will be finally provided to the GPU for rendering through OpenGL.
* Skia is the underlying image rendering engine of Flutter.
   * Skia is already the official image rendering engine of Android, so the Flutter Android SDK can get natural Skia support without embedding the Skia engine; for the iOS platform, since Skia is cross-platform, it serves as the iOS rendering of Flutter The engine is embedded in the Flutter iOS SDK, replacing the closed-source Core Graphics/Core Animation/Core Text of iOS, which is why the app package packaged by the Flutter iOS SDK is larger than that of Android.
   * The underlying rendering capabilities are unified, and the upper-layer development interface and functional experience are also unified immediately. Developers no longer have to worry about platform-related rendering features. In other words, Skia guarantees that the rendering effect of the same code call on the Android and iOS platforms is exactly the same.
* Why Dart?
Because Dart supports both JIT (JUST-IN-TIME) and AOT (Ahead-Of-Time), it has high development efficiency, good running speed, and high execution performance. Choose JIT during the development period, which is very convenient for development and debugging (hot overload); use AOT during the release period, and the execution performance of local code is more efficient.

## Flutter JIT and AOT compilation
The compilation mode can be roughly divided into two types, AOT compilation and JIT compilation. The full name of JIT is Just In Time. The code can be compiled during program execution, because it needs to be analyzed and compiled before the program is executed. JIT compilation may lead to slower program execution time; while AOT compilation, the full name of Ahead Of Time, is when the program is running. It has been compiled before, and it is slow for developers to modify the code and compile, but it does not need to be analyzed and compiled at runtime, so the execution speed is faster.

Flutter uses a unique compilation mode. In the development phase, Kernel Snapshot mode (corresponding to JIT compilation) is used to generate tokenized source code from the dart code, compiled at runtime, interpreted and executed; in the release phase, ios uses AOT compilation, and the compiler will The dart code generates assembly code, and finally generates app.framwork, android uses Core JIT compilation, dart is converted into binary mode, and loaded before the VM starts.

Therefore, based on the Kernel Snapshot compilation mode in the development stage, we can know that Hot Reload scans the project files, converts the changed dart files into tokenized source code kernel files, sends them to the running DartVM, DartVM replaces resources, and then notifies Flutter Framework rebuilds, re-layouts, and re-draws WidgetsTree to see the effect of the changes.

## Principle of Flutter
* Architecture diagram of Flutter:
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/3hj8g8xrsj.jpeg?imageView2/2/w/1620">

* The Flutter architecture adopts a layered design, which is divided into three layers from bottom to top, in order: Embedder, Engine and Framework
   * Embedder is the operating system adaptation layer, which realizes the adaptation of platform-related features such as rendering Surface settings, thread settings, and platform plug-ins. From here we can see that there are not many related features of the Flutter platform, which makes the cost of maintaining cross-terminal consistency at the framework level relatively low.
   * The Engine layer mainly includes Skia, Dart, and Text, which implements functions such as Flutter's rendering engine, text layout, event processing, and Dart runtime. Skia and Text provide the upper-layer interface with the ability to call the underlying rendering and typesetting, while Dart provides Flutter with the ability to call Dart and the rendering engine at runtime. The role of the Engine layer is to combine them to achieve view rendering from the data they generate.
   * The Framework layer is a UI SDK implemented with Dart, which includes functions such as animation, graphics drawing and gesture recognition. In order to provide a more intuitive and convenient interface when drawing fixed-style graphics such as controls, Flutter also encapsulates a set of UI component libraries based on the two visual design styles of Material and Cupertino based on these basic capabilities. When we develop Flutter, we can directly use these component libraries.
  
## layout
Flutter uses a depth-first mechanism to traverse the rendering object tree to determine the position and size of each rendering object in the rendering object tree on the screen. During the layout process, each rendering object in the rendering object tree will receive the layout constraint parameters of the parent object to determine its own size; then the parent object determines the position of each child object according to the control logic to complete the layout process. As shown below
 
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/fzbm29uz9l.jpeg?imageView2/2/w/1620">
In order to prevent the entire control tree from being re-layouted due to changes in the factor nodes, Flutter has added a new mechanism - the layout boundary (Relayout Boundary), which can automatically or manually set the layout boundary at certain nodes. When any object within the boundary occurs When re-layouting, objects outside the bounds are not affected, and vice versa. As shown below:
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/w4ovjmpuqh.jpeg?imageView2/2/w/1620">
  
## draw
After the layout is completed, each node in the rendered object tree has a definite size and position. Flutter will draw all rendering objects to different layers. Like the layout process, the drawing process is also a depth-first traversal, and always draws itself first, and then draws child nodes.
Take the following figure as an example. After node 1 draws itself, it will draw node 2, then draw child nodes 3, 4 and 5, and finally draw node 6.
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/wgaxa6i6tr.jpeg?imageView2/2/w/1620">
It can be seen that due to some other reasons (such as manual merging of views), the child node 5 of 2 is on the same layer as its sibling node 6, which will cause node 6, which has nothing to do with it, to be redrawn when node 2 needs to be redrawn. It will also be redrawn, causing performance loss.

In order to solve this problem, Flutter proposes a mechanism corresponding to the layout boundary - Repaint Boundary. Within the redrawing boundary, Flutter will force a new layer to be switched, so as to avoid the mutual influence inside and outside the boundary, and avoid unnecessary redrawing caused by unrelated content placed on the same layer.
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/2p3dnw909o.jpeg?imageView2/2/w/1620">
A typical scenario for redrawing bounds is ScrollView. When ScrollView scrolls, it needs to refresh the view content, which triggers content redrawing. When the scrolling content is redrawn, in general, other content does not need to be redrawn. At this time, redrawing the border comes in handy.

## Summary
Skia and Dart are the key technologies for building the bottom layer of Flutter, and they are also the core that distinguishes Flutter from other cross-platform solutions.

The limitation of the cross-platform solution is that it is difficult to fully guarantee the true multi-terminal consistency. Needless to say, RN, the performance and behavior of many components are different at both ends. Even Flutter can only achieve multi-terminal consistency above the rendering layer, and there are still some native things (such as Push, map, positioning, Bluetooth, WebView) that cannot be bypassed, and need to be solved by writing plug-ins on the native. But having said that, if it is really bypassed, then Flutter will become an operating system, and the typed out package may not be able to handle hundreds of megabytes.

Flutter refers to the knowledge system map:
<image src="https://ask.qcloudimg.com/http-save/yehe-4984806/iyi15cv1ue.jpeg?imageView2/2/w/1620">

The inheritance relationship of common objects in FLutter:
<image src="https://upload-images.jianshu.io/upload_images/4185621-fe1e969c84d57481.png">

Widget life cycle:
<image src="https://upload-images.jianshu.io/upload_images/4185621-77330034d352258b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp">
More about this source textSource text required for additional translation information
