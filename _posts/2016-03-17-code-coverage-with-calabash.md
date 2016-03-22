---
layout: post
title: "Code Coverage with Calabash"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Even though XCode7 is deprecating GCov output, it is still possible to generate .gcda and .gcno files for UI Automation.  However, it is my experience that code coverage files are only generated for simulator. This means it won't be possible to get code coverage for any tests that are limited to physical devices.  (If you do find a way to get it to generate on a device, please let me know!)

<b>It is important to note that you will get less accurate code coverage results with this deprecated gcov output from XCode7 than running tests with XCode6. It will, however, give you a minimum number.  I am actively working to get the numbers closer and will report back.</b>

<!--more-->

<h2>Here are the steps you would follow...</h2>

<b>Before the run (one time):</b>
<ol>
  <li>Clean up from your previous run. Delete everything in your output directory.  </li>
  <li>Generate a baseline file.  </li>
</ol>

<b>Each time you run a test case (scenario):</b>
<ol>
  <li>Remove old .gcda files (code coverage results from the previous run).</li>
  <li>Run the test case.</li>
  <li>Flush (generate those .gcda files).</li>
  <li>Process all the .gcda files into a single .info file.</li>
</ol>

<b>After all the tests are complete (one time):</b>
<ol>
  <li>Combine all the .info files into a single file.</li>
  <li>Generate a report from that file using lcov.</li>
</ol>

<br><h2>Install this:</h2>
<h3>Homebrew</h3>
If you don't have it, it's a great way to install tools without having to compile everything yourself
```/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"```

<h3>lcov</h3>
This tool is used to generate reports. Gcovr is another option, but it looked like the results weren't as accurate.
```brew install lcov```


<br><h2>Implementation</h2>
<h3>XCode</h3>
<h4>Environment Variables</h4>

```
// Activating this setting causes '.gcno' files to be produced that the gcov code-coverage utility can use to show program coverage.
GCC_GENERATE_TEST_COVERAGE_FILES = YES
 
// Activating this setting indicates that code should be added so program flow arcs are instrumented.
GCC_INSTRUMENT_PROGRAM_FLOW_ARCS = YES
 
// Lets us know if we have coverage enabled so we can turn the __gcov_flush() lines on/off as appropriate and avoid build errors
GTM_CONFIGURATION_GCC_PREPROCESSOR_DEFINITIONS = COVERAGE_ENABLED=1
```

<h4>Flushing</h4>

I've seen some people have some success with adding ```__gcov_flush()``` to their AppDelegate class. I decided to manually flush so I had more control.

<h5>Flushing in your Calabash backdoor (recommended flush method)</h5>
```
- (NSString*)flushGCov:(NSString*)ignored
{
#if defined(COVERAGE_ENABLED)
	extern void __gcov_flush(void);
	__gcov_flush();
	
	return @"GCov Flushed";
#endif
	return @"GCov not Flushed";
}
```

<h5>Flushing on applicationDidEnterBackground (alternative flush method)</h5>
<font color="red">Use at your own risk!</font>

You would place this before your @implementation:

```
#if defined(COVERAGE_ENABLED)
#include <stdio.h>
extern void __gcov_flush();
#endif
```

And you would add this to your applicationDidEnterBackground method:

```
#if defined(COVERAGE_ENABLED)
	__gcov_flush();
#endif
```

<h3>Ruby Code</h3>
This is how we have ours set up. You can implement it your own way if you'd like :).
<a href="https://gist.github.com/TeresaP/3fe3abdba01d47d02847">Gist with the Ruby code</a>

<br><h2>Reports</h2>
When the run completes, assuming you have called the methods like I did, you should have a CodeCoverage folder that contains a lot of .info files (including a *combined.info file with all the other .info files merged into it) and an html report folder.


