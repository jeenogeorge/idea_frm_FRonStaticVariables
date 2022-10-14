## Background

After finding the weights for each variable using the FR model, the
weights has to be applied on the original layer to be used further. Her
we explain how we read the FR weights from the save table and apply it
on the variable.

For the wrok we have to load the following libraries:

    packages <- c("terra","tidyverse","dplyr")

    f <- read.table('conditioningfactors/frequencyratio_Aug')
    colnames(f) <- c('factor', 'Class', 'FR')
    FR <- f
    FR$FR_norm <- FR$FR

    state_4326 <- vect("assam_State.shp")

Next, we have to read the flood conditioning factors.

    soil <- rast("conditioningfactors/soil.tif")
    lith <- rast("conditioningfactors/lithogy.tif")

    drnDens <- rast("conditioningfactors/drainDensity.tif")
    rivdist <- rast("conditioningfactors/RiverDistance.tif")
    elv <- rast("conditioningfactors/elevation.tif")
    slope <- rast("conditioningfactors/slope.tif")
    gcn <- rast("conditioningfactors/GCN.tif")

    #latest static
    dist_natReserv <- rast("state/ind_wdpa_dst_cat1_100m_2017.tif")
    dist_rd <- rast("state/ind_osm_dst_road_100m_2016.tif")
    dist_rdIntrs <- rast("state/ind_osm_dst_roadintersec_100m_2016.tif")

We show the rest of the scripts for a single layer - Drainage Density.
The same is applied for the rest of the variables.

    FR_wt <- FR %>% filter(factor == "drainagedensity")
    drnDens_FR <- classify(drnDens, cbind(id=FR_wt$Class, v=FR_wt$FR_norm))
    drnDens_FR <- drnDens_FR / max(drnDens_FR[], na.rm = T)
    plot(drnDens_FR, main = "FR values of Drainage Density")

![](weights_FR_on_static_variables_files/figure-markdown_strict/Create%20a%20raster%20having%20FR%20values%20of%20the%20respective%20layer-1.png)

We normalise the values of the original layer

    drnDens_rl <- rast("conditioningfactors/gmted_drainage_density_without_1_real.tif") 
    drnDens_rl <- drnDens_rl / max(drnDens_rl[], na.rm = T)
    plot(drnDens_rl)

![](weights_FR_on_static_variables_files/figure-markdown_strict/Normalise%20the%20values%20of%20drainage%20density-1.png)

    drnDens_fin <- drnDens_rl * drnDens_FR

Finally, re-sample the weighted layer to a common raster format.

    slope <- rast("conditioningfactors/slope.tif")
    dummy <- rast(ext=ext(slope),crs=crs(slope, proj=T, describe=FALSE, parse=FALSE),
                  nrow=dim(slope)[1],ncol=dim(slope)[2])
    drnDens_fin <-resample(drnDens_fin, dummy)
    plot(drnDens_fin)

![](weights_FR_on_static_variables_files/figure-markdown_strict/unnamed-chunk-2-1.png)
