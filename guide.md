## Getting the Most out of Scope 📈

### Objectives
- Know more about Scope
  - Practical use
  - Sales arguments
  - Advanced features

### Agenda
- Intro / Presentation
- Project Basics
  - Basic YT
  - Triggering
- Lab 1 - The Basics
  - **ALL** the properties
  - Chart formatting, grouping
  - Triggering
  - Marker table
- Lab 2 - Scope Server
  - Headless data acquisition
  - Conversion tool
- Lab 3
- Lab 4 - New charts!
  - Array time chart
  - Table

## Lab 1 - The Basics

Open the solution. The PLC project is a simple reversing motion sequence. Activate and run, and let's create a new YT Scope project. Start by adding 3 charts to show the `SetPos` vs. `ActPos`, `SetVel` vs. `ActVel`, and `AxisState`. Use Target Browser and just drag your signals in.
> These signals can all be found in `P_ReverseSequence.fbAxis.stAxis.NcToPlc`. Use the Target Browser's filter!

Arrange the windows so that all trends are visible, and record some data.

#### Observe:
- Project Properties
  - Save/record
  - View detail
- Chart Properties
  - Test modifying display width and overwrite mode (you can do this "live")
- Axis Group Properties
  - Mostly visual properties of X/Y axis
- Channel Properties
  - Test modifying line width and mark size

Compare disparate values:

Move the `AxisState` channel up to the Position chart. When sharing an Axis Group with the position values, the auto Y-scaling makes the state unreadable. Create another axis group for the Position chart (right-click Chart -> New Axis) and drag `AxisState` into it. Now you have two auto-scaling Y-axes that keep your data visually aligned.

#### Triggering

Triggers are a way to automate certain behavior in Scope — most commonly, to start recording.

- Add a new Channel Trigger Set
  - In the Group, set the Trigger Action to **Start Record**
- We will use the `bStartSeq` BOOL as our trigger
  - Trigger channels must be in the DataPool first!
  - Channel Trigger Set properties:
    - Release = Rising Edge
    - Threshold = 1
    - Used Data = Acquisition: bStartSeq
  - Scope Project property: 
      - Record->Start Mode = Trigger Set

Press Start Record - it now waits for the trigger condition(s). Go over to the PLC and stop the motion sequence with `bStopSeq`. Once back in the `nSeq` 0 state, restart the sequence by writing `TRUE` to `bStartSeq`. Check back over at your Scope. Did it start? Why not? 
> Hint: The SampleMode for all items in the DataPool (default) = **Task Sampletime**.

We need at least one scan or Scope will miss the trigger. Modify the reset of `bStartSeq` with an online change (move it down to `CASE nSeq OF 1:`), then restart the motion sequence again. Now our trigger should work appropriately. Something to watch out for when trying to use internal flags as triggers.

For an analog trigger, let's say we only want to see when the axis is moving in the reverse direction (`ActVel < 0`). We can create two trigger groups with actions **Start Record**, and **Stop Record**, then set the channel trigger properties accordingly.

> Hint: If we combine this with the **Restart Record** Project-level property, the **Start Record** trigger will work without having to manually re-initiate the recording (via the button). This is particularly useful if you are also auto-saving and want to capture multiple cycles. This capability does require a **Scope Pro** license.

#### Layering

Layering is a way to overlay and visually compare how a signal changes over time. It is particularly useful with cyclic data like we are analyzing here.

- Hide all channels (via toggling visibility) on your position chart *except* for `ActPos`
- Record 3 or 4 full cycles
- Create two new markers (right-click Project -> **New Time Marker**)
  - Put them on either side of the last full peak in the chart
  - Try to get the markers at the lowest Y-point on both sides of the peak; equidistant from the high point
- Right-click the project again and this time open the **Layer Editor**
- Select the appropriate YT Chart in the Layer editor, and set the start/end times to the respective markers.
  - Note: If you do not have a Scope Pro license, you will have to manually copy/paste the timestamps from the marker properties.
- Click the green '+' to add a chart layer, and then change it to an 'Echo layer' via the goofy looking button next to the eye.
- Set the echo amount and echo duration
  - Duration should be ~4.860s. You can calculate it by subtracting your marker times.

Observe the previous cycles overlayed with the latest. Notice that you can also use triggers as start/end points.