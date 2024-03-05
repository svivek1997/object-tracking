---
title: object tracking
---
<SwmSnippet path="/write_in_air.py" line="1">

---

These are the imports.

```python
import math

import cv2
import numpy as np
import random
from collections import deque
```

---

</SwmSnippet>

<SwmSnippet path="/write_in_air.py" line="8">

---

video capture lib

```python
cap = cv2.VideoCapture(1)
# To keep track of all point where object visited
center_points = deque()
```

---

</SwmSnippet>

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBb2JqZWN0LXRyYWNraW5nJTNBJTNBc3ZpdmVrMTk5Nw==" repo-name="object-tracking"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
