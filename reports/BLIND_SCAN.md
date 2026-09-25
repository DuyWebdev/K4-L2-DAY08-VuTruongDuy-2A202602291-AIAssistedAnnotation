# Blind Scan

## Image
- Filename: frame_0099.jpg

## Observed vehicle count
- Observed count of 4+ wheel vehicles: 19

## Likely AI miss/error areas

### 1. Dark/partially visible vehicles in the lower-right area
There are vehicles near the lower-right edge of the image that are partially cut off by the frame and are difficult to distinguish from the dark background. These areas are likely candidates for missed detections or incomplete bounding boxes.

### 2. Closely spaced/dark vehicles in the middle-left to center roadway
Several vehicles are traveling close together, with strong headlights and dark vehicle bodies. The bright headlights can make the vehicle boundaries difficult to distinguish, so the AI may miss a vehicle, merge adjacent vehicles, or place a box around the light/reflection rather than the actual vehicle body.
