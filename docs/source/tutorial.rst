Tutorial
========

Simulation of datasets is split into a number of different commands. Each
command takes a set of command line options or a configuration file in YAML
format.

Show default configuration
--------------------------

The default configuration parameters can be seen by typing the following
command:

.. code-block:: bash

  parakeet.config.show

This will give some output like the following which can then be copied to a
yaml file (e.g. config.yaml):

.. code-block:: bash

  Configuration:
      cluster:
          max_workers: 1
          method: null
      device: gpu
      microscope:
          beam:
              acceleration_voltage_spread: 8.0e-07
              drift: null
              electrons_per_angstrom: 30
              energy: 300
              energy_spread: 3.3e-07
          detector:
              dqe: true
              nx: 4000
              ny: 4000
              pixel_size: 1
          model: null
          objective_lens:
              c_10: 20000
              c_12: 0.0
              c_21: 0.0
              c_23: 0.0
              c_30: 2.7
              c_32: 0.0
              c_34: 0.0
              c_41: 0.0
              c_43: 0.0
              c_45: 0.0
              c_50: 0.0
              c_52: 0.0
              c_54: 0.0
              c_56: 0.0
              c_c: 2.7
              current_spread: 3.3e-07
              inner_aper_ang: 0.0
              m: 0
              outer_aper_ang: 0.0
              phi_12: 0.0
              phi_21: 0.0
              phi_23: 0.0
              phi_32: 0.0
              phi_34: 0.0
              phi_41: 0.0
              phi_43: 0.0
              phi_45: 0.0
              phi_52: 0.0
              phi_54: 0.0
              phi_56: 0.0
          phase_plate: false
      sample:
          box:
          - 4000
          - 4000
          - 4000
          centre:
          - 2000
          - 2000
          - 2000
          coords:
              filename: null
              recentre: false
          ice:
              density: 940
              generate: false
          molecules:
              4v1w: 0
              4v5d: 0
              6qt9: 0
          shape:
              cube:
                  length: 4000
              cuboid:
                  length_x: 4000
                  length_y: 4000
                  length_z: 4000
              cylinder:
                  length: 10000
                  radius: 1500
              type: cube
      scan:
          axis:
          - 0
          - 1
          - 0
          exposure_time: 1
          mode: still
          num_images: 1
          start_angle: 0
          start_pos: 0
          step_angle: 10
          step_pos: auto
      simulation:
          division_thickness: 100
          ice: false
          margin: 100
          padding: 100
          slice_thickness: 3.0


Generate sample model
---------------------

Once the configuration file has been generated a new sample file can be created
with the following command:

.. code-block:: bash

  parakeet.sample.new -c config.yaml


This will result in a file "sample.h5" being generated. This file contains
information about the size and shape of the sample but as yet doesn't contain
any atomic coordinates. The atomic model is added by running the following
command which adds molecules to the sample file. If a single molcule is
specified then it will be placed in the centre of the sample volume. If
multiple molecules are specified then the molecules will be positioned at
random locations in the sample volume. This command will update the "sample.h5"
file with the atomic coordinates but will not generated any new files.

.. code-block:: bash

  parakeet.sample.add_molecules -c config.yaml


Simulate EM images
------------------

Once the atomic model is ready, the EM images can be simulated with the
following commands. Each stage of the simulation is separated because it may be
desirable to simulate many different defocused images from the sample exit wave
for example or many different doses for the sample defocusses image. Being
separate, the output of one stage can be reused for multiple runs of the next
stage. The first stage is to simulate the exit wave. This is the propagation of
the electron wave through the sample. It is therefore the most computationally
intensive part of the processes since the contribution of all atoms within the
sample needs to be calculated.


.. code-block:: bash

  parakeet.simulate.exit_wave -c config.yaml


This command will generate a file "exit_wave.h5" which will contain the exit
wave of all tilt angles. The next step is to simulate the micropscope optics
which is done with the following command:

.. code-block:: bash

  parakeet.simulate.optics -c config.yaml


This step is much quicker as it only scales with the size of the detector image
and doesn't require the atomic coordinates again. The command will output a
file "optics.h5". Finally, the response of the detector can be simulated with
the following command:

.. code-block:: bash

  parakeet.simulate.image -c config.yaml


This command will add the detector DQE and the Poisson noise for a given dose
and will output a file "image.h5".

Other functions
---------------

Typically we cant to output an MRC file for further processing. The hdf5 files
can easily be exported to MRC by the following command:

.. code-block:: bash

  parakeet.export file.h5 -o file.mrc
  
The export command can also be used to rebin the image or select a region of interest. 
