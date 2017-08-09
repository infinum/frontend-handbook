**BEM** stands for block - element - modifier, and is a philosophy of naming and structuring HTML and CSS classes. This guide will not only touch on the SASS side of things, but also on the HTML layout.

Basic idea of BEM is to separate content into manageable blocks - a type of componentization.

##Block
A block is a piece of content that is encapsulated, meaning it doesn't rely on any other part of the page to be complete. Simplest example of a block could be a form. A block can be repeated endlessly on a site and should be able to fit into any other content without breaking too much (or at all, for that matter).

##Element
An element is a part of a block. An element on in its own is meaningless and should always be below a block in the HTML tree. An element describes a part of a block, and by combining multiple elements a block is defined.

##Modifier
A modifier (or modificator) can be put on both the block and element, and is used to change the visual properties of a block or element. It is used so a block/element can be flexibly reused on other parts of a page, where a small modification is required.
