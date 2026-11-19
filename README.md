![](../../workflows/gds/badge.svg) ![](../../workflows/docs/badge.svg)

# Ultra-Low-Frequency GIDL/BTBT Low-Pass Filter

A compact SKY130 low-pass filter with a cutoff of about 3 Hz, implemented as
a custom analog layout for Tiny Tapeout. The filter uses the off-state
leakage of a single transistor as a very large resistance, so no resistors
or capacitors are added.

## General Description

This repository contains the design files for a first-order low-pass filter
for slow signals (a few hertz or below), such as bio-signals. A floating-body
NMOS in a deep N-well acts as the filter, and a PMOS source follower buffers
its high-impedance output so it can drive a 100 pF load.

## Academic Motivation

The design is based on the band-to-band tunneling (BTBT) low-pass filter of
Kadam et al. (IEEE TCAS-II, 2023) and the related spike delay element
(ISCAS 2025). Those works were demonstrated in 45 nm RF-SOI. This project
adapts the idea to the open SKY130 bulk CMOS process and adds an output
buffer. It was developed as a project for the Integrated Circuit Design
Laboratory at UABC, Mexicali.

## Project Objectives

- Build a low-pass filter with a cutoff below 10 Hz in SKY130, without
  external passives.
- Buffer the output to drive a 100 pF load with about 0 dB gain.
- Verify the design with pre-layout and post-layout simulations.
- Export GDS and LEF files compatible with the Tiny Tapeout analog flow.

## Project Summary

| Item | Value |
| --- | --- |
| Top module | `tt_um_b2b` |
| Process | SKY130 |
| Target | Tiny Tapeout analog/custom layout |
| Tile size | `<1x2 ` |
| Design type | Analog low-pass filter |
| Transistors | 5 (1 filter NMOS with 8 fingers, 4 in the output stage) |
| Analog pins | `ua[0]` input, `ua[1]` output, `ua[2]` source_2 |
| Supply / power | 1.8 V, about 10 µW |

## Design Concept

With its channel off, a MOSFET still conducts picoampere currents through
gate-induced drain leakage (band-to-band tunneling). Seen from the floating
body, the drain-body and source-body junctions become resistances of hundreds
of gigaohms. Combined with the junction capacitances (a few femtofarads),
they form a low-pass filter with a cutoff of a few hertz in a very small area.

The filter transistor (L = 2 µm to suppress the parasitic lateral bipolar)
has its gate grounded, drain and source tied together as the input, and the
body as the output. A PMOS source follower with a current-mirror load,
biased through Vbias, buffers the output.

## Simulation Results

Pre-layout simulation (Spectre, SKY130 models):

- Cutoff frequency (-3 dB): about 3.16 Hz 
- Passband gain: about -0.07 dB
- Output DC level: about 1.38 V for Vin = 350 mV and Vbias = 680 mV

Post-layout (parasitic-extracted) results: ~16 Hz.

## Layout Strategy

Full-custom analog layout:

- **Deep N-well isolation:** the filter NMOS sits in a deep N-well tied to
  1.8 V, so its body can float.
- **Separate N-wells:** the mirror PMOS devices share one well, and the
  source-follower PMOS has its own well tied to its source.
- **Short body route:** the body-to-follower-gate connection is kept as short
  as possible to limit coupling into the high-impedance node.

## Repository Structure

- `src/project.v`: Tiny Tapeout wrapper module.
- `gds/`: final GDS file.
- `lef/`: LEF abstract for the Tiny Tapeout flow.
- `info.yaml`: project metadata and pinout.
- `<xschem/>`, `<mag/>`: schematics and layout sources (adjust to your folders).

## Acknowledgments

Developed at the Faculty of Engineering Mexicali, Universidad Autónoma de Baja
California (UABC). Thanks to SkyWater and Tiny Tapeout for enabling open-source
silicon fabrication.
