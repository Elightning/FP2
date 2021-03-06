### My Library: Portaudio
 
For this assignment I checked out the portaudio library. I constructed a basic program that allows users to pan (or position) a generated sine-wave in the stereo feild. 

The default levels of each channel of audio is .5 out of 1. A user is intended to input the pan position (which is provided as an argument to the "PanPos->PanScalar" procedure I wrote) within the range of [-100, 100], where -100 is far left and 100 is far right. The opposite channel of the one the pan position is set towards is attenuated is proportion to how far the pan is set. So, if the pan is set to -100, the right channel is fully attenuated while the left channel contains normal audio data with an amplitude of 0.5.

The mechanics are discussed in greater detial within the comment sections found in the code below.

Unfortunately, the Portaudio library doesn't have any built-in write functions, so in order to produce some audio samples generated from my program I had to include rsound library. The rsound library is dependent on the portaudio library, so I 
didn't need to include both libraries in the require field. However, other than the last two lines, this entire program is based on the Portaudio library. 

My code is as follows:

```
#lang racket

(require rsound
         ffi/vector)
 
(define pitch 426)

;Values of left and right channels set to 0.5 out of 1 by default.

(define panRight 0.5)
(define panLeft 0.5)

;Valid pan position values are [-100,100], where -100 is far left and 100
;is far right. In the following procedure, the pan position argument is scaled 
;by a factor of 0.5. Then that value is deducted from 0.5. This procedure is used
;to garner a value that is later used in the PanL/R procedure to help determine
;the level that the channel opposite to the direction of the pan should be attenuated,
;thereby making it appear as though the sound has moved in the direction panned.

(define (PanPos->PanScalar panPos) 
  (- 0.5 (* 0.5 (/ panPos 100))))

;The else statement should be straight forward enough. The first consequent expression
;is necessary because a negative (left) pan position will generate a number greater
;than 0.5, and the greater the negative number is (the farther left the pan) the closer
;the result of the PanPos->PanScalar procedure will be to zero, thus attenuating the 
;right channel.

(define (PanL/R panVal)
  (cond ((< 0.5 panVal) (set! panRight (- 1 panVal)))
        (else (set! panLeft panVal))))

;Here the pan position is set to 0 by default, meaning that the audio is centered.

(define panVal (PanPos->PanScalar 0))
(PanL/R panVal)
 
;defualt sample rate

(define sample-rate 44100.0)

;tpisr is a default constant that is used later to generate values of the 
;sine wave at particular samples. tpsir is later multiplied by the desired
;frequency and then an iterator from 0 to (sample-rate x duration) acts as
;the variable used to calculate the value of the sine wave for each sample.

(define tpisr (* 2 pi (/ 1.0 sample-rate)))

;Scales the value of the amplitude of the current sampeled sine wave value
;by 32767. Since the minimum value of the sine wav is -1 and the max value
;is 1, this scaling factor provides a dynamic range of (32767 - (-32767)) = 2^16,
;or 16 bits, which is the default bit depth of this library and a large amount
;of pro-audio systems.

(define (realnum->s16 x)
  (inexact->exact (round (* 32767 x))))
 
;44100 x 2 (two channels- left/right) = 88200.
;88200 x 2 (duration is 2 seconds) =  176400. 
;Therefore, vec contains 176400 elements, each of which are 2 bytes.

(define vec (make-s16vector (* 88200 2)))

;There are 88200 frames each of which contains two channels, whose
;values are of by some constant factor as determined by the result
;PanPos->PanScalar procedure. 

(for ([t (in-range 88200)])
  
;Sample (left) is multiplied by the panLeft constant for all of the values
;generated by the sin function. Similar idea for sample2 except the constant
;mulitplier is the value of panRight.
  
  (define sample (realnum->s16 (* panLeft (sin (* tpisr t pitch)))))
  (define sample2 (realnum->s16 (* panRight (sin (* tpisr t pitch)))))
  
;The first element in each frame is the left channel (evens), which is set equal to
;sample. The second element in each frame is the right channel (odds), which is 
;set equal to sample2.
  
  (s16vector-set! vec (* 2 t) sample)
  (s16vector-set! vec (add1 (* 2 t)) sample2))
 
;Converts vector to type rsound.

(define audio (vec->rsound vec 44100))

;Writes rsound to path.

(rs-write audio "some path...")

```
[Click Here](https://soundcloud.com/you/tracks) for some audio samples that this code generated.
