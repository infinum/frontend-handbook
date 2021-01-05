**BEM** stands for block - element - modifier. It is a philosophy of naming and structuring HTML and CSS classes. This guide will not only touch on the SASS side of things, but also on the HTML layout.

The basic idea of BEM is to separate content into manageable blocks - a type of componentization.

##Block
A block is a piece of content that is encapsulated, meaning it doesn't rely on any other part of the page to be complete. Simplest example of a block could be a form. A block can be repeated endlessly on a site and should be able to fit into any other content without breaking too much (or at all, for that matter).

##Element
An element is a part of a block. An element on its own is meaningless and should always be below a block in the HTML tree. An element describes a part of a block. By combining multiple elements, a block is defined.

##Modifier
A modifier (or modificator) can be put on both a block or an element to change their visual properties. It is used so that a block/element can be flexibly reused on other parts of a page, where a small modification is required.
