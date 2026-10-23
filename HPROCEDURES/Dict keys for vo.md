# Dict keys for vo

### vo_create_box_model

#### Input dict

The whole input dict is stored as `TrainDict` in the output, so every key below is
later read by the find procedures from `RecipeDict.TrainDict`.

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
  - `Large` — *real*
  - `Width` — *real*
  - `High` — *real*  
    Box height. Used to compute `GetTopROI.DeltaZ` and the layer height. Overridden by `RealHeight` when received.

**Optional Params**

- `Filters` — *dict*  
  Dict of all filters used to find the shape models in the best way. Applied in this order:
  - `Channel` — *int*  
    Select channel you want to use, number 4 is used for rgb_to_gray function
  - `ErosionRectBefore` — *int*  
    Size of the gray erosion applied before scaling
  - `DilationRectBefore` — *int*  
    Size of the gray dilation applied before scaling
  - `ScaleRange` — *tuple [Min, Max]*  
    Gray value range passed to `scale_image_range`
  - `ErosionRectAfter` — *int*  
    Size of the gray erosion applied after scaling
  - `DilationRectAfter` — *int*  
    Size of the gray dilation applied after scaling
- `GetTopROI` — *dict*  
  Params to get top ROI
  - `MinArea` — *real* — default: area of `BoxRegion` / 3  
    Min area to consider a correct top region
  - `DeltaZ` — *real* — default: `BoxSize.High` / 2.5  
    DeltaZ used to determine the steps needed to go down to find a complete layer
  - `Maxcompactness` — *real* — default: `4.5`  
    Max compactness of the region to consider it a correct top region
  - `MinRectangularity` — *real* — default: `0.8`  
    Min Rectangularity of the region to consider it a correct top region
  - `DilationRect` — *int* — default: `10`  
    Dilate final region (pixels)
- `SMParams` — *dict*  
  Params for shape model. Each key is a `set_generic_shape_model_param` parameter name
  (e.g. `min_score`, `num_matches`) and its value is the value to set.
- `CheckCandidates` — *dict*  
  Params to determine if it is a correct candidate
  - `MinScoreCandidate` — *real* — default: `0.4`  
    Once the possible candidates have been found, their Score value is checked, a loop is executed starting from `ScoreCandidate` down to `MinScoreCandidate` in steps of 0.1.
  - `MinScaleCandidate` — *real* — default: `0.5`  
    Lowest scale (row and column) accepted while the loop lowers `ScaleCandidate`.
  - `ScoreCandidate` — *real* — default: `0.8`  
    Initial min score a candidate must reach to be accepted.
  - `ScaleCandidate` — *real* — default: `0.8`  
    Initial min scale (row and column) a candidate must have to be accepted. Lowered together with `ScoreCandidate`.
  - `MinScoreSM` — *real* — default: trained `min_score` of the shape model  
    Lowest `min_score` used on the shape model search. If it is lower than the trained value, the search is repeated lowering `min_score` in steps of 0.1.
  - `PercentageBox` — *int* — default: `65`  
    Percentage of the best box area that an uncovered region of the layer must reach to be considered a missing box.
- `OrderCandidates` — *dict*  
  Arrange the detected candidates
  - `OrderType` — *string* — default: `'mY_mX'`  
    Order of the candidates. `'mY_mX'`: rows from top to bottom, then columns. `'mX_mY'` is not implemented.
- `GetPose` — *dict*  
  Manipulate box pose
  - `ErosionCore` — *int* — default: `30`  
    Erosion (pixels) applied to the box region before averaging its XYZ points to get the pose
  - `OffsetX` — *real* — default: `0`  
    Offset added to X of the camera pose (mm)
  - `OffsetY` — *real* — default: `0`  
    Offset added to Y of the camera pose (mm)
  - `OffsetZ` — *real* — default: `0`  
    Offset added to Z of the camera pose (mm)
  - `OffsetAngle` — *real* — default: `0`  
    Offset added to the rotation around Z of the camera pose (degrees)
- `SecondRecipe` — *dict*  
  Second Recipe for multireference. When present, `vo_process_data` calls `vo_find_multi_boxes` instead of `vo_find_boxes`.
  - `Name` — *string*  
    Name of second recipe
  - `Recipe` — *dict*  
    Complete second recipe (same structure as `RecipeDict`), embedded in this recipe

#### Output Dict

This dict is the recipe (`RecipeDict`) used by the find procedures.

- `TrainDict` — *dict*  
  Copy of the input dict (all trained params)
- `FiltersResult` — *dict*  
  Dict to see how the filters work. One image per applied filter, with the same key as in `Filters`. Only present when `Filters` is defined.
- `ModelID` — *handle*  
  Shape Model ID
- `ModelContour` — *XLD contour*  
  Contour of the shape model found in the training image
- `StartAngle` — *real*  
  initial angle of box region in radian
- `Error` — *dict*  
  Show all procedure errors
  - `General` — *string*  
    Show general description of exception. Also used for the warning `'Warning : No model was detected in testing'`.
  - `Exception` — *exception*  
    Show complete description of exception

### vo_find_multi_boxes

`vo_find_boxes` uses the same input and output dicts. Both are called from `vo_process_data`,
which selects `vo_find_multi_boxes` when `RecipeDict.TrainDict.SecondRecipe` exists.

#### Input dict

- `RecipeDict` — *dict*  
  Output dict of vo_create_box_model. Keys used:
  - `TrainDict` — *dict*  
    Main params and optional params (see vo_create_box_model Input dict)
  - `ModelID` — *handle*  
    Shape Model ID
  - `StartAngle` — *real*  
    initial angle of box region in radian
- `InputImage` — *RGB Image*  
  Image where find candidates
- `DepthMap` — *Depth map image (X, Y, Z)*  
  where obtain candidate position
- `RobotPose` — *dict*  
  Required, but all its keys are optional
  - `ToolHCam` (optional) — *pose* — default: zero pose  
    Calibration pose
  - `BaseHTool` (optional) — *pose* — default: zero pose  
    Real robot pose
  - `ZReinforce` (optional) — *real* — default: `0` (disabled)  
    high of pallet from robot base position. When not 0, `LayerHigh` is computed.
- `RealHeight` (optional) — *real*  
  Real box height received by TCP. Replaces `BoxSize.High` for this call.

#### Output Dict

- `Candidates` — *dict*  
  Detected boxes, ordered by `OrderType`
  - `[Num of candidate]` — *dict*  
    Dict of candidate (keys `0`, `1`, `2`…)
    - `Score` — *real*  
      Score of candidates
    - `ScaleRow` — *real*  
      Scale of the match in rows
    - `ScaleColumn` — *real*  
      Scale of the match in columns
    - `Angle` — *real*  
      Angle of the match (radian)
    - `Contour` — *XLD contour*  
      Contour of candidate
    - `BoxRegion` — *Region*  
      Box Region
    - `Cross` — *XLD contour*  
      Cross on the box center, used in `ImageResult`
    - `CamHobj` — *pose*  
      Box pose in camera coordinates (mm, degrees), with `GetPose` offsets applied
    - `BaseHObj` — *pose*  
      Box pose in robot base coordinates (mm, degrees)
- `InputImage` — *RGB Image*  
  Input image
- `ImageResult` — *RGB Image*  
  Show all candidates detected
- `FiltersResult` — *dict*  
  Dict to see how the filters work. Only present when `Filters` is defined.
- `LayerRegion` — *Region*  
  Top layer region.
- `MissingBox` — *dict*  
  Uncovered regions of the top layer
  - `Region` — *Region*  
    Uncovered regions that could be a missing box
  - `Area` — *real*  
    Area of the biggest uncovered region
- `LayerHigh` — *real*  
  Height of the top layer from the robot base (mm). In `vo_find_boxes` only present when `ZReinforce` is not 0.
- `Error` — *dict*  
  Show all procedure errors
  - `General` — *string*  
    Show general description of exception. Also set to `'Not all candidates was found '` when not all boxes were found.
  - `Exception` — *exception*  
    Show complete description of exception
