now we are using 
``` python
from pathlib import Path
```
instead of 
``` python
import os
file_path = os.path.join("folder", "subfolder", "file.txt")
```

because mac,windows,linux handles this path differently.. and it is pain and error prone. 
**pathlib** treats file paths as objects instead of plain text strings. This lets you interact with files using intuitive methods and properties.

