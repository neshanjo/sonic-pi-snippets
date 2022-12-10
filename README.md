# sonic-pi-snippets <!-- omit in toc -->

Some code snippets that help me making music with [sonic pi](https://sonic-pi.net/).

- [Template with synced live loops](#template-with-synced-live-loops)
- [Playing](#playing)
  - [Playing a sample](#playing-a-sample)
  - [Play a sample stretched to a certain time](#play-a-sample-stretched-to-a-certain-time)
  - [Playing notes with a synthesizer, global synth settings](#playing-notes-with-a-synthesizer-global-synth-settings)
  - [Playing notes with a synthesizer, individual synth settings](#playing-notes-with-a-synthesizer-individual-synth-settings)
  - [Playing portamento (changing pitch of note while playing)](#playing-portamento-changing-pitch-of-note-while-playing)
  - [Playing chords](#playing-chords)
  - [Playing several notes with one command](#playing-several-notes-with-one-command)
  - [Changing the octave](#changing-the-octave)
- [Sequencing techniques](#sequencing-techniques)
  - [Playing a looping sequence](#playing-a-looping-sequence)
  - [Playing a sample (or a note) every 8th time](#playing-a-sample-or-a-note-every-8th-time)
  - [Playing a sample (or a note) with 0.5% probability](#playing-a-sample-or-a-note-with-05-probability)
  - [Playing random notes from a scale](#playing-random-notes-from-a-scale)
  - [Playing random chords](#playing-random-chords)
  - [Using Euclidean rhythms](#using-euclidean-rhythms)
- [Effect automation](#effect-automation)
  - [Independent ticks in one live\_loop](#independent-ticks-in-one-live_loop)
  - [Moving panorama (left to right to left to right...)](#moving-panorama-left-to-right-to-left-to-right)
  - [Moving cutoff filter](#moving-cutoff-filter)
- [Global variables](#global-variables)
- [Favorite effects](#favorite-effects)
  - [Ping pong echo](#ping-pong-echo)
  - [Adding moving lpf to any sound](#adding-moving-lpf-to-any-sound)
- [Favorite samples](#favorite-samples)
- [Favorite synth sounds](#favorite-synth-sounds)
  - [Bass or lead: soft and fat](#bass-or-lead-soft-and-fat)
  - [Fat FM Bass (in octave 2)](#fat-fm-bass-in-octave-2)
  - [Gnarly FM Bass (in octave 3)](#gnarly-fm-bass-in-octave-3)
  - [Piano: FM electric piano](#piano-fm-electric-piano)
  - [Pad: soft and dark](#pad-soft-and-dark)
- [Risers and fallers](#risers-and-fallers)
  - [Riser: Snare roll](#riser-snare-roll)
  - [Faller: lunar land](#faller-lunar-land)
- [License](#license)

## Template with synced live loops

The live_loop :pulse is used for synchronizing the other loops which wait until the loop starts. Delay makes sure that no other loop is started first.

```ruby
use_bpm 120

live_loop :pulse, delay: 0.1 do
  sample :bd_haus
  sleep 1
end

live_loop :hihat, sync: :pulse do
  sleep 0.5
  sample :drum_cymbal_open, finish: 0.2, amp: 0.4
  sleep 0.5
end

# other live loops with 'sync: :pulse' ... 
```

## Playing

### Playing a sample

```ruby
sample :drum_cymbal_closed, finish: 0.2, amp: 0.4
```

### Play a sample stretched to a certain time

```ruby
sample :loop_amen, beat_stretch: 4
```

### Playing notes with a synthesizer, global synth settings

```ruby
use_synth :tb303
use_synth_defaults cutoff: 60, res: 0.5, cutoff_attack: 0,
      attack: 0, decay: 0, sustain: 8, 
      release: 0.4, amp: 0.9
play :e1
play :f1
```

### Playing notes with a synthesizer, individual synth settings

```ruby
use_synth :tb303
play :e1, cutoff: 60, res: 0.5, cutoff_attack: 0,
      attack: 0, decay: 0, sustain: 8, 
      release: 0.4, amp: 0.9
play :f1, cutoff: 40, res: 0.5, cutoff_attack: 0,
      attack: 0, decay: 0, sustain: 1, 
      release: 0.7, amp: 0.5
```

### Playing portamento (changing pitch of note while playing)

```ruby
s = (play :d4, sustain: 5)
sleep 2
control s, note: :g4
sleep 1
control s, note: :f4
sleep 0.5
control s, note: :d4
sleep 0.5
sleep 2
```

Usually, the sleep times should add up to the sustain value.

### Playing chords

```ruby
play_chord chord(:a3, :minor)
play_chord [:a4, :c4, :e4, :a5]
```

### Playing several notes with one command


```ruby
play_pattern_timed [:e3, :f3, :g3, :g3, :r],
    [0.5, 0.5, 1, 1, 1], release: 1
```

`:r` is a rest.

### Changing the octave

```ruby
use_octave +1
play :e1 # actually, :e2 is played!
```

## Sequencing techniques

### Playing a looping sequence

```ruby
play (ring :e2, :b5, :d5, :b3, :a4).tick
sleep 1
```

### Playing a sample (or a note) every 8th time

```ruby
sample :bd_ada, amp: 1.5 if (bools 0, 0, 0, 0, 0, 0, 1).tick
```

### Playing a sample (or a note) with 0.5% probability

```ruby
sample :drum_bass_soft if one_in(2)
```

### Playing random notes from a scale

```ruby
play (scale :c3, :minor, num_octaves: 2).choose
```

### Playing random chords

```ruby
play (ring [:g3, :d4, :g4], [:g3, :d4, :a4], [:g3, :d4, :c4], [:g3, :d4, :f4]).choose
```

### Using Euclidean rhythms

```ruby
play :a4 if (spread 3, 8, rotate: 1).tick
```

[Euclidean rhythms](https://en.wikipedia.org/wiki/Euclidean_rhythm) define patterns that are typical in different rhythms, e.g. `(spread 3, 8) = (ring true, false, false, true, false, false, true, false)` is a typical clave pattern.

Other example that can be copied to be used for different notes (e.g. bass notes):

```ruby
volume = 1.5
16.times do
  play :c2, amp: volume if (spread 5, 8, rotate: 2).tick
  sleep 0.25
end
```

## Effect automation

### Independent ticks in one live_loop

When using multiple rings in one live loop, `tick` advances all of them together. Use `look` to get the
current position of the tick index without advancing it.

```ruby
(range -1.0, 1.0, 0.1).reflect.tick 
(ring 0, 1).tick # this tick advances the tick index again, so always 0 is returned

# to avoid this, use:
tick
(range -1.0, 1.0, 0.1).reflect.look
(ring 0, 1).look
```

Alternatively, you can add add a parameter to the tick function which then creates individual tick counters:

```ruby
(range -1.0, 1.0, 0.1).reflect.tick(:t1)
(ring 0, 1).tick(:t2)
```

See also [Naming Ticks](https://sonic-pi.net/tutorial.html#section-9-4).

### Moving panorama (left to right to left to right...)

```ruby
pan: (range -1.0, 1.0, 0.1).reflect.tick(:pan)
```

Explanation: The `range` function creates values from -1.0 to 1.0 in increments of 0.1. `reflect` then adds the inversion of the ring to it, so the values wander between -1 and 1.

### Moving cutoff filter

```ruby
cutoff: (range 40, 60, 6).reflect.tick(:co)
```

## Global variables

It is possible to put values into a [global memory store](https://sonic-pi.net/tutorial.html#section-10-1). These values can then be accessed in all live loops. Here is an example how to implement a simple mixer with this method and control the bass pattern:

```ruby
live_loop :mixer, delay: 0.1 do
  set :amp_bd, 2
  set :amp_hh, 2
  set :amp_bass, 1.5
  set :bass_extended, true
  sleep 1
end

live_loop :kick, sync: :mixer do
  sample :bd_haus, amp: get[:amp_bd]
  sleep 1
end

live_loop :hihat, sync: :mixer do
  sleep 0.5
  sample :drum_cymbal_closed, amp: get[:amp_hh]
end

live_loop :bass, sync: :mixer do
  use_synth :tb303
  play :e1, amp: get[:amp_bass]
  sleep 4
  if (get[:bass_extended]) then
    play :b1, amp: get[:amp_bass]
    sleep 4
  end
end
```

## Favorite effects

### Ping pong echo

```ruby
with_fx :ping_pong, mix: 1, phase: 4, feedback: 0.8
```

### Adding moving lpf to any sound

... even if the sound does not have a `cutoff`-property!

```ruby
with_fx :lpf, cutoff: (line 70, 100, steps: 64).reflect.tick do
  # code
end
```

## Favorite samples

- `:bd_haus`: fat house bassdrum
- `sample :drum_cymbal_open, finish: 0.2, amp: 0.4`: tamed open hihat

## Favorite synth sounds

### Bass or lead: soft and fat

```ruby
use_synth :dtri
```

### Fat FM Bass (in octave 2)

```ruby
live_loop :fatbass do
  use_synth :fm
  use_synth_defaults divisor: 2, depth: 2
  play_pattern_timed [:c2, :eb2, :f2, :g2, :bb2, :f2, :r],
    [0.5, 0.5, 0.5, 1, 1, 1, 3.5],
    release: 0.5
end
```

### Gnarly FM Bass (in octave 3)

```ruby
live_loop :gnarlybass do
  use_synth :fm
  use_synth_defaults divisor: 4, depth: 2
  play_pattern_timed [:c3, :eb3, :f3, :g3, :bb3, :f3, :r],
    [0.5, 0.5, 0.5, 1, 1, 1, 3.5],
    release: 0.5
end
```

### Piano: FM electric piano

```ruby
live_loop :fmpiano do
use_synth :fm
  use_synth :fm
  use_synth_defaults sustain_level: 0.8, decay: 0.1, sustain: 0.5, amp: 2, cutoff: 110
  use_octave -1
  play_pattern_timed [(chord :c5, :minor), (chord :g5, :minor),
                      (chord :g5, :minor),(chord :eb5, :major),
                      (chord :f5, :major),  (chord :f5, :major)
                      ], [2, 1, 1, 2.5, 1, 0.5]
end

```

### Pad: soft and dark

```ruby
live_loop :amb do
  use_synth :dark_ambience
  play [:g3, :d4, :a4],
    attack: 0, sustain: 8, release: 1, amp: 1.5
  sleep 8
end
```

## Risers and fallers

### Riser: Snare roll

```ruby
live_loop :snareriser do
  sample :drum_snare_hard,
    amp: (line 0, 1.0, inclusive: true, steps: 64).ramp.tick
  sleep 0.25
end
```

Explanation: `line` creates a ring going from 0 to 1.0 in 64 steps. `ramp` make the last value of the ring repeat forever.

### Faller: lunar land

```ruby
live_loop :fill do
  with_fx :panslicer, wave: 3 do
    sample :ambi_lunar_land, amp: 0.7
    sleep 20
  end
end
```

## License 

This work is licensed under a [Creative Commons Attribution 4.0 International License.](https://creativecommons.org/licenses/by/4.0/). The code can be used within the conditions of the [MIT license](LICENSE).

(C) Johannes Schneider
