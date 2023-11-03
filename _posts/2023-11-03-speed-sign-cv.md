---
layout: post
author: Ákos Strack
tags: [computer vision, openCV, template matching, image processing, C++]
title:  "Speed Limit Sign Recogniser with Classical Image Processing Techniques"
date:   2015-10-03 11:09:34 +0200
---

# Speed Limit Sign Recogniser

## Overview

This blog post introduces an algorithm designed to detect and recognize speed limit signs in the United States from images captured by car-mounted cameras using classical image processing techniques. In contrast to artificial intelligence methods, this algorithm relies solely on traditional image processing, posing a unique challenge. The chosen approach draws inspiration from the "Real-time Recognition of U.S. Speed Signs"[^1] paper, which focuses on form-based sign detection, emphasizing the difficulties of reliably segmenting signs based on color. The post highlights the algorithm's capacity to efficiently identify distinct characteristics of American speed limit signs and implement low-level image processing. It also explores unaddressed aspects and potential modifications, making it an intriguing project for implementation.

![School zone](/assets/images/speedsign/speed_sing_with_school.jpg)
<div style="text-align:center"><i>School zone speed limit sign detection.</i></div>

## Preprocessing

The preprocessing phase aims to normalize input images by converting Bayer-patterned color images into high-quality grayscale images using a demosaicing algorithm. This step also involves [histogram equalization][hist-equ] and scaling with [Lanczos interpolation][Lanczos], followed by image averaging. Additionally, a Gaussian smoothing filter is applied for noise reduction, based on observations from the LISA dataset[^2].

In the "Rectangle Detection" step, the algorithm acts as a pre-filtering mechanism to segregate the image into potential rectangles and non-rectangular regions. Its primary objective is to quickly eliminate areas where traffic signs cannot exist. This early filtering not only reduces processing time but also minimizes false positives. The algorithm simplifies gradient directions to vertical and horizontal orientations and separately counts votes in the up, down, left, and right directions for each potential rectangle, enabling efficient rejection of false positives, typically arising from partial rectangles.

![Rectangle detection](/assets/images/speedsign/rectangle_detection.jpg)
<div style="text-align:center"><i>Red rectangles indicate potential signs and green rectangles indicate accepted signs.</i></div>

## Speed Sign Detection

The objective is to identify speed limit signs. Unlike most related works that rely on machine learning algorithms, this approach maintains a strict no-Machine Learning stance. Instead, it initially employs a simple template matching method[^4] for its time-efficiency. While other algorithms like SURF[^7] and SIFT[^6] were briefly explored, they couldn't capture distinctive features in the sample images. The template matcher is only executed in areas indicated as potential sign locations from the previous stage, substantially improving processing speed. The algorithm searches for the best match within varying-sized templates around the center of a potential sign, applying a predefined threshold to identify and precisely locate the sign when a match exceeds the threshold.

In "Limit Recognition," classic image processing techniques are employed, mainly utilizing the [k-nearest neighbors][knear] algorithm to recognize numbers on the signs. Pre-processing helps in the exact localization and normalization of numbers on the sign. The steps include binarization by Otsu[^5] thresholding, blob detection, rejection of unsuitable blobs based on size, aspect ratio, and area criteria, and selection of exactly two remaining candidate blobs as there are always two digits in a speed limit. In the final step, the k-Nearest Neighbors algorithm matches the recognized patterns to a predefined dataset of speed limit numbers, providing the final value of the recognized speed limit. The comparison is based on normalized pixel intensities.

![Otsu thresholding](/assets/images/speedsign/otsu_binarization.png)
<div style="text-align:center"><i>Otsu thresholding.</i></div>
![Blob detection](/assets/images/speedsign/blobs.png)
<div style="text-align:center"><i>Relying on blob detection to find the digits.</i></div>

Recognizing additional restrictions, such as conditions linked to traffic signals or "School" labels, involves strategies similar to the core speed limit detection process, which may include template matching with specific thresholds and accounting for sign tilt in position calculations. However, detecting conditions that depend on flashing lights might require analyzing multiple frames, introducing additional complexity.

## Results

The performance evaluation was conducted on the LISA[^2] dataset. The results include:

 * 29 True Positives
 * 1134 True Negatives
 * 8 False Positives
 * 327 False Negatives

These metrics translate to the following performance indicators:

 * Recall: 0.0814607
 * Precision: 0.783784
 * F1 score: 0.147583

The results indicate a relatively good precision value but a very low recall. However, it's important to note that these results may not accurately represent the algorithm's practical performance. The LISA annotation file is incomplete, and in some cases, False Positives occur because there actually is a speed limit sign in the image, but it's annotated differently. The issue with False Negatives is also compounded because it doesn't consider that in a sequence of images, the algorithm may not initially recognize a distant sign but does so as the car approaches. Currently, distant signs are categorized as False Negatives in the evaluation.

![Missed detection](/assets/images/speedsign/table_mis.jpg)
<div style="text-align:center"><i>Missed detection. The edge detector fails to detect the sign due the low light surroundings.</i></div>

## Conclusion

In this project, a prototype program was developed to showcase a broad spectrum of programming techniques, encompassing basic algorithms, high-level image processing libraries, and low-level image processing implementations. While the program provides reasonable recognition results, it maintains extensibility for future development. Upcoming steps involve code refactoring and optimization for each component, such as the rectangle-finding algorithm. New methods like the Viola-Jones[^3] algorithm will be explored for enhanced recognition. Furthermore, tools for efficient testing and evaluation, GPU utilization, and other features like supplementary restriction recognition will be implemented, making it a comprehensive and adaptable image processing solution.

## Usage

The program is a command-line application with potential cross-platform compatibility since it avoids OS-specific dependencies. It has been tested on Windows 7 and 10. The command-line arguments are as follows:

```
--help: Displays program usage instructions.
--f: Processes the specified image file. Cannot be used in combination with the --d switch.
--d: Processes all ".png" image files found in the specified directory. Cannot be used in combination with the --f switch.
--a: Specifies the annotation file for evaluation.
--force: Forces the program to recalculate results for each image. Without this flag, the program attempts to load previously saved results.
--debug: Saves debug images.
--separate: Separates result images and data into TruePositive, TrueNegative, FalseNegative, and FalsePositive directories.
```

An usage example:

```shell
SpeedLimitSignRecogniser.exe --d testFolder --a testFolder\frameAnnotations.csv --debug --separate
```

This command will process all ".png" images in the "testFolder," use the annotation file for evaluation, save debug images, and separate the results into directories based on their classification.

![Windows output](/assets/images/speedsign/windows_run.png)
<div style="text-align:center"><i>Windows command-line output.</i></div>


## Implementation

The entire source code, resources, and documentation are managed with Git and stored on GitHub at the address [https://github.com/HunBug/SpeedLimitSignRecogniser][github-slr].

The development was carried out in C++ using the Eclipse IDE and the MinGW compiler on Windows 10 and 7 operating systems. The following programming libraries were used:

 * Boost 1.59
 * OpenCV 3.0

The program's structure is characterized by separating different processing steps, as discussed in previous sections, into separate interface classes within the code. While this may seem excessive for a prototype program, it proves highly beneficial during the research phase. Many combinations need to be explored, and automatic (parameter) optimization and testing systems can be built more easily, allowing for easy changes in the processing flow, even during runtime.

The main components of the program include "SpeedLimitSignRecogniser" where parameter input and processing initiation occur, "Recogniser" which combines the processing flow and performs result evaluation based on annotation files, and other classes such as "FileSource," "SourceNormaliser," "Segmentation," "CandidateFinder," "Detector," "Recogniser," and "FeatureExtractor," each with specific functions related to image processing and object recognition. Additionally, there are helper classes like "ImagingTools" for common image processing functions and "DebugTools" for debugging aids.


### Bibliography
[^1]: 10 Christoph Gustav Keller , Christoph Sprunk , Claus Bahlmann , Jan Giebel3 and Gregory Baratoff, "Real-time Recognition of U.S. Speed Signs", Intelligent Vehicles Symposium, 2008

[^2]: LISA Traffic Sign Dataset , Laboratory for intelligent & safe automobiles, [LISA Dataset][LISA]

[^3]: P. Viola and M. Jones, "Robust real-time object detection", Technical report, Compaq Cambridge Research Laboratory, Feb. 2001. CRL 2001/1.

[^4]: R. Brunelli, "Template Matching Techniques in Computer Vision: Theory and Practice", Wiley, ISBN 978-0-470-51706-2, 2009

[^5]: Nobuyuki Otsu, A threshold selection method from gray-level histograms. IEEE Trans. Sys., Man., Cyber. 9 (1): 62--66. doi:10.1109/TSMC.1979.4310076., 1979

[^6]: Lowe D G, "Distinctive image features from scale-invariant keypoints". International journal of computer vision, 60(2): 91-110, 2004

[^7]: Bay H, Ess A, Tuytelaars T, et al., "Speeded-up robust features (SURF)". Computer vision and image understanding, 110(3): 346-359, 2008

[^8]: Wikipedia, [k-nearest neighbors algorithm][knear]

[^9]: Wikipedia, [Histogram equalization][hist-equ]

[^10]: Wikipedia, [Lanczos resampling][Lanczos]

[github-slr]: https://github.com/HunBug/SpeedLimitSignRecogniser
[LISA]: http://cvrr.ucsd.edu/LISA/lisa-traffic-sign-dataset.html
[knear]: https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm
[hist-equ]: https://en.wikipedia.org/wiki/Histogram_equalization
[Lanczos]: https://en.wikipedia.org/wiki/Lanczos_resampling