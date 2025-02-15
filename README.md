# key-ripper

A keyboard personalized for myself, designed in KiCad, with custom firmware written in Rust.

The story behind this board can be found in [this reddit post](https://www.reddit.com/r/MechanicalKeyboards/comments/vtlujd/i_built_a_keyboard_pcb_and_wrote_firmware_for_it/).

## Main Features and Requirements

* Be cheap to manufacture (can easily be ordered and fabricated by JLCPCB, uses an RP2040)
* USB C Connector
* Have dedicated function keys
* Reduce friction changing between a macbook keyboard and this one
* Stay relatively compact
* Full sized arrow keys with a gap (on the left and right) for easily finding the up arrow without looking
* No right shift key (this allows for the arrow keys to sit below the Enter key, and I never use right shift)
* Support for screw-in stabilizers first and foremost, along with snap-in stabilizers, and plate-mounted stabilizers when using a top plate
* Easy to assemble - this will apply once I design a case for it
* Firmware written in Rust, debuggable with knurling-rs

## Current Version (v0.2)

[v0.2 JLCPCB Production Files](https://github.com/bschwind/key-ripper/releases/tag/v0.2)

Version 0.2 improves upon some of the issues in v0.1, with some intentional choices that could be seen as a downgrade:

* Added a copper pour on both sides for a flatter PCB
* Shifted a few components away from the screw-in stabilizer holes to avoid screw washers touching the components
* Fixed some silkscreens for the reset and USB boot buttons
* Fixed the alignment of the F5 key
* **Controversial** - Changed the switch footprint to normal MX soldered switches instead of Kailh hotswap footprints. This allows for simpler boards that don't require a top plate.

(pictures to be added later)

## Version v0.1

Version 0.1 _works_ but there are some issues:

* The PCB warps during SMT reflow because there is a copper pour on only one side
* The silkscreens are not labeled well (SW1, SW2, SW3, etc. instead of Q, W, E, R, etc.)
* The F5 key is slightly more to the left of the F6 key than it should be, which leads to incompatible top plates between v0.1 and v0.2

Version 0.1 uses Kailh hotswap sockets. Combined with the top plate, this is good for trying out different switches, but adds complexity
in the build process. It requires more parts, and I had to 3D print 3.4mm M2 spacers.

[v0.1 JLCPCB Production Files](https://github.com/bschwind/key-ripper/releases/tag/v0.1)

![PCB Front](https://i.imgur.com/pEsHZqL.jpeg)
![PCB Back](https://i.imgur.com/iWDIUL9.jpeg)
![Assembled Front](https://i.imgur.com/rEM1gKr.jpeg)
![Assembled Back](https://i.imgur.com/TvYkt1t.jpeg)
![Plugged in and working](https://i.imgur.com/G42Mo8T.jpeg)

## Production Files

For every PCB fabrication order, I make a github release and upload the files I used to make the order.

You can see a list of releases [here](https://github.com/bschwind/key-ripper/releases).

## Create a Gerber File

For most PCB fabrication shops, they expect gerber files, which are essentially files that describe the layout of pads/traces/geometry for each layer in a PCB. They also contain the drill locations if your PCB has any holes. All these layers should be placed in a directory and compressed to a zip file for uploading.

To create a gerber zip in KiCAD:

* Open up PCBNEW (KiCAD's PCB design tool)
* Select `File` -> `Plot`
* Select the following layers:
    * F.Cu
    * B.Cu
    * F.Paste
    * B.Paste
    * F.SilkS
    * B.SilkS
    * F.Mask
    * B.Mask
    * Edge.Cuts
* Leave all the other settings to their defaults
* Output to a directory named `gerber` (this is just convention, any name will do)
* Click `Plot`
* Next click `Generate Drill Files` with an `Excellon` drill file format, and a `Gerber` map file format
* Make sure the `Drill Units` are in `Millimeters`
* Output to the same directory you did for the previous gerber layers
* Click `Generate Drill File`
* Zip the `gerber` directory into a `gerber.zip` file and it's ready to upload!

**Note**: Check the PCB vendor's website for special KiCAD instructions, as they sometimes prefer certain settings when exporting.

OSHPark can directly accept `*.kicad_pcb` files so you don't need to export gerbers when ordering there.

## Create a Bill of Materials (for JLCPCB)

From the schematic viewer:

* Click the "BOM" button in the upper right toolbar
* Use the `bom_csv_grouped_by_value_with_fp`
* The output file won't have a `.csv` extension, so add it

Modify the CSV file:
* Rename the following columns:
    * `Ref` -> `Designator`
    * `Qnty` -> `Quantity`
* Delete the following columns:
    * `Cmp name`
    * `Description`
    * `Vendor`
* Add the following column:
    * `LCSC Part #`
* Remove any part rows you don't need to populate
* For each part, look it up on `jlcpcb.com/parts` and copy over both the footprint and the LCSC part number into their respective columns

In general, try to use as many "basic parts" as you can from JLCPCB. Each "extended part" costs an extra 300 yen per board.

## Create the Position File (for JCLPCB's pick-and-place machines)

From KiCad's PCB design tool:

* `File` -> `Fabrication Outputs` -> `Footprint Position (.pos) File`
* `Format`: `CSV`
* `Units`: `Millimeters`
* `Files`: `Single file for board`
* `Include footprints with SMD pads even if not marked Surface Mount`: `checked`
* Click `Generate Position File`

Modify the CSV file:

* Rename the following columns:
    * `Ref` -> `Designator`
    * `Qnty` -> `Quantity`
    * `Val` -> `Value`
    * `PosX` -> `Mid X`
    * `PosY` -> `Mid Y`
    * `Rot` -> `Rotation`
    * `Side` -> `Layer`

When uploading to JLCPCB, you may need to modify the rotation values. It will show you red dots on pin 1 for the relevant components, as well as a red `+` for components with polarity, so double check against your silkscreen and placement. Positive rotation goes counter-clockwise, so if you need to rotate a part counter-clockwise one turn, add 90 degrees. Subtract 90 to rotate one turn clockwise, and modulo 360 degrees to keep the overall rotation value positive.

## Firmware Debugging

Set the `DEFMT_LOG` environment variable.
