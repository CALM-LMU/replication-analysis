# README

## System Requirements

The scripts should run on any operating system, though they were only tested on Windows 10 and Linux.

The following tools are required:
* ilastik (1.3.3post3, https://www.ilastik.org)
* Anaconda Python distribution (2020.07, https://www.anaconda.com)
* read_lif (install via ```pip install read_lif```)

All Python code is in the form of Jupyter notebooks that can be run in the Browser after starting Jupyter (```jupyter notebook``` or ```jupyter lab``` in a terminal). **Not all cells in the notebooks are necessary, so we suggest to run them one after another**, skipping sections marked as ```test``` or ```old data```.


## 1) data conversion and metadata extraction

Our raw files are Leica LIF files containing multiple image series, each a 3-color confocal z-stack.
Using ```extract_metadata_from_lif.ipynb```, the metadata (especially pixel size) is read from the .lif files and saved in JSON format.
In this script, the input .lif images (```files```) and the output path (```outfile```) can be specified.

Furthermore, we have a set of manual crops around single cells, saved as 3-channel TIFF stacks. These are placed in subfolders of the folder containing the LIF files, with the folder name specifying the S-phase stage.
Cropped images follow the naming convention: ```[original LIF filename] - [Series name]-[index].tif```
Using ```replication_data_export.ipynb```, we generate single channel TIFF stacks named ```[channel name]_[stage]_[chase duration]_[index].tif``` and split the metadata generated earlier into a ```metadata_[stage]_[chase duration]_[index].json``` for each crop.
In the notebook, the folder containing the raw LIF files and TIF crops (```rootdir```), the folder to save results to (```outdir```), the channel names (```channel_names```) and the metadata file generated above (```metadata_file```) can be specified.

## 2) semantic segmentation and manual annotation using ilastik

Using the singlechannel images created above, in ilastik, we created 3 pixel classification projects for the following tasks:
* 3-class segmentation (background-nuclear border-nuclear interior) using the DAPI channel
* 2-class segmentation (background-RFi) for the EdU and PCNA channels

The segmentation results (probabilites) for each file were exported as HDF5 files (dataset name ```probs```) to subfolders (```classification_[channel name]```) of the single-channel TIFF folder under the same name as the input TIF file.

Using another 2-class pixel classification project, we manually annotated the center of each nucleus in the DAPI images as foreground. The manual labels are exported as HDF5 to a subfolder called ```markers```.

## 3) Feature extraction

Using ```replication_feature_extraction.ipynb```, an instance segmentation is performed and features for RFi are extracted and saved in tabular form.
Since the pixel sizes in our images were not equal, we need the metadata JSON files (```metadata_files```) to use equal dimensions in the features.
For the feature extraction, if the steps above were followed, only the marker folder (```marker_dir```) has to be changed in the script.

Fine adjustment of parameters (e.g. blur radii, quantiles for normalization, ...) could be done in the function definitions for ```segment()``` and ```get_features()```.

The path of the output CSV files can be specified in the ```df_edu.to_csv()``` and ```df_agg.to_csv()``` calls.

## 4) Visualization and analysis on get_features

Code for plotting histograms of RFi volume and Distance from periphery can be found at the end of ```replication_feature_extraction.ipynb```.

Code for t-SNE visualization, S-phase stage classification, feature importance assessment and creation of boxplots for single features can be found in ```replication_cluster_feature_importance.ipynb```. The path to the CSV file containing aggregated features (```df_agg = pd.read_csv(...)```) or, alternatively, single RFi features (```df_edu = pd.read_csv(...)```) can be specified in the code.
