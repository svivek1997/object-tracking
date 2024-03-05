---
title: Object Tracking
---
# Introduction

This document will walk you through the implementation of the object tracking feature. This feature allows us to track the movement of a specific object in a video stream and draw a line following its path.

We will cover:

1. How we capture and process the video frames.


1. How we detect and track the object.


1. How we draw the line following the object's path.

# Capturing and processing video frames

The first step in our object tracking feature is to capture the video frames. We use OpenCV's VideoCapture function to capture video from the webcam. The frames are then flipped horizontally to mirror the user's movements more intuitively.

<SwmSnippet path="/write_in_air.py" line="1">

---

To improve the object detection, we apply a Gaussian blur to the frame. This helps to reduce high-frequency noise in the image and make the object's edges smoother. We then convert the frame from the BGR color space to the HSV color space. This conversion makes it easier to define a range of colors to detect the object.

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
```

---

</SwmSnippet>

# Detecting and tracking the object

The next step is to detect the object in the frame. We define a range of blue colors in the HSV color space and create a mask that isolates the pixels within this range. This mask is then processed with morphological operations to reduce noise and fill in gaps in the detected object.

We then find all contours in the mask. A contour is a curve joining all the continuous points along the boundary of an object with the same color or intensity. If there are any contours, we find the biggest one, which we assume to be the object we want to track.

<SwmSnippet path="/write_in_air.py" line="1">

---

We calculate the center of this contour and draw a filled circle at this point. We also fit an ellipse to the contour and draw this ellipse on the frame. The center of the contour is then added to a deque of center points.

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
```

---

</SwmSnippet>

# Drawing the line following the object's path

The final step in our object tracking feature is to draw a line following the object's path. We do this by iterating over the deque of center points and drawing a line between each pair of points.

We add some randomness to the color of the line to make the path more visually interesting. We also check that the distance between two points is not too large before drawing the line, to avoid connecting points between which the object did not actually move.

<SwmSnippet path="/write_in_air.py" line="1">

---

The processed frame and the mask are then displayed using OpenCV's imshow function. The program continues to process frames until the user presses the ESC key.

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
```

---

</SwmSnippet>

This implementation of object tracking is relatively simple but effective for tracking a single, clearly defined object. It could be extended in various ways, such as supporting multiple objects, using more sophisticated object detection algorithms, or adding more features to the path visualization.

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBb2JqZWN0LXRyYWNraW5nJTNBJTNBc3ZpdmVrMTk5Nw==" repo-name="object-tracking"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
