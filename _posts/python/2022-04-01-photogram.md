---
layout: post
title: "COLMAP"
subtitle: "COLMAP pipeline"
category: python
tags:  photogram
#image:
#  path: "/assets/img/2022-04-02/BG.gif"
---
In this post I describe the piple used to create awesome 3D models created from just images. <br>
Using *COLMAP*'s pipe line we create the sparse cloud. check my next post how we acutally render a 3D model from this sparse cloud using *OPENMVS*.

<!--more-->
* this unordered seed list will be replaced by the toc
{:toc}


## COLMAP
### Installation

### Configure

### Scripts

#### Set up directory locations

```bat
 file: `example.bat`
@echo on
::These parameters are specific to computer
:: Set colmap directory all the way to and including the bat file.
set colDir=COLMAP_directory
set oMVS=OPENMVS_directory
set workDir=%CD%

```
#### run COLMAP

```bat

::Run the colmap part - reconstructing camera positions + sparse point cloud
::extract features
call %colDir% feature_extractor --database_path database.db --image_path .
 ::match features
call %colDir% exhaustive_matcher --database_path database.db
::make a directory for the sparse reconstruction to go in.  this will be a sub-folder in working directory
mkdir sparse
 ::mapper - create depth maps?
call %colDir% mapper --database_path %workDir%\database.db --image_path . --output_path %workDir%\sparse
::undistort images - necessary for openMVS colmap interface
call %colDir% image_undistorter --image_path . --input_path .\sparse\0\ --output_path myundist
```

#### run OPENMVS
```bat

::OpenMVS part - densification, meshing, and texturing.
::NEW - OpenMVS now interfaces directly with the colmap folders, rather than having to go through nvm format
%oMVS%\InterfaceColmap.exe -i %workDir%\myundist -o proj.mvs
::move the shell into the colmap project folders.
cd myundist
::Create a dense point cloud
%oMVS%\DensifyPointCloud.exe proj.mvs
::Reconstruct the mesh
%oMVS%\ReconstructMesh.exe proj_dense.mvs
::Refine the mesh.
%oMVS%\RefineMesh.exe --resolution-level 1 proj_dense_mesh.mvs
::Texture that mesh!
%oMVS%\TextureMesh.exe --export-type obj -o model_name.obj proj_dense_mesh_refine.mvs

pause
```
### Results

<div class="sketchfab-embed-wrapper"> <iframe title="Hanuman Model" frameborder="0" allowfullscreen mozallowfullscreen="true" webkitallowfullscreen="true" allow="autoplay; fullscreen; xr-spatial-tracking" xr-spatial-tracking execution-while-out-of-viewport execution-while-not-rendered web-share src="https://sketchfab.com/models/54d824ef5dc54307b94421a21de90961/embed?ui_theme=dark"> </iframe> <p style="font-size: 13px; font-weight: normal; margin: 5px; color: #4A4A4A;"> <a href="https://sketchfab.com/3d-models/hanuman-model-54d824ef5dc54307b94421a21de90961?utm_medium=embed&utm_campaign=share-popup&utm_content=54d824ef5dc54307b94421a21de90961" target="_blank" style="font-weight: bold; color: #1CAAD9;"> Hanuman Model </a> by <a href="https://sketchfab.com/LSP7?utm_medium=embed&utm_campaign=share-popup&utm_content=54d824ef5dc54307b94421a21de90961" target="_blank" style="font-weight: bold; color: #1CAAD9;"> LSP7 </a> on <a href="https://sketchfab.com?utm_medium=embed&utm_campaign=share-popup&utm_content=54d824ef5dc54307b94421a21de90961" target="_blank" style="font-weight: bold; color: #1CAAD9;">Sketchfab</a></p></div>


### Next step


Continue with [OPENMVS](){:.heading.flip-title}
{:.read-more}
