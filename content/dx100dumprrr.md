---
Title: "DX100 SysEx data dumper"
Description: "human readable synth settings from DX100 Sysex files."
Tags: ["San Francisco", "synth", "dx100", "c++"]
Date: "2024-11-17"
Categories:
  - "blog"
Slug: "dx100-sysex"
---

<div id="story">

<div>
<img src="/FMsynth.png"
<br><br></div>

<div><br>
One of my software synths is based on the DX100. I programmed it by following along with <a href="https://www.willpirkle.com/">Will Pirkle’s 'Designing Software Synthesizer Plug-Ins in C++'</a>, a most awesome book. FM synthesis has a reputation for being notoriously difficult to program, so most people with one of the original  Yamaha DX range tended to just modify the presets. Building my own software implementation means I don’t have any presets, so I mostly stuck to randomizing the settings and saving the ones which sounded decent.<br><br></div>

<div><br>
More recently I’ve started learning how to create FM patches from scratch, and when you dig in a little, you find it’s not actually that hard to start sculpting decent sounds. With FM synths, oscillators and envelope generators are combined into a unit called an Operator.  These Operators can either be used as Carriers, where the sound they generate can be heard, or as Modulators, which modulate the sound of the Carriers, creating interesting timbres. These Operators can have multiple configurations of how they are connected, i.e. which are Carriers and which are Modulators. That makes more sense with a diagram. Here’s the layout of the 4 Operators for the 8 x DX100 algorithms:
<img src="/8algos.png">
</div>
<div><br>
The button row of each algorithm are the Carriers, the ones which we can directly hear. The Operators above them are modulating the ones below.<br>
<br>
The secret to designing your own is start simple, use one of the first 4 Algorithms, and switch off all but the first Operator, gradually adding modulation from higher Operators. The first 4 algorithms all have only one Carrier, and the other 3 Operators apply modulation in various combinations. From Algorithm 5, you start to get combinations of Carriers, up to Algorithm 8 which is no longer applying FM, but instead is a purely additive synthesis with the 4 Operators acting as Carriers. <br><br>

As I’ve been designing my own patches, I have become more curious about the original DX presets, some of which have their own kinda cult following, especially the Jazz Organ preset, which is maybe mostly widely down from Robin S’s "Show Me Love". (See an excellent wee blog post on its history and mix from Mark Fell <a href="https://markfell.com/web/work?id=902">here</a>.)<br><br>

I discovered there are many resources online for sharing DX patches, which can be loaded into compatible synths. The DX line of synths use a binary format called <a href="https://notebook.zoeblade.com/SysEx_file.html">SysEx</a> to store and transmit patches. SysEx messages became an unofficial but de facto standard, and you can find lots of SysEx files online. The ones I found were specifically Midi bulk data messages containing 32 Voice VMEM. Specifically this has a 6 byte header, then 32 voices stored in 4096 bytes, and a closing 2 bytes footer.<br><br>

While searching I found many Sysex librarian programs which allow you to manage and apply the voices but I couldn’t find what I was looking for - a way to just display the settings stored within the DX100 SysEx files. The closest I did find was this <a href="https://github.com/rogerallen/dxsyx">dxsyx</a>, a "C++ library for manipulating DX7 SysEx files", which offers a human readable YAML output, which is exactly what I was looking for, however that library is only the DX7 with its 6 operators, and not compatible with DX100 Sysex format. The original Yamaha DX synth, the DX7, has 6 Operators which are combined into 32 algorithms. The later DX100 and compatible DX21 and DX27, all used 4 Operators organized in 8 algorithms.<br><br>

I decided to write my own parser, I figured it could be quite a fun exercise! I borrowed quite a lot of code from the before-mentioned dxsyx, a lot of the header checks and the general approach. Quite simple really, read the file into a data array (std::vector<uint8_t>), and peel it off byte by byte, mapping the bytes to the expected Sysex fields. Where my dumper differs from dxsyx is in the parsing of the bytes, which need to be mapped correctly to the values needed for the Voice and Operator values, specific to the synth model.
The hardest part was finding an exact specification of the DX100 Sysex format to ensure the binary data I was reading was being mapped to the correct data field.<br><br>

The DX-100 manuals I could find listed the Sysex VMEM data fields in detail:<br>
<img src="./vmem.png">
</div>

<div><br>
But not how the byes were laid out in memory. For that I found a related manual for the Yamaha TX81Z, which “uses the same 4-operator, 8 algorithm FM synthesis as the DX21, DX27 and DX100, and voice data can be transmitted and received between them”
<br>
Here’s the binary layout of fields:
<img src="./vmem-binary.png">
</div>

<div><br>
With that information, I could loop and parse each of the 32 voices. Most of the data are byte length, but a few are packed and require some bitmasking to extract.
<br>
Full code is on my github.<br>
<br>
Build:<br>
clang++ -o dx -std=c++20 main.cpp<br>
<br>
Usage: ./dx inputfile.syx<br>
</div>

<div><br>
Here in my ‘dx100_1.syx’ i found the illustrious ‘Jazz Org’
<img src="./jazzorg.png">
</div>


<div><br>
It seems to work pretty legit. The data still needs some translation for use however. The ‘algorithm’ field is 0 indexed, so in this case its Algorithm 8, which is correct according to Mark’s blog post
<img src="./algo8.png">
</div>

<div><br>
Another example is the frequency ratio field, which is crucial in the timbre of FM synthesis. In the data output, the ratio value is an index into a table of frequency values. I had learned this already from the Will Pirkle book. Using the indexes listed above, the frequency ratios from my dump show ratios of 3, 1, 6, and 1, which also agree with Mark’s description.
<img src="./dxratios.png">
</div>

<div><br>
The values for the envelope length are in “rates”, a value between 31 and 0, where 31 is an instantaneous one, and 0 is a long change. I haven’t yet found a mapping between these envelope rates and milliseconds, so i’ve just been mapping them between 0 and 1000ms.<br>
<br>
Applying the values to my own dx software synth, the Jazz Organ preset sounds decently convincing, but some of the other presets sound a little wonky when imported to my synth, so i think i still have a bit more interpretation to play with, most likely around feedback. But yeah, pretty fun wee project! Check it <a href="https://github.com/sideb0ard/DX100Dumprrr/">here</a>

</div>




</div>
