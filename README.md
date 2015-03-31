# Final Project Assignment 2: Explore One More! (FP2) 
DUE March 30, 2015 Monday (2015-03-30)

```
#lang racket

(require net/url)
```

### My Library: Portaudio
 
For this assignment I checked out the portaudio library. I constructed a basic program that allows users to pan (or position) a generated sine-wave in the stereo feild. 

The default levels of each channel of audio is .5 out of 1. A user is intended to input the pan position (which is provided as an argument to the "PanPos->PanScalar" procedure I wrote) within the range of [-100, 100], where -100 is far left and 100 is far right. The opposite channel of the one the pan position is set towards is attenuated is proportion to how far the pan is set. So, if the pan is set to -100, the right channel is fully attenuated while the left channel contains normal audio data with an amplitude of 0.5.

Unfortunately, this library doesn't have any write functions, so I couldn't save any of the produced audio to disk.

My code is as follows:

```
#lang racket

(require portaudio
         ffi/vector)
 
(define pitch 426)

(define panRight 0.5)
(define panLeft 0.5)

(define (PanPos->PanScalar panPos) 
  (- 0.5 (* 0.5 (/ panPos 100))))

(define (PanL/R panVal)
  (cond ((< 0.5 panVal) (set! panRight (- 1 panVal)))
        (else (set! panLeft panVal))))

; Sample value of 75
(define panVal (PanPos->PanScalar 75))
(PanL/R panVal)
 
(define sample-rate 44100.0)
(define tpisr (* 2 pi (/ 1.0 sample-rate)))
(define (real->s16 x)
  (inexact->exact (round (* 32767 x))))
 
(define vec (make-s16vector (* 88200 2)))
(for ([t (in-range 88200)])
  (define sample (real->s16 (* panLeft (sin (* tpisr t pitch)))))
  (define sample2 (real->s16 (* panRight (sin (* tpisr t pitch)))))
  (s16vector-set! vec (* 2 t) sample)
  (s16vector-set! vec (add1 (* 2 t)) sample2))
 
(s16vec-play vec 0 88200 sample-rate)

```

