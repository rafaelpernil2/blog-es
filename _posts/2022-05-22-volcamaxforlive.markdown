---
title: volca sample2 polyphonic chromatic player - A Max4Live plugin to use a KORG
  volca sample2 as a polyphonic chromatic sample player
date: 2022-05-22 01:13:00 +02:00
categories:
- MIDI
tags:
- MIDI
- Software
---

![Sin título.png](https://feranern.sirv.com/Images/Sin%20t%C3%ADtulo.png)

This little project started from my goal of learning MIDI programming and Max programming language, joining my both passions, software and music.

I was reading the MIDI Implementation of my new volca sample2 to see what can be done and to my surprise, I could transform this device into a sample-based synth with a very simple transformation.

CC#49 is used for chromatic speed (i.e, setting the exact note of the sample), so I thought I could translate MIDI Note On messages to CC#49 values and a note on message to trigger the sound.

I opened a MIDI Monitor and checked how the notes and CC#49 correlated. It turns out for CC#49 middle C has the value 64 while MIDI specification uses a value 60 as middle C. Simple math: add 4 to the note value

Well, that was easy, it took me only a few hours with little knowledge of Max. But I thought... Okay, this device has 8 note polyphony, if I map a note to a channel, I can have 8-voice polyphony! But here the complexity skyrocketed, mainly because of Ableton Live/Max4Live limitations.

Let me explain the flow: Key x is pressed on MIDI keyboard → NoteON message is sent → Obtain MIDI channel to use, trying to avoid channels in use → Transform note value to CC#49 message → Send CC#49 and NoteOn to Volca

But I can only output to one MIDI channel per instance of plugin... So, how do I determine where to output? By having one instance per track and manually setting which MIDI channel each instance has to output to.

And how do I communicate between different instances? It’s complicated, you can’t connect with specific instances, you can only broadcast messages. How do you know which channel to use if you don’t know which is the last one used?

I had to resort to a simple but imperfect algorithm, “Note count”. Using the “borax” component, you can get the amount of notes being active, so...

1 note active → Channel 1, ...8 notes active → Channel 8

Problems:

With melodies, the next note collapses with the previous one... Polyphony is destroyed, it works like a monophonic synth
The same happens with chords and it easy to lose notes while playing
Good things:

It works and serves as a Proof of Concept
It can be quite musical and provide interesting limitations for playing differently
As a last feature, I added a global selector for samples. Using message broadcasting (send and receive components) I send the same sample selected to all 8 channels. I think it was absolutely needed for this plugin to be usable. Changing samples one by one for all tracks is tedious.

There’s some interesting logic here too! The volca sample2 has a maximum capacity of 200 samples, but CC/Program change messages only go from 0 to 127 (7 bits), that means 2 messages are used for creating the message.

First, a MSB (Most Significant Bit, CC#3) that represents if the sample is between 0-99 or 100-199 and is either 0 or 1

Then, the LSB (Least Significant Bit, CC#35) goes from 0 to 99

So, the math is the following for getting the sample number from the messages: MSB*100 + LSB

And this is the math for obtaining each part:

MSB = Sample Number / 100

LSB = Sample Number % 100 (Modulo operation)

It was a great learning experience, feel free to modify it and play with the code. Since Max is a visual programming language, I can also show in one screenshot how it works:

![volcasamplemax.png](https://feranern.sirv.com/Images/volcasamplemax.png)


Extending and improving this project, I created a new version based on WebMIDI. It’s WAY better, you’ll see :P