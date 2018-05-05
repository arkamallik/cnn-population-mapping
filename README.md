# cnn-population-mapping

This project requires datasets and csv annotations provided by the instructors of CS325B in Stanford. 

The required data and files include:
* Groud truth India village population survey datasets, available at Google Cloud Buckets: 
  - gs://es262-poverty/India_pov_pop.csv
* Landsat-8 and Sentinel-1 Satellite RGB images for each village, also available at Google Cloud Buckets: 
     - gs://es262-poverty/l8_median_india_vis_500x500_*.0.tif
     - gs://es262-poverty/s1_median_india_vis_500x500_*.0.tif
* The metadata info for images and csv are in:
     - data/Readme_poverty.rtf

## Data Preprocessing

The required data should be preprocessed to have the format needed for CNN training, which can be done by modify and run the codes in: 

 1. code/pop_data_preprocess.ipynb 
     - Input:
        - data/annos_csv/India_pov_pop_Feb4.csv
        - data/output.txt: contained all image filenames
     - Output:
        - data/annos_csv/India_pov_pop_Feb4_density_class.csv
        - data/annos_csv/all_paths_density_labels_564k_Feb10.csv
        - data/annos_csv/state24_paths_density_labels_13k_Feb10.csv
        - (data/annos_csv/state24_paths_density_labels_13k_Feb26-NoOverlapDistrict.csv)
        - data/annos_csv/state24_jpgpaths_density_nolaps_12k_Mar6.csv
        - (data/annos_csv/ state24_jpgpaths_density_nolaps_12k_Mar8.csv)
    - Sections:
        - initial cleanup: 
           - Select necessary field of csv
           - remove rows with missing are/lat/long
        - calculate density and class
           - area in units of m2, calculate density as nums/km2
           - Calculate log2 density: pop_density_log2
           - pool the log2 population < 1 and > 13 into same class
           - shift the class to [0,xx] classes
           - => India_pov_pop_Feb4_density_class.csv
        - Village area distribution
           - Used to determine the crop size of image
        - train/val/test partition
           - 70% train, 20% Val, 10% test
        - add tif imge path
           - Relative path for l8_median_india_vis and s1_median_india_vis
           - Takes 15min to add 500k image paths as seperate column
           - => all_paths_density_labels_564k_Feb10.csv
        - Select single state (24)
           - outlier removal of extrme small/big densities
           - df_state24 = df_state24[(df_state24.pop_density_class >= 3) & (df_state24.pop_density_class <= 9)]
           - => state24_paths_density_labels_13k_Feb10.csv
        - Remove Overlaps 
           - jupyter notebook -> Overlap-Solution.ipynb
           - Fill in any required paths and then 'Cell' -> 'Run All'
           - => state24_paths_density_labels_13k_Feb26-NoOverlapDistrict.csv
        - create jpg path column in csv
           - => state24_jpgpaths_density_nolaps_12k_Mar6.csv
           - Then run jpg_convertion.py 
        - Save Nightlight Images 
           - jupyter notebook -> Nightlights.ipynb
           - Fill in any required paths and then 'Cell' -> 'Run All'
           - => state24_jpgpaths_density_nolaps_12k_Mar8.csv
                        
 2. code/jpg_convertion.py (convert satellite TIF image to jpg format)
    - Input: 
        - data/annos_csv/state24_paths_density_labels_13k_Feb10.csv
        - TIF_DIR = '/home/timhu/all_tif’
    - output:
        - JPG_DIR = '/home/timhu/all_jpg’
        - Cropped jpg images saved in JPG_DIR

## Data Loading Test

Once the data are preprocessed, they can be tested and visualized in the notebook:

- tim_input_test.ipynb
    * tif load test
        * Input:
            * data/annos_csv/state24_paths_density_labels_13k_Feb10.csv
            * IMG_DIR = '/home/timhu/all_tif’
        * Output:
            * L8 and S1 cropped images
    * Night light load test
        * Input:
            * all_jpg/nl_median_india_vis/nl_median_india_vis_5x5_453544.0.jpg
        * Output:
            * Nightlight numpy array
    * CNN input test
        * Input:
            * vgg_deeep_combo.py
            * data_input_jpg.py
            * weights/vgg_16.ckpt
            * data/annos_csv/state24_jpgpaths_density_nolaps_12k_Mar8.csv
            * JPG_DIR = '/home/timhu/all_jpg/‘
        * Output:
            * Batch input of S1, L8, Nightlinght, pop, lat, long, village id, etc.
            
## CNN training

This projects uses seperate training script for different combination of datasets and archetectures. The Training script filenames their uses are listed below. To run the train, firstly modify the parameters at the beginning section of the script, and then run it with "python <train_script.py>".

(L8: Landsat-8, S1: Sentinel-1, NL: Nightlight)

 - scripts:
    * code/train_regression.py
        * L8/S1 only: DATA = ‘l8’/’s1’, IMAGE_CHANNEL = 3
        * L8+S1 concat initially: DATA = ‘l8s1’, IMAGE_CHANNEL = 6
    * code/train_shallow_combo_regression.py
        * L8+S1 combined after conv1
    * code/train_deep_combo_regression.py
        * L8+S1 combined after all conv1-conv5
    * code/train_deep_combo_nl_regression.py
        * L8+S1+NL, deep combo
 - parameters to change before training:
    * JPG_DIR: jpg images directory 
    * LOG_DIR: directory that save checkpoints
    * FLAGS: training hyperparameters
    * PRETRAIN_WEIGHTS
    * IMAGE_HEIGHT, IMAGE_WIDTH, IMAGE_CHANNEL
    * ANNOS_CSV 
    * DATA: l8, s1, l8s1, l8s1nl
