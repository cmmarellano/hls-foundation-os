# Multi-Temporal Cloud Gap Imputation With HLS Data Across CONUS

## Dataset Description

### Dataset Summary

This dataset contains temporal Harmonized Landsat and Sentinel-2 (HLS) imagery of diverse land covers across the Contiguous United States (CONUS) for the year 2022 along with binary cloud masks for the same area and year.

### Dataset Structure
```
hls-multi-temporal-cloud-gap-imputation/
├── chip_catalog.csv
├── cloud_catalog.csv
├── train/
│ ├── hls_chips/
│ ├── cloud_masks/
├── test/
│ ├── hls_chips/
│ ├── hls_chips_masked/
```
## Data Catalogs
Data catalogs containing metadata for both HLS scenes and cloud masks are provided. The catalog for HLS scenes is provided in `chip_catalog.csv` and the catalog for cloud masks is provided in `cloud_catalog.csv`.

### Cloud Catalog

`cloud_catalog.csv` contains the following columns:
| Column |  Contains  |
|---------|--------|
| cloud_mask_id       |  'chip_XXX_YYY' corresponding to the file name  |
| cloud_pct       |  ratio of cloudy pixels within the chip, ranges from 0.0 to 1.0 |
| usage       |  'train' or 'validate' |
| bin       |  which of 10 bins the cloud mask fall within, e.g. 0.1-0.2  |

### HLS Chip Catalog

`chip_catalog.csv` contains the following columns:
| Column |  Contains  |
|---------|--------|
| chip_id       |  'chip_XXX_YYY' corresponding to the file name  |
| chip_x       |  x coordinate of bounding box centroid |
| chip_y       |  y coordinate of bounding box centroid   |
| tile       |  HLS tile ID   |
| valid_first       |  count of valid pixels in the first time step  |
| valid_second       |  count of valid pixels in the second time step  |
| valid_third       |  count of valid pixels in the thrid time step  |
| bad_pct_first       |  percent of invalid pixels in the first time step |
| bad_pct_second       |  percent of invalid pixels in the second time step   |
| bad_pct_third       |  percent of invalid pixels in the third time step   |
| first_image_date       |  date of first time step in `YYYY/mm/dd` |
| second_image_date       |  date of second time step in `YYYY/mm/dd`  |
| third_image_date       |  date of third time step in `YYYY/mm/dd`  |
| bad_pct_max       |  maximum of invalid pixels in all time steps  |
| na_count       |  count of pixels in all time steps with no data |
| usage | 'train' or 'validate' |

Invalid pixels are those which intersect with any QA mask.

## Ground Truth HLS Scenes
The ground truth HLS scenes are stored in GeoTIFF format under `train/hls_chips/` and `test/hls_chips/`. Each GeoTIFF file covers a 224 x 224 pixel area at 30m spatial resolution. Each file contains 18 bands consisting of 6 spectral bands in 3 steps stacked together. The file name structure is `chip_XXX_YYY.tif` where `XXX` and `YYY` refer to row and column of a tile grid imposed on the Continental US. Since the dataset is sampled from this country-wide grid not all `XXX`s and `YYY`s are present in the dataset.

## Masked HLS Scenes for Testing
Testing scenes are pre-masked to ensure that all models are evaluated using the same test set. These scenes are stored in GeoTIFF format under `test/hls_chips`. The file name structure for masked scenes is `chip_XXX_YYY_masked.tif`. Each masked scene corresponds to the ground truth scene with the same value of `XXX_YYY` in the file name. For example, the file `test/hls_chips/chip_373_294.tif` is the ground truth for `test/hls_masked/chip_373_294_masked.tif`, with the latter having values of 0 at cloud-masked locations. Cloud masks are present in all possible combinations of time steps in equal proportion. Possible combinations given time steps t1, t2, and t3 are:

- t1
- t2
- t3
- t1, t2
- t1, t3,
- t2, t3,
- t1, t2, t3

So, for example, 1/7th of test scenes are masked at ONLY t2, and 1/7th at t2 AND t3, etc. for each of the possible combinations.

Cloud masks for the test scenes range from 0.01% coverage to 100% coverage, and are equally sampled from 10 equally sized bins between 0-100%.

## Cloud Masks
The training cloud masks are stored in GeoTIFF format under `train/cloud_masks/`. The file name structure for cloud masks is `chip_XXX_YYY_T_cmask.tif` where `XXX` and `YYY` refer to row and column of a tile grid imposed on the Continental US. `T` refers to the time step of each cloud mask and is meant only to distinguish cloud masks derived from the same location from each other. The intent for training is that these cloud masks are randomly paired with training HLS scenes in all time steps. The distribution of cloud mask coverage for the training set does not correspond to the distribution of cloud mask coverage for the validation set, as the distribution of the latter has been equalized. This may lead to higher validation accuracy if the user chooses not to equalize the training dataset - it is left to the user's discretion.

## Band Order
In each HLS GeoTIFF the following bands are repeated for each of three observations throughout the year:
| Channel |  Name  |  HLS   S30 Band number |
|---------|--------|------------------------|
| 1       |  Blue  |  B02                   |
| 2       |  Green |  B03                   |
| 3       |  Red   |  B04                   |
| 4       |  NIR   |  B8A                   |
| 5       |  SWIR 1  |  B11                   |
| 6       |  SWIR 2  |  B12                   | 

Masks are a stored as a single-band binary image where 1 denotes the presence of the cloud mask and 0 denotes the absence of the cloud mask.

## Dataset Creation

Code used to generate HLS scenes and cloud masks is available [here](https://github.com/ClarkCGA/cloud-gap-filling-td). Code used to generate masked test scenes is available [here](https://github.com/ClarkCGA/gfm-gap-filling-baseline/blob/main/gfm-gap-filling-baseline/gap-filling-baseline/datasets/gapfill.py). `usage='validate'` was used along with default parameters when initializing the dataset using the `gapfill.py` code. Refer to [Seeing Through the Clouds: Cloud Gap Imputation with Prithvi Foundation Model](https://arxiv.org/abs/2404.19609) for further information about the creation and initial use of this dataset.

### Chip Generation and Partitioning
Three HLS scenes were selected between Mar and Sep 2022 with time difference between scenes varying between 1 and 200 days. After filtering for missing values and cloudy pixels, a total of 7,852 cloud-free chips evenly distributed across the CONUS were generated. This set was randomly partitioned into training (80%) and validation (20%) sets, resulting in 6,231 training chips and 1,621 validation chips.

### Cloud Generation and Partitioning
Cloud masks were generated from the same region of CONUS using HLS cloud mask quality flag and exported as a binary layer of cloudy and non-cloudy pixels. This yielded 21,642 cloud masks, of which 1,600 were randomly selected and reserved for validation, resulting in 20,042 training cloud masks

### License and Citation
This dataset is published under a CC-BY-4.0 license. If you find this dataset useful for your application, you can cite it as following:

```
@misc{hls-multi-temporal-cloud-gap-imputation,
    author = {Godwin, Denys and Li, Hanxi (Steve) and Alemohammad, Hamed},
    doi    = {https://doi.org/10.5281/zenodo.11281740},
    title  = {{Multi-Temporal Cloud Gap Imputation With HLS Data Across CONUS}},
    version = {1.0},
    year   = {2024}
}
```
### Contact
For any questions about the dataset, you can contact [Dr. Hamed Alemohammad](mailto:halemohammad@clarku.edu). 


### Funding
This dataset is generated with funding from a grant awarded to [Clark University Center for Geospatial Analytics (CGA)](https://www.clarku.edu/cga/) by NASA. 