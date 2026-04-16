# How to initialize the environmet

1. SSH into your Data Tailor instance.
2. Verify the installation by running:
```bash
conda env list
```
The output should look like:

```
# conda environments:
#
base                  *  /usr/local/miniconda
epct-desktop             /usr/local/miniconda/envs/epct-desktop
```
3. Add conda environment to your shell and then restart it:

```bash
conda init
```
The output may look as follows:
```
no change     /usr/local/miniconda/condabin/conda
no change     /usr/local/miniconda/bin/conda
no change     /usr/local/miniconda/bin/conda-env
no change     /usr/local/miniconda/bin/activate
no change     /usr/local/miniconda/bin/deactivate
no change     /usr/local/miniconda/etc/profile.d/conda.sh
no change     /usr/local/miniconda/etc/fish/conf.d/conda.fish
no change     /usr/local/miniconda/shell/condabin/Conda.psm1
no change     /usr/local/miniconda/shell/condabin/conda-hook.ps1
no change     /usr/local/miniconda/lib/python3.11/site-packages/xontrib/conda.xsh
no change     /usr/local/miniconda/etc/profile.d/conda.csh
modified      /home/USERNAME/.bashrc
 
==> For changes to take effect, close and re-open your current shell. <==
```

4. Restart your shell (close and re-open it)
5. Activate the conda environment and test the application
```bash
conda activate epct-desktop
```

The output should resemble something like:
```
epct_version: 3.4.0
etc_dir: /usr/local/miniconda/envs/epct-desktop/etc/epct
workspace_dir: /usr/local/miniconda/envs/epct-desktop/var/cache/epct
customisations_log_dir: /usr/local/miniconda/envs/epct-desktop/var/log/epct
output_dir: /
installed_plugins:
  - epct-plugin-netcdf-generator, 3.1.1
  - epct-plugin-ncarrays, 1.1.0
  - epct-plugin-umarf, 3.2.0
  - epct-plugin-gis, 3.3.0
registered_backends:
  - epct_1d_array
  - epct_2d_array
  - epct_ascatl1szf
  - epct_eps_gome2l1b
  - epct_eps_grasl1
  - epct_eps_iasil1c
  - epct_eps_iasisnd02
  - epct_eps_native
  - epct_eps_native_gome2l1_iasisnd02
  - epct_gis
  - epct_gis_eps
  - epct_gis_eps_grib2
  - epct_gis_fcil2
  - epct_gis_glbsst
  - epct_gis_glbsst_grib2
  - epct_gis_grib2
  - epct_gis_hrit
  - epct_gis_hrit_grib2
  - epct_gis_hrit_png_16_bit
  - epct_gis_msg
  - epct_gis_msg_meteo_bufr
  - epct_gis_msg_png_16_bit
  - epct_gis_mtgfcil1
  - epct_or1sww025
  - epct_plugin_netcdf_generator
  - epct_plugin_umarf_isccp
  - epct_plugin_umarf_isccp_nat
  - epct_plugin_umarf_msgclmk_grib
  - epct_plugin_umarf_msgclmk_netcdf
  - epct_plugin_umarf_msgnative
  - epct_plugin_umarf_msgnative_hrv_nat
  - epct_plugin_umarf_msgnative_nat
  - epct_plugin_umarf_openmtp
  - epct_plugin_umarf_rgb_dc_jpeg
  - epct_plugin_umarf_rgb_dc_jpeg_nat
  - epct_plugin_umarf_rgb_dc_png
  - epct_plugin_umarf_rgb_dc_png_nat
  - epct_plugin_umarf_xrit
  - epct_plugin_umarf_xrit_nat
executable_paths:
  epct_plugin_netcdf_generator: null
```

The `Data Tailor Standalone` instance is now ready to use. 
Whenever you want to use the application, just activate your conda environment with

```bash
conda activate epct-desktop
```
The you can start consuming data.

## Resources

- [How to consume data](../how-to/how-to-consume-data.md)
- [Data Tailor Standalone Guide](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide)
