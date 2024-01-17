# STFGEN.py
STFGEN (Single-Type Fiber GENerator) is a hardcoded Python script for the generation of random fiber network according to input parameters. Current script allow to adjust input data on fiber structural (number of fibers, its length range, diameter, angle displacement, etc.) and spatial (volume occupation) properties. and provide output data of 3D bead coordinates in .xyz file format, as well as output information on generator run in .txt file format. 

- Visual Studio Code version: 1.85.1
- Python version: 3.7.7.

> Warning! There are no guaranties that this software will run on your machine.


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










