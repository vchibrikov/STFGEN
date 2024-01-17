# STFGEN.py
STFGEN (Single-Type Fiber GENerator) is a hardcoded Python script for the generation of random fiber network according to input parameters. Current script allow to adjust input data on fiber structural (number of fibers, its length range, diameter, angle displacement, etc.) and spatial (volume occupation) properties. and provide output data of 3D bead coordinates in .xyz file format, as well as output information on generator run in .txt file format. 

- Visual Studio Code version: 1.85.1
- Python version: 3.7.7.

> Warning! There are no guaranties that this software will run on your machine.

Following script consist of several principle blocks of the code, which are explained below.
### Importing necessary libraries
 ```
# Import libraries
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import random
import os
from datetime import datetime
 ```
### Defining date and time 
> Here, it is done for the purpose of proper naming convention of output files.
```
# Get the current date and time
current_date = datetime.now().strftime('%H.%M.%S_%d.%m.%Y')
```
### Defining folders for output files
> Make sure to modify this segment prior to the code use
```
# Define output folders
output_xyz_folder = ''
output_stats_folder = ''
output_length_folder = ''
```
### Defining filename for the output .xyz file
```
# Define .xyz filename and output folder
output_xyz_filename = ['STFGEN', '_', current_date, '_NETWORK_VC.xyz']
output_xyz_filename = ''.join(output_xyz_filename)
```
### Defining input variables
> Make sure to modify this segment prior to the code use.
```
# Define variables
box_length = # Length of simulation box
box_width = # Width of simulation box
box_thickness = # Thickness of simulation box
sphere_radius = # Radius of the single bead
fiber_length_mean = # Mean length of modeled fibers. Experimental value
fiber_length_sd = # Standard deviation of modeled fibers. Experimental value
volume_occupied = 0 # Minimum (starting) volume of simulation box occupied (always 0)
max_volume_occupied = # Maximum volume of simulation box occupied (0 < max_volume_occupied < 1)
cutoff_distance = # Minimum distance between the beads of two different fibers
sphere_overlay = sphere_radius # Distance, on which beads of the same fiber are allowed to overlay, mimicking bead-bead interaction
bead_bead_angle_mean = # Mean of an angle between consequtive beads of fiber (in deg.)
bead_bead_angle_sd = # Standard deviation of an angle between consequtive beads of fiber (in deg.)
fiber_index = 0 # Starting fiber number in box
bead_volume = (4/3) * np.pi * sphere_radius**3 # Volume of single bead
overlay_volume = ((np.pi / 12) * (4 * sphere_radius + sphere_radius) * (2 * sphere_radius - sphere_radius)**2) # Area of bead-bead overlay of two similar fibers
bead_bead_overlay_ratio = overlay_volume / (bead_volume) # Area of bead-bead overlay as a ratio to bead area
total_beads = int((box_length * box_width * box_thickness * max_volume_occupied) / (bead_volume - bead_volume * bead_bead_overlay_ratio)) # Total number of beads in box according to its radius and total volume occupied
total_fibers = int(total_beads / (fiber_length_mean / sphere_radius)) # Total number of fibers in box according to its radius and total volume occupied
```
### Generate normal distribution of fiber length
> This block of code generates normal distribution of fiber length according to the input values of its mean and standard deviation. Additionally, code ensures that the minimum fiber length is the diameter of one bead. At the very end of the block, array of generated values is sorted from highest to lowest. It allows to efficiently generate longer fibers within the network, allowing to overcome sterical hindrances, which may occur while shorter fibers are already in simulation box.
```
random_fiber_length = np.random.normal(0, 1, total_fibers)
random_fiber_length = random_fiber_length * fiber_length_sd + fiber_length_mean
random_fiber_length = np.clip(random_fiber_length, 0, None)
random_fiber_length = np.maximum(random_fiber_length, sphere_radius)
random_fiber_length = np.round(random_fiber_length / sphere_radius) * sphere_radius
random_fiber_length[random_fiber_length < sphere_radius] = sphere_radius
random_fiber_length = np.sort(random_fiber_length)[::-1]
```
### Initializing arrays to store bead positions and fiber indices
```
# Initialize arrays to store sphere positions and fiber indices
x_positions = []
y_positions = []
z_positions = []
fiber_indices = []
```
### Defining function to check overlay of the beads of different values
> Current script allows overlay of the beads of the same fiber, assuming it as an interbead junction. However, bead overlay of the different fibers if restricted, so that the minimum distance between the beads of different fibers equals to two radii and some cutoff distance. Cutoff distance defines closes distance between the bead edges.
```
# Function to check beads of different fibers overlap
def check_overlap(pos1, pos2, radius, same_fiber):
    if same_fiber:
        return False  # Allow overlap for the same fiber
    else:
        distance = np.linalg.norm(pos1 - pos2)
        return distance < radius * 2 + cutoff_distance # Interfiber bead overlay restricted
        # return distance < 0 # Allow bead overlay
```
### While loop for bead and fiber generating up to some volume of the simulation box to be occupied
```
# Generate fiber network until desired volume is occupied
while volume_occupied < max_volume_occupied:
    print(f'Starting generation for volume occupied: {volume_occupied}')
```
### Defining function to check overlay of the beads of different values
> Current script allows overlay of the beads of the same fiber, assuming it as an interbead junction. However, bead overlay of the different fibers if restricted, so that the minimum distance between the beads of different fibers equals to two radii and some cutoff distance. Cutoff distance defines closes distance between the bead edges. To allow bead overlay, use return command in commented line.
```
# Function to check beads of different fibers overlap
def check_overlap(pos1, pos2, radius, same_fiber):
    if same_fiber:
        return False  # Allow overlap for the same fiber
    else:
        distance = np.linalg.norm(pos1 - pos2)
        return distance < radius * 2 + cutoff_distance # Interfiber bead overlay restricted
        # return distance < 0 # Allow bead overlay
```
### Starting fiber generation
> Current block takes the fiber length from a previously generated array, calculates beads number within a fiber, and defines some random position in 3D for the first bead.
```
    # Start generate beads of each fiber within array of fiber length
    for fiber_length in random_fiber_length:

        # Define number of beads in each fiber
        n_beads = int(fiber_length / sphere_radius) 

        # Generate random position for the first bead in the fiber
        fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                     random.randint(sphere_radius, box_width - sphere_radius),
                                     random.randint(sphere_radius, box_thickness - sphere_radius)]])
```
### Defining bead within the boundaries of simulation box
> Current block assume first bead should be located within the simulation box.
```
        if (fiber_positions < sphere_radius).any() or (fiber_positions > box_length - sphere_radius).any():

            # Generate random position for the first bead in the fiber
            fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                        random.randint(sphere_radius, box_width - sphere_radius),
                                        random.randint(sphere_radius, box_thickness - sphere_radius)]])
```
### Handling bead overlay
> While generated bead overlay existing one, some new random position is generated for that bead.
```
        # Collision detection for the first bead of each fiber
        while any(check_overlap(fiber_positions[0], np.array([x_positions[i], y_positions[i], z_positions[i]]), sphere_radius, fiber_indices[i] == fiber_index) for i in range(len(x_positions))):
            
            # Generate random position for the first bead in the fiber
            fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                        random.randint(sphere_radius, box_width - sphere_radius),
                                        random.randint(sphere_radius, box_thickness - sphere_radius)]])
```
### Handling bead overlay
> While generated bead overlay existing one, some new random position is generated for that bead.
```
        # Collision detection for the first bead of each fiber
        while any(check_overlap(fiber_positions[0], np.array([x_positions[i], y_positions[i], z_positions[i]]), sphere_radius, fiber_indices[i] == fiber_index) for i in range(len(x_positions))):
            
            # Generate random position for the first bead in the fiber
            fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                        random.randint(sphere_radius, box_width - sphere_radius),
                                        random.randint(sphere_radius, box_thickness - sphere_radius)]])
```








Figure representing the logics of bead generation in terms of spherical coordinate system.
![Figure 8](https://github.com/vchibrikov/STFGEN/assets/98614057/f0aae5da-48e9-4d87-8651-e2b786281d79)


Figure of randomly generated first bead of each fiber at 5% volume occupation of fibers, with beads of the radius of 25 units. X-Y axis view.
![Figure 7](https://github.com/vchibrikov/STFGEN/assets/98614057/0086d7cb-f772-470b-b687-f02e4423e7dd)

Figure of randomly generated fiber networks at 1% volume occupation with beads of the radius of 25 units. X-Y axis view.
![Figure 1](https://github.com/vchibrikov/STFGEN/assets/98614057/b81a64a0-bb66-4b5e-a116-083f0e4dafe2)

Figure of randomly generated fiber networks at 1% volume occupation with beads of the radius of 25 units. X-Z axis view.
![Figure_2](https://github.com/vchibrikov/STFGEN/assets/98614057/f374494a-84d3-4160-be29-0d11c051be39)

Figure of randomly generated fiber networks at 1% volume occupation with beads of the radius of 25 units. Y-Z axis view.
![Figure 3](https://github.com/vchibrikov/STFGEN/assets/98614057/fc560869-99dd-4530-af24-135e04064374)

Figure of randomly generated fiber networks at 5% volume occupation with beads of the radius of 25 units. X-Y axis view.
![Figure 4](https://github.com/vchibrikov/STFGEN/assets/98614057/7a9df709-bd09-443b-a548-ea7fc96eff2a)

Figure of randomly generated fiber networks at 5% volume occupation with beads of the radius of 25 units. Z-X axis view.
![Figure 5](https://github.com/vchibrikov/STFGEN/assets/98614057/8dfeb2c4-0a9b-4f87-b992-b6ef63faab42)

Figure of randomly generated fiber networks at 5% volume occupation with beads of the radius of 25 units. Z-Y axis view.
![Figure 6](https://github.com/vchibrikov/STFGEN/assets/98614057/230dc2e0-0345-470c-8f09-229d60381e0e)

> Limitations:
> 1. It is recommended to generate fiber network, in which bead radius is no more than 1% of the length of simulation box
> 2. Despite parameters of the simulation box are provided, current script does allow generated fibers to run over the box boundaries. The volume of the beads that run over the boundaries of simulation box are not considere in current script.










