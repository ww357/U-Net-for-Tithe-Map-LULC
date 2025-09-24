# U-Net-for-Tithe-Map-LULC
All the files needed to train an improved U-Net model for vectorising land cover based on boundary line information in Tithe Maps for historic land use and cover mapping

Making training data:
- trace boundaries in QGIS and save polyline vector layer as a geopackage
- run `\training-data\CreateMatchinglMasksAndImage.ipynb` to create matching/overlapping images for the georeferenced raster map scan and the traced features 
- run `\training-data\PatchifyTitheMap.ipynb` to patchify the tithe map
- run `\training-data\PatchifyTitheMask.ipynb` to patchify the map's mask
- ~run `\training-data\AddAugmentedPatches.ipynb` to first filter out most of the ~ not made this yet


Training process:
- run `\U-Net\train.ipynb` to train the model on the patches

Land cover patch vectorisation:
- If you want to use the pre-trained model run `\U-Net\predict.ipynb` with model weights 'U-net-AG-D-ASPP-tithemaps.h5'
- run `\post-processing\Predicted` to predict and save masks for patches
  - *I need to make BATCH_SIZE scale relative to the number of input patches (maybe make BATCH_SIZE = (total input patches / 10))
-  run `\post-processing\full-map-from-patches.ipynb` to stitch the patches into a full map prediction image and spatially project it using metadata from the reference map  
  - *change stitching patches into full maps.ipynb to this name*
- Using GRASS GIS processes (can be done more easily in QGIS with 'GRASS GIS provider' plug-in installed and activated) run the following tools to convert the binary raster prediction mask image into a set of vectorised polylines
  - r.mapcalc.simple
  	- if(A>0,1,null())
  	- input: {...}stitched_mask
  	- output: {...}NoData_Set
  - r.thin
  	- input: {...}NoData_Set
  	- output: {...}thinned
  - r.to.vect
  	- select line
  	- input: {...}thinned
  	- output: {...}polylines
  - simplify or v.generalize
  	- input: {...}vectorised
  	- output: {...}simplified
  	- Simplification method: Douglas-Peucker Algorithm (Distance)
  	- Tolerance: 0.5 m
- Visually inspect and fix errors
- Run vector-to-vector polygonisation to produce land cover patches.


Built on Ran and colleagues (2022) improved U-Net model for segmenting line elements from historic map documents.
Paper Citation: Ran, W. et al. (2022) ‘Raster Map Line Element Extraction Method Based on Improved U-Net Network’, ISPRS International Journal of Geo-Information, 11(8), p. 439. Available at: https://doi.org/10.3390/ijgi11080439.
Link to GitHub: [Raster-Map](https://github.com/FutureuserR/Raster-Map)


