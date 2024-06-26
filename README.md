
# SDL_gfx-cmake

This is an SDL2 fork of <https://github.com/ferzkopp/SDL_gfx>, which applies a [modified version](https://github.com/ferzkopp/SDL_gfx/pull/6) of the `SDL_gfx-SDL2.patch`.

In addition to that, I've written my own CMakeLists.txt, as the one provided in the patch is broken.

I've also modified the source to reflect new features and changes introduced in the repo <https://github.com/rtrussell/BBCSDL>.

## Example Usage with CMake

Be aware that the CMakeLists.txt that I crafted does not identify SDL2's headers on its own, so it is *ESSENTIAL* that you include SDL2's headers onto the SDL_gfx target.

```cmake
FetchContent_Declare(
  SDL_gfx
  GIT_REPOSITORY https://github.com/zoogies/SDL_gfx-cmake
  GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(SDL_gfx)
target_include_directories(SDL_gfx PUBLIC ${SDL2_SOURCE_DIR}/include) # REPLACE WITH YOUR SDL2 HEADER LOCATION
target_include_directories(YOUR_TARGET_HERE PUBLIC ${SDL_gfx_SOURCE_DIR})
```

## Original README

```txt
/*!


\mainpage SDL_gfx - SDL-1.2 graphics drawing primitives, rotozoom and other supporting functions


\section contact_sec Contact and License

Email aschiffler at ferzkopp dot net to contact the author or better check
author's homepage at http://www.ferzkopp.net for the most up-to-date
contact information.

This library is licenced under the zlib License, see the file LICENSE for details. 


\section intro_sec Introduction

The SDL_gfx library evolved out of the SDL_gfxPrimitives code which
provided basic drawing routines such as lines, circles or polygons for 
SDL Surfaces and adding a couple other useful functions for zooming 
images for example and doing basic image processing on byte arrays.

Note that SDL_gfx is compatible with SDL version 1.2 (not SDL2).
A contributed patch to enable compilation of SDL_gfx against SDL2 
with CMake is provided in the "Other Builds" folder.

The current components of the SDL_gfx library are:
- Graphic Primitives (SDL_gfxPrimitives.h, SDL_gfxPrimitives.c)
- Rotozoomer (SDL_rotozoom.h, SDL_rotozoom.c)
- Framerate control (SDL_framerate.h, SDL_framerate.c)
- MMX image filters (SDL_imageFilter.h, SDL_imageFilter.c)
- Custom Blit functions (SDL_gfxBlitFunc.h, SDL_gfxBlitFunc.c)
- Build-in 8x8 Font (SDL_gfxPrimitives_font.h)



\subsection notes_gfx Notes on Graphics Primitives

Care has been taken so that all routines are fully alpha-aware and can 
blend any primitive onto the target surface if ALPHA<255. Surface depths 
supported are 1,2,3 and 4 bytes per pixel. Surface locking is implemented
in each routine and the library should work well with hardware 
accelerated surfaces. 

\htmlonly
<a href="../Screenshots/SDL_gfxPrimitives.jpg" target="_blank" title="SDL_gfxPrimitives Screenshot"><img src="../Screenshots/SDL_gfxPrimitives-thumb.jpg" border="0" hspace="5"></a><br />
\endhtmlonly

Currently, The following Anti-Aliased drawing primitives are available:
- AA-line
- AA-polygon
- AA-circle
- AA-ellipse

Note: All ___Color routines expect the color to be in the format 0xRRGGBBAA.

\subsection notes_roto Notes on Rotozoomer

The rotozoom code is not ASM-speed quality, but it should be fast enough 
even for some realtime effects if the CPU is good or bitmaps small.
With interpolation the routines are typically used for pre-rendering stuff 
in higher quality (i.e. smoothing) - that's also the reason why the API differs 
from SDL_BlitRect() - as they create a new target surface each time rotozoom 
is called. The final rendering speed is dependent on the target surface
size as it is beeing xy-scanned when rendering the new surface.

\htmlonly
<a href="../Screenshots/SDL_rotozoom.jpg" target="_blank" title="SDL_rotozoom Screenshot"><img src="../Screenshots/SDL_rotozoom-thumb.jpg" border="0" hspace="5"></a><br />
\endhtmlonly

Note also that the smoothing toggle is dependent on the input surface bit 
depth. 8bit surfaces will \b never be smoothed - only 32bit surfaces will.

Note that surfaces of other bit depth then 8 and 32 will be converted 
on the fly to a 32bit surface using a blit into a temporary surface. This 
impacts performance somewhat.

Smoothing (interpolation) flags work only on 32bit surfaces:
\verbatim
 #define SMOOTHING_OFF		0
 #define SMOOTHING_ON		1
\endverbatim
 
\subsection notes_rate Notes on Framerate Manager

The framerate functions are used to insert delays into the graphics loop
to maintain a constant framerate.

The implementation is more sophisticated that the usual
\verbatim
	SDL_Delay(1000/FPS); 
\endverbatim
call since these functions keep track of the desired game time per frame 
for a linearly interpolated sequence of future timing points of each frame. 
This is done to avoid rounding errors from the inherent instability in the 
delay generation and application.

\htmlonly
<a href="../framerate.png" target="_blank" title="Framerate Diagram"><img src="../framerate-thumb.png" border="0"></a><br />
\endhtmlonly

i.e. the 100th frame of a game running at 50Hz will be accurately
2.00sec after the 1st frame (if the machine can keep up with the drawing).

The functions return 0 or 'value' for sucess and -1 for error. All functions
use a pointer to a framerate-manager variable to operate.


\subsection notes_filter Notes on ImageFilters

The imagefilter functions are a collection of MMX optimized routines that
operate on continuous buffers of bytes - typically greyscale images from 
framegrabbers and such - performing functions such as image addition and 
binarization. All functions (almost ... not the the convolution routines) 
have a C implementation that is automatically used on systems without MMX 
capabilities.

The compiler flag -DUSE_MMX toggles the conditional compile of MMX assembly.
An assembler must be installed (i.e. "nasm").


\subsection notes_blitters Notes on Custom Blitters

The custom blitter functions provide (limited) support for surface
compositing - that is surfaces can be blitted together, and then
still blitted to the screen with transparency intact.

\subsection platforms Supported Platforms

\subsubsection platformlinux Unix/Linux

The library compiles and is tested for a Linux target (gcc compiler) via the
the usual configure;make;make install sequence.

\subsubsection platformwindows Windows

A Win32 target is available (VisualC, mingw32, xmingw32 cross-compiler).

VS2022

- Download SDL sources from https://github.com/libsdl-org/SDL-1.2
- Place files into directory along the SDL_gfx source folder as "SDL-1.2" folder
- Recompile SDL using VS2022 (note SDL's README for instructions of config file)
- Open SDL_gfx.sln in VS2022 and recompile all
- Manually copy the SDL.sll into the target folder of Test programs

See "Other Builds" for additional makefiles (likely out of date, since not maintained).

When using the cross-compiler (available on the author's homepage, very
out of date), the build process generates .DLLs. You can use the command 
line 'LIB.EXE' tool to generate VC6 compatible .LIB files for linking 
purposes. 

\subsubsection platformosx Mac OSX 

The usual autotools build chain should be used. MacPorts or fink may 
be required (that's what the author uses).

Xcode is supported via templates. See "Other Builds" folder Xcode3+.zip -
this template only supports SDL_gfx and not the tests. For this template, the
Deployment Target (the lowest version to run on) is set to 10.5 and expects
the SDL.framework preinstalled in some default location
(either /Library/Frameworks, or ~/Library/Frameworks). 

Older targets are also reported to work (10.3+ native and Project Builder). 

\subsubsection platformqnx QNX

QNX was reported to build (see .diff in "Other Builds").

\subsubsection platformzune Zune

ZuneHD (WinCE 6 ARM) is reported to build (see OpenZDK in "Other Builds").
Note that between rendering on the Zune's ARM CPU and constantly uploading
textures to the GPU, SDL_gfx is going to be slow. Also, the libc math 
functions all use software FP emulation even when VFP floating point
support is turned on in the compiler, so there's extra overhead due to that
as well.

\subsubsection platformothers Others

Other platforms might work but have not been tested by the author.
Please check the file "INSTALL" as well as the folder "Other Builds".

See also section "Installation" below for more build instructions.

\section install_sec Installation

\subsection unix Unix/Linux

To compile the library your need the SDL 1.2 installed from source or 
installed with the 'devel' RPM package. For example on Mandriva, run:
\verbatim
	urpmi  libSDL1.2-devel
\endverbatim

Then run
\verbatim
	./autogen.sh	# (optional, recommended)
	./configure
	make
	make install
	ldconfig
\endverbatim

to compile and install the library. The default location for the 
installation is /usr/local/lib and /usr/local/include. The libary 
path might need to be added to the file:
	/etc/ld.so.conf

Run the shell script 'nodebug.sh' before make, to patch the makefile 
for optimized compilation:
\verbatim
	./autogen.sh	# (optional, recommended)
	./configure
	./nodebug.sh
	make
	make install
	ldconfig
\endverbatim

Check the folder "Other Builds" for alternative makefiles.

\subsection prep Build Prep

Run autogen.sh or manually:
\verbatim
	aclocal --force
	libtoolize --force --copy
	autoreconf -fvi
\endverbatim

\subsection nommx No-MMX

To build without MMX code enabled (i.e. PPC or for AMD64 architecture
which is missing pusha/popa):
\verbatim
	./configure --disable-mmx
	make
	make install
\endverbatim
i.e. to build on MacOSX 10.3+ use:
\verbatim
	./configure --disable-mmx && make
\endverbatim

\subsection vs9 Windows (VC9, VS2010)

Open SDL_gfx_VS2010.sln solution file and review README.

\subsection vs8 Windows (VC8, VS2008)

Open SDL_gfx_VS2008.sln solution file and review README.


\subsection vc6 Windows (VC6/7)

See folder Other Builds.

To create a Windows DLL using VisualC6:
\verbatim
	unzip -a VisualC6.zip
	vcvars32.bat
	copy VisualC/makefile
	nmake
\endverbatim
or
\verbatim
	unzip -a VisualC7.zip
\endverbatim
and open the project file.


\subsection wince WindowsCE

See folder Other Builds.

May need workaround for missing lrint.


\subsection cross Cross-Compilation

To build using mingw32 on Win32, check the makefile contained in mingw.zip

To create a Windows DLL using the xmingw32 cross-compiler:
\verbatim
	cross-configure
	cross-make
	cross-make install
\endverbatim

Make sure the -DBUILD_DLL is used (and only then) when creating the DLLs.
Make sure -DWIN32 is used when compiling the sources (creating or using
the DLLs.

Specify the path to your cross-compiled 'sdl-config', and invoke
'./configure' with the '--host' and '--build' arguments. For example,
to cross-compile a .DLL from GNU/Linux:
\verbatim
	SDL_CONFIG=/usr/local/cross-tools/i386-mingw32msvc/bin/sdl-config \
		./configure --host=i586-mingw32msvc --build=i686-pc-linux-gnu
	make
	make install
\endverbatim

\subsection qnx QNX

To build on QNX6, patch first using:
\verbatim
	patch -p0 <QNX.diff
\endverbatim

\subsection osx OSX

Use standard unix build sequence.

To build on MacOS X with Project Builder, follow these steps:
- Update your developer tools to the lastest version.
- Install the SDL Developers framework for Mac OS X.
- Download the latest SDL_gfx source distribution and extract the
	archive in a convenient location.
- Extract the included OSX-PB.tgz archive into the
	top directory of the SDL_gfx distribution (from step 3). This will create a
	PB that contains the project files.
- The project has targets for the SDL_gfx framework and the four test
	programs. All can be built using the 'deployment' or 'development' build
	styles. 

A newer version for MaxOS X is included in the OSX-PB-XCode.zip archive. The 
updated version uses relative pathnames where appropriate, and pointers to 
the standard  installation location of SDL. However, it may require XCode in 
order to be used.

\section interfaces_sec Language Interfaces

SDL_gfx has been integrated with the following language interfaces:
- Pascal: http://www.freepascal-meets-sdl.net
- Perl: http://sdl.perl.org
- Python: http://www.pygame.org
- C#: http://cs-sdl.sourceforge.net
- Lua: http://www.egsl.retrogamecoding.org/
- Oberon: http://sourceforge.net/projects/sdl-for-oberon/

\section test_sec Test Programs

Change to the ./Test directory and run
\verbatim
	./autogen.sh
	./configure
	make
\endverbatim
to create several test programs for the libraries functions. This requires
the library to be previously compiled and installed.


See the source code .c files for some sample code and implementation hints.


\section contrib_sec Contributors

- Fix for filledbox by Ingo van Lil, inguin at gmx.de - thanks Ingo.

- Non-alpha line drawing code adapted from routine 
  by Pete Shinners, pete at shinners.org - thanks Pete.

- More fixes by Karl Bartel, karlb at gmx.net - thanks Karl.

- Much testing and suggestions for fixes from Danny van Bruggen,
  danny at froukepc.dhs.org - thanks Danny.

- Original AA-circle/-ellipse code idea from Stephane Magnenat, 
  nct at wg0.ysagoon.com - thanks Stephane.

- Faster blending routines contributed by Anders Lindstroem,
  cal at swipnet.se - thanks Anders.

- New AA-circle/-ellipse code based on ideas from Anders Lindstroem - 
  thanks Anders.

- VisualC makefile contributed by Danny van Bruggen, 
  danny at froukepc.dhs.org - thanks Danny.

- VisualC7 project file contributed by James Turk, 
  jturk at conceptofzero.com - thanks James.

- Project Builder package contributed by Thomas Tongue, 
  TTongue at imagiware.com - Thanks Thomas.

- Fix for filledPolygon contributed by Kentaro Fukuchi 
  fukuchi at is.titech.ac.jp - Thanks Kentaro.

- QNX6 patch contributed by Mike Gorchak,
  mike at malva.ua - Thanks Mike.

- Pie idea contributed by Eike Lange,
  eike.lange at uni-essen.de - Thanks Eike.

- Dynamic font setup by Todor Prokopov,
  koprok at dir.bg - Thanks Todor.

- Horizontal/Vertical flipping code by Victor (Haypo) 
  Stinner, victor.stinner at haypocalc.com - Thanks Victor.

- OSX build fixes by Michael Wybrow, 
  mjwybrow at cs.mu.oz.au - Thanks Michael.

- gcc3.4 build fixes by Dries Verachtert, 
  dries at ulyssis.org - Thanks Dries.

- Updated OSX build by Brian Rice,
  water451 at gmail.com - Thanks Brian.

- aaellipse issues pointed out by Marco Wertz,
  marco.wertz at gmx.de - Thanks Marco.

- texturedPolygon idea and code by Kees Jongenburger,
  kees.jongenburger at gmail.com - Thanks Kees.
 
- Several bugfixes contributed by Sigborn Skjaeret, 
  cisc at broadpark.no - Thanks CISC.

- Syntax error for C++ found by Olivier Boudeville,
  olivier.boudeville at online.fr - Thanks Olivier.

- hline/vline clipping fixes found by Daniel Roggen,
  droggen at gmail.com and Mikael Thieme, 
  mikael.thieme at his.se - Thanks Daniel+Mikael.

- rotozoom fix for big-endian machines (i.e. PPC)
  by Julian Mayer, julianmayer at mac.com - Thanks
  Julian.

- Cross-compilation notes contributed by Sylvain
  Beucler, beuc at beuc.net - Thanks Sylvain.

- Fix duplicate pixel renders in circleColor contributed
  by David Raber, lotharstar at gmail.com - Thanks David.

- New arcColor (and arcRGBA) routine contributed
  by David Raber, lotharstar at gmail.com - Thanks David.

- Idea for polygonColorMT and texturePolygonMT routines for
  multithreaded operation contributed by "unknown" -
  Thank you.

- Multiple patches applied and repackaged version 2.0.18
  by Paul, sweetlilmre at gmail.com - Thanks Paul and
  other contributors of patches.

- Change to avoid gcc4 compiler warnings contributed by Thien-Thi
  nguyen, ttn at gnuvola.org - thanks Thi. 

- hline 1bpp off-by-one patch contributed by Manuel Lausch
  mail at manuellausch.de - Thanks Manuel.

- pkg-config support contributed by Luca Bigliardi, shammash
  at artha.org - thanks Luca.

- Updated mingw Makefile contributed by Jan Leike, jan dot leike 
  at gmx dot net - thanks Jan.

- Rotozoom debugging help by Jeff Wilges, heff at ifup dot us -
  thanks Jeff.

- Build updates provided by Barry deFreese, bdefreese at debian 
  dot org - thanks Barry.

- Fix for 1-pixel postponement with 8bit scaling by Sylvain Beucler, 
  beuc at beuc dot net - thanks Sylvain.

- Updates to headers and configure to allow for cross-compiling 
  to DLL (not just static .a) and fixes for compiling on Windows 
  using autotools by Sylvain Beucler, beuc at beuc dot net - thanks Sylvain.

- Added Visual CE Project to Builds. Contributed by vrichomme at smartmobili
  dot com - thanks.

- Added Symbian and Windows 64bit fix with lrint function by Rene 
  Dudfield, renesd at gmail dot com - thanks Rene.

- Fixes for Rotate90 0-degree case by Chris Allport, chris dot allport
  at gmail dot com - thanks Chris.

- Fixed for Win32 build support defines by tigerfishdaisy (SF) - thanks
  Tiger Tiger.

- Patch for OpenSDK contributed by itsnotabigtruck at ovi dot com - thanks
  IsNotABigTruck.

- OSX Xcode 3+ template ZIP contributed by marabus at meta dot ua - thanks
  Vasyl.

- Memory leak/error check patch for zoomSurface contributed by 
  RodolfoRG - thanks Rodolfo.

- Zoom scaling factor rounding bugfix patch contributed by WanderWander
  Lairson Costa - thanks Wander.

- Suggestions for speed improvements contributed by inkyankes - thanks.

- Pixel blend routine patches contributed by mitja at lxnav dot com -
  thanks Mitja.

- ImageFilter patches contributed by beuc at beuc dot net - thanks Sylvain.

- Bug reports contributed by Yannick dot Guesnet at univ-rouen dot fr -
- thanks Yannick.

\section changelog_sec Change Log

\verbinclude ChangeLog

*/
```
