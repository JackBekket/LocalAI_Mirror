## Package: elements

### Imports:

- strings
- github.com/chasefleming/elem-go
- github.com/chasefleming/elem-go/attrs
- github.com/mudler/LocalAI/core/gallery

### External Data, Input Sources:

- galleryName (string)
- m (gallery.GalleryModel)
- galleryID (string)

### Summary:

This package provides functions to create buttons for various actions related to models in a gallery.

#### installButton:

This function creates an install button for a given gallery name. The button has a specific set of attributes, including data attributes for ripple effects, class for styling, and a data attribute for the target element to be swapped. The button also has a post action that sends the gallery name to the specified URL.

#### reInstallButton:

This function creates a reinstall button for a given gallery name. Similar to the install button, it has a set of attributes for styling and functionality. The button also has a post action that sends the gallery name to the specified URL.

#### infoButton:

This function creates an info button for a given gallery model. The button has attributes for styling and functionality, including a data attribute for the modal target and toggle. The button also has an icon and text for the info action.

#### deleteButton:

This function creates a delete button for a given gallery ID. The button has attributes for styling and functionality, including a confirmation message for the delete action. The button also has a post action that sends the gallery ID to the specified URL.

#### dropBadChars:

This function replaces any "@" characters in a string with "__" to avoid issues with Javascript/HTMX.

