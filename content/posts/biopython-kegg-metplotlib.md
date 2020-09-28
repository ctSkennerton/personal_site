---
title: 'Drawing KEGG pathway maps using biopython and matplotlib'
date: Sun, 27 Sep 2020 22:11:47 -0700
draft: false
tags: [Bioinformatics]
---

I use [KEGG](https://kegg.jp) a lot to understand microbial metabolism.
KEGG is one of the largest resources of enzymes, biochemical
reactions, genes, and molecules, all cross-linked and organized
into what's called metabolic maps. These maps are well-constructed
images of enzymes that functions together for the same overall
purpose like amino acid synthesis, or the metabolism of glucose.
One of the great things about the website is the ability to color
on your data to their metabolic maps. Doing this in bulk though can
be very tedious as you need to view and download individual maps.
Luckily, there is a [great tutorial on how to dynamically color KEGG maps](https://nbviewer.jupyter.org/github/widdowquinn/notebooks/blob/master/Biopython_KGML_intro.ipynb)
using biopython that I've used in the past to create PDF images.

While this works well, the current biopython graphics module is
based on [reportlab](https://www.reportlab.com), which is a more niche graphics system than
[matplotlib](https://matplotlib.org). This means that the images made in biopython can't
easily be combined with other plots generated with matplotlib and
the biopython implementation of drawing KEGG maps is restricted to
producing PDF documents.

I was recently working on a project where I needed to combine the
KEGG maps with other types of plots, generated via matplotlib and
didn't want to do it manually in a separate program. Instead I
looked into converting the `KEGG_vis` module of biopython from
reportlab to matplotlib. Thankfully, the code is quite short and
self-contained, thus making the conversion process easy.

Visualizing KEGG data in biopython is driven by KGML, an XML markup
which describes the placement of objects in a pathway map. Biopython
parses this file and draws on the graphics elements based on 
this information. Converting the drawing code was pretty
straightforward as many of the concepts between reportlab and
matplotlib are the same. The bulk of the drawing code happens in
the `__add_graphics` method, which is responsible for adding in the
lines, circles, and rectangles described in KGML. There is almost
a 1:1 mapping between the reportlab constructs and the equivalent
matplotlib Patch API.  For example drawing a line path in the
original reportlab version looked like

```python
p = self.drawing.beginPath()
x, y = graphics.coords[0]
p.moveTo(x, y)
for (x, y) in graphics.coords:
    p.lineTo(x, y)
self.drawing.drawPath(p)
self.drawing.setLineWidth(1)  # Return to default
```

Which starts a line at an xy-coordinate and then iterates through all of the remaining point in the path
using the `p.lineTo` method. The translation to matplotlib results in very similar code

```python
x, y = graphics.coords[0]
verts = [
(x, y),  # left, bottom
]

codes = [
	Path.MOVETO,
]
for (x, y) in graphics.coords:
	codes.append(Path.LINETO)
	verts.append((x,y))

path = Path(verts, codes)
patch = patches.PathPatch(path)    
self.ax.add_patch(patch)        
```

The only major difference was that reportlab seems to use a global
state to keep track of things like the line color and weight;
throughout the original code there are calls to set the font, color,
and line and then return to the original state after certain calls
to draw a graphics object have been made. Alternatively, in matplotlib,
these modifications are passed in directly when creating a new patch
object. Internally I made this change to the API by adding a
`**kwargs` argument to the `__add_graphics` method.

```python
# The following code snippet demonstrates the change from reportlab
# which used calls like setStrokeColor to globally change the state
# and the new interface which passes these arguments into __add_graphics
# as kwargs, which will be applied to the matplotlib patches.
for ortholog in self.pathway.orthologs:
    for g in ortholog.graphics:
	#self.drawing.setStrokeColor(color_to_reportlab(g.fgcolor))
	#self.drawing.setFillColor(color_to_reportlab(g.bgcolor))
	self.__add_graphics(g, fc=g.bgcolor, ec=g.fgcolor)
```

This new version of the `KEGG_vis` library lives on [my fork of biopython](https://github.com/ctSkennerton/biopython/tree/kegg_matplotlib) for now. 

