# omepslid
Script for loading Murphy lab image collections into OMERO.servers.

# Upload Scripts

## Installation (Mac)
1. Install [homebrew](http://brew.sh/).  
`$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
2. Tap the OMERO homebrew tap.  
`$ brew tap ome/alt`
3. Install python through homebrew. This python will be located in `/usr/local/bin/`.  
`$ brew install python`
4. Install OMERO's version of [Ice](https://zeroc.com/products/ice) through homebrew with python bindings.  
`$ brew install zeroc-ice35 --with-python`
5. Install [numpy](http://www.numpy.org/) (this must be installed first, at least before javabridge).  
`$ /usr/local/bin/pip install numpy`
6. Install the rest of the required python packages: [Pillow](http://python-pillow.org/), [javabridge](https://pypi.python.org/pypi/javabridge), and [python-bioformats](https://pypi.python.org/pypi/python-bioformats).  
`$ /usr/local/bin/pip install Pillow javabridge python-bioformats`
7. Download and unzip [OMERO.py](https://downloads.openmicroscopy.org/omero/5.2.2/#py) (link might not be latest version).

## Usage (Mac)
Before running any of the scripts, you need to set your `PYTHONPATH` environment variable to know about OMERO.py:  
`$ export PYTHONPATH=$PYTHONPATH:/path/to/OMERO.py/lib/python`

Then, you can run any of the scripts using the homebrew python:  
`$ /usr/local/bin/python /path/to/script <arg> ...`

# Synthetic Image Sampler

## Download Script
Make sure the `OMERO_ROOT` variable is set to the url of the right OMERO server (defaults to MurphyLab's OMERO server).

```
$ python sample_synthetic_images.py -h
usage: sample_synthetic_images.py [-h] [--dimensionality DIMENSIONALITY]
                                  [--cell-line CELL_LINE]
                                  [--downsampling-factor DOWNSAMPLING_FACTOR]
                                  [--number-of-images NUMBER_OF_IMAGES]
                                  [--channel0-model-class CHANNEL0_MODEL_CLASS]
                                  [--channel0-model-type CHANNEL0_MODEL_TYPE]
                                  [--channel1-model-class CHANNEL1_MODEL_CLASS]
                                  [--channel1-model-type CHANNEL1_MODEL_TYPE]
                                  [--channel2-model-class CHANNEL2_MODEL_CLASS]
                                  [--channel2-model-type CHANNEL2_MODEL_TYPE]
                                  [--output-dir OUTPUT_DIR]

Download synthetic images from an OMERO server

optional arguments:
  -h, --help            show this help message and exit
  --dimensionality DIMENSIONALITY
                        2D or 3D (default: 2D)
  --cell-line CELL_LINE
                        type of cell line (default: HeLa)
  --downsampling-factor DOWNSAMPLING_FACTOR
                        amount to downsample the images by using bicubic
                        interpolation (default: None)
  --number-of-images NUMBER_OF_IMAGES
                        number of images to download (default: 1)
  --channel0-model-class CHANNEL0_MODEL_CLASS
                        model class for channel 0 (e.g. framework) (default:
                        None)
  --channel0-model-type CHANNEL0_MODEL_TYPE
                        model type for channel 0 (e.g. cylindrical,
                        diffeomorphic) (default: None)
  --channel1-model-class CHANNEL1_MODEL_CLASS
                        model class for channel 1 (e.g. framework) (default:
                        None)
  --channel1-model-type CHANNEL1_MODEL_TYPE
                        model type for channel 1 (e.g. ratio, diffeomorphic)
                        (default: None)
  --channel2-model-class CHANNEL2_MODEL_CLASS
                        model class for channel 2 (e.g. vesicles, microtubule)
                        (default: None)
  --channel2-model-type CHANNEL2_MODEL_TYPE
                        model type for channel 2 (e.g. gmm, network) (default:
                        None)
  --output-dir OUTPUT_DIR
                        directory to save the images in (default: .)
```

If downsampling is requested, the image is resized using bicubic interpolation to the nearest size that would result in downsampling less than or equal to the amount requested in each axis. For example, an 11x14 image with requested downsampling of 2 would be resized to 6x7, with a downsampling factor of 1.83 in the x dimension and 2 in the y dimension.

Metadata such as resolution and PhysicalSize attributes are updated appropriately. In addition, metadata containing the details of the downsampling requested and performed is stored in the OME XML embedded metadata. Under the StructureAnnotations element, the following DoubleAnnotation elements are stored (with corresponding Description and Value sub-elements):

| ID  | Description |
| --- | ----------- |
| Annotation:RequestedDownsamplingFactor | downsampling factor requested |
| Annotation:XDownsamplingFactor | downsampling factor performed in the x dimension |
| Annotation:YDownsamplingFactor | downsampling factor performed in the y dimension |
| Annotation:OriginalPhysicalSizeX | original length of a pixel in the x dimension before downsampling (see the PhysicalSizeX attribute of the Pixels element) |
| Annotation:OriginalPhysicalSizeY | original length of a pixel in the y dimension before downsampling (see the PhysicalSizeY attribute of the Pixels element) |

Example OME XML might look like:
```
<OME ...>
  <Image ...>
    ...
  </Image>
  <StructuredAnnotations>
    ...
    <DoubleAnnotation ID="Annotation:RequestedDownsamplingFactor">
      <Description>This element contains the requested downsampling factor the client passed to the OMERO server.</Description>
      <Value>1.05</Value>
    </DoubleAnnotation>
    <DoubleAnnotation ID="Annotation:XDownsamplingFactor">
      <Description>This is the amount by which the x dimension of the image was downsampled by the OMERO server.</Description>
      <Value>1.0375</Value>
    </DoubleAnnotation>
    <DoubleAnnotation ID="Annotation:YDownsamplingFactor">
      <Description>This is the amount by which the y dimension of the image was downsampled by the OMERO server.</Description>
      <Value>1.03448275862</Value>
    </DoubleAnnotation>
    <DoubleAnnotation ID="Annotation:OriginalPhysicalSizeX">
      <Description>This is the PhysicalSizeX attribute of the Pixels object before the downsampling was performed on the image.</Description>
      <Value>0.049</Value>
    </DoubleAnnotation>
    <DoubleAnnotation ID="Annotation:OriginalPhysicalSizeY">
      <Description>This is the PhysicalSizeY attribute of the Pixels object before the downsampling was performed on the image.</Description>
      <Value>0.049</Value>
    </DoubleAnnotation>
  </StructuredAnnotations>
</OME>
```

## OMERO Webapp Installation
1. On the OMERO server, install the needed dependencies.  
`$ sudo apt-get install python-lxml`  
`$ sudo pip install tifffile`
2. Make a new webapps directory in the home directory (for example).  
`$ mkdir ~/webapps`
3. Add this directory to your PYTHONPATH on login by adding the following line to your `~/.profile` (in this case, the user is named 'omero').  
`export PYTHONPATH=$PYTHONPATH:/home/omero/webapps`
4. Source `~/.profile` so the changes take effect in the current session.  
`$ source ~/.profile`
5. Move the syntheticsampler directory into the newly created webapps directory.  
`$ mv /path/to/syntheticsampler/ ~/webapps/`
6. In `syntheticsampler/views.py`, update the `SYNTHETIC_DATASETS` variable according to the instructions in the file.
7. Tell OMERO about the webapp.  
`$ omero config append omero.web.apps '"syntheticsampler"'`
8. Restart the OMERO web server.  
`$ omero web restart`
