Avisynth+ v3.6.0
----------------

Use the installer or copy files directly
- 64 bit OS:
  copy Avisynth.dll from x86 folder to the windows SysWOW64 folder
  copy Avisynth.dll from x64 folder to the windows System32 folder
- 32 bit OS
  copy Avisynth.dll from x86 folder to the windows System32 folder
- All OS: Copy appropriate files from the plugins+/plugins64+ to the appropriate Avisynth+ folder

Useful links:
-------------
Source (from 3.4): https://github.com/AviSynth/AviSynthPlus
Source (before 3.4): https://github.com/pinterf/AviSynthPlus/tree/MT
Forum: https://forum.doom9.org/showthread.php?t=168856
Forum on some avs+ filters: https://forum.doom9.org/showthread.php?t=169832
Avisynth+ info page: http://avisynth.nl/index.php/AviSynth%2B
Info on Avisynth+ new color spaces: https://forum.doom9.org/showthread.php?p=1783714#post1783714
Avisynth Universal Installer by Groucho2004: https://forum.doom9.org/showthread.php?t=172124

Short info for plugin writers
-----------------------------
  Avisynth+ header (and helper headers) are available here, or choose SDK during install:
  https://github.com/AviSynth/AviSynthPlus/tree/master/avs_core/include

  Use these headers for building x86/x64 plugins, and to use Avisynth+'s high-bitdepth related VideoInfo functions
  without any concern. When a VideoInfo function is non-existant on a system that uses your plugin,
  it won't crash, just fall back to a default behaviour. E.g. VideoInfo.BitsPerComponent() will always return 8
  when your plugin calls it on a Classic Avisynth, or pre-high bit depth Avisynth+ host.

(see readme_history.txt for details, syntax element, etc. They also appear on avisynth.nl)

20200331 3.6.0
--------------
- New function:
  SetMaxCPU(string feature)

  string "feature"

    "" or "C" for zero SIMD support, no processor flags are reported
    "mmx", "sse", "sse2", "sse3", "ssse3", "sse4" or "sse4.1", "sse4.2", "avx, "avx2"

    parameter is case insensitive.
    Note: "avx2" triggers FMA3 flag as well.

  Processor options w/o any modifier will limit the CPU flag report to at most the processor level.
  When "feature" is ended by '+', relevant processor feature flag will be switched on
  When "feature" is ended by '-', relevant processor feature flag will be removed
  Multiple options can be put in a comma separated list. They will evaluated in that order.

  Examples:
    SetMaxCPU("SSE2") reports at most SSE2 processor (even if AVX2 is available)
    SetMaxCPU("avx,sse4.1-") limits to avx2 but explicitely removes reporting sse4.1 support
    SetMaxCPU("C,avx2+") limits to plain C, then switches on AVX2-only support

- Script array for NEW_AVSVALUE define are working again. (default in Linux build - experimental)
  Memo:
    Compiler define NEW_AVSVALUE means script arrays
    - AVSValue deep copy for arrays (arrays in arrays in ...)
    - untyped and unconstrained element number
    - access with indexes or in a dictionary-like associative way
      array_variable = [[1, 2, 3], [4, 5, 8], "hello"]
      dictionary = [["one", 1], ["two", 2]]
      empty = []
      subarray = array_variable[0]
      val = subarray[2]
      val2 = array_variable[1, 3]
      str = array_variable[2]
      n = ArraySize(array_variable) #3
      n2 = ArraySize(empty) #0
      val3 = dictionary["two"]
    - arrays as filter parameters(named and unnamed)
      new 'a' type or use '.+' or '.*' and check AVSValue IsArray()
    - Concept is incompatible with avs 2.5 plugins due to their 'baked' interface code
    - Demo script (to be cleaned up)
      '''
      ColorBars()
      clip=last
      a = [[1,2],[3,4]]
      aa = [1]
      b = a[1,1] + ArrayGet(a, 1,0) + aa[0]

      empty_array = []
      empty_array_2 = empty_array
      #n3 = empty_array_2.ArrayGet(0) # array index out out range error!

      black_yuv_16 = [0,32768,32768]
      grey_yuv_16 = [32768,32768,32768]
      white_yuv_16 = [65535,32768,32768]
      aSelectColors = [\
        ["black", black_yuv_16],\
        ["grey", grey_yuv_16],\
        ["white",white_yuv_16],\
        ["empty",empty_array]\
      ]
      test_array = [99, 1.0, "this is a string"] # mixed types
      test_array2 = [199, 2.0, "This is a string"]

      n = ArraySize(test_array) # 3
      n2 = ArraySize(empty_array_2) # 0
      sum = FirstNSum(grey_yuv_16,2)
      b = b

      clip = clip.Text(e"Array size = " + String(n) +\
       e"\n Empty array size = " + String(n2) +\
       e"\n sum = " + String(sum) +\
       e"\n b = " + String(b) +\
       e"\n white_yuv_16[1]=" + String(aSelectColors["white"][1]) + \
       e"\n [0]=" + String(ArrayGet(test_array,0)) + \
       e"\n [1]=" + String(ArrayGet(test_array,1)) + \
       e"\n [2]=" + ArrayGet(test_array,2), lsp=0, bold=true, font="info_h")

      return clip

      function FirstNSum(array x, int n)
      {
        a = 0
        for (i=0, x.ArraySize()-1) {
          a = a + x[i]
        }
        return a
      }
      '''
- Fix: Mix/Max Runtime function 32bit float chroma: return -0.5..0.5 range (was: 0..1 range)
- AviSynth+ enhancements by Nekopanda (Neo fork)
  - Allow multiple prefetchers (MT) (mentioned earlier)
  - Multithreading and deadlock fixes for ScriptClip
    (originally I intended to pull only Neo changes which were fixing an old AVS+ bug,
     namely ScriptClip and multithreading. But I was not able to do that without pulling
     nearly everything from Neo)
  - Caching enhancements.
    SetCacheMode(0) or SetCacheMode(CACHE_FAST_START) start up time and size balanced mode
    SetCacheMode(1) or SetCacheMode(CACHE_OPTIMAL_SIZE) slow start up but optimal speed and cache size
    Latter can do wonders especially at really low memory environment
  - ScriptClip and variable stability in multithreading.
    Compared to Neo, I extended it a bit.
    Regarding visibility of variables: I had to change back from Neo's new default behaviour,
    which was similar to GRunT's local=true and yielded incompatibility with the past.
    So scripts written on the assumption of local = false did not work.
    Neo's (valid) point: they would not allow the following script to show the behavior to print "3".
    '''
      # prints 3 - Avisynth default but seems incorrect
      global foo=2
      function PrintFoo(clip c)
      { c.ScriptClip("Subtitle(string(foo))", local = false) }
      Version()
      PrintFoo()
      foo = 3
      last

      # prints 2 - Neo's default behaviour
      global foo=2
      function PrintFoo(clip c)
      { c.ScriptClip("Subtitle(string(foo))", local = true) }
      Version()
      PrintFoo()
      foo = 3
      last
    '''

    So all runtime filters* got a bool "local" parameter which acts same as in GRunT
    * ConditionalSelect, ConditionalFilter, ScriptClip, ConditionalReader, FrameEvaluate,
      WriteFile, WriteFileIf, WriteFileStart, WriteFileEnd

    If local=true the filter will evaluate its run-time script in a new variable scope,
    avoiding unintended sharing of variables between run-time scripts.
    (As of current state (20200328): Neo's WriteFileStart and WriteFileEnd works like
    local=false, in Avisynth compatible mode; all other runtime filters work in local=true mode in Neo)

    In our present Avisynth+ all runtime filters are compatible with the usual behaviour,
    but one can set the other mode by "local=true" parameter. This is when script is passed as string
    Except: when the script parameter is not a string but a function (yes! Nekopanda made it available!)
    then we in classic Avs! (haha, Classic+) kept Neo's new recommendation
    and the default value of "local" is true.

    (as a memo, ConditionalFilter still has the GRunT-like "shortcut" style parameter for bool results which implies "=" and "true"
     and parameter "condvarsuffix" for ConditionalReader)
  - UseVar, special filter, opens a clean variable environment in which only the
    variables in the parameter list can be seen.

  - "escaped" string constants: with e prefix right before the quotation mask
    n"Hello \n" will store actual LF (10) control character into the string
    \n \r \t \0 \a \f \\ and \" are converted
  - Introduce function objects into scripts
    Functions can appear as standard Avisynth variables and parameters (AVSValue type='n')
    https://github.com/nekopanda/AviSynthPlus/wiki/Language-New-Features
    Even with variable capture [] (like in GRuntT args)

  - Filter graph. Switch it on by putting SetGraphAnalysis(true) at the beginning of the script.
    Dump to text file with DumpFilterGraph. E.g. DumpFilterGraph("graph.txt", mode=2)
    Output is in "dot" format, it can be converted to an image with Graphviz as follows:
    '''
    dot -Tsvg graph.txt -o graph.svg
    '''

    SetGraphAnalysis (bool)

      Enables (True) or disables (False) graph node insertion into the instantiated filter.
      To output a filter graph, a graph node must be inserted in the filter.
      When a graph node is inserted, performance may decrease slightly due to the increase of internal function calls.
      (In most cases, there is no observable performance degradation.)

    DumpFilterGraph (clip, string "outfile", int "mode", int "nframes", bool "repeat")

      Outputs a filter graph.

      clip
          Clip to output filter graph
      string outfile = ""
          Output file path
      int mode = 0 *
      int nframes = -1
          Outputs the filter graph when processing the specified frame. The cache size and memory usage of each filter at that time are output together. This is effective when you want to know the memory usage of each filter. If -1, output when DumpFilterGraph is called (before the frame is processed).
      bool repeat = false
          Valid only when nframes> 0. Outputs a filter graph repeatedly at nframes intervals.

  - Frame properties (still from Neo!)
    (experimental, we have planned it in Avs+, probably we'll try to follow the VapourSynth methods(?))

20200322 3.6.0
--------------
- Fix: Multithreading enhancements and fixes (Nekopanda, from Neo fork)
  - Fix old ScriptClip (runtime filters) issue
    In this example "current_frame" variable was not seen by YDifferenceFromPrevious scripted within SubTitle
    resulting in "ERROR: Plane Difference: This filter can only be used within run-time filters" message
    Now this script finally works:
      SetLogParams("log.txt", LOG_DEBUG)
      ColorBars(width=640, height=480, pixel_type="yv12")
      ScriptClip(last, "Subtitle(String(YDifferenceFromPrevious))")
      Prefetch(4)
  - Fix deadlock of ScriptClip on MT
  - MT improvement
    - Allow multiple Prefetchers
    - Add argument to Prefetch to change # of prefetch frames without changing # of threads

    Prefetch (clip c, int threads, int "frames")

    In the original Plus, you could use only one Prefetch, but you can use any number of CUDA versions.
    Also, an argument has been added to specify the number of frames to prefetch.
    Prefetch (1,4) # Make 1 thread stand and prefetch 4 frames
    By doing so, flexible parallelization configuration is possible, such as pipeline parallelization.

    threads
    Number of threads. If it is 0, it passes without doing anything.

    frames
    Number of frames to prefetch.
    Again, if it is 0, it passes without doing anything.
- Fix: BuildPixelType: chroma subsampling of sample clip was ignored.
- POSIX: better behaviour under non-Windows because of having multiple sized fixed fonts, not only a single size=20 one.
  e.g. MessageClip(), Info(), Version(), ColorYUV "show", internal ApplyMessage
- Text filter:
  - font types with
    - "Terminus" fixed fonts added (12-14-16-18-20-22-24-28-32, regular + bold)
    - "Info_h" good old 10x20 fixed font kept under this name
  - much more international unicode characters (1354), use utf8=true under Windows
  - use fontname parameter (default "Terminus", other choice is "info_h")
  - use font_filename parameter (accepts BDF fonts at the moment - import is probably not too smart but worked for Terminus)
  - use size parameter (12 to 32, if no size is available, a smaller one is chosen but at least the smallest one)
  - new parameter: bold (default false)
- Info() filter: when parameter "size" < 0, font is automatically enlarged over 640x480
  (POSIX limit: minimum size is 12, maximum size is 32 - limited by available fixed fonts)")
- SIL OPEN FONT LICENSE added because of usage of Terminus fonts)
- able to build w/o GDI and font rendering engine under Windows, so that text-overlay filters
  work like in POSIX version of AviSynth+ (mainly for my development test)
  Use with NO_WIN_GDI define.
- Fix: ReplaceStr when the pattern string to be replaced is empty
- New:
  Exist() to have bool utf8 parameter
  This is another function to have utf8 option:
  Usage: b = Exist("Здравствуй.mkv",utf8=true). Avs file is saved as utf8 w/o BOM
- Fix: broken Exist for directories (regression appeared in 3.5.0)
- Fix: ColorYUV: really disable variable search when parameter "conditional" is false
- Development:
  ScriptEnvironment::VSprintf: parameter (void *) is changed back to va_list.
  May affect C interface (avs_vsprintf) and CPP interface (ScriptEnvironment::VSprintf)
- Enhanced: Planar RGB to YUV 444 10-14 bits: more precision (32 bit float internally)
- Enhanced: Planar RGB to YUV 444 10-16 bits: AVX2 (speed improvement)

20200302 3.5.0
--------------
- New: Native Linux, macOS, and BSD support.
- Fix: ConvertBits 32->8 for extremely out of range float pixel values.
  When pixel value in a 32 bit float format video was way out of range and greater than 128 (e.g. instead of 0 to 1.0 for Y plane) then the ConvertBits(8) had artifacts.
- Fix potential crash on exit or cache shrink (linux/gcc only?)
- Layer: support RGB24 and RGB48 (internally processed as Planar RGB - lossless pre and post conversion)
- Fix: RGBP to 444 8-14bit right side artifacts at specific widths
- Fix: "scalef" and "scaleb" for 32 bit input, when scale_inputs="floatf" produced wrong result
- Fix: missing rounder in V channel calculation of PlanarRGB->YUV 8-14bits SSE2 code
- Overlay: show error when Overlay is fed with clips with different bit depths
- New function:
  bool IsVersionOrGreater(int majorVersion, int minorVersion [,int bugfixVersion]) function
  Returns true if the current version is equal or greater than the required one in the parameters.
  The function is checking the current version against the given parameters (similar to a Windows API function)
  e.g. IsVersionOrGreater(3, 4) or IsVersionOrGreater(3, 5, 8)
- New: "Expr" helpers:
  - Constants "yrange_min", "yrange_half", "yrange_max"
    Unlike the luma/chroma plane adaptive "range_min", "range_half", "range_max" these constants always report the luma (Y) values
  - new parameter: bool clamp_float_UV (default false)
    this parameter affects clamping of chroma planes: chroma is clamped between 0..1.0 instead of -0.5..0.5s
  - "clamp_float" is not ignored (and set to true) when parameter "scale_inputs" auto-scales 32 bit float type pixels
  - new "yscalef" and "yscaleb" keywords similar to "scalef" and "scaleb" but scaling is forced to use rules for Y (non-UV) planes
  - new allowed value "floatUV" for scale_inputs.
    In short: chroma pre-shift by 0.5 for 32 bit float pixels
    Affects the chroma plane expressions of 32 bit float formats.
    Shifts the input up by 0.5 before processing it in the expression, thus values from -0.5..0.5 (zero centered) range are converted to the 0..1 (0.5 centered) one.
    Since the expression result internally has 0..1.0 range, this then is shifted back to the original -0.5..0.5 range. (since 2.2.20)
    Note: predefined constants such as cmin, cmax, range_min, range_max and range_half will be shifted as well, e.g. the expression will see range_half = 0.5
  - These modifications are similar to Masktools2 2.2.20+
- Fix: TemporalSoften possible access violation after SeparateFields (in general: after filters that only change frame pitch)
- Fix: Shibatch.DLL Access Violation crash when exit when target rate is the same as vi.audio_samples_per_second or audio_samples_per_second is 0
- Build system: Cmake: use platform toolset "ClangCL" for using built-in Clang support of Visual Studio.
  Since VS2019 v16.4 there is LLVM 9.0 support.
  Use: Tools|Get Tools and Features|Add Individual Components|Compilers, build tools, and runtimes
       [X] C++ Clang compiler for Windows
       [X] C++ Clang-cl for v142 build tools (x64/x86)
  (for LLVM Clang installed separately, please use llvm or LLVM - no change in its usage)
- New: AddBorders, LetterBox: new color_yuv parameter like in BlankClip
- Fix: Resizers to really resize alpha channel (YUVA, RGBPA)
- Fix: crash when outputting VfW (e.g. VirtualDub) for YUV444P16, other fixes for r210 and R10k formats
- WavSource: really use "utf8" parameter, fix some debug asserts
- TimeStrech: pass internal errors as Avisynth exception text (e.g. proper "Excessive sample rate!" instead of "unhandled C++ error")

20191021 3.4.0
--------------

- Merges in the MT branch, the current state of pinterf/MT, and packaging fixes
- Development HEAD is the master repo again in https://github.com/AviSynth/AviSynthPlus
- Bumps version to 3.4

20190829 r2915
--------------
- Changed: Trim, FreezeFrame, DeleteFrame, DuplicateFrame, Reverse and Loop are using frame cache again (similar to classic Avs 2.6)
- Enhanced: Expr: faster exp, log, pow for AVX2 (sekrit-twc)
- ConditionalReader: allow empty value in text file when TYPE string
- Fix: Expr: fix non-mod-8 issues for forced RGB output and YUV inputs
- New: AviSource support v308 and v408 format (packed 8 bit 444 and 4444)
- Fix: AviSource v410 source garbage (YUV444P10)
- Fix: Expr: when using parameter "scale_inputs" and the source bit depth conversion occured, predefined constants
  (ymin/max, cmin/max, range_min/max/half) would not follow the new bit depth
- Fix: ConvertToRGB from 32bit float YUV w/ full scale matrixes (pc.601, pc.709, average)
- Fix: FlipHorizontal RGB48/64 artifacts
- Enhanced: a bit quicker FlipHorizontal
- Fix: RGB64 Blur leftmost column artifact
- Enhanced: quicker RGB24/48 to Planar RGB for AVX2 capable processors
- Fix: Strip alpha channel when origin is YUVA and using ConvertToYV12/ConvertToYV16/ConvertToYV24 funtions
- Fix: garbage with ConvertToYUY2 from 8 bit YUVA colorspaces
- Enhanced: Colorbars to accept RGB24, RGB48, YV411 and 4:2:2 formats for pixel_type (now all colorspaces are supported)
- Fix: shifted chroma in ColorBars and ColorBarsHD for YUV444PS
- Fix: ConvertToY8, ConvertToYV12, ConvertToYV16, ConvertToYV24 are now allowed only for 8 bit inputs.
  Formerly these functions were allowed for 10+ bit colorspaces but were not converted to real 8 bit Y8/YV12/16/24.
  Use ConvertToY, ConvertToYUV420, ConvertToYUV422, ConvertToYUV444 instead which are bit depth independent
- New parameter in ColorYUV, RGBAdjust, Overlay, ConditionalReader: string "condvarsuffix"
  Allows multiple filter instances to use differently named conditional parameters.
- Fix: ColorBars: pixel_type planar RGB will set alpha to 0 instead of 255, consistent with RGB32 Alpha channel
- Fix: text colors for YUV422PS - regression since r2728 (zero-centered chroma)
- New: VirtualDub2 to display 8 bit planar RGB (needs up-to-date VirtualDub2 as well)
  VfW interface to negotiate 8 bit Planar RGB(A) with FourCCs: G3[0][8] and G4[0][8], similar to the 10-16 bit logic
- Fix: planar RGBA Alpha on VfW was uninitialized because it wasn't filled.
- Layer: big update
  - Support for all 8-32 bit Y and planar YUV/YUVA and planar RGB/RGBA formats
  - New parameter: float "opacity" (0.0 .. 1.0) optionally replaces the previous "level". Similar to "opacity" in "Overlay"
  - threshold parameter (used for lighten/darken) is autoscaled. Keep it between 0 and 255, same as it was used for 8 bit videos.
  - new parameter: string "placement" default "mpeg2". Possible values: "mpeg2" (default), "mpeg1".
    Used in "mul", "darken" and "lighten", "add" and "subtract" modes with planar YUV 4:2:0 or 4:2:2 color spaces (not available for YUY2)
    in order to properly apply luma/overlay mask on U and V chroma channels.
  - Fix some out-of-frame memory access in YUY2 C code
  - Fix: Add proper rounding for add/subtract/lighten/darken calculations. (YUY2, RGB32, 8 bit YUV and 8 bit Planar RGB)
  - Fix: "lighten" and "darken" gave different results between yuy2 and rgb32 when Threshold<>0
    Fixed "darken" for RGB32 when Threshold<>0
    Fixed "lighten" and "darken" for YUY2 when Threshold<>0
- Avisynth C interface header (avisynth_c.h):
  - cosmetics: functions regrouped to mix less AVSC_API and AVSC_INLINE, put together Avisynth+ specific stuff
  - cosmetics: remove unused form of avs_get_rowsize and avs_get_height (kept earlier for reference)
  - use #ifndef AVSC_NO_DECLSPEC for AVSC_INLINE functions which are calling API functions
  - define alias AVS_FRAME_ALIGN as FRAME_ALIGN (keep the AVS_xxxx naming convention)
  - dynamic loader (avs_load_library) uses fallback mechanism for non-existant Avisynth+ specific functions, functions are usable for classic avisynth
- filter "Version": update year, removed avs-plus.net link
- Updated: TimeStretch plugin with SoundTouch 2.1.3 (as of 07.Jan 2019)
- Source/Build system
  - rst documentation update (qyot27) in distrib\docs\english\source\avisynthdoc\contributing\compiling_avsplus.rst
  - GCC-MinGW build, GCC 8.3 support
  - CMake: Visual Studio 2019 generator support
  - Clang (LLVM) support

20181220 r2772
--------------
- Fix: Expr: possible Expr x64 crash under specific memory circumstances
- Fix: Expr: safer code for internal variables "Store and pop from stack" (see: Internal variables at http://avisynth.nl/index.php/Expr)

20181218 r2768
--------------
- New: Expr: allow input clips to have more planes than an implicitely specified output format
  Expr(aYV12Clip, "x 255.0 /", format="Y32") # target is Y only which needs only Y plane from YV12 -> no error
- New: Expr: Y-plane-only clip(s) can be used as source planes when a non-subsampled (rgb or 444) output format implicitely specified
  Expr(Y, "x", "x 2.0 /", "x 3.0 /", format="RGBPS") # r, g and b expression uses Y plane
  Expr(Grey_r, Grey_g, Grey_b, "x", "y 2.0 /", "z 3.0 /", format="RGBPS") # r, g and b expression uses Y plane
- Fix: ConvertToYUY2() error message for non-8 bit sources.
- Fix: Y32 source to 32 bit 420,422,444 (introduced in big the zero-chroma-center transition)
- Fix: ShowY, ShowU, ShowV crash for YUV (non-YUVA) sources
- Speedup: ConvertToY12/16... for RGB or YUY2 sources where 4:4:4 or YV16 intermediate clip was used internally
  (~1.5-2x speed, was a regression in Avs+, use intermediate cache again)
- Fix: Allow ExtractY on greyscale clips
- ImageReader/ImageSource: use cache before FreezeFrame when result is a multiframe clip (fast again, regression since an early AVS+ version)
- Resizers: don't use crop at special edge cases to avoid inconsistent results across different parameters/color spaces
- Fix: Histogram 'classic': rare incomplete histogram shown in multithreading environment
- Fix: ImageReader and ImageWriter: if path is "" then it works from/to the current directory.
- GeneralConvolution: Allow 7x7 and 9x9 matrices (was: 3x3 and 5x5)
- GeneralConvolution: All 8-32 bit formats (was: RGB32 only): YUY2 is converted to/from YV16, RGB24/32/48/64 are treated as planar RGB internally
  Since 32 bit float input is now possible, matrix elements and bias parameter now is of float type.
  For 8-16 bit clips the matrix is converted to integer before use.
- GeneralConvolution: Allow chroma subsampled formats to have their luma _or_ chroma processed. E.g. set chroma=false for a YV12 input.
- GeneralConvolution: new parameters: boolean luma (true), boolean chroma(true), boolean alpha(true)
    Default: process all planes. For RGB: luma and chroma parameters are ignored.
    Unprocessed planes are copied. Using alpha=false makes RGB32 processing faster, usually A channel is not needed.
- GeneralConvolution: MT friendly parameter parsing
- New: UTF8 filename support in AviSource, AVIFileSource, WAVSource, OpenDMLSource and SegmentedAVISource
  All functions above have a new bool utf8 parameter. Default value is false.
- Experimental: new syntax element (by addewyd): assignment operator ":=" which returns the assigned value itself.
  (Assignment within an expression)

20180702 r2728
--------------
- Fix: Expr: expression string order for planar RGB is properly r-g-b like in original VapourSynth version, instead of counter-intuitive g-b-r.
- Fix: Expr: check subsampling when a different output pixel format is given
- Fix: ColorYUV: round to avoid green cast on consecutive TV<>PC
- Fix: RGBAdjust memory leak when used in ScriptClip
- Fix: RGB64 Turnleft/Turnright (which are also used in RGB64 Resizers)
- Fix: Rare crash in FrameRegistry
- Fix: couldn't see variables in avsi before plugin autoloads (colors_rgb.avsi issue)
- Fix: LoadVirtualdubPlugin: Fix crash on exit when more than one instances of a filter was used in a script
- New: Expr: implement 'clip' three operand operator like in masktools2
- New: Expr: Parameter "clamp_float" (like in masktools2 2.2.15)
- New: Expr: parameter "scale_inputs" (like in masktools2 2.2.15)
- New: function bool VarExist(String variable_name)
- New function: BuildPixelType:
  Creates a video format (pixel_type) string by giving a colorspace family, bit depth, optional chroma subsampling and/or a
  template clip, from which the undefined format elements are inherited.
- Enhanced: Limiter to work with 32 bit float clips
- Enhanced: Limiter new parameter bool 'autoscale' default false, parameters now are of float type to handle 32 bit float values.
- Enhanced: RGBAdjust new parameter: conditional (like in ColorYUV)
  The global variables "rgbadjust_xxx" with xxx = r, g, b, a, rb, gb, bb, ab, rg, gg, bg, ag are read each frame, and applied.
- Enhanced: RGBAdjust: support 32 bit float ('analyze' not supported, 'dither' silently ignored)
- Enhanced: AviSource to support much more formats with 10+ bit depth.
- Changed (finally): 32bit float YUV colorspaces: zero centered chroma channels.
  U and V channels are now -0.5..+0.5 (if converted to full scale before) instead of 0..1
- New function: bool IsFloatUvZeroBased() for plugin or script writers who want to be compatible with pre r2672 Avisynth+ float YUV format:
- Enhanced: Allow ConvertToRGB24-32-48-64 functions for any source bit depths
- Enhanced: ConvertBits: allow fulls-fulld combinations when either clip is 32bits
  E.g. after a 8->32 bit fulls=false fulld=true:
  Y: 16..235 -> 0..1
  U/V: 16..240 -> -0.5..+0.5
  Note: now ConvertBits does not assume full range for YUV 32 bit float.
  Default values of fulls and fulld are now true only for RGB colorspaces.
- New: LoadVirtualdubPlugin update: Update from interface V6 to V20, and Filtermod version 6 (partial)
- Source: move to c++17, 'if constexpr' requires. Use Visual Studio 2017 (or GCC 7?). CMakeLists.txt changed.
- Source: C api: AVSC_EXPORT to dllexport in capi.h for avisynth_c_plugin_init
- Source: C api: avs_is_same_colorspace VideoInfo parameters to const
- Project struct: changelog to git.
- Include current avisynth header files and def/exp file in installer, when SDK is chosen

20180328 r2664
--------------
#Fix
YUY2 Sharpen overflow artifacts - e.g. Sharpen(0.6)
Levels: 32 bit float shift in luma
Merge sse2 for 10-14bits (regression)
AVX2 resizer possible access violation in extreme resizes (e.g. 600->20)
32bit float PlanarRGB<->YUV conversion matrix
VfW: fix b64a output for OPT_Enable_b64a=true

#Enhanced
VfW output P010 and P016 conversion to SSE2 (VfW output is used by VirtualDub for example)
ColorYUV: recalculate 8-16 bit LUT in GetFrame only when changed frame-by-frame (e.g. in autowhite)
ConvertBits 32->8 sse2/avx2 and 32->10..16 sse41/avx2 (8-15x speed)

20180302 r2636
--------------
#Fix
Blur/Sharpen crashed when YUY2.width<8, RGB32.width<4, RGB64.width<2
ColorYUV: don't apply TV range gamma for opt="coring" when explicit "PC->TV" is given
ColorbarsHD: 32bit float properly zero(0.5)-centered chroma

v2632 (20180301)
----------------
#Fix
Fix: IsInterleaved returned false for RGB48 and RGB64 (raffriff42)
Fix: SubTitle for Planar RGB/RGBA: wrong text colors (raffriff42)
Fix: Packed->Planar RGB conversion failed on SSE2-only computers (SSSE3 instruction used)
Fix: Resizers for 32 bit float rare random garbage on right pixels (simd code NaN issue)

#Enhanced
Blur, Sharpen
    AVX2 for 8-16 bit planar colorspaces (>1.35x speed on i7-7770)
    SSE2 for 32 bit float formats (>1.5x speed on i7-7770)
Completely rewritten 16bit and float resizers, much faster (and not only with AVX2)
8 bit resizers: AVX2 support
Speed up converting from RGB24/RGB48 to Planar RGB(A) - SSSE3, approx. doubled fps
Enhanced: VfW: exporting Y416 (YUV444P16) to SSE2.

#New/Modded
ConvertFPS supports 10-32 bits, planar RGB(A), YUV(A)
New script function: int BitSetCount(int[, int, int, ...])
Modded script function: Hex(int , int "width"=0), new "width" parameter
Modded script function: HexValue(String, "pos"=1) new pos parameter
Modded script function: ReplaceStr(String, String, String[, Boolean "sig"=false]) New parameter: sig for case insensitive search (Default false: exact search)
New script functions: TrimLeft, TrimRight, TrimAll for removing beginning/trailing whitespaces from a string.
New in ColorYUV: New parameter: bool f2c="false". When f2c=true, the function accepts the Tweak-like parameters for gain, gamma and contrast
New/Fixed in ColorYUV: Parameter "levels" accepts "TV". (can be "TV->PC", "PC->TV", "PC->TV.Y")
New: Now gamma calculation is TV-range aware when either levels is "TV->PC" or coring = true or levels is "TV"
New in ColorYUV:  32 bit float support.
ColorYUV: can specify bits=32 when showyuv=true -> test clip in YUV420PS format
Modded: remove obsolate "scale" parameter from ConvertBits.
Internal: 8-16 bit YUV chroma to 32 bit float: keep middle chroma level (e.g. 128 in 8 bits) at 0.5. Calculate chroma as (x-128)/255.0 + 0.5 and not x/255.0
(Note: 32 bit float chroma center will be 0.0 in the future)
New: Histogram parameter "keepsource"=true (raffriff42) for "classic", "levels" and "color", "color2"
New: Histogram type "color" to accept 8-32bit input and "bits"=8,9,..12 display range
New: Histogram parameter "markers"=true. Markers = false disables extra markers/coloring for "classic" and "levels"

v2580 (20171226)
----------------
# Fix
Fix (workaround): Merge: Visual Studio 2017 15.5.1/2 generated invalid AVX2 code (x86 crashed)
Fix: Temporalsoften 10-14 bits: an SSE 4.1 instruction was used for SSE2-only CPU-s (Illegal Instruction on Athlon XP)

v2574 (20171219)
----------------
# Fix
Fix: MaskHS created inverse mask. Regression after r2173
Fix: jitasm code generation at specific circumstances in Expr filter

# Build
Build: changed avisynth.h, strict C++ conformity with Visual Studio 2017 /permissive- flag

# Other
Installer in two flavours: simple or full (with Microsoft Visual C++ Redistributables)

# New
Expr tweaks:
  - Indexable source clip pixels by relative x,y positions.
  Syntax: x[a,b] where
  'x': source clip letter a..z
  'a': horizontal shift. -width < a < width
  'b': vertical shift. -height < b < height
  'a' and 'b' should be constant. e.g.: "x[-1,-1] x[-1,0] x[-1,1] y[0,-10] + + + 4 /"
  When requested pixels come from off-screen the off-screen values are cloned from the appropriate top-bottom-left-right edge.
  Optimized version requires SSSE3 (and no AVX2 version is available). On non-SSSE3 CPUs falls back to C.

  - sin cos tan asin acos atan (no SSE2/AVX2 optimization, when they appear in Expr a slower C code runs the expression)

  - % (modulo). result = x - trunc(x/d)*d.
  Note: internally everything is calculated as a 32 bit float.
  A float can only hold a 24 bit integer number, don't expect 32 bit accuracy here.

  - Variables: uppercase letters A..Z for storing and reuse temporary results, frequently used computations.
  Store, result can still be used from stack: A@ .. Z@
  Store and remove from stack: A^ .. Z^
  Use: A..Z
  Example: "x y - A^ x y 0.5 + + B^ A B / C@ x +"

  - 'frameno' : use current frame number in expression. 0 <= frameno < clip_frame_count

  - 'time' : calculation: time = frameno/clip_frame_count. Use relative time position in expression. 0 <= time < frameno/clip_frame_count

  - 'width', 'height': currently processed plane width and height

v2542-2544 (20171114,20171115), changes since v2508
-------------------------------------
# New
  Expr filter

  Syntax ("c+s+[format]s[optAvx2]b[optSingleMode]b[optSSE2]b")
  clip Expr(clip c[,clip c2, ...], string expr [, string expr2[, string expr3[, string expr4]]] [, string format]
      [, bool optSSE2][, bool optAVX2][, bool optSingleMode])

  Clip and Expr parameters are unnamed
  'format' overrides the output video format
  'optSSE2' to disable simd optimizations (use C code)
  'optAVX2' to disable AVX2 optimizations (use SSE2 code)
  'optSingleMode' default false, to generate simd instructions for one XMM/YMM wide data instead of two. Experimental.
     One simd cycle processes 8 pixels (SSE2) or 16 pixels (AVX2) at a time by using two XMM/YMM registers as working set.
     Very-very complex expressions would use too many XMM/YMM registers which are then "swapped" to memory slots, that can be slow.
     Using optSingleMode = true may result in using less registers with no need for swapping them to memory slots.

  Expr accepts 1 to 26 clips as inputs and up to four expression strings, an optional video format overrider, and some debug parameters.
  Output video format is inherited from the first clip, when no format override.
  All clips have to match their dimensions and plane subsamplings.

  Expressions are evaluated on each plane, Y, U, V (and A) or R, G, B (,A).
  When an expression string is not specified, the previous expression is used for that plane. Except for plane A (alpha) which is copied by default.
  When an expression is an empty string ("") then the relevant plane will be copied (if the output clip bit depth is similar).
  When an expression is a single clip reference letter ("x") and the source/target bit depth is similar, then the relevant plane will be copied.
  When an expression is constant, then the relevant plane will be filled with an optimized memory fill method.
  Expressions are written in Reverse Polish Notation (RPN).

  Expressions use 32 bit float precision internally

  For 8..16 bit formats output is rounded and clamped from the internal 32 bit float representation to valid 8, 10, ... 16 bits range.
  32 bit float output is not clamped at all.

  - Clips: letters x, y, z, a, ... w. x is the first clip parameter, y is the second one, etc.
  - Math: * / + -
  - Math constant: pi
  - Functions: min, max, sqrt, abs, neg, exp, log, pow ^ (synonyms: "pow" and "^")
  - Logical: > < = >= <= and or xor not == & | != (synonyms: "==" and "=", "&" and "and", "|" and "or")
  - Ternary operator: ?
  - Duplicate stack: dup, dupN (dup1, dup2, ...)
  - Swap stack elements: swap, swapN (swap1, swap2, ...)
  - Scale by bit shift: scaleb (operand is treated as being a number in 8 bit range unless i8..i16 or f32 is specified)

  - Scale by full scale stretch: scalef (operand is treated as being a number in 8 bit range unless i8..i16 or f32 is specified)

  - Bit-depth aware constants
    ymin, ymax (ymin_a .. ymin_z for individual clips) - the usual luma limits (16..235 or scaled equivalents)

    cmin, cmax (cmin_a .. cmin_z) - chroma limits (16..240 or scaled equivalents)

    range_half (range_half_a .. range_half_z) - half of the range, (128 or scaled equivalents)

    range_size, range_half, range_max (range_size_a .. range_size_z , etc..)

  - Keywords for modifying base bit depth for scaleb and scalef: i8, i10, i12, i14, i16, f32

  - Spatial input variables in expr syntax:
    sx, sy (absolute x and y coordinates, 0 to width-1 and 0 to height-1)
    sxr, syr (relative x and y coordinates, from 0 to 1.0)

  Additions and differences to VS r39 version:

  ------------------------------
--------------
  (similar features to the masktools mt_lut family syntax)

  Aliases:

    introduced "^", "==", "&", "|"
  New operator: != (not equal)

  Built-in constants

    ymin, ymax (ymin_a .. ymin_z for individual clips) - the usual luma limits (16..235 or scaled equivalents)

    cmin, cmax (cmin_a .. cmin_z) - chroma limits (16..240 or scaled equivalents)

    range_half (range_half_a .. range_half_z) - half of the range, (128 or scaled equivalents)

    range_size, range_half, range_max (range_size_a .. range_size_z , etc..)

  Autoscale helper functions (operand is treated as being a number in 8 bit range unless i8..i16 or f32 is specified)

    scaleb (scale by bit shift - mul or div by 2, 4, 6, 8...)

    scalef (scale by stretch full scale - mul or div by source_max/target_max

  Keywords for modifying base bit depth for scaleb and scalef
: i8, i10, i12, i14, i16, f32

  Built-in math constant

    pi

  Alpha plane handling
. When no separate expression is supplied for alpha, plane is copied instead of reusing last expression parameter.
  Proper clamping when storing 10, 12 or 14 bit outputs

  (Faster storing of results for 8 and 10-16 bit outputs, fixed in VS r40)
  16 pixels/cycle instead of 8 when avx2, with fallback to 8-pixel case on the right edge. Thus no need for 64 byte alignment for 32 bit float.
  (Load zeros for nonvisible pixels, when simd block size goes beyond image width, to prevent garbage input for simd calculation)


  Optimizations: x^0.5 is sqrt, ^1 +0 -0 *1 /1 to nothing, ^2, ^3, ^4 is done by faster and more precise multiplication
  Spatial input variables in expr syntax:
    sx, sy (absolute x and y coordinates, 0 to width-1 and 0 to height-1)
    sxr, syr (relative x and y coordinates, from 0 to 1.0)
  Optimize: recognize constant plane expression: use fast memset instead of generic simd process. Approx. 3-4x (32 bits) to 10-12x (8 bits) speedup
  Optimize: Recognize single clip letter in expression: use fast plane copy (BitBlt)
    (e.g. for 8-16 bits: instead of load-convert_to_float-clamp-convert_to_int-store). Approx. 1.4x (32 bits), 3x (16 bits), 8-9x (8 bits) speedup

  Optimize: do not call GetFrame for input clips that are not referenced or plane-copied

  Recognize constant expression: use fast memset instead of generic simd process. Approx. 3-4x (32 bits) to 10-12x (8 bits) speedup
    Example: Expr(clip,"128","128,"128")

  Differences from masktools 2.2.10
  --------------------------------
-
  Up to 26 clips are allowed (x,y,z,a,b,...w). Masktools handles only up to 4 clips with its mt_lut, my_lutxy, mt_lutxyz, mt_lutxyza

  Clips with different bit depths are allowed

  Works with 32 bit floats instead of 64 bit double internally

  Less functions (e.g. no bit shifts)

  No float clamping and float-to-8bit-and-back load/store autoscale magic

  Logical 'false' is 0 instead of -1

  The ymin, ymax, etc built-in constants can have a _X suffix, where X is the corresponding clip designator letter. E.g. cmax_z, range_half_x

  mt_lutspa-like functionality is available through "sx", "sy", "sxr", "syr"

  No y= u= v= parameters with negative values for filling plane with constant value, constant expressions are changed into optimized "fill" mode

  Sample:

  Average three clips:
  c = Expr(clip1, clip2, clip3, "x y + z + 3 /")
  using spatial feature:
  c = Expr(clip1, clip2, clip3, "sxr syr 1 sxr - 1 syr - * * * 4096 scaleb *", "", "")

# Additions
  - Levels: 32 bit float format support

# Fixes
  - RGB (full scale) conversion: 10-16 bits to 8 bits rounding issue; pic got darker in repeated 16<->8 bit conversion chain
  - ConvertToY: remove unnecessary clamp for Planar RGB 32 bit float
  - RGB ConvertToY when rec601, rec709 (limited range) matrix. Regression since r2266

# Optimizations
  - Faster RGB (full scale) 10-16 bits to 8 bits conversion when dithering

# Other
  - Default frame alignment is 64 bytes (was: 32 bytes). (independently of AVX512 support)

# Source and build things.
  - Built with Visual Studio 2017, v141_xp toolset

    Download Visual Studio 2017 Redistributable from here (replaces and compatible with VS2015 redist)

    X64:
    https://go.microsoft.com/fwlink/?LinkId=746572

    x86:
    https://go.microsoft.com/fwlink/?LinkId=746571

  - Not for live yet - experimental - use of size_t in video frame fields

    You can now build avs+ with optional define: SIZETMOD. Type of offset-related video frame fields and return values changed from int to size_t.
    Affects x64 where size_t is 8 bytes while int is 4 bytes.
    With Avisynth+ built with this SIZETMOD option, plugins or apps that are using C interface may be broken, when they access AVS_VideoFrameBuffer or AVS_VideoFrame fields directly.

    Known plugins/applications that are broken (as of 20171114): x264_64.exe
    Possible problems come from such as using field "pitch" in AVS_VideoFrame directly instead of calling avs_get_pitch_p through the interface.
    For the future: please take the warnings "DO NOT USE THIS STRUCTURE DIRECTLY" seriously.

    Plugin writers, Who are using the usual (non-C) interface, will probably see a change as frame->GetOffset which returns size_t instead of int. Nevertheless the content
    will possibly never exceed old 32 bit limit.

    Another broken x64 plugin for SIZETMOD version: LSmashSource (as of 20171114). Possible reason: it uses CPP 2.5 (baked code) interface.
    (How much CPP 2.5 plugins exist still in 2017?)

  - avisynth_c.h (C interface header file) changed:
    Optional define SIZETMOD. Experimental. Offsets are size_t instead of int (x64 is different!)
    Fix: avs_get_row_size calls into avs_get_row_size_p, instead of direct field access
    Fix: avs_get_height calls into avs_get_row_size_p, instead of direct field access.

v2508 (20170629), changes since v2506
-------------------------------------
# Fixes
  Fix TemporalSoften: threshold < 255

v2506 (20170608), changes since v2504
-------------------------------------
# Fixes
  Fix CombinePlanes: feeding YV16 or YV411 target with Y8 sources

v2504 (20170603), changes since v2502
-------------------------------------
# Fixes
  Fix broken XP support

v2502 (20170602), changes since v2489
-------------------------------------
# Fixes
  - (Important!) MT_SERIALIZED mode did not always protect filters (regression since r2069)
    Such filters sometimes were called in a reentrant way (like being MT_NICE_FILTER), which
    possibly resulted in using their internal buffers parallel.
  - ImageWriter crash when no '.' in provided filename
  - Overlay: correct masked blend: keep exact clip1 or clip2 pixel values for mask extremes 255 or 0.
    Previously 0 became 1 for zero mask, similarly 255 changed into 254 for full transparency (255) mask

# other modification, additions
  - New script functions: StrToUtf8(), StrFromUtf8(): Converting a 8 bit (Ansi) string to UTF8 and back.
  - PluginManager always throws error on finding wrong bitness DLL in the autoload directories
  - increased x64 default MemoryMax from 1GB to 4GB, but physicalRAM/4 is still limiting
  - allow conversions between RGB24/32/48/64 (8<->16 bits) w/o ConvertBits
  - Added VS2017 and v141_xp to CMakeList.txt

v2489 (20170529), changes since v2487
-------------------------------------
# Fixes
  - Memory leak in CAVIStreamSynth (e.g. feeding vdub)
  - ConvertToY for RGB64 and RGB48

v2487 (20170528), changes since v2455
-------------------------------------
# Fixes
  - Blur width=16 (YV12 width=32)
  - Overlay Lighten: artifacts when base clip and overlay clip have different widths (regression since r2290)
  - YUY2 HorizontalReduceBy2 did nothing if target width was not mod4

# optimizations
  - Blur, Sharpen 10-16 bits planar and RGB64: SSE2/SSE4 (2x-4x speed)

# other modification, additions
  - New script function: int GetProcessInfo([int type = 0])
    Without parameter or type==0 the current bitness of Avisynth DLL is returned (32 or 64)
    With type=1 the function can return a bit more detailed info:
    -1: error, can't establish
    0: 32 bit DLL on 32 bit OS
    1: 32 bit DLL on 64 bit OS (WoW64 process)
    2: 64 bit DLL
  - ImageReader: 16 bit support; "pixel_type" parameter new formats "RGB48", "RGB64" and "Y16"
  - ImageWriter: 16 bit support; save RGB48, RGB64, Y16, planar RGB(A) 8 and 16 bit formats
    (note: greyscale through devIL can be corrupt with some formats, use png)
  - ImageWriter: flip greyscale images vertically (except "raw" format)
  - SubTitle: new parameter "font_filename" allows using non-installed fonts
  - Allows opening unicode filenames through VfW interface (virtualdub, MPC-HC)
  - Script function Import: new parameter bool "utf8" to treat the filenames as UTF8 encoded
    (not the script text!)
  - SubTitle: new parameter bool "utf8" for drawing strings encoded in UTF8.
      Title="Cherry blossom "+CHR($E6)+CHR($A1)+CHR($9C)+CHR($E3)+CHR($81)+CHR($AE)+CHR($E8)+CHR($8A)+CHR($B1)
      SubTitle(Title,utf8=true)
  - New script functions: ScriptNameUtf8(), ScriptFileUtf8(), ScriptDirUtf8(),
    they return variables $ScriptNameUtf8$, $ScriptFileUtf8$ and $ScriptDirUtf8$ respectively

v2455 (20170316), changes since v2440
-------------------------------------
# Fixes
  IsY() script function returned IsY8() (VideoInfo::IsY was not affected)

# other modification, additions
  ConvertBits, dither=1 (Floyd-Steinberg): allow any dither_bits value between 0 and 8 (0=b/w)

v2440 (20170310), changes since v2420
-------------------------------------
# Fixes
  Merge for float formats
  error text formatting under wine (_vsnprintf_l issue)
  Regression: YUY2 UToY copied V instead of U, since August, 2016 (v2150)

# optimizations
  Merge: float to sse2 (both weighted and average)
  Ordered dither to 8bit: SSE2 (10x speed)

# other modification, additions
  - ColorBars allows any 4:2:0, 4:4:4 formats, RGB64 and all planar RGB formats
  - ColorBarsHD accepts any 4:4:4 formats
  - Dithering: Floyd-Steinberg
    Use convertBits parameter dither=1: Floyd-Steinberg (was: dither=0 for ordered dither)
  - Dithering: parameter "dither_bits"
    For dithering to lower bit depths than the target clip format
    Usage: ConvertBits(x, dither=n [, dither_bits=y])
    - ordered dither: dither_bits 2, 4, 6, ... but maximum difference between target bitdepth and dither_bits is 8
    - Floyd-Steinberg: dither_bits 1, 2, 4, 6, ... up to target bitdepth - 2
    (Avisynth+ low bitdepth, Windows 3.1 16 bit feeling I was astonished that dither_bits=6 still resulted in a quite usable image)
  - Dithering is allowed from 10-16 -> 10-16 bits (was: only 8 bit targets)
  - Dithering is allowed while keeping original bit-depth. clip10 = clip10.ConvertBits(10, dither=0, dither_bits=8)
    (you still cannot dither from 8 or 32 bit source)
  - ConditionalFilter syntax extension like Gavino's GConditional: no "=" "true" needed
  - Revert: don't give error for interlaced=true for non 4:2:0 sources (compatibility, YATTA)
  - CombinePlanes: silently autoconvert packed RGB/YUY2 inputs to planar
  - ConvertBits: show error message on YV411 conversion attempt: 8 bit only
  - ConvertBits: Don't give error message if dither=-1 (no dithering) is given for currently non-ditherable target formats
  - Script function: IsVideoFloat. returns True if clip format is 32 bit float. For convenience, same as BitsPerComponent()==32
  - ConvertToDoubleWidth and ConvertFromDoubleWidth: RGB24<->RGB48, RGB32<->RGB64
  - New MT mode: MT_SPECIAL_MT. Specify it for MP_Pipeline like filters, even if no Prefetch is used (MP_Pipeline issue, 2 fps instead of 20)

v2420 (20170202), changes since v2294
--------------------------------------
# Fixes
  - MinMax-type conditional functions (min, max, median): return float value for float clips
  - ScriptClip would show garbage text when internal exception occurs instead of the error message
  - DV chroma positioning (UV swapped), interlaced parameter check for 4:2:0
  - YUVA->PlanarRGBA and YUVA42x->444 missing alpha plane copy

# optimizations

  - TemporalSoften: much faster average mode (thres=255)
    radius=1 +70%, radius=2 +45%,
    16bit: generally 7-8x speed (SSE2/4 instead of C)
  - Difference-type conditional functions: Simd for 10-16 bits
  - RemoveAlphaPlane (subframe instead of BitBlt copy)
  - RGB48->RGB64 SSSE3 (1,6x), RGB64->RGB48 SSSE3 (1.5x speed)
  - RGB24,RGB48->PlanarRGB: uses RGB32/64 intermediate clip
  - Crop: Fast crop possible for frames with alpha plane (subframe)
  - YUV444->RGB48/64: fast intermediate PlanarRGB(A) then RGB48/64 (not C path)
  - RGB48/64->YUV4xx target: Planar RGB intermediate (instead of C, 10x faster)
  - Planar RGB <-> YUV: SSE2 (SSE4)
  - Planar RGB <-> Packed RGB32/64: SSE2
  - Merge, MergeChroma, MergeLuma: AVX2 (planar formats)
  - Possibly a bit faster text overlay
  - Overlay: 10-16bit SSE2/SSE4 for 420/422<->444 conversions
  - Overlay: blend: SSE4 for 10-16 bits, SSE2 for float
  - ConvertBits for planar RGB(A): SSE2/SSE4 for 10-16 bit <-> 10-16 bit Planar RGB (and Alpha plane) full scale conversions
    (also used internally for automatic planar RGB -> packed RGB VfW output conversions)

# other modification, additions

  - TemporalSoften: Planar RGB support
  - SeparateColumns: 10-16bit,float,RGB48/64
  - WeaveColumns: 10-16bit,float,RGB48/64,PlanarRGB(A)
  - SwapUV: YUVA support
  - ConvertToRGB32/64: copy alpha from YUVA
  - SeparateRows,SeparateFields: PlanarRGB(A),YUVA support
  - WeaveRows: PlanarRGB(A), YUVA
  - Weave (fields,frames): YUVA,PlanarRGB(A)
  - ConvertToPlanarRGB(A):
    PlanarRGB <-> PlanarRGBA is now allowed
  - ConvertToPlanarRGB(A):
    YUY2 source is now allowed (through automatic ConvertToRGB proxy)
  - New CPU feature constants (cpuid.h and avisynth_c.h)
    Detect FMA4 and AVX512F,DQ,PF,ER,CD,BW,VL,IFMA,VBMI
    See: https://github.com/pinterf/AviSynthPlus/blob/MT/avs_core/include/avs/cpuid.h
    Constants for AVX/AVX2/FMA3/F16C... were already defined in previous releases
    Note that for live AVX and AVX2 proper OS support is needed:
      64 bit OS (Avisynth+ can be x86 or x64)
      Windows7 SP1 or newer
    On a non-supported OS AVX or better processor features are not even reported to plugins through GetCPUInfo
  - BitBlt in 32 bit Avisynth:
    for processors with AVX or better ignore tricky isse memcpy replacement, trust in memcpy (test)
    (x64 is O.K., it always used memcpy)
  - VDubFilter.dll:
    Avisynth allows using filters written for Virtualdub (? version) with similar parameters that Avisynth uses
    New: allow parameters 'd' and 'l' types
    Avisynth+ converts 'd' double and 'l' long typed parameters to 'f' float and 'i' int as
    such types are non-existant in Avisynth
  - Internals: add SubframePlanarA to IScriptEnvirontment2 for frames with alpha plane
    General note: unlike IScriptEnvironment (that is rock solid for the time beeing), IScriptEnvironment2 is still not final.
    It is used within Avisynth+ core, but is also published in avisynth.h.
    It contains avs+ specific functions, that could not be stuffed into IScriptEnvironment without killing compatibility.
    Although it changes rarely, your plugin may not work with Avisynth+ versions after a change

#################
# VfW interface for high bit depth colorspaces
Output high bit depth formats on VfW interface.

One example is Virtualdub Deep Color mod, forum: https://forum.doom9.org/showthread.php?p=1795019
that can accept high bit depth avisynth plus script natively.

Non-existent formats will automatically converted to an existing one:
- 12, 14 and float YUV formats to 16 bit for 4:2:0 and 4:2:2
  Note: OPT_Enable_Y3_10_16 is still a valid override option
- 10, 12, 14 and float YUV formats to 16 bit for 4:4:4 (e.g. YUV444P10->YUV444P16)
  if alpha channel is present (YUVA), it will we copied, else filled with FFFF
- 10, 12, 14 and float planar RGB formats to RGB64
  Conversion of 8 bit planar RGB formats to RGB24
  Conversion of 8 bit planar RGBA formats to RGB32
  Note: Planar RGB autoconvert is triggered when global Avisynth variable Enable_PlanarToPackedRGB is true

Note:
    use OPT_VDubPlanarHack=true for YV16 and YV24 for older versions of VirtualDub
    when you experience swapped U and V planes

Supported formats:
    BRA[64],b64a,BGR[48],P010,P016,P210,P216,Y3[10][10],Y3[10][16],v210,Y416
    G3[0][10], G4[0][10], G3[0][12], G4[0][12], G3[0][14], G4[0][14], G3[0][16], G4[0][16]

Default format FourCCs:
    Avisynth+ will report these FourCC-s, override them with defining OPT_xxx global variables

    RGB64: BRA[64]
    RGB48: BGR[48]
    YUV420P10: P010
    YUV420P16: P016
    YUV422P10: P210
    YUV422P16: P216
    YUV444P16 and YUVA444P16: Y416
    Planar RGB  10,12,14,16 bits: G3[0][10], G3[0][12], G3[0][14], G3[0][16]
    Planar RGBA 10,12,14,16 bits: G4[0][10], G4[0][12], G4[0][14], G4[0][16]

Global variables to override default formats:
    Put them at the beginning of avs script.

    OPT_Enable_V210 = true --> v210 for YUV422P10
    OPT_Enable_Y3_10_10 = true --> Y3[10][10] for YUV422P10
    OPT_Enable_Y3_10_16 = true --> Y3[10][16] for YUV422P16
    OPT_Enable_b64a = true --> b64a for RGB64
    Enable_PlanarToPackedRGB = true --> RGBP8->RGB24, RGBAP8->RGB32, all other bit depths to RGB64

# Functions, Filters
[Overlay]
  - Conversionless "blend"/"luma"/"chroma" for Y, 4:2:0, 4:2:2, 4:4:4 and RGB clips
    Input clips are not converted automatically to 4:4:4 (YV24 for 8 bits) then back
    If you want to keep conversion, specify parameter use444=true
    Packed RGB (RGB24/32/48/64) are losslessly converted to and from planar RGB internally.
    That also means that RGB source clip is not converted to YUV for processing anymore.

    In this mode Overlay/mask automatically follows input clip format.
    For compatibility: when greymask=true (default) and mask is RGB then mask source is the B channel
  - blend for float formats

[Histogram]
    "levels" mode for RGB (Planar and RGB24/32/48/64 input)
    Color legends show R, G and B channels instead of Y, U and V

    Reminder:
      Histogram "levels" and "Classic" allows bits=xx parameter, xx=8..12
      Original display is drawn for 8 bits wide resolution.
      Now you can draw with 9..12 bits precision. Get a wide monitor though :)

[ConvertBits]: new parameters, possibly for the future.
    bool fulls, bool fulld

    For YUV and greyscale clips the bit-depth conversion uses simple bit-shifts by default.
    YUV default is fulls=false

    RGB is converted as full-stretch (e.g. 0..255->0..65535)
    RGB default is fulls=true

    If fulld is not specified, it takes the value of fulls.
    Use case: override greyscale conversion to fullscale instead of bit-shifts

    Note 1: conversion from and to float is always full-scale
    Note 2: alpha plane is always treated as full scale
    Note 3: At the moment you cannot specify fulld to be different from fulls.

[AddAlphaPlane]

Full Syntax:
    AddAlphaPlane(clip c, [int mask]) or
    AddAlphaPlane(clip c, [clip mask])

Adds and/or sets alpha plane
    YUV->YUVA, RGBP -> RGBAP, RGB24->RGB32, RGB48->RGB64
    If the current video format already has alpha channel and mask is provided, then the alpha plane will be overwritten with the new mask value.

Initialize alpha plane with value
    If optional mask parameter is supplied, the alpha plane is set to this value.
    Default: maximum pixel value of current bit depth (255/1023/4095/16383/65535/1.0)

Initialize alpha plane from clip
    Function can accept initializer clip with Y-only or alpha (YUVA/PRGBA/RGB32/64) for alpha source
    New alpha plane is copied from the greyscale clip or from the alpha channel.

[RemoveAlphaPlane]
    Stripes alpha plane from clip
    YUVA->YUV, RGBAP->RGBP, RGB32->RGB24, RGB64->RGB48

[Info]
    Info display was made a bit more compact.
    Bit depth info moved after color space info
    Does not display pre-SSE2 CPU flags when at least AVX is available
    Display AVX512 flags in separate line (would be too long)

    Reminder: Info() now has more parameters than is classic Avisynth:
      Info(): c[font]s[size]f[text_color]i[halo_color]i
    Added font (default Courier New), size (default 18), text_color and halo_color parameters, similar to (but less than) e.g. in ShowFrameNumber.

[ReplaceString]
New script function:
    string ReplaceStr(string s, string pattern, string replacement)
    Function is case sensitive, parameters are unnamed

[NumComponents]
New script function:
    int NumComponents(clip)
    returns 1 for greyscale, 3 for YUVxxx, YUY2, planar RGB or RGB24/RGB48, 4 for YUVAxxx, Planar RGBA or RGB32/64
    Same as VideoInfo::NumComponents()

[HasAlpha]
New script function:
    bool HasAlpha(clip)
    returns true when clip is YUVA, Planar RGBA, or packed RGB32 or RGB64

[Plane extraction shortcuts]
New functions:
  ExtractR, ExtractG, ExtractB,
  ExtractY, ExtractU, ExtractV
  ExtractA

Functions extract the specified plane to a greyscale clip.

For Y,U and V the function they work the same way as UToY8 and VToY8 and ConvertToY/ConvertToY8 did.
All colorspaces are accepted.
Although packed RGB formats (RGB24/32/48/64) are not planar, they can be used.
They are converted to planar RGB(A) on-the-fly before plane extraction

[YToUV addition]
  YToUV accepts optional alpha clip after Y clip

  old: YToUV(clip clipU, clip clipV [, clip clipY ] )
  new: YToUV(clip clipU, clip clipV [, clip clipY [, clip clipA] ] )

  Example
    U = source.UToY8()
    V = source.VToY8()
    Y = source.ConvertToY()
    A = source.AddAlphaPlane(128).AToY8()
    # swaps V, U and A, Y
    YToUV(V,U,A,Y).Histogram("levels").Info().RemoveAlphaPlane()

[CombinePlanes]
New function for arbitrary mixing planes from up to four input clips.

  CombinePlanes(clip1 [,clip2, clip3, clip4], string planes [, string source_planes, string pixel_type, string sample_clip])

  Combines planes of source clip(s) into a target clip

  If sample_clip is given, target clip properties are copied from that clip
  If no sample_clip is provided, then clip1 provides the template for target clip
  An optional pixel_type string (e.g."YV24", "YUV420PS", "RGBP8") can override the base video format.

  If the source clip count is less than the given planes defined, then the last available clip is
  used as a source.

  Target planes are set to default RGBP(A)/YUV(A), when all input clips are greyscale and no source planes are given
  Decision is made by the target plane characters, if they are like R,G,B then target video format will be planar RGB
  Same logic applies for YUV.
    Example:
    Y1, Y2 and Y3 are greyscale clips
    Old, still valid: combineplanes(Y1, Y2, Y3, planes="RGB", source_planes="YYY", pixel_type="RGBP8")
    New:              combineplanes(Y1, Y2, Y3, planes="RGB") # result: Planar RGB

  string planes
    the target plane order (e.g. "YVU", "YYY", "RGB")
    missing target planes will be undefined in the target

  string source_planes (optional)
    the source plane order, defaulting to "YUVA" or "RGBA" depending on the video format

  Examples#1
    #combine greyscale clips into YUVA clip
    U8 = source.UToY8()
    V8 = source.VToY8()
    Y8 = source.ConvertToY()
    A8 = source.AddAlphaPlane(128).AToY8()
    CombinePlanes(Y8, U8, V8, A8, planes="YUVA", source_planes="YYYY", sample_clip=source) #pixel_type="YUV444P8"

  Examples#2
    # Copy planes between planar RGB(A) and YUV(A) without any conversion
    # yuv 4:4:4 <-> planar rgb
    source = last.ConvertBits(32) # 4:4:4
    cast_to_planarrgb = CombinePlanes(source, planes="RGB", source_planes="YUV", pixel_type="RGBPS")
    # get back a clip identical with "source"
    cast_to_yuv = CombinePlanes(cast_to_planarrgb, planes="YUV", source_planes="RGB", pixel_type="YUV444PS")

  Examples#3
    #create a black and white planar RGB clip using Y channel
    #source is a YUV clip
    grey = CombinePlanes(source, planes="RGB", source_planes="YYY", pixel_type="RGBP8")

  Examples#4
    #copy luma from one clip, U and V from another
    #source is the template
    #sourceY is a Y or YUV clip
    #sourceUV is a YUV clip
    grey = CombinePlanes(sourceY, sourceUV, planes="YUV", source_planes="YUV", sample_clip = source)

  Remark:
    When there is only one input clip, zero-cost BitBlt-less subframes are used, which is much faster.

    e.g.: casting YUV to RGB, shuffle RGBA to ABGR, U to Y, etc..
    Target planes that are not specified, preserve their content.

    Examples:
    combineplanes(clipRGBP, planes="RGB",source_planes="BGR") # swap R and B
    combineplanes(clipYUV, planes="GBRA",source_planes="YUVA",pixel_type="RGBAP8") # cast YUVA to planar RGBA
    combineplanes(clipYUV, planes="Y",source_planes="U",pixel_type="Y8") # extract U


Earlier (pre v2294) modifications and informations on behaviour of existing filter
----------------------------------------------------------------------------------
[ColorSpaceNameToPixelType]
New script function: ColorSpaceNameToPixelType()
Parameter: video colorspace string
Returns: Integer

Returns a VideoInfo::pixel_type integer from a valid colorspace string

In Avisynth+ we have way too many pixel_type's now, this function can be useful for plugins for parsing a target colorspace string parameter.

Earlier I made this function available from within avisynth core, as I made one function from the previous 3-4 different places where colorspace name parameters were parsed in a copy-paste code.

In Avisynth the existing PixelType script function returns the pixeltype name of the current clip.
This function reverses this.

It has the advantage that it returns the same (for example) YV24 constant from "YV24" or "YUV444" or "Yuv444p8", so it recognizes some possible naming variants.

csp_name = "YUV422P8"
csp_name2 = "YV16"
SubTitle("PixelType value of " + csp_name + " = " + String(ColorSpaceNameToPixelType(csp_name))\
+ " and " + csp_name2 + " = " + String(ColorSpaceNameToPixelType(csp_name2)) )

[New conditional functions]

Conditional runtime functions have 10-16 bit/float support for YUV, PlanarRGB and 16 bit packed RGB formats.

Since RGB is also available as a planar colorspace, the plane statistics functions logically were expanded.

New functions
• AverageR, AverageG AverageB like AverageLuma
• RDifference, GDifference, BDifference like LumaDifference(clip1, clip2)
• RDifferenceFromPrevious, GDifferenceFromPrevious, BDifferenceFromPrevious
• RDifferenceToNext, GDifferenceToNext, BDifferenceToNext
• RPlaneMin, GPlaneMin BPlaneMin like YPlaneMin(clip [, float threshold = 0, int offset = 0])
• RPlaneMax, GPlaneMax BPlaneMax like YPlaneMax(clip [, float threshold = 0, int offset = 0])
• RPlaneMinMaxDifference, GPlaneMinMaxDifference BPlaneMinMaxDifference like YPlaneMinMaxDifference(clip [, float threshold = 0, int offset = 0])
• RPlaneMedian, GPlaneMedian, BPlaneMedian like YPlaneMedian(clip [, int offset = 0])

For float colorspaces the Min, Max, MinMaxDifference and Median functions populate pixel counts for the internal statistics at a 16 bit resolution internally.

[Tweak]
See original doc: http://avisynth.nl/index.php/Tweak
The original 8 bit tweak worked with internal LUT both for luma and chroma conversion.
Chroma LUT requires 2D LUT table, thus only implemented for 10 bit clips for memory reasons.
Luma LUT is working at 16 bits (1D table)

Above these limits the calculations are realtime, and done pixel by pixel.
You can use a new parameter to force ignoring LUT usage (calculate each pixel on-the-fly)
For this purpose use realcalc=true.

[MaskHS]
Works for 10-16bit,float.

MaskHS uses LUT for 10/12 bits. Above this (no memory for fast LUTs) the calculation is done realtime for each.
To override LUT for 10-12 bits use new parameter realcalc=true

[ColorKeyMask]:
Works for RGB64, Planar RGBA 8-16,float.
ColorKeyMask color and tolerance parameters are the same as for 8 bit RGB32.
Internally they are automatically scaled to the current bit-depth

[ResetMask]
New extension.
Accepts parameter (Mask=xxx) which is used for setting the alpha plane for a given value.
Default value for Mask is 255 for RGB32, 65535 for RGB64 and full 16 bit, 1.0 for float.
For 10-12-14 bit it is set to 1023, 4095 and 16383 respectively.

Parameter type is float, it can be applied to the alpha plane of a float-type YUVA or Planar RGBA clip.

[Layer]
Layer() now works for RGB64.
Original documentation: http://avisynth.nl/index.php/Layer

By avisynth documentation: for full strength Level=257 (default) should be given.
For RGB64 this magic number is Level=65537 (this is the default when RGB64 is used)

Sample:
lsmashvideosource("test.mp4", format="YUV420P8")
x=last
x = x.Spline16Resize(800,250).ColorYUV(levels="TV->PC")
x = x.ConvertToRGB32()

transparency0_255 = 128 # ResetMask's new parameter. Also helps testing :)
x2 = ColorBars().ConvertToRGB32().ResetMask(transparency0_255)

x_64 = x.ConvertToRGB32().ConvertBits(16)
x2_64 = ColorBars().ConvertToRGB32().ConvertBits(16).ResetMask(transparency0_255 / 255.0 * 65535.0 )

#For pixel-wise transparency information the alpha channel of an RGB32 overlay_clip is used as a mask.

op = "darken" # subtract lighten darken mul fast
level=257         # 0..257
level64=65537     # 0..65537
threshold=128                   # 0..255   Changes the transition point of op = "darken", "lighten."
threshold64=threshold*65535/255 # 0..65535 Changes the transition point of op = "darken", "lighten."
use_chroma = true
rgb32=Layer(x,x2,op=op,level=level,x=0,y=0,threshold=threshold,use_chroma=use_chroma )
rgb64=Layer(x_64,x2_64,op=op,level=level64,x=0,y=0,threshold=threshold64,use_chroma=use_chroma ).ConvertBits(8)
StackVertical(rgb32, rgb64)

[Levels]
Levels: 10-16 bit support for YUV(A), PlanarRGB(A), 16 bits for RGB48/64
No float support yet

Level values are not scaled, they are accepted as-is for 8+ bit depths

Test scripts for Levels
# Gamma, ranges (YUV):
x=ConvertToYUV420()
dither=true
coring=true
gamma=2.2
output_low = 55
output_high = 198
clip8 = x.Levels(0, gamma, 255, output_low, output_high , dither=dither, coring=coring)
clip10 = x.ConvertBits(10).Levels(0,gamma,1023,output_low *4,(output_high +1)*4 - 1, dither=dither, coring=coring)
clip16 = x.ConvertBits(16).Levels(0,gamma,65535,output_low *256,(output_high+1) *256 -1,dither=dither, coring=coring)
stackvertical(clip8.Histogram("levels"), clip10.ConvertBits(8).Histogram("levels"), Clip16.ConvertBits(8).Histogram("levels"))

# packed RGB 32/64
xx = ConvertToRGB32()
dither=false
coring=false
gamma=1.4
clip8 = xx.Levels(0, gamma, 255, 0, 255, dither=dither, coring=coring)
clip16 = xx.ConvertBits(16).Levels(0,gamma,65535,0,65535,dither=dither, coring=coring)
stackvertical(clip8.ConvertToYUV444().Histogram("levels"), Clip16.ConvertBits(8).ConvertToYUV444().Histogram("levels"))

[ColorYUV]
Now it works for 10-16 bit clips

• Slightly modified "demo" mode when using ColorYUV(showyuv=true)

#old: draws YV12 with 16-239 U/V image (448x448)
#new: draws YV12 with 16-240 U/V image (450x450)

• New options for "demo" mode when using ColorYUV(showyuv=true)
New parameter: bool showyuv_fullrange.
Description: Draws YV12 with 0-255 U/V image (512x512)
Usage: ColorYUV(showyuv=true, showyuv_fullrange=true)

New parameter: bits=10,12,14,16
Result clip is the given bit depth for YUV420Pxx format.
As image size is limited (for 10 bits the range 64-963 requires big image size), color resolution is 10 bits maximum.
#This sample draws YUV420P10 with 64-963 U/V image
ColorYUV(showyuv=true, bits=10).Info()

Luma steps are 16-235-16../0-255-0.. up to 0-65535-0... when bits=16

• Additional infos for ColorYUV

- Fixed an uninitialized internal variable regarding pc<->tv conversion,
  resulting in clips sometimes were expanding to pc range when it wasn't asked.
- No parameter scaling needed for high bit depth clips.
  For 8+ bit clips parameter ranges are the same as for the 8 bit clips.
  They will be scaled properly for 10-16 bitdepths.
  e.g. off_u=-20 will be converted to -204 for 10 bits, -20256 for 16 bits
- ColorYUV uses 8-10-12-14-16 bit lut.
- ColorYUV is not available for 32 bit (float) clips at the moment

[Other things you may have not known]
Source filters are automatically detected, specifying MT_SERIALIZED is not necessary for them.

[Known issues/things]
GRunT in MT modes (Avs+ specific)
[done: v2502] Overlay blend with fully transparent mask is incorrect, overlaying pixel=0 becomes 1, overlaying pixel=255 becomes 254.
[done: v2676-] Float-type clips: chroma should be zero based: +/-0.5 instead of 0..1
