# How to consume data

The Data Tailor standalone has multiple ways of use including:

* [GUI Web-App (graphical user-interface)](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide#ID-Using-the-GUI)
* [CLI (command line interface)](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide#ID-Using-the-CLI)
* [Python Library](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide#ID-The-DT-Standalone-APIs)
* [EUMDAC](https://user.eumetsat.int/resources/user-guides/eumetsat-data-access-client-eumdac-guide#ID-Data-Tailor-Standalone-in-EUMDAC)

For more information, please refer to the [Data Tailor Standalone Guide](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide). 

## Example

> 💡 The `HRSEVIRI` product (`MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA.zip`) is downloaded and untouced. The download is done directly from the Data Store, using `eumdac`. 

> 💡 `eumdac` is included in your Data Tailor instance. An example command for downloading data with it is: `eumdac download -c EO:EUM:DAT:MSG:HRSEVIRI --limit 1`

Above command is downloading latest available product from HRSEVIRI collection.

Below is a small example of using the application with CLI. 

1. Activate the conda environment:

```bash
conda activate epct-desktop
```

2. Create a `chain.yaml` file:
```bash
product: HRSEVIRI
format: netcdf4
projection: geographic
roi: western_europe
filter:
  bands:
    - channel_9
    - channel_10
    - channel_11
```

3. Run your customisation using the command below:
```bash
epct run-chain -f chain.yaml MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA.zip
```

The output would like like:
```
2024-06-13 10:05:25 - PROCESSING.chain_runner[291] - INFO - Start process "4c15df87"
2024-06-13 10:05:25 - PROCESSING.chain_runner[241] - INFO - WORKER: localhost
2024-06-13 10:05:25 - PROCESSING.chain_runner[242] - INFO - PID: 17361
2024-06-13 10:05:25 - PROCESSING.chain_runner[243] - INFO - backend: epct_gis_msg
2024-06-13 10:05:25 - PROCESSING.chain_runner[244] - INFO - user: None
2024-06-13 10:05:28 - PROCESSING.preprocessing[677] - INFO - Processing details - product: HRSEVIRI chain-name: None chain-details: -product: HRSEVIRI -format: NetCDF4 (simplified) -projection: Geographic / Plate-Carree -roi: Western Europe -filter: Custom
2024-06-13 10:05:28 - PROCESSING.preprocessing[683] - INFO - ROI name: Western Europe - NSWE: [67, 35, -23, 16] - process: 4c15df87
2024-06-13 10:05:28 - PROCESSING.preprocessing[684] - INFO - Input products: MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA.nat
2024-06-13 10:05:28 - PROCESSING.epct_gis[742] - INFO - Expected steps: 6
2024-06-13 10:05:28 - PROCESSING.epct_gis[744] - INFO - Starting step "IMPORT" 1/6 ...
2024-06-13 10:05:28 - PROCESSING.vrt[90] - INFO - Command line and its output ...
 
gdal_translate -of VRT -a_srs "+proj=geos +h=35785831 +lon_0=0.0 +ellps=GRS80 +no_defs" -a_ullr -5565747.7 5565747.7 5565747.7 -5565747.7 -b 9 -b 10 -b 11  RAD:/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/decompressed_data/MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA.nat /usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA.vrt
 
2024-06-13 10:05:28 - PROCESSING.epct_gis[397] - INFO - a preliminary ROI is applied to the intermediate VRT
2024-06-13 10:05:28 - PROCESSING.vrt[90] - INFO - Command line and its output ...
 
gdalwarp -cutline /usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/roi.shp -crop_to_cutline -cblend 10 -r near --config OGR_ENABLE_PARTIAL_REPROJECTION YES /usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA.vrt /usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA_roi.vrt
 
2024-06-13 10:05:28 - PROCESSING.epct_gis[747] - INFO - ... step "IMPORT" finished!
2024-06-13 10:05:28 - PROCESSING.epct_gis[754] - INFO - Starting step "FILTER" 2/6 ...
2024-06-13 10:05:28 - PROCESSING.epct_gis[758] - INFO - ... step "FILTER" finished!
2024-06-13 10:05:28 - PROCESSING.epct_gis[754] - INFO - Starting step "PROJECTION" 3/6 ...
2024-06-13 10:05:28 - PROCESSING.vrt[90] - INFO - Command line and its output ...
 
gdalwarp -overwrite --config CPL_MAX_ERROR_REPORTS 1 -of GTiff -co BIGTIFF=IF_SAFER -co COMPRESS=DEFLATE -ot Float32 -t_srs "EPSG:4326" --config THRESHOLD 0.005 -r near -srcnodata -1000.0 "/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/MSG3-SEVI-MSG15-0100-NA-20240613095742.896000000Z-NA_roi.vrt" "/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/intermediate_warped.tif"
 
2024-06-13 10:05:32 - PROCESSING.epct_gis[647] - INFO - ... step "PROJECTION" finished!
2024-06-13 10:05:32 - PROCESSING.epct_gis[754] - INFO - Starting step "ROI" 4/6 ...
2024-06-13 10:05:32 - PROCESSING.vrt[90] - INFO - Command line and its output ...
 
gdalwarp -overwrite --config CPL_MAX_ERROR_REPORTS 1 -te -23.0 35.0 16.0 67.0 -te_srs "EPSG:4326" --config THRESHOLD 0.005 -r near "/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/intermediate_warped.tif" "/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/intermediate_warped_cut.tif"
 
2024-06-13 10:05:32 - PROCESSING.epct_gis[647] - INFO - ... step "ROI" finished!
2024-06-13 10:05:32 - PROCESSING.epct_gis[754] - INFO - Starting step "FORMAT" 5/6 ...
2024-06-13 10:05:32 - PROCESSING.vrt[90] - INFO - Command line and its output ...
 
gdal_translate -of netCDF -co FORMAT=NC4 -ot Float32 "/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/intermediate_warped_cut.tif" "/usr/local/miniconda/envs/epct-desktop/var/cache/epct/EPCT_HRSEVIRI_FPC_4c15df87/OUTPUTS/EPCT_HRSEVIRI_FPC_4c15df87.nc"
 
2024-06-13 10:05:32 - PROCESSING.epct_gis[647] - INFO - ... step "FORMAT" finished!
2024-06-13 10:05:32 - PROCESSING.postprocessing[468] - INFO - Starting step "POST-PROCESSING" 6/6 ...
2024-06-13 10:05:32 - PROCESSING.postprocessing[493] - INFO - ... step "POST-PROCESSING" finished!
2024-06-13 10:05:32 - PROCESSING.postprocessing[495] - INFO - output-product: ./HRSEVIRI_20240613T094510Z_20240613T095742Z_epct_4c15df87_FPC.nc
2024-06-13 10:05:32 - PROCESSING.postprocessing[503] - INFO - customisation time: 7 - process: 4c15df87
2024-06-13 10:05:32 - PROCESSING.postprocessing[504] - INFO - *** STOP PROCESSING - Status DONE ***
 
processing-time: 6.8 [sec]
```

## Resources

- [How to initialize the environmet](../how-to/how-to-initialize-environment.md )
- [Data Tailor Standalone Guide](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide)