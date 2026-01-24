---
Title: "Adventures In Pitch Shifting"
Description: "the wonders of windowed sinc!"
Tags: ["audio", "video", "code", "ai"]
Date: "2025-04-07"
Categories:
  - "blog"
Slug: "adventures-in-pitch-shifting"
---

<div>
<img src = "/img/samplez.png">

<p>I utilize audio sample playback in a few ways in my Soundb0ard application - I have one shot sample playback, and I have a Looper which uses a form of granular synthesis to time stretch.
</p>

<p>Sample files are stored in PCM Wav files which have a header, followed by the audio data stored in an array of numbers, one per sample. Normal playback entails playing those samples back at sample rate at which it was recorded, e.g. 44,100 samples per second.<p>

<p>In order to pitch shift a sample playback, i.e. slow it down or speed it up, you have a few options. You can think of pitch shifting as resampling, e.g. to play a sample back at twice the speed, you could resample at half the original sample rate, i.e. remove half the samples, and then playback the resampled audio at the original sample rate; or to slow down playback to half-speed, you could play every sample twice.</p>

<p>However, what happens if you want a fractional pitch, such as 1.1 x original speed or 0.8? The naive way, which I’ve been using up till now, was to progress through the array at the fractional speed, i.e. instead of moving through the array 1 sample at a time, I would maintain a float read_idx, that would increment at the sample ratio, e.g. 1.1 x and then calculate the playback value as a linear interpolation between the two closest points in the audio data array. This works ok for some ratios, but some can sound a bit too gnarly.</p>

<p>Recently via a reddit thread I came across this wonderful resource -
<a href="https://cs.gmu.edu/~sean/book/synthesis/">Sean Luke, 2021, Computational Music Synthesis, first edition, available for free at http://cs.gmu.edu/~sean/book/synthesis/</a></p>

<p>
"But it turns out that there exists a method which will, at its limit, interpolate along the actual band-limited function, and act as a built-in brick wall antialiasing filter to boot. This method is windowed sinc interpolation."</p>

<p>Windowed Sinc Interpolation relies on this <a href="https://en.wikipedia.org/wiki/Sinc_function">Sinc Function</a>:<p>
<img src="/img/sinc.png">

<p>"you can use sinc to exactly reconstruct this continuous signal from your digital samples."</p>

<p>The links on this page can explain the math better, but basically in order to convert the frequency / sample rate, you walk through your original samples as the new sample rate and apply this sinc operation over a window of neighboring samples before and after your current sample, applying and summing the result of the sinc function.</p>

<p>From the Sean Luke book above, i converted this algorithm into code:</p>
<img src="/img/algo18.png">

<p>My first implementation didn’t work. The pitched signal was recognisable but was amped too high and sounded a lil janky. I think I mixed up some indexes with the value they should be representing.</p>

<p>I then found this amazing <a href="https://www.nicholson.com/rhn/dsp.html#3">Ron's Digital Signal Processing Page</a>, which has a clear concise implementation in Basic:</p>

<img src="/img/basic_interp.png">

<p>I implemented this in C++, and the code was clearer to read. After applying the repitch my signal was still clean but no matter what pitch ratio I used, my return signal was always double the original pitch. I must have made a calculation wrong. Possibly to do with handling stereo values.</p>

<p>Lazily I turned to Google Gemini…<br>
<br>
&gt; can you give me some example c++ code that will change the frequency of an array of samples using sinc ?<br>
..<br>
&lt;boom&gt;><br>
&gt; can you expand that example to handle a stereo signal?<br>
&lt;boom&gt;><br>
&gt; using an interleaved stereo signal, please<br>
&lt;boom&gt;><br>
&gt; can you improve the algorithm using a hann window?<br>
&lt;boom&gt;><br>
<br>
Ok, quite impressed. I dropped the code into my Looper, and it worked great.<br></p>

<p>Heres the before, with linear playback:</p>
<div class="video-container">
<iframe width="560" height="315" src="//www.youtube.com/embed/LXSMsztnHHs" frameborder="0" allowfullscreen></iframe>
</div>

<p>Heres the after using windowed sinc.</p>
<div class="video-container">
<iframe width="560" height="315" src="//www.youtube.com/embed/6Jv6lGHXwzo" frameborder="0" allowfullscreen></iframe>
</div>

<p>I think it sounds cleaner and better, so i think the implementation works? I’ll play with it a while and see if I prefer it. Here’s the <a href="https://github.com/sideb0ard/SoundB0ard/blob/c6f61d0f0bd324985b13cdbfb15234b1ad2c8804/src/audioutils.cpp#L426">current code</a>:<p>
<img src="/img/my_resample_code.png">

<p>Job done?<br>
No, there are some performance trade-offs.<p>

<p>I initially implemented it for the granular playback system, which meant only dealing with small arrays of data. However this meant I was doing redundant work, recalculating the same values upon each loop.</p>
<p>I moved the window sinc operation to be run once when you call the RePitch function. This becomes a performance bottleneck as those samples can be large arrays, and you dont want this being run on your audio thread as if it takes too long to run, you’ll experience audio drop outs. I looked to a newer feature of C++ to run the repitch algorithm, using <a href-"https://en.cppreference.com/w/cpp/thread/async">std::async</a> from &lt;future&gt;.<p>


</div>


