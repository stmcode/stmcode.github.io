---
layout: post
title:  "Algorithmic Music with Strudel"
date:   2022-11-27 15:16:33 +0000
categories: jekyll update
---
Today we're going to learn Javascript and functional programming by using the [*Strudel* Javascript framework](https://strudel.tidalcycles.org/), which is a Javascript implementation of the [*TidalCycles* Haskell framework](https://tidalcycles.org).

Strudel aims to be a drop-in replacement for TidalCycles, which currently requires a bit of an annoying setup to run - however it is one of the most popular [livecoding](https://en.wikipedia.org/wiki/Live_coding) frameworks used by artists to generate new sounds. Some examples of algorithmic music artists include PC Music and Charli XCX collaborator Lil Data ([example 1](https://youtu.be/EqbXAk0Xm-I), [example 2](https://youtu.be/KdSiv7unrx8)), and dance music duo Autechre ([example 1](https://www.youtube.com/watch?v=qkTJTf7Yvk8), [example 2](https://youtu.be/-F665Mkc0D0)).

If this kind of music isn't your kind of thing - don't worry, it doesn't all sound like this! We are just going to use the process of algorithmic music to learn more about how to code. First off; you will need to visit the [Strudel Website](https://strudel.tidalcycles.org/). From here, you'll be greeted with a scary looking page:

![start page](/assets/strudel_intro/start_screen.png)

There's a good chance this screen looks slightly different to yours - by default, Strudel opens with a random code example from the creators. To play what this code produces, click the "Play" button in the top right (in the yellow box)!

You can see another example by clicking the "Shuffle" button. These are all examples written by the Strudel team as examples of what the framework is capable of!

## How does Strudel Work?

Strudel is interpreting Javascript code and outputting sound from your browser - it operates by having a constant *cycle* running in the background that governs what sounds are played when, by distributing patterns over this cycle. We can define patterns in a number of ways - the easiest being the `s()` function (we'll talk about what functions are in more detail later). Try deleting all the code in the editor and just putting in the following:

```javascript
s("bd bd")
```

Now try hitting play. You should hear a kick drum play - this is because `bd` stands for "bass drum", and by supplying the `s()` function with a pattern (`"bd bd"`), we are telling it to play two bass drums per cycle. Try changing the pattern in the `s()` function to `"bd bd bd bd"`. What happens? Can you think of why? As with all the following csnippets of code, have a play around with them to see what you can come up with!

If you guessed that we are trying to fit twice the amount of drums into one cycle, well done!

This way of writing patterns is called the *mini notation*, and is a codified way of writing down patterns. These patterns can be used to control a number of things, such as rhythms for drums, notes, parameters for effects and more. Mini notation doesn't stop there - we have more tools at our disposal!

By default, Strudel will try and evenly distribute your pattern over a single cycle - but sometimes you want more complex rhythms. By using `[]`, you can tell strudel to treat a segement of a pattern as one note - try the following:

```javascript
s("bd bd bd [bd bd bd bd]")
```

Strudel will compress all of those kick drums into the last quarter of the cycle. We can also define "rests" by using the `~` character: 

```javascript
s("bd bd ~ [~ bd]")
```

We can slow or speed patterns up using the `/` and `*` operators, distributing them over part of or many cycles:

```javascript
s("[bd bd bd [bd bd bd]]/2")
```

Finally, can alternate between different options per cycle using the `<>` operator (try and guess what `sn` and `cp` stand for!):

```javascript
s("bd bd bd <sn cp>")
```

There's more to the mini notation than this, with some shortcuts for commonly used rythms - but we don't need to know them for now. Let's move onto something cooler!

If we want two or more patterns playing at once, we have to use the `stack` function, with our different patterns seperated by commas:

```javascript
stack(
  s("bd bd"),
  s("~ cp"),
  s("[~ hh]*2")
)
```

Now it sounds like we're getting somewhere! What about loading other sounds in?

Well, try changing the above code to this:

```javascript
samples({
  bd: '808bd/BD0050.WAV',
  cp: '808/CP.WAV',
  hh: '808oh/OH00.WAV',
}, 'github:tidalcycles/Dirt-Samples/master/');

stack(
  s("bd bd"),
  s("~ cp"),
  s("[~ hh]*2")
)
```

What does this do?

Well the `samples` function is in charge of loading new samples into Strudel. It takes 2 arguments (separated by a comma) - the second is the location of where Strudel should start looking for the files - in our case, this is a GitHub code repository [here](https://github.com/tidalcycles/Dirt-Samples). This repository has lots of folders containing `.wav` sound files that we need to tell Strudel what to do with. That's where the first argument comes in - it's a list of pairs (with a `:` in between), first telling what strudel should map the sound do (in our case, we are asking it to override `bd`, `cp`, and `hh`, but you can use custom names if you want), and the second telling it where to find these samples in the directory we just pointed to.

Is this a huge headache? Yes - but this allows us to programatically download samples from other areas of the internet! If you want to upload your own files to Strudel, I'd recommend using the same method as above with your own GitHub repository.

## We have Rhythm - but what about Pitch?

Using just the above, we can already crate some complex percussive patterns - but Strudel also allows us to pitch samples using another function - the `note()` function. For example:

```javascript
note("<[c3 e3 g3 c4] [f2 a2 c3 f3] [a2 c3 e3 g3] [g2 d3 g3 b3]>")
    .s("piano")
    .pianoroll()
```

(Note: This may get very annoying! I've also moved each function to it's own line to improve readability - this is totally fine and encouraged for readable code!)

Some notes here:

* We are now *chaining functions together* - this is the essence of functional programming - every funcion takes in an input and output, and applies some effect to it. Here, the `note()` function sees it has nothing before it in the chain, so just outputs a `Pattern` object that is passed forward to the `s()` function. This function, seeing it has two inputs (the previous link in the chain, and the `"piano"` argument), takes these and outputs a new pattern with each of the notes replaced by the piano sound. If we remove the `.s("piano")` link in the chain, the code will still run, but with a sad sounding default sample.
* We can write the notes either as numbers (half-steps away from the root note), or as letter notes as we would on a piano (sharps are represented by an "s" (e.g. `cs3`) and flats by a "b" (e.g. `cb3`)). Later we can generate these patterns algorithmically using arpeggiation and more!
* I've included the `pianoroll()` function - this just visualises the pitches in the background in a pretty way!

The crazy thing is that the input to `note()` is a pattern - but so is the `s()` input. What happens if we change the sound like follows? Take the following:

```javascript
note("<[c3 e3 g3 c4] [f2 a2 c3 f3] [a2 c3 e3 g3] [g2 d3 g3 b3]>")
  .s("[piano ocarina organ_8inch sax_stacc]/4")
  .decay(0.7).sustain(0)
  .pianoroll()
```

(Note: I've added a `.decay(0.7).sustain(0)` to this to stop you from going mad - this means that the sample decays after `0.7` seconds to a volume of `0` - otherwise all the samples play on top of each other! Also note that these parameters accept patterns and we could change them over a cycle!!!)

Amazing!

We can now make a suitable festive song using Strudel with a bit of work (bonus points if you can guess the song without running the code!):

```javascript
note(`
  [
    [e3 e3 e3 ~]
    [e3 e3 e3 ~]
    [e3 g3 c3 d3]
    [e3 ~ ~ ~]
    [f3 f3 f3 f3]
    [f3 e3 e3 [e3 e3]]
    <[e3 d3 d3 e3 d3 ~ g3 ~ ] [g3 g3 f3 d3 c3 ~ ~ ~]>@2
  ]
/8`).s("piano").decay(0.7).sustain(0)
```

(Backticks are used for a multi line pattern and the `@` symbol means that the pattern should given twice the normal share of the cycle)

## Effects and modifiers

The great thing about functional programming is it lends itself nicely to building chains of effects for audio outputs. In some ways, we have already seen one effect - that modifies the pattern from the default sound to the one given - however obviously this isn't what we think of when we talk about effects!

As of writing, Strudel is still fairly new - so [the documentation](https://strudel.tidalcycles.org/tutorial/) is not complete. The [TidalCycles documenation](https://tidalcycles.org/docs/reference/cycles) can give a clearer picture as to what is going on - sometimes though you'll just need to poke around to see if you can understand something!

Effects can be applied to a sound by simply chaining the function to the pattern. Consider the following - I assign the above sound to the variable `jingle_bells` to organise the code a bit better:

```javascript
var jingle_bells = note(`
  [
    [e3 e3 e3 ~]
    [e3 e3 e3 ~]
    [e3 g3 c3 d3]
    [e3 ~ ~ ~]
    [f3 f3 f3 f3]
    [f3 e3 e3 [e3 e3]]
    <[e3 d3 d3 e3 d3 ~ g3 ~ ] [g3 g3 f3 d3 c3 ~ ~ ~]>@2
  ]
/8`).s("piano").decay(0.7).sustain(0)

jingle_bells.cutoff(100)
```

This applies a low pass filter to the sound at 100Hz - making it sound a bit muffled. As with most things in Strudel, we can make this a pattern:

```javascript
...
jingle_bells.cutoff("[100 500 1000 10000]/4")
```

It's fairly common to want to oscillate the cutoff of a signal according to a simple wave (e.g. a sine wave) - fortunately in Strudel we can do this! You can also control the filter resonance using the `.resonance()` function!

```javascript
...
jingle_bells.cutoff(sine.range(10, 1000).slow(4)).resonance(10)
```

## Higher Order Functions

Most functions in Strudel take in a pattern - however some need to take in another function to run. Two examples of this are `off()` and `sometimes()`.

```javascript
var jingle_bells = `
  [
    [e3 e3 e3 ~]
    [e3 e3 e3 ~]
    [e3 g3 c3 d3]
    [e3 ~ ~ ~]
    [f3 f3 f3 f3]
    [f3 e3 e3 [e3 e3]]
    <[e3 d3 d3 e3 d3 ~ g3 ~ ] [g3 g3 f3 d3 c3 ~ ~ ~]>@2
  ]
/8`.off(1/8, x=>x.add(12)).note().s("piano").decay(0.7).sustain(0)

jingle_bells
```

The syntax here is a bit confusing for two reasonse:

* We have to change the order of how we apply the functions. This is because what one function outputs needs to be acceptable for the next output; the `add` function within the `off` function works on note values before they are transformed - so it must be applied at this step!
* The second argument in the `off` function is an *anonymous function* - that is, we define a custom function to let `off` what to do with the offset notes. Here we are just taking their note value and adding `12` semitones (or an octave).

`sometimes()` works very similarly:

```javascript
var jingle_bells = `
  [
    [e3 e3 e3 ~]
    [e3 e3 e3 ~]
    [e3 g3 c3 d3]
    [e3 ~ ~ ~]
    [f3 f3 f3 f3]
    [f3 e3 e3 [e3 e3]]
    <[e3 d3 d3 e3 d3 ~ g3 ~ ] [g3 g3 f3 d3 c3 ~ ~ ~]>@2
  ]
/8`.sometimes(x=>x.add(12)).note().s("piano").decay(0.7).sustain(0)

jingle_bells
```

By combining all these tools together, you can get some pretty interesting outputs! For example - take the following:

```javascript
let melody = "[~ db3 f3 c4]!6 [~ bb2 eb3 bb3] [~ bb2 e3 bb3] [~ c3 f3 c4]!4";
let harmony = "[db2,f2,ab2]!6 [eb2,gb2,bb2] [e2,gb2,bb2] [f2,ab2,c3]!4";
let bass = "bb1@6 eb2 e2 f2@4";

stack(
  melody
    .add(12)
    .rarely(x=>x.off(0, x=>x.add(12).velocity(0.2)))
    .note()
    .s("ocarina")
    .sometimes(x=>x.jux(rev))
    .decay(1/2)
    .sustain(0)
    .color("salmon"),
  harmony
    .add(12)
    .note()
    .s("ocarina")
    .hcutoff(100)
    .cutoff(2000)
    .decay(1)
    .sustain(0)
    .color("darkseagreen"),
  bass
    .note()
    .s("sine")
    .shape(0.4)
    .gain(0.5)
).slow(8).pianoroll()
```

That's it for this post - have a play around with some of the functions and see what you can come up with!
