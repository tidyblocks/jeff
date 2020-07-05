# jeff

A modern JavaScript library for building blocks-based interfaces.

## Overview

-   Jeff reproduces the look and feel of [Scratch](http://scratch.mit.edu),
    which is built using [Google Blockly](https://developers.google.com/blockly/).
    When in doubt, do what they do :-)

-   Jeff is an interface toolkit.
    It lets developers define blocks that users can click together,
    but does *not* execute blocks---that's up to the application that uses it.

-   The **developer** defines the blocksets and blocks they want
    by creating a data structure that contains Jeff classes (for common blocks)
    and/or subclasses of those classes (for custom blocks).
    This is explained in more detail in *Initialization*.

-   The **developer** creates an `svg` element in their web page
    and calls `jeff.initialize(svgElementId, blockStructure)`,
    where `svgElementId` is the ID of the SVG element
    and `blockStructure` is the data structure defining their blocksets and blocks.

-   The **user** can then construct block stacks.
    This is explained in more detail in *Interaction*.

-   When the **user** invokes the function `jeff.toJson(svgElementId)`,
    Jeff returns a JSON structure representing the current configuration of blocks.
    This function will typically be called from the event handler for a `Run` button
    and from the event handler for a `Save` button.
    In the first case,
    the **developer** will have written code to walk the JSON and perform actions
    (e.g., execute the block stacks).
    In the second case,
    the **developer** will have written code to save the JSON.

-   The **user** can also invoke the function `jeff.fromJson(svgElementId, json)`
    (typically via the event handler for a `Load` button).
    This function will recreate the block stacks defined by `json`.

## Initialization

Jeff defines both the appearance of standard blocks (as SVG shapes)
and the code that handles their interactions (as JavaScript classes).
The **developer** can use the standard blocks directly
or subclass those blocks to create new ones
with the same appearance but different behavior.

Jeff has five basic kinds of block:

-   A *top block* can only be used as the first block in a block stack.
    A top block has a rounded edge on top (to indicate that nothing can be joined to its top)
    and a standard connector edge on bottom.

-   A *bottom block* can only be used as the last block in a block stack.
    A bottom block has a standard connector edge on top,
    and a rounded edge on bottom.

-   A *middle block* can be used anywhere in a stack,
    and has a standard connector edge on both its top and bottom.

-   Top blocks, middle blocks, and bottom blocks are also called *main blocks*.

-   A *nesting block* fits inside a hole in a main block or in another nesting block;
    the containing block expands as necessary to holds its contents.
    Nesting blocks are typically used to construct expressions;
    we discuss them in more detail below.

-   A *container block* (also called a "C block") is shaped like an upper-case 'C'
    and holds one block stack.
    Container blocks are typically used to construct compound statements;
    we discuss them in more detail below.

## Creating Blocks

When Jeff initializes the SVG drawing area,
it creates a *toolbox* containing blocks the **user** can clone.
The toolbox contains blocks and *blocksets*.
A blockset acts like a folder that contains actual blocks
(but not nested blocksets: the hierarchy is only one level deep).
Blocks and blocksets may be of different colors,
but typically all of the blocks in a blockset are of the same color.

## Widgets

Blocks of all kinds may contain *widgets* for customization,
including:

-   A *pulldown* (e.g., for selecting an arithmetic operator).

-   A small one-line *textbox* (e.g., for entering a variable name or filename).

-   A *checkbox* for turning a feature on or off.

Each widget is represented as a class in Jeff.
Each block's constructor is responsible for creating the widgets it needs,
but Jeff lays them out within the block.

## Nesting Blocks

Nesting blocks are typically used to create expressions such as `3 * x + 2`,
which as blocks would be:

```
[[number _3_] _*_ [variable _x_]] + [number _2_]]
```

and as JSON:

```
{
  'type': 'binary',
  'operator': '+',
  'left': {
    'type': 'binary',
    'operator': '*',
    'left': {
      'type': 'number',
      'value': 3,
    },
    'right': {
      'type': 'variable',
      'name': 'x'
    }
  },
  'right': {
    'type': 'number',
    'value': 2
  }
}
```

Nesting blocks grow horizontally and vertically to be as large as they need to be.
For more details about the kinds of nesting blocks we need to support,
please see the [briq](http://github.com/tidyblocks/briq) project.

## Container Blocks

C-blocks are typically used to create compound statements such as loops.
The top arm of a C-block may contain widgets and/or nesting blocks
(e.g., to define the range of a loop).
The open space in a C-block may contain one stack of middle blocks.
Note that a C-block may itself be a middle block,
which is how the **user** creates things like nested loops
or loops that contain conditionals.
C-blocks shrink and grow vertically to be as large as they need to be.

## Interaction

-   When the **user** clicks on a blockset in the toolbox,
    Jeff expands the blockset to show the blocks it contains.

-   When the **user** clicks on a block in the toolbox,
    Jeff clones that block and attaches it to the pointer.

-   As the **user** moves the pointer,
    the block follows it as long as the mouse is down.
    When the **user** releases the block,
    it is dropped in its current location.

-   When a block comes near other blocks that it can snap to,
    the matchable edges of the moving block and the target block light up.
    This can be the top or bottom of a main block,
    a hole that can contain a nesting block,
    etc.

-   Blocks snap into position when they are dropped close enough to a matching location.

-   If the **user** selects the top block of a stack and moves the mouse,
    the whole stack moves.
    If the **user** selects any other block in the stack and moves the mouse,
    that block and all of the blocks below it move.
    (This is how the **user** repositions or breaks stacks.)

-   When Jeff initializes the SVG area,
    it automatically creates a *trash bin*.
    If the **user** drags a block or block stack into the trash bin,
    it is deleted.
    There is no undo.

-   Note that block stacks don't have to start with top blocks
    or end with bottom blocks.
    It is up to the application using Jeff to complain
    if a set of blocks and block stacks are improperly configured.

## Optional: Connectors

It is sometimes necessary for block stacks to communicate with each other.
Scratch does this using signals and listeners:
one type of block will emit a named signal when it executes,
while another type of block will pause the execution of its stack
until it receives a named signal.
This allows for many-to-many communication,
but is hard to debug because the connections between signallers and listeners are not visible.

Instead,
Jeff provides *connectors* between *signal blocks* and three kinds of *listener block*.
A signal block is a bottom block with a single connection point on its bottom (unconnectable) edge.
The listener blocks are top blocks with various decorations on their top (unconnectable) edges:

-   A *single listener* has a single connection point on its top edge.
    It runs each time it receives a signal.
-   An *either listener* has two connection points on its top edge.
    It runs whenever it receives a signal on either incoming connection (logical OR).
-   A *both listener* has two connections points on its top edge.
    It runs whenever it has received a signal on both of its incoming connections
    (logical AND).

The **user** can connect signal blocks to listener blocks by clicking on a connection point,
dragging,
and clicking on the other connection point.
Clicking on a connection point that already has a connection selects that end of the connection;
dropping that end on empty space breaks the connection.

## Use Cases

1.  *User opens web page containing a Jeff application.*
    User sees a rectangular workspace with:

    1.  A vertical column on the left side containing icons for blocksets.
    1.  An empty area for building stacks.
    1.  A trash bin in the lower right of the empty area for disposing of blocks.

    There will usually be other items on the page,
    such as an area for displaying output
    and buttons for loading, saving, and running,
    but they are not Jeff's responsibility.

1.  *User adds the first block to the workspace.*
    1.  User clicks on a blockset in the block palette.
    1.  Palette displays the blocks in that blockset.
    1.  User clicks and holds on a block from that blockset.
    1.  Jeff creates a new block of that type.
    1.  User moves the pointer into the construction area.
    1.  The new block follows the pointer.
    1.  User releases the pointer.
    1.  The new block is dropped in its last location.

1.  *User adds a second block to the workspace to create a stack.*
    1.  User creates a new block (as above).
        This block is *not* a top block or a nesting block.
    1.  User moves the pointer near the bottom of the existing block
        (which is *not* a bottom block or a nesting block).
    1.  Jeff highlight the lower edge of the existing block
        and the upper edge of the new block to show that they can be joined.
    1.  User releases the pointer.
    1.  Jeff clicks the two blocks together to create a stack.

1.  *User moves a stack.*
    1.  User clicks and holds the top block of a stack.
    1.  Jeff highlights the whole stack.
    1.  User moves the pointer.
    1.  Jeff moves the entire stack to follow the pointer.
    1.  User releases the pointer.
    1.  Jeff drops the stack in the pointer's last location.

1.  *User breaks a stack.*
    1.  User clicks and holds any block in the stack except the top one.
    1.  Jeff highlights that block and all the blocks below it.
    1.  User moves the pointer.
    1.  Jeff moves the portion of the stack
        consisting of the selected block and the blocks below it.
    1.  User releases the pointer.
    1.  Jeff drops the new stack in the pointer's last location.
    1.  Note: the user may also drag the stack close to an existing stack,
        in which case the two stacks are joined when the moving stack is dropped.

1.  *User inserts a block into a stack.*
    1.  User creates a new block (as above) or selects an existing stack (as above).
        The block or stack is connectable on its top and bottom edges.
    1.  User moves the pointer over the join between two blocks in an existing stack.
    1.  Jeff highlights the join to show where the block or stack being moved can be inserted.
    1.  User releases the pointer.
    1.  Jeff inserts the block or stack in the indicated location.

1.  *User deletes a stack.*
    1.  User clicks and holds a block (either on its own or in a stack).
    1.  User moves the pointer to the trash can.
    1.  Jeff moves the block or stack to the trash and removes it from the workspace.

1.  *User nests a block.*
    1.  User creates or selects a nesting block.
    1.  User moves the pointer to an empty hole in an existing block.
    1.  User releases the pointer.
    1.  Jeff inserts the nesting block in the hole and resizes the containing block as needed.

1.  *User edits a block.*
    1.  User selects a widget in an existing block.
    1.  If the widget is a text entry widget, Jeff sets the text edit focus.
    1.  If the widget is a pulldown, Jeff shows the options.
    1.  If the widget is a checkbox, Jeff toggles its state.
