**BEM** stands for block—element—modifier, and is a philosophy of naming and structuring HTML and CSS classes. This guide will not only touch on the Sass side of things, but also on the HTML layout.

The basic idea of BEM is to separate content into manageable blocks—a kind of componentization.

##Block
A block is a piece of content that is encapsulated, meaning it doesn't rely on any other part of the page to be complete. The simplest example of a block could be a form. A block can be repeated endlessly on a site and should be able to fit into any other content without breaking too much (or at all, for that matter).

##Element
An element is a part of a block. An element is meaningless on its own and should always be below a block in the HTML tree. An element describes a part of a block, and a block is defined by combining multiple elements.

##Modifier
A modifier (or modificator) can be put on both the block and element, and is used to change the visual properties of a block or element. It is used so that a block/element can be flexibly reused on other parts of a page, where a small modification is required.
