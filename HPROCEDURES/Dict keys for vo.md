# Dict keys for vo

### vo_create_box_model

#### Input dict

**Main Params**

- `ImageToTrain` — *RGB Image*  
  Image used to create the shape model
- `BoxRegion` — *Region*  
  Define the size of the box in pixels
- `WorkRegion` — *Region*  
  Define ROI where the boxes can be found.
- `ShapeROI` — *Region*  
  Region where the image can be trained to obtain the shape model contour
- `BoxSize` — *dict*  
  Real box size in millimetres
  - `Large`
  - `Width`
  - `High`

**Optional Params**

- `Filters` — *dict*  
  Dict of all filters used to find the shape models in the best way.
  - `Channel` — *Int*  
    Select channel you want to use, number 4 is used for rgb_to_gray function
  - More filters …
- `GetTopROI` — *dict*  
  Params to get top ROI
  - `MinArea` — *int*  
    Min area to consider a correct top region
  - `DeltaZ` — *int*  
    DeltaZ used to determine the steps needed to go down to find a complete layer
  - `Maxcompactness` — *int*  
    Max compactness of the region to consider it a correct top region
  - `MinRectangularity` — *int*  
    Min Rectangularity of the region to consider it a correct top region
  - `DilationRect` — *int*  
    Dilate final region
- `SMParams` — *dict*  
  Params for shape model
- `CheckCandidates` — *dict*  
  Params to determine if it is a correct candidate
  - `MinScoreCandidate` — *int*  
    Once the possible candidates have been found, their Score value is checked, a loop is executed starting from the nominal score value to the "MinScoreCandidate".
  - `MinScaleCandidate` — *int*
  - `ScoreCandidate` — *int*
  - `ScaleCandidate` — *int*
  - `MinScoreSM` — *int*
  - `PercentageBox` — *int*
- `OrderCandidates` — *string*  
  Arrange the detected candidates
  - `OrderType` — *string*
- `GetPose` — *dict*  
  Manipulate box pose
  - `ErosionCore` — *int*  
    Get core pose
  - `OffsetX` — *int*
  - `OffsetY` — *int*
  - `OffsetAngle` — *int*
- `SecondRecipe` — *dict*  
  Second Recipe for multireference.
  - `Name` — *String*  
    Name of second recipe
  - `Recipe` — *HDict*  
    Second recipe

#### Output Dict

- `TrainDict` — *dict*  
  Dict where see all trained params
- `FiltersResult` — *dict*  
  Dict to see how the filters work
- `ModelID` — *Handle*  
  Shape Model ID
- `ModelContour` — *Region*  
  Region of trained shape model
- `StartAngle` — *real*  
  initial angle of box region in radian
- `SecondRecipe` — *real*  
  initial angle of box region in radian
- `Error` — *Dict*  
  Show all procedure errors

### vo_find_multi_boxes

#### Input dict

- `RecipeDict` — *dict*  
  Output dict of vo_create_box_model
- `InputImage` — *Image RGB*  
  Image where find candidates
- `DepthMap` — *Depth map image*  
  where obtain candidate position
- `RobotPose` — *Dict*
  - `ToolHCam` (optional) — *Pose*  
    Calibration pose
  - `BaseHTool` (optional) — *Pose*  
    Real robot pose
  - `ZReinforce` (optional) — *real*  
    high of pallet from robot base position
- `RealHeight` (optional) — *int*

#### Output Dict

- `Candidates` — *dict*  
  Dict where see al train params
  - `[Num of candidate]` — *dict*  
    Dict of candidate
    - `Score` — *int*  
      Score of candidates
    - `Contour` — *Region*  
      Contour of candidate
    - `BoxRegion` — *Region*  
      Box Region
    - `Pose` — *Tuple[X, Y, Z]*  
      Box pose (mm)
    - More information…
- `FiltersResult` — *dict*  
  Dict to see how the filters work
- `LayerRegion` — *Region*  
  Top layer region.
- `ImageResult` — *Image RGB*  
  Show all candidates detected
- `Error` — *Dict*  
  Show all procedure errors
  - `General` — *String*  
    Show general description of exception
  - `Exception` — *Exception string*  
    Show complete description of exception
