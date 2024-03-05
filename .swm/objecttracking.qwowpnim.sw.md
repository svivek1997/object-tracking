---
title: ObjectTracking
---
# Introduction

This document will walk you through the implementation of the ObjectTracking feature. The feature is designed to track the movement of a specific object in a video stream and draw a line following the path of the object.

We will cover:

1. How the video stream is processed and the object is detected.


1. How the path of the object is tracked and visualized.


1. How the feature handles noise and other potential disruptions in the video stream.

# Processing the video stream and detecting the object

The first part of the implementation involves processing the video stream and detecting the object of interest. This is done in the `write_in_air.py` file.

The video stream is captured using OpenCV's `VideoCapture` function. Each frame of the video is then processed to detect the object. The frame is first blurred using a Gaussian blur to reduce noise and then converted from the BGR color space to the HSV color space. This is done because the HSV color space allows for better color detection.

<SwmSnippet path="/write_in_air.py" line="1">

---

The object is detected by creating a mask of the frame where the pixels within a certain color range are set to white and all other pixels are set to black. The color range is defined to detect blue objects in this case, but it can be adjusted to detect objects of other colors.

```python
import math

import cv2
import numpy as np
import random
from collections import deque

cap = cv2.VideoCapture(1)
# To keep track of all point where object visited
center_points = deque()

while True:
    # Read and flip frame
    _, frame = cap.read()
    frame = cv2.flip(frame, 1)

    # Blur the frame a little
    blur_frame = cv2.GaussianBlur(frame, (7, 7), 0)

    # Convert from BGR to HSV color format
    hsv = cv2.cvtColor(blur_frame, cv2.COLOR_BGR2HSV)

    # Define lower and upper range of hsv color to detect. Blue here
    lower_blue = np.array([100, 50, 50])
    upper_blue = np.array([140, 255, 255])
    mask = cv2.inRange(hsv, lower_blue, upper_blue)

    # Make elliptical kernel
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (15, 15))

    # Opening morph(erosion followed by dilation)
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)

    # Find all contours
    contours, hierarchy = cv2.findContours(mask.copy(), cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)[-2:]

    if len(contours) > 0:
        # Find the biggest contour
        biggest_contour = max(contours, key=cv2.contourArea)

        # Find center of contour and draw filled circle
        moments = cv2.moments(biggest_contour)
        centre_of_contour = (int(moments['m10'] / moments['m00']), int(moments['m01'] / moments['m00']))
        cv2.circle(frame, centre_of_contour, 5, (0, 0, 255), -1)

        # Bound the contour with circle
        ellipse = cv2.fitEllipse(biggest_contour)
        cv2.ellipse(frame, ellipse, (0, 255, 255), 2)

        # Save the center of contour so we draw line tracking it
        center_points.appendleft(centre_of_contour)

    # Draw line from center points of contour
    for i in range(1, len(center_points)):
        b = random.randint(230, 255)
        g = random.randint(100, 255)
        r = random.randint(100, 255)
        if math.sqrt(((center_points[i - 1][0] - center_points[i][0]) ** 2) + (
                (center_points[i - 1][1] - center_points[i][1]) ** 2)) <= 50:
            cv2.line(frame, center_points[i - 1], center_points[i], (b, g, r), 4)

    cv2.imshow('original', frame)
    cv2.imshow('mask', mask)

    k = cv2.waitKey(5) & 0xFF
    if k == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

---

</SwmSnippet>

# Tracking the object's path and visualizing it

Once the object is detected in a frame, the center of the object is calculated and a circle is drawn at that location. The center of the object is then added to a deque of center points.

<SwmSnippet path="/write_in_air.py" line="1">

---

A line is drawn between consecutive center points in the deque. This line represents the path that the object has taken. The color of the line is randomly generated for each pair of points to create a colorful trail.

```python
import math

import cv2
import numpy as np
import random
from collections import deque

cap = cv2.VideoCapture(1)
# To keep track of all point where object visited
center_points = deque()

while True:
    # Read and flip frame
    _, frame = cap.read()
    frame = cv2.flip(frame, 1)

    # Blur the frame a little
    blur_frame = cv2.GaussianBlur(frame, (7, 7), 0)

    # Convert from BGR to HSV color format
    hsv = cv2.cvtColor(blur_frame, cv2.COLOR_BGR2HSV)

    # Define lower and upper range of hsv color to detect. Blue here
    lower_blue = np.array([100, 50, 50])
    upper_blue = np.array([140, 255, 255])
    mask = cv2.inRange(hsv, lower_blue, upper_blue)

    # Make elliptical kernel
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (15, 15))

    # Opening morph(erosion followed by dilation)
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)

    # Find all contours
    contours, hierarchy = cv2.findContours(mask.copy(), cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)[-2:]

    if len(contours) > 0:
        # Find the biggest contour
        biggest_contour = max(contours, key=cv2.contourArea)

        # Find center of contour and draw filled circle
        moments = cv2.moments(biggest_contour)
        centre_of_contour = (int(moments['m10'] / moments['m00']), int(moments['m01'] / moments['m00']))
        cv2.circle(frame, centre_of_contour, 5, (0, 0, 255), -1)

        # Bound the contour with circle
        ellipse = cv2.fitEllipse(biggest_contour)
        cv2.ellipse(frame, ellipse, (0, 255, 255), 2)

        # Save the center of contour so we draw line tracking it
        center_points.appendleft(centre_of_contour)

    # Draw line from center points of contour
    for i in range(1, len(center_points)):
        b = random.randint(230, 255)
        g = random.randint(100, 255)
        r = random.randint(100, 255)
        if math.sqrt(((center_points[i - 1][0] - center_points[i][0]) ** 2) + (
                (center_points[i - 1][1] - center_points[i][1]) ** 2)) <= 50:
            cv2.line(frame, center_points[i - 1], center_points[i], (b, g, r), 4)

    cv2.imshow('original', frame)
    cv2.imshow('mask', mask)

    k = cv2.waitKey(5) & 0xFF
    if k == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

---

</SwmSnippet>

# Handling noise and disruptions

The implementation includes several measures to handle noise and other potential disruptions in the video stream.

Firstly, the frame is blurred using a Gaussian blur before the object detection is performed. This helps to reduce noise in the video stream.

Secondly, an opening morphological operation (erosion followed by dilation) is performed on the mask. This helps to remove small noise pixels and to separate objects that are close together.

<SwmSnippet path="/write_in_air.py" line="1">

---

Finally, when drawing the line that represents the object's path, a distance check is performed between consecutive center points. If the distance is greater than a certain threshold, the line is not drawn. This helps to prevent the line from jumping around when the object is not detected in a frame.

```python
import math

import cv2
import numpy as np
import random
from collections import deque

cap = cv2.VideoCapture(1)
# To keep track of all point where object visited
center_points = deque()

while True:
    # Read and flip frame
    _, frame = cap.read()
    frame = cv2.flip(frame, 1)

    # Blur the frame a little
    blur_frame = cv2.GaussianBlur(frame, (7, 7), 0)

    # Convert from BGR to HSV color format
    hsv = cv2.cvtColor(blur_frame, cv2.COLOR_BGR2HSV)

    # Define lower and upper range of hsv color to detect. Blue here
    lower_blue = np.array([100, 50, 50])
    upper_blue = np.array([140, 255, 255])
    mask = cv2.inRange(hsv, lower_blue, upper_blue)

    # Make elliptical kernel
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (15, 15))

    # Opening morph(erosion followed by dilation)
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)

    # Find all contours
    contours, hierarchy = cv2.findContours(mask.copy(), cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)[-2:]

    if len(contours) > 0:
        # Find the biggest contour
        biggest_contour = max(contours, key=cv2.contourArea)

        # Find center of contour and draw filled circle
        moments = cv2.moments(biggest_contour)
        centre_of_contour = (int(moments['m10'] / moments['m00']), int(moments['m01'] / moments['m00']))
        cv2.circle(frame, centre_of_contour, 5, (0, 0, 255), -1)

        # Bound the contour with circle
        ellipse = cv2.fitEllipse(biggest_contour)
        cv2.ellipse(frame, ellipse, (0, 255, 255), 2)

        # Save the center of contour so we draw line tracking it
        center_points.appendleft(centre_of_contour)

    # Draw line from center points of contour
    for i in range(1, len(center_points)):
        b = random.randint(230, 255)
        g = random.randint(100, 255)
        r = random.randint(100, 255)
        if math.sqrt(((center_points[i - 1][0] - center_points[i][0]) ** 2) + (
                (center_points[i - 1][1] - center_points[i][1]) ** 2)) <= 50:
            cv2.line(frame, center_points[i - 1], center_points[i], (b, g, r), 4)

    cv2.imshow('original', frame)
    cv2.imshow('mask', mask)

    k = cv2.waitKey(5) & 0xFF
    if k == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

---

</SwmSnippet>

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBb2JqZWN0LXRyYWNraW5nJTNBJTNBc3ZpdmVrMTk5Nw==" repo-name="object-tracking"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
