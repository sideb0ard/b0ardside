---
Title: "Autechre-like beat written by Claude CLI"
Description: "I was surprised that Claude CLI could write a music program in my custom language!"
Tags: ["video", "AI", "music", "livecode", "soundb0ard"]
Date: "2026-01-08"
Categories:
- "blog"
Slug: "autechre-claude"
---

<p>
I’m working on a track within my Soundb0ard Shell environment, and it’s missing a lil something. I’ve been using Claude CLI as a regular coding companion, and its helped me with lots of C++ code, adding dsp effects, fixing threading issues and a whole lot more. Being able to interact with a shell which can read and edit your code is a lot more natural than having to context switch to a browser looking for stack overflow answers.
Anyhoo, I asked Claude:
"hi! I'd like to program an Autechre like crunchy hip hop beat. Can you help?"
</p>
<p>
I expected a text answer, describing the sounds, and hopefully with some sixteen step patterns i could use. Claude CLI churned away for a few minutes, before coming back with a complete program written in my custom programming language SbLang - slang. Knowing how LLMs work, I guess it's not super surprising, but something about it still blows me away. No-one else has actually used this language yet, so this is the first time a program has been written in my language that wasn’t by me!
I guess what makes it so fascinating is that even a few years ago, this wouldn't have seemed possible. The idea of a Claude terminal style program written in an imperative style is just umpossible!
</p>


<pre><code class="terminal">

❯ I'm going to write a blog post about this. Can you show me my original prompt
asking you to create an autechre inspired crunchy hip hop beat, and explain how
you went about learning my language and using it

⏺ Your Original Prompt

"hi! I'd like to program an Autechre like crunchy hip hop beat. Can you help?"

How I Learned Your Language

Here's the process I used to understand and write in your custom .sb language:

1. Explored the Codebase Structure

I started by looking at the project structure and quickly identified:
  - SBTraxx/ directory containing .sb script files (your language in action)
- src/interpreter/ containing the language implementation (lexer, parser,
        evaluator, AST)
  - Command files in src/interpreter/sound_cmds.cpp showing available audio
  functions

  2. Studied the Grammar from Parser Code

  I read through src/interpreter/parser.cpp and ast.hpp to understand:
  - Syntax: C-style expressions with custom audio extensions
  - Control flow: for, if, let for variables
  - Built-in functions: note_on_at(), rand(), incr(), len()
  - Special syntax: set object:param value at = time for automation

  3. Read Existing Examples

  I looked at other .sb files in SBTraxx/ to see patterns like:
  - How drum patterns are defined: let pat = [1,0,1,1,0,1,0,0]
  - How samples are loaded: let kick = "/kick.wav"
  - How loops are structured: for (let i = 0; i < 16; i++)
  - Timing conventions: positions measured in pulse units (pp for pulse per
          16th)

  4. Understood the Audio Model

  From sound_cmds.cpp and object code:
  - DrumSampler objects for playing samples
  - Parameters: pitch, pan, vol, attack_ms, decay_ms, release_ms
  - MIDI-style: velocity values (0-127), note scheduling
  - Pattern language: Grid-based sequencing at 16th-note resolution

  5. Synthesized the Knowledge

  For an "Autechre-like crunchy hip hop beat," I knew I needed:
  - Glitchy elements: Rapid pitch modulation
  - Polyrhythms: Different pattern lengths playing simultaneously
  - Randomization: Unpredictable variations
  - Crunch: Short decay times, varied velocities

  6. Created AUTECHRE_CRUNCH.sb

  // Load diverse percussive samples
  let percs = [
    load_drum_sampler("./wavs/hihat_edgy.wav"),
    load_drum_sampler("./wavs/click2.wav"),
    load_drum_sampler("./wavs/glitch2.wav"),
    // ... more samples
  ];

  // Define glitchy pitch variations
  let pitchz = [1.0, 0.5, 2.0, 1.5, 0.75, 1.33, 0.66];

  // Polyrhythmic patterns
  let pat1 = [1,0,1,1,0,1,0,0,1,0,1,0,0,1,1,0];  // 16 steps
  let pat2 = [1,0,0,1,1,0,1,0,0,1,0,1];          // 12 steps (polyrhythm)
  let pat3 = [1,1,0,1,0,0,1];                    // 7 steps (odd timing)

  // Schedule notes with pitch automation
  for (let i = 0; i < 16; i++) {
    if (pat[i] > 0) {
      note_on_at(percs[pidx], 1, i * pp + offset,
          dur = durz[drx], vel = 40 + rand(60));
      set percs[pidx]:pitch pitchz[pz] at = i * pp;
    }
  }

The key insight: I treated your language like any other codebase - the code IS
the documentation. By reading the interpreter implementation and existing
examples, I could infer the language's semantics and idioms without needing a
separate language manual.

</code></pre>

<p>
Rad! It does a much better job of documentation than i've been doing!
</p>
<p>You can find the full file contents <a href="https://github.com/sideb0ard/SBShell/blob/d34d7dc2ea59c8deea2e7f490329eb48958ba493/SBTraxx/AUTECHRE_CRUNCH.sb">here</a> and hear the results in the youtube video below..</p>

<div class="video-container">
<iframe width="560" height="315" src="//www.youtube.com/embed/s_790pWyi-s" frameborder="0" allowfullscreen></iframe>
</div>
