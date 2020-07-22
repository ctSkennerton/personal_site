---
title: 'Python ete3: formatting organism names the way I want'
date: Thu, 17 Nov 2016 00:58:40 +0000
draft: false
tags: ['Bioinformatics']
---

I feel like I'm on a life-long quest to make all of my phylogenetic tree figures completely programmatically. The best tool I've found for making them is the [ete library](http://etetoolkit.org) for the python programing language. I've already figured out how to get [trees drawn in the style that I like](/2016/07/09/drawing-phylogenetic-trees-connor-style/) but there was still one thing left to do: making organism names italicize correctly. I work with microorganisms where the convention is for the genus and species names to be italicized but the strain name to be in regular font. For example: _Methanoregula formicica_ SMSP, DSM 22288. There are exceptions to this as well; if an organism is not in pure culture then the name _Candidatus_ is prepended and any genus and species names remain in regular format. Unfortunately, ete doesn't have an interface for doing this kind of name formatting, you can either have it italic, bold or regular, but you can't mix and match. Happily though it does contain an interface for dropping down into the lower-level drawing engine (pyQt4), which enables you to do all sorts of custom things. Below is my solution to making fancy formatted organism names. I think it might be a little hacky but seems to work. The organism name should be in the "name" attribute of the leaf for this to work. (See the example below) Here are the formatting rules:

1.  If the name has less than 2 words: do nothing. This is to catch internal names or accession numbers that are not valid scientific names
2.  If the first word of the name is "Candidatus": abbreviate it to "Ca." and italicize just that part.
3.  If there are more than 2 words: italicize the first two.
4.  If there are exactly 2 words: italicize both

```python
from ete3 import Tree, faces, TreeStyle
from PyQt4 import QtCore
from PyQt4.QtGui import QGraphicsRectItem, QGraphicsSimpleTextItem, \
QGraphicsPolygonItem, QGraphicsTextItem, QPolygonF, \
     QColor, QPen, QBrush, QFont

def scientific_name_face(node, *args, **kwargs):
    scientific_name_text = QGraphicsTextItem()
    words = node.name.split()
    text = []
    if len(words) &lt; 2:
        # some sort of acronym or bin name, leave it alone
        text = words
    elif len(words) &gt; 2:
        if words[0] == 'Candidatus':
            # for candidatus names, only the Candidatus
            # part is italicised
            # name shortening it for brevity
            text.append('<i>Ca.</i>')
            text.extend(words[1:])
        else:
            # assume that everything after the
            # second word is strain name
            # which should not get italicized
            text.extend(['<i>'+words[0],words[1]+'</i>'])
            text.extend(words[2:])
    else:
        text.extend(['<i>'+words[0],words[1]+'</i>'])

    scientific_name_text.setHtml(' '.join(text))

    # below is a bit of a hack - I've found that the height of the bounding
    # box gives a bit too much padding around the name, so I just minus 10
    # from the height and recenter it. Don't know whether this is a generally
    # applicable number to use
    masterItem = QGraphicsRectItem(0, 0,
        scientific_name_text.boundingRect().width(),
        scientific_name_text.boundingRect().height() - 10)

    scientific_name_text.setParentItem(masterItem)
    center = masterItem.boundingRect().center()
    scientific_name_text.setPos(masterItem.boundingRect().x(),
        center.y() - scientific_name_text.boundingRect().height()/2)

    # I don't want a border around the masterItem
    masterItem.setPen(QPen(QtCore.Qt.NoPen))

    return masterItem

def master_layout(node):
    if node.is_leaf():
        F = faces.DynamicItemFace(scientific_name_face)
        faces.add_face_to_node(F, node, 0)

if __name__ == '__main__':
    t = Tree()
    # this is fake data to show the rendering
    t.populate(4)
    leaves = t.get_leaves()
    # give the leaves some different types of names
    leaves[0].name = 'Methanosarcina barkeri'
    leaves[1].name = 'Candidatus Methanoperedens nitroreducens'
    leaves[2].name = 'ANME_bin_23'
    leaves[3].name = 'Methanosaeta thermophila PT'

    ts = TreeStyle()
    ts.show_leaf_name = False
    ts.layout_fn = master_layout
    t.show(tree_style=ts)
```