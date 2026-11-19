# CS-61A-Animator-Surgical-Robotics-Lab
Project for CS 61A (UCB)
AI was used to create extensions for the project following the given tasks.

How to try it out
------------------
Open "Drawings & Fractals" and pick mandelbrot, mandelbrot_zoom,
apollonian_gasket, lorenz_attractor or clifford_attractor from the drawing
menu, then drag the slider or press morph. Open "Surgical Robotics Lab" and
step through lessons 1-8 with the arrows on the lesson card. Edit each
lesson's code and follow its "try this" exercises, watching the readout's
measured numbers change.

ANIMATOR EXTENSIONS: FRACTALS, A SURGICAL ROBOTICS LAB, AND A FINISHED WEBSITE

Summary
-------
I extended Animator in three directions. First, new shapes that draw
mathematical objects too heavy to build out of Lines and Points: the
Mandelbrot set (with deep zoom), the Apollonian gasket, and the Lorenz and
Clifford strange attractors. Second, a "Surgical Robotics Lab": a second area
of the app with eight interactive lessons, each led by code the user can
edit, that teach the core ideas behind surgical robots (signals, tremor
filtering, kinematics, the remote center of motion, state estimation, feedback
control and haptics), using realistic numbers from published research. I utilized 
AI to create it as a learning tool for myself, since this is the niche I want to
specialize in. Third, I turned the app into something that can be posted as a public 
website: deployment, usability, accessibility and performance work.

Constraints AI kept: It didn't change anything in test/, or Point, Line or
Path. parser.gleam, builtins.gleam, errors.gleam, limits.gleam and
mutable_dict.gleam are unchanged, and init, update, view, public_url and
video_url keep their names and arguments. All new drawing features are new
Shape variants, added to evaluator.gleam's list of constructors, so they're
written in the same Gleam subset as every other drawing.

The extensions in the order I built them
----------------------------------------

1. The Mandelbrot set.
   A new Shape variant, Mandelbrot(center, width, iterations). The renderer
   escape-time iterates z -> z^2 + c over a grid and draws each row of
   points that escape at the same iteration as one path. The animation
   reveals the set one iteration at a time, and the colour shifts gradually
   along the spectrum with each new iteration. I added it to the library
   (examples.gleam) so it can be chosen from the drawing menu like any
   built-in example.

2. Faster Mandelbrot calculations and deep zoom.
   A second variant, MandelbrotZoom(center, zoom, iterations), where every 10
   steps of zoom double the magnification, so the shape slider zooms
   smoothly into the set's boundary. The speed-ups:
   - Skip the main cardioid and the period-2 bulb with closed-form tests.
   - Brent-style periodicity checking, to stop early on points that will
     never escape.
   - Merge neighbouring cells into runs, so far fewer paths are drawn.
   - Cache results in an LRU of recent computations (in JavaScript, via FFI),
     so going back to a slider value is instant.
   The canvas also batches consecutive paths of the same style into a single
   stroke.

3. Better colouring.
   I replaced the always-cycling palette with histogram colouring, the scheme
   most fractal programs use. Colours are given by each point's escape count
   relative to how many points escaped by then, so the colours follow the
   density of escape times. They spread evenly at every zoom level instead
   of being washed out or banded.

4. The Apollonian gasket and strange attractors.
   - Apollonian(...): circles packed into a circle, built by Descartes'
     circle theorem and repeated Vieta reflections. The slider changes how
     the first circles split the space.
   - Lorenz(...): the Lorenz system integrated with fourth-order Runge-Kutta
     and projected to 2D. The slider sets rho, so you can watch the system
     go from settling to a point to chaos.
   - Clifford(a, b, c, d, points): a million-point attractor drawn as a
     density image with the same histogram colouring, computed in an FFI
     helper for speed.

5. The first surgical robotics example: tremor filtering.
   I wanted something related to my interest in surgical robotics, focused
   on signals (thanks EE66). TremorFilter(...) simulates a surgeon's hand tracing a
   running suture, with physiological tremor added. It passes the motion
   through a 2nd-order Butterworth low-pass filter (a biquad from the
   bilinear transform) and through motion scaling. It draws four panels: the
   hand, the instrument tip, tremor over time, and the frequency response.

6. The mode switcher and home screen.
   A header with two areas, "Drawings & Fractals" and "Surgical Robotics
   Lab", and a home screen with a card for each. Every drawing belongs to the
   area its code uses, and each area has its own accent colour.

7. Smoother sliders.
   - Parsed programs are cached, so dragging a slider doesn't parse again.
   - While dragging, redraws wait for the browser to paint (after_paint), so
     they never pile up.
   - Heavy shapes are drawn at lower detail while the slider moves, then in
     full detail when it's let go.
   - A "morph" button sweeps the slider back and forth automatically.

8. The Surgical Robotics Lab: eight lessons.
   Each lesson has a card (explanation, key formula, "try this" exercises,
   next/previous, and a button to restore the original code) above code the
   user edits. Each also has a live dashboard on the canvas with plots and a
   readout of measured numbers. New Shape variants:
   1. Signals are made of frequencies: Epicycles. A DFT rebuilds a path from
      rotating circles, and dropping the fast ones acts as a low-pass filter.
   2. Tremor filtering and motion scaling: TremorFilter (from step 5).
   3. Forward kinematics: ArmPose, a two-link arm with its joint angles and
      reachable workspace.
   4. Inverse kinematics: ArmTrack, damped least squares with adaptive
      damping near singularities. It respects joint limits and a joint speed
      limit, and reports how much of the path was reached.
   5. The remote center of motion: Laparoscope, a rigid instrument that
      pivots about the incision. It shows how hand motion is reversed and
      scaled by insertion depth.
   6. State estimation with a Kalman filter: Kalman, a constant-velocity
      filter tracking a tool from noisy position measurements, compared with
      the raw readings.
   7. Feedback control and time delay: Pid, a joint with PID control
      (derivative on measurement, anti-windup) and adjustable measurement
      delay. The slider shows a stable loop becoming unstable as the delay
      grows.
   8. Virtual fixtures and force feedback: Fixture, a forbidden-region
      constraint pushing back on the tool with adjustable stiffness.

9. Bug fixes and stability.
   I reviewed every lesson at extreme slider values and fixed drawings that
   left their panels:
   - Out-of-range values are now clamped and labelled.
   - Inverse kinematics no longer flips its elbow at the edge of reach.
   - The Fourier chain now starts at the path's centre.
   - The tremor filter no longer has a start-up jump.
   - Instrument paths outside the body are reported as a code error.
   - A better random-number generator (mulberry32) replaced a correlated one.

10. Grounding the lab in real data.
    I asked AI to check every lesson against published research and switched to real
    units (micrometres, millimetres, degrees, milliseconds):
    - Tremor: physiological tremor is about 50-200 micrometres RMS at
      8-12 Hz, and intended movement stays below about 1 Hz.
    - Motion scaling: published studies used ratios like 2.5:1 and 7:1.
    - Tracking noise: electromagnetic and optical trackers are accurate to
      about 0.1-0.7 mm.
    - Instruments: laparoscopic shafts are 33-45 cm long.
    - Delay: the 2001 transatlantic remote surgery had 155 ms of delay,
      performance holds up to about 100 ms, and it degrades badly near 1 s.
    - Haptics: about 3.3 N of force for common haptic devices.
    - Robot speed: joint speed limits are similar to UR5 arms'.
    Each readout reports results in these units, so the exercises teach
    numbers that match practice.

11. Preparing the app to be a public website.
    - Deployment: the CodeMirror editor is bundled locally, so the site needs
      no other sites. It's plain static files that work from a subfolder (for
      example GitHub Pages), with metadata for search and sharing.
    - Slider labels: each drawing can name its slider with a
      "// slider: <label>" comment and a slider_value function, so the value
      shown is in real units (for example "cutoff (Hz) = 6").
    - A timeline under the controls to scrub through the animation, with the
      current and total time.
    - Rename and delete for the user's own drawings, with confirmation.
    - Share links that carry a drawing's code in the address, and a button
      to save the frame as a PNG.
    - Errors: the error's line is highlighted in the editor, and clicking
      the error jumps to it.
    - Keyboard shortcuts: Space, R, M, E and the arrow keys. They're ignored
      while typing.
    - A first-visit tip, a one-column layout for phones, labels for screen
      readers, visible keyboard focus, higher-contrast text, and starting
      paused for users who prefer reduced motion.
    - Heavy fractals (Mandelbrot and Clifford) compute in a Web Worker, with
      a "computing..." badge, so the page stays responsive while sliders
      move.
