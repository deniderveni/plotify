Plotify
=======
Display images on a webpage and provide the viewer with controls to change the parameters used to find them.

This fork is a modern refresh of Benjamin Krikler's Plotify project https://github.com/ic-coders-club/plotify .
This fork is designed with simplified code architecture in mind, minimising custom functions, using more modern features, and making things as easy as possible for both users and developers.

While this was built for the particle physics group at Imperial College London, this is completely generic and can be used for anyone who has many figures related to each other and needs to be shown or toggled through in one place.

Concept
-------
Often in particle physics, we produce hundreds of similar plots for variations of the input data.
The challenge is finding a sensible way to show these plots so that the viewer can compare the differences between the input parameters.

With that in mind, you can use the javascript code in this repository to easily set up a webpage that places the adjustable variables on one side of the screen, and the plots in the centre.
The different values are clickable so that in doing so the plots update to match the new parameters. Images can be clicked on to download the image in some other format (such as pdf, or root) unless the browser can display that type of image, in which case they open in a new window/tab.

On the whole, I've tried to keep the code as simple as possible and have tried to leave as many of the decisions up to the user.
For instance, the Parameter class does not request a title.  Writing and placing the HTML code for a title is left to the user to arrange as desired.
I hope that this provides both flexibility and easier-to-read HTML.

Authors 
-------
Ben Krikler, Roden Derveni

There should probably be some sort of open-source license on this, but for now just give us a mention when (if) you use this, put a citation at the bottom of the webpage or something to that effect.

HTML, CSS and Javascript
------------------------

For those unfamiliar with HTML, CSS and javascript, you may want to consult [this website](http://www.w3schools.com) 
for tutorials and explanations. 


Usage
-----
The aim in writing this code was to make it as easy as possible to use.
To that end, there are only four javascript functions needed to create the functionality described above:

1. A Parameter constructor
2. A method to create and place the HTML for a Parameter
3. An Image constructor
4. A method to create and place the HTML for an Image

### 1) Add a Parameter
The `Parameter` function constructs some variables that can be modified. The idea being that your parameter is something that changes in an otherwise consistent filename, for example '{particle}-momentum.png' which could be 'mu-momentum.png' or 'e-momentum.png'.

This is defined as:
```javascript
const SomeVariableName = new Parameter('VisibleVariableName', ["Visible Option 1", "Visible Option 2"], ["FileNameOption1", "FilenameOption2"]); 
```

This constructs a parameter `SomeVariableName`, the webpage will show "VisibleVariableName", and underneath it a drop-down menu with "Visible Option 1" and "Visible Option 2". Each one of these options relates to the literal part of the filename `FileNameOption1` and `FileNameOption2`

For example, maybe the file is actually called "-13\_momentum.png" but you want the user to select "muon" from the "particle" drop-down menu:
```javascript
const particle = new Parameter('Particle', ["muon-", "e-"], ["13", "-11"]);
```

This then gives you a parameter that you can use to programatically select filenames later on.
We must add this to an array of parameters that is parsed through the .html file:
```javascript
const parameters = [SomeVariableName, ..., ..., ...]
```

### 2) Add an Image
To set up the image filename we simply use:
```javascript
function generateImageFilename(image_name) {
    return `${particle.values[particle.selectedIndex]}_momentum.png`;
}
```

And hurray! The .html script sets up the right filename, having pulled out the value from whatever index in the list was selected by the user.

However, realistically this would only give us 1 image per page. This is a little silly, perhaps we want multiple figures.

Let's say we want each page to have 2 figures. We must set up some list of standard 'image names':
```javascript
var image_name_list = ["MC1", "MC2"];
```
For the sake of this example, I want to show, side-by-side, two similar distributions from MC1 and MC2.
This `image_name_list` variable contains an array of the section of the filename for all the images I want to show on one page.

For example, let's say we have 4 files:
"""
MC1-13\_momentum.png
MC1-11\_momentum.png
MC2-13\_momentum.png
MC2-11\_momentum.png
"""

Then to generate the filenames for the page on the image, we now do:

```javascript
function generateImageFilename(image_name) {
    return `${image_name_list}${particle.values[particle.selectedIndex]}_momentum.png`;
}
```
Where `${image_name_list}` has been given as a whole array.

This of course requires you to engineer your files to be named appropriately...
RootWriter and ComparisonTool were created to engineer filenames in this way, consider using them towards this.

Recipe to create a page
-----------------------
1. Produce your plots and analyses and make sure they are stored in a logical way, using the same string to represent each value of a parameter. 
   (Personally, I prefer creating a directory for the full set of parameters using the values separated with underscores. I then put all the various plots for a given set of values in the corresponding directory.)
2. Create the layout of the webpage using normal HTML, creating a  parameter to be added.
3. Instantiate all the Parameters you need in the .js file
4. Adapt any messages as desired on the .html file; the parameter options are automatically populated
5. Adapt the CSS styles for the parameters if you would like to (classes: value, current_value) and for the images (classes: plot if default value used).

Viewing the page
================

You can upload these files to any standard HTML server that you or your organisation hosts.
Alternatively, you can download the files locally and run `./localserver.sh` if you have a BASH-based terminal (this just uses a Python command, you can do it on any terminal with `python3 -m http.server 8000` instead)
This will run a local server with port 8000 and open up the webpage in your default browser automatically (probably in an existing window).
You can also go to the page manually at `http://localhost:8000`
If the port is already in use somewhere, just change the value in `localserver.sh` and the web link. 

Notes
=====
1. All images use relative links to find the image source.
2. Many servers can only do static server-side includes, so to achieve the desired effect for this project, we must use javascript that is run by the client's machine.
