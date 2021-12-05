---
title: Frequency Visualizer Technicals
date: 2021-12-04
layout: post
tags: [retro, gif, fft, GLUT, OpenGL, numpy, frequency, visualizer]
excerpt: "The ins and outs of live audio analysis"
comments: true
---
As part of the [audio visualizer project](https://github.com/TimtheStew/AudioSpectrumVisualizer) described in the [first real post](/Frequency-Visualizer-Retrospective/) on this blog
we had to write up descriptions of the operation of our programs. Now that I'm
rebooting my blog (after not touching it for 2+ years 😬) it seemed a shame to let such an easy potential blog post fall by 
the wayside, so I figured I'd post the write-up here (slightly edited). There's 
also a wishlist/TODO section at the bottom from the original project. 

In case you haven't read my previous post, here's a gif of the program running so you know
what it looks like.
<figure>
	<a href="/assets\img\sine-tones.gif"><img src="/assets\img\sine-tones.gif"></a>
</figure>
<p align=center>
    <cite> Sine tones, increasing in frequency(Hz) and decreasing in decibels(DBFS) </cite>
</p>

## GUI Operation Description
- It's an OOP tkinter GUI. It is very simple and tkinter is well documented. I
won't go into too much detail about it here. It uses buttons with lamda
callback function to register events, and updates it's state accordingly.

- When you have selected your desired .wav file, the play button will create
a header file for the display and audio processes and compile them according to 
our Makefile.

- Then the program will create processes for the fft and display processes.
It pipes the stdout of the fft process to the stdin of the display process.

## FFT Program Operation Description

### The Fast Fourier Transform in General:
- The fft is an O(nlogn) time complexity algorithm for computing the discrete
Fourier transform (the discrete analog of the continous Fourier Transform)
which utilizes the the collapsing nature of the roots of unity to divide
and conquer what is essentially a matrix multiplication. Numpy's fft (which
we are using, uses Cooley-Turkey with butterflies).

- The fft transforms a signal in the time domain into a signal in the
frequency domain, we are doing a 1024 pt fft, which means our output will be
half the size (as we're only using real components for input, our output
would be 1024 bins mirrored in the middle around the Nyquist frequency
(samplerate of the file/2)) This is fine as the human audible range is
only 20Hz-20kHz and wav files are encoded at 41.5kHz-48kHz

### Getting the samples:
- we use the python standard library module wave to read the metadata and
samples from the .wav files

### Dealing with channels:
- if the file only has one channel, we simply pass it along to the next step.
But if it has two channels, we can add them pre-fft (which due to the
linearity of the Fourier transform is the same as adding them after, just
with much less fft computation)

### High Pass Filtering:
- Because of how the fft is defined, the value in output bin 0 will be the
average frequency of the entire signal. To avoid this data from "corrupting"
our output, (by artificially increasing values closer to 0) what we want to
do is subtract from the entire signal (pre-fft) the value of that 0th bin.
In order to do this, we fft the signal once, then set the 0th bin of the
output of that fft to 0, then perform an inverse fft on that data,
reproducing the original signal but without the "DC component".

- This can also be done (and is perhaps more intuitive) by averaging the time
domain data per sample, and subtracting that amount from each value in the
sample.

### Windowing:
- The fourier transform, being something which decomposes data into it's
periodic elements, assumes that the input it's being given is periodic. This
can present a problem if the shape of the input curve (the measure of
voltage across the membrane of a microphone) does not have a period of
exactly 1024. As you can imagine that sort of thing doesn't happen often
(unless you specifically generate a sine tone for it). So to solve that
problem we use a window function, there are several at our disposal, but
currently we are using the Hanning, as it has good side lobe reduction, and
we are not overly concerned with minor peak spreading.

- A window function is basically a curve that "windows" our input (makes it
0 valued everywhere outside of the input, and squishes down the sides so
they approach 0 at the beginning and end of the interval)

- For example, using no window function is equivalent to using a window that
is 0 everywhere except the sample (0-1024). This is sometimes also called
the uniform window.

- We apply the window by multiplying our high pass filtered signal with it,
term by term.

### The "Actual" FFT:
- Now that our data is prepared, we fft it to decompose our time chunk into
a frequency chunk. What we get out is a 512 array of complex numbers. At
this point we take the magnitudes of those vectors in the complex plane,
which give us our frequency bin amplitudes on a scale of the width of the
original samples (in our case 16 bit).

### Adjusting for Windowing:
- Windowing may magically fix our discontinuity problem,but it generates
several of it's own. We did, after all, transform our data quite heavily.
What we did is reduce the "gain" of the signal, and the amount of gain we've
lost is equal to the average value of the terms of the window. Looking at
the definition of the elements of the Hanning window:

		Hanning(n) = 1/2 - 1/2 * cos(n2pi/(N-1)), where N = window size

- It's easy to see, as cos moves between -1 and 1, that the average value of
those terms is going to be 1/2. So in order to remove that we multiply by
the reciprocal, 2.*

*note: this is kind of an oversimplification in ways I don't yet fully
understand. see TODO/WISHLIST > Multiple Overlapped FFT's section below.

### Converting to DBFS:
- To convert to DBFS, we scale the amplitudes back by dividing them by our
dbfs reference constant (32768, the max value on the scale) and then take
20log_10() of that in order to end up with DBFS


## Display Program Operation Description

### Mainloop:
- After being kicked off by the GUI, the display process first starts a new thread 
to begin reading the FFT data from it's stdin into a buffer array. Then it allocates some
arrays to represent state of the grid of square cells it will be displaying, as well 
as the corresponding grid of color values for the cells. Then it sets some metadata for
GLUT (our windowing service), execv's the actual audio process, and enters the main loop.

- The glutmainloop() attempts through various standard callbacks to, in whatever order 
it can, update the buffer index, render the grid of squares, and update the state of 
our cells from the buffer of FFT data.

### Callback Functions:
- We set up callback functions, to tell the openGL context (basically a giant
state machine) how to respond when certain things happen, for example a key
press (which we use to check if 'escape' has been hit) but also for things
like rendering and displaying, and the idle process that runs when there are no
events to process to maintain the framerate.

### Creating Vertices:
- We draw the squares by defining a small unit in both the x and y directions
which is the size of the gap between the colored cells. (like 1/16 of 1/64 of the
screen) Then the cells are just the remaining proportion of the larger
fraction of the cell. (14/16 of 1/64, each way)

### Defining Colors:
- We're using RGB floats, since they're valued from 0 to 1 we can create each rows
color as a slight variation on the previous. Both this and the vertex
definition step are done ahead of entering the render loop.



## TODO/WISHLIST
At first I wasn't going to leave this section in, but I am still even now
very satisfied and entranced when I watch this program throw up some
colored squares in time to music, and part of me would love to make 
a better version of it. So maybe this section is just flavor, but maybe 
someday I'll make my own rhythm game. 🤷 Or at least a frequency visualizer
that isn't so rough around the edges. 

### Multiple Overlapped FFT's:
- this would help with our time resolution, as well as let us do more
precise windowing with asymmetric windows (as long as their overlaps sum
to a constant over a window length).

### Units:
- It would be really great it we could display the frequency values below
the bins and the DBFS level up the side, especially because it would
allow verification of data without directly reading the values int the
file

### OpenGL 3/4 SDL:
- This was the original dream, and still something I'd really like to do.
definitely beyond the scope of this project (or at least beyond the scope
of us in the timeframe of this project) While I am a little disheartened
that we couldn't use something more up to date, I still think we made a
really cool thing, that I'm excited to keep working on in the future. And
even if we don't move into the core profile of OpenGL we could probably
switch from GLUT to SDL for a little more control and

### More Options in GUI:
- I'd like for overlap percentage, window type, and color parameters to
be selectable from the GUI

### Port to using FFTW instead of Numpy:
- Now that we're much more familiar with the fourier transform and digital
signal processing, I feel I could effectively wield a tool like fftw,
which I was not sure of when we started this project.

### Clean up inter process communication and global variable dependencies
