Terrain for Panda3D
===================

Just throwing some academic papers at the GPU to see what sticks.


Current Model
-------------

* Hypermodel
  * `boundary_condition`: Whether the edges of the simulation are...
    * `OPEN`: Water flows over the edge as if the terrain and water
      heights were `0.0` beyond it.
    * `CLOSED`: No water flows beyond the edge.
    * `WRAPPING`: Water flowing out of one edge of the map gets added on
      the opposite edge.
* Map Data
  * `terrain_height`: Height of the terrain
  * `water_height`: Current height of the water
  * `water_influx`: Water influx rate to simulate e.g. rain, or springs.
* Intermediate Maps
  * `water_height_after_influx`
  * `water_crossflux`
  * `water_height_after_crossflux`
  * `water_height_after_evaporation`
* Scalar model parameters
  * `cell_distance`: Distance between centers of neighboring cells, default `1.0`
  * `pipe_coefficient`:  `gravity g * pipe cross area A / pipe length l`, default `98.1`
  * `evaporation_constant`: Percentage of water that would evaporate in
    one second if the amount of evaporation was constant. `0.0` for no
    evaporation. Default `0.01`
* Frame parameters
  * `dt`: time step. The amount of time for which the simulation will be
    advanced.
* Process
  * Influx of water: Add new water to the simulation.
    `water_height_after_influx = water_height + water_influx * dt`
  * Calculate outflux
    For each direction/channel: `water_outflux = current_outflux + height_difference * hight_difference * pipe_coefficient * dt`, lower border of `0.0`.
    If the amount of outflowing water is greater than that of water
    actually present in this cell, scale the outflux so that exactly
    zero water remains.
  * Crossflux of water
    `water_height_after_crossflux = water_height_after_influx - own water_crossflux + neighbors' water_crossflux to this cell`
  * Evaporate water: Remove a fraction of water from each cell.
    `water_height_after_evaporation = water_height_after_crossflux * (1 - evaporation_constant * dt)`
  * Update main data
    `waterHeight = waterHeightAfterEvaporation`


TODO
----

* Bug: Why is water leaking out of a CLOSED/WRAPPING simulation?
* Aesthetics
* More papers


Papers
------

* "Fast Hydraulic Erosion Simulation and Visualization on GPU": https://xing-mei.github.io/files/erosion.pdf
  * Overview
    * Influx of water
    * Calculating the pressure-based outflux from each cell to its neighbors
    * Update water height
    * Calculate water velocity
    * Erode material from terrain into water, or deposit it
    * Transport sediment through water flow
    * Evaporate water
  * Map Data
    * `w`: Water influx rate during this frame
    * `b`: Terrain height
    * `d`: Water height
    * `s`: Suspended sediment amount
    * `f`: Outflow flux
    * `v`: Velocity field
  * Scalar model parameters
    * `g`: Gravity
    * `l`: Length of the pipe connecting cells
    * `A`: Cross-section of the pipe connecting cells
    * `Ks`: Dissolving constant
    * `Kd`: Deposition constant
    * `Ke`: Evaporation constant
  * Frame parameters
    * `dt`: time step.
  * Intermediate Maps
    * `d1`: Water height after the initial influx
    * `d2`:
  * Process
    * DONE: Increase water
    * DONE: Compute outflux flow; We treat neighboring cells as if they were connected by pipes.
    * DONE: Update water surface
    * Update velocity field
      Consider only vertical flow.
      `u` / `v` are velocity in X / Y direction
      For X and Y direction separately:
      * delta Water `dWX` = sum up the delta flows with the neighbors.
      * equation: `dWX` = distance Y (???) * (average of `d1` and `d2`) * u
      * derive u (and v accordingly for `dWY`).
      For simulation to be stable, dt * u/v velocity <= distance to neighbor
    * Erosion-deposition
      Note: local tilt angle may require setting a lower bound, otherwise sediment transport capacity will be near zero at low tilt.
      sediment transport capacity `C` = scaling factor * sin(local tilt angle) * |velocity|
      if sediment transport capacity > suspended sediment amount:  # add soil to water
          dissolved amount = Ks * (C - s)
          new terrain height = b - dissolved amount
          intermediate sediment amount `s1` = s + dissolved amount
      else:  # deposit sediment
          deposited amount = Kd * (s - C)
          new terrain height = b + deposited amount
          intermediate sediment amount `s1` = s - deposited amount
    * Transport sediment
      new suspended sediment amount = s1 at position - uv * dt, interpolating the four nearest neighbors
    * DONE: Evaporate water
* "Fast Hydraulic and Thermal Erosion on GPU": http://diglib.eg.org/bitstream/handle/10.2312/EG2011.short.057-060/057-060.pdf?sequence=1
* "Hydraulic Erosion Simulation on the GPU for 3D terrains": https://www.diva-portal.org/smash/get/diva2:1646074/FULLTEXT01.pdf