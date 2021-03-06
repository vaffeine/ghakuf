ghakuf
======

A Rust library for parsing/building SMF (Standard MIDI File).

[![Build Status](https://travis-ci.org/acknak/ghakuf.svg?branch=master)](https://travis-ci.org/acknak/ghakuf)
[![Crates.io](https://img.shields.io/crates/v/ghakuf.svg)](https://crates.io/crates/ghakuf)

## Examples

`ghakuf` has parse module and build module separatory.

### Perser

`ghakuf`'s parser is made by Event-driven online algorithm. You must prepare original handler implementing Handler trait to catch SMF messages. Any number of handlers you can add for parser if you want.

```rust
use ghakuf::messages::*;
use ghakuf::reader::*;
use std::path;

let path = path::Path::new("test.mid");
let mut handler = HogeHandler{};
let mut reader = Reader::new(
    &mut handler,
    &path,
).unwrap();
let _ = reader.read();

struct HogeHandler {}
impl Handler for HogeHandler {
    fn header(&mut self, format: u16, track: u16, time_base: u16) {
      // Something
    }
    fn meta_event(&mut self, delta_time: u32, event: &MetaEvent, data: &Vec<u8>) {
      // you
    }
    fn midi_event(&mut self, delta_time: u32, event: &MidiEvent) {
      // want
    }
    fn sys_ex_event(&mut self, delta_time: u32, event: &SysExEvent, data: &Vec<u8>) {
      // to
    }
    fn track_change(&mut self) {
      // do
    }
}
```

### Builder

`ghakuf` build SMF by Message enums. Message enum consists of MetaEvent, MidiEvent, SysExEvent, and TrackChange. You can use running status if you want. At track change, you should use not only MetaEvent::EndOfTrack message, but also TrackChange message.

```rust
use ghakuf::messages::*;
use ghakuf::writer::*;
use std::path;

let tempo: u32 = 60 * 1000000 / 102; //bpm:102
let write_messages: Vec<Message> = vec![
    Message::MetaEvent {
        delta_time: 0,
        event: MetaEvent::SetTempo,
        data: [(tempo >> 16) as u8, (tempo >> 8) as u8, tempo as u8].to_vec(),
    },
    Message::MetaEvent {
        delta_time: 0,
        event: MetaEvent::EndOfTrack,
        data: Vec::new(),
    },
    Message::TrackChange,
    Message::MidiEvent {
        delta_time: 0,
        event: MidiEvent::NoteOn {
            ch: 0,
            note: 0x3c,
            velocity: 0x7f,
        },
    },
    Message::MidiEvent {
        delta_time: 192,
        event: MidiEvent::NoteOn {
            ch: 0,
            note: 0x40,
            velocity: 0,
        },
    },
    Message::MetaEvent {
        delta_time: 0,
        event: MetaEvent::EndOfTrack,
        data: Vec::new(),
    }
];
let path = path::Path::new("examples/example.mid");
let mut writer = Writer::new();
writer.running_status(true);
for message in &write_messages {
    writer.push(&message);
}
let _ = writer.write(&path);
```

## Supported SMF Event

You can use three type events. In Message enum, these events have delta time and data.

### Meta Event

* SequenceNumber
* TextEvent
* CopyrightNotice
* SequenceOrTrackName
* InstrumentName
* Lyric
* Marker
* CuePoint
* MIDIChannelPrefix
* EndOfTrack
* SetTempo
* SMTPEOffset
* TimeSignature
* KeySignature
* SequencerSpecificMetaEvent

### MIDI Event

* NoteOff { ch: u8, note: u8, velocity: u8 }
* NoteOn { ch: u8, note: u8, velocity: u8 }
* PolyphonicKeyPressure { ch: u8, note: u8, velocity: u8 }
* ControlChange { ch: u8, control: u8, data: u8 }
* ProgramChange { ch: u8, program: u8 }
* ChannelPressure { ch: u8, pressure: u8 }
* PitchBendChange { ch: u8, data: i16 }

### System Exclusive Event

* (F0 event)
* (F7 event)

## License

`ghakuf` is primarily distributed under the terms of both the MIT license and the Apache License (Version 2.0), with portions covered by various BSD-like licenses.

See LICENSE-APACHE, and LICENSE-MIT for details.
