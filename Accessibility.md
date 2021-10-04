# Accessibility

> "The power of the Web is in its universality. Access by everyone regardless of disability is an essential aspect." – Tim Berners-Lee

## Motivation

According to (Datareportal)[https://datareportal.com/global-digital-overview], there are 4.72 billion users that use the internet, which is around 60% of the world population. Among those users, many of them have some sort of disability. To say it with numbers, approximately 15% of the world population live with disabilities which translates to about **702 million internet users**. Because of this, it is mandatory that each web app is adequately created and follows accessibility guidelines so all users can use it.

Accessibility in web applications, or web accessibility for short, is a practice used by web developers to make their applications accessible to all users, regardless of disability. Today, this practice makes an application stand out and increases the search engine optimization (SEO) rating.

## Introduction

Accessibility is a practice that ensures that people with disabilities can do what they need to do in a similar amount of time and effort as someone that does not have a disability. In terms of web accessibility, each user should perceive, navigate, use, interact and contribute to the web application the same as someone who does not have a disability.

Web accessibility encompass all disabilities that affect access to web applications: auditory, cognitive, neurological, physical, speech, and visual. Also, accessible web applications benefit users without disabilities:

- users that are using mobile phones, smartwatches, TVs, different input modes, etc.
- older people with changing abilities
- users with "temporary disabilities" such as a broken arm, ear infection, etc.
- users with "situational limitations" such as bright sunlight, noisy environment
- users with slow internet connection

The web is an increasingly important resource in all aspects of life, especially today when there is a global pandemic. Everything is transferring to an "online world" from government, education, health care, shopping, etc. Because of this migration, everyone must have equal access whether they have a disability or not. Further, this is a fundamental human right in the [United Nations Convention on the Rights of Persons with Disabilities](https://www.un.org/development/desa/disabilities/convention-on-the-rights-of-persons-with-disabilities.html). On many occasions, and this is becoming a trend now, web accessibility is required by law.

## Accessibility tree

An accessibility tree is a data structure that contains data from the web application. Every page has an accessibility tree, and this is generated automatically by the browser. If a user uses an assistive technology, then the assistive technology will use this tree to present data to a user in a way they can perceive it. The two of them communicate over an API.

For an accessibility tree to be an excellent representation of an application, the developer should be careful to express the page's semantics correctly. The developer makes sure that the crucial elements on the page have the correct accessible roles, states, and properties and that all elements specify accessible names and descriptions. To make your life easier, there are a lot of semantic elements already built into the browser, and we can rely on them. E.g., use buttons instead of div and span elements, use proper input type, make sure there are labels and/or text alternatives, etc.

![Google Chrome Accessibility Tree](/img/accessibility_tree.png)

## Assistive technologies

Assistive technology is any item, piece of equipment, software or product system that is used to improve functional capabilities of a person with disabilities. Some examples of assistive technology are special-purpose computers, prosthetics, special switches, special keyboards, wheelchairs, etc. However, there is also a wide variety of assistive technology used on the web.

Assistive technologies scan the page from the top to the bottom. E.g., we have a search box where the search (submit) button is placed before an input in the Document Object Model (DOM). As a result, a button will be accessed before the input in the accessibility tree, which might be a bad user experience, so developers and designers should watch out for such situations.

### Screen readers

Screen readers are software used by blind or visually impaired people. This software processes the content on the desktop and in the web browser and converts it to other forms of data that user can use. Often content is converted to speech so users can listen to that, but there are also variants when content is converted to Braille. Typically, screen readers provide other functions such as shortcut keys, different modes for processing and interacting with content. Some examples of screen readers are Apple VoiceOver and Window Eyes.

#### Apple's Voice Over

Every Apple device is equipped with a service called VoiceOver Utility. This service is a software that helps users perceive, use and navigate the content shown on the screen. By default it is not enabled so to enable it press <kbd>Command</kbd> + <kbd>F5</kbd> on your keyboard or follow these steps:

1. Go to System Preferences
2. From the menu, select Accessibility
3. Select VoiceOver
4. Check Enable VoiceOver.

To do things with the VoiceOver, it is necessary to learn the VO key. VO stands for VoiceOver and the key is a combination of <kbd>Control</kbd> and <kbd>Option</kbd> keys on a keyboard. Now, once this is known, it is possible to do the following

- Help: VO key + <kbd>H</kbd>
- Navigation (next/previous): VO key + <kbd>Left/Right arrow</kbd>
- Navigation by heading: VO key + <kbd>Command</kbd> + <kbd>H</kbd>
- Etc.

## Semantics in HTML

By definition semantics in programming refers to the meaning of a piece of code, e.g., _"What effect does running that line of JavaScript have?"_, or _"What purpose or role does that HTML element have?"_ (rather than _"What does it look like?"_).

In the HTML world semantics refers to the self-explanatory HTML elements. These elements give you context and purpose simply by looking at the code without seeing the page. Elements like: `aside`, `header`, `main`, `button`, `nav`, `section`, etc. are semantic HTML elements. The whole list of semantic elements can be found in [MDN docs](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

Semantic elements are good to use since they are:

- **Easier to work with** — you most of the time get some functionality built in plus it is easier to understand the page without looking at the final product
- **Better on slow networks** — semantic HTML elements are smaller in file size than non-semantic (generic) elements. E.g., button element will give you default styles, built-in keyboard event listeners, semantic value, will be focusable, etc.; on the other hand, to make the same component using a div element everything mentioned would mean that more code is required to write which will result in a bigger bundle size.
- **SEO benefits** — search engines give more importance to keywords inside semantic elements like headings, links, etc. than keywords included in non-semantic divs, etc., so your documents will be more findable by searchers.

## ARIA

ARIA stands for Accessible Rich Internet Application. ARIA is a set of attributes that define ways to make content and web applications more accessible to people with disabilities. Using ARIA attributes, any component should be accessible so assistive technologies can read the data. Today, there are many widgets and components that are ready out of the box and require little to no extra work to make them accessible. If there is no HTML element that would match our case, ARIA attributes take care to make that component accessible. To learn how to use ARIA there is great documentation from The World Wide Web Consortium (W3C) on ARIA attributes as well as ARIA guidelines on how to use it in a real case.

- [Introduction and description of all ARIA properties](https://www.w3.org/TR/wai-aria/)
- [ARIA design patterns](https://www.w3.org/TR/wai-aria-practices/)
- [Accessibility guidelines](https://www.w3.org/TR/WCAG21/)

### ARIA roles

An ARIA role is the main indicator of a type. This semantic association allows assistive technologies to present and support interaction with the element. It is set on an element using a role attribute and must not be changed over time or with an interaction. If there is a reason to do this, the developer should remove the old element and create a new one with a new role.

The role taxonomy uses the following relationships to relate different roles in relation to each other: a superclass role, subclass roles, related concepts and a base concept. Superclass role is the role that the current role extends in the taxonomy. Subclass roles is the list of roles to which this role is the superclass. Related concepts are informative data about similar concepts from other specifications and those concepts are not necessarily identical. E.g., progressbar and status roles are related concepts. Base concept is information about objects that are considered prototypes for this role, e.g., HTML element checkbox is a base concept of a checkbox role.

ARIA roles are categorized into six categories:

- Abstract roles are used to support role taxonomy for the purpose of defining general roles. These roles are used for the ontology and should not be used in the content.
- Widget roles describe common interactive patterns for elements that can be used as a standalone user interface or as a part of a larger, composite widget or as a composite widget. Some roles from this category already have a proper semantic HTML element and it is recommended to use those instead of using ARIA roles, e.g., `button`, `checkbox`, `link`.
- Document structure roles provide a structural description for a section and these roles are often non-interactive. Some document roles, e.g., `form`, map onto existing HTML tags and are only meant for cases when using the native tag is not possible.
- Landmark roles identify large content areas and are used by assistive technology for navigation. All content of a page should be placed inside an element with a landmark role. As a result, all content could be navigated to by use of landmarks. For some roles in this category, there are HTML elements with the same name, e.g., `<main>` and `role="main"`.
- Live region is a part of an application where some data is updated because of some event, e.g., toast notifications and chat log. Since the user's focus might be elsewhere on a page when the data in this region updates, ARIA has provided a collection of ARIA properties that are used with this role: `aria-live`, `aria-relevant`, `aria-atomic`, and `aria-busy`. With these properties developers can tell assistive technologies when to move focus on to this region. Live region role is a role that indicates a live region.
- Window roles help in creating a window within the application. The two roles that inherit a window role are `dialog` and `alertdialog`.

### ARIA attributes

Term _ARIA attributes_ is a synonym for _ARIA states and ARIA properties_. This synonym is used because ARIA states and ARIA properties have similar definitions and are often hard to tell apart.

ARIA state is a dynamic property that expresses characteristics of an element that may change due to user interaction. ARIA states do not affect the essential nature of the element, but they give further information about the element.

ARIA property is an element’s attribute that is crucial to the nature of the element. A change of a property may remarkably affect the presentation of an element.

As described, both ARIA states and ARIA properties provide some information about an element, and both are part of the definition of an element’s role. In terms of web accessibility, they both have the same markup: the name of an ARIA attribute is prefixed by `aria-`, e.g., `aria-current`. Even though they are similar, they are maintained conceptually distinct to clarify minor differences between them. One major difference is that the values of ARIA properties are usually unchanged throughout the life cycle of an application than the values of ARIA states which are expected to change due to user interactions.

ARIA attributes are categorized into four categories:

- Widget attributes are attributes used for common user interface elements that receive and process user actions. Such attributes help assistive technology to better represent the state of a user interface element to a user, e.g., `aria-required="true"` will tell the user that this field is required before submitting a form.
- Live region attributes are a set of four ARIA attributes: `aria-atomic`, `aria-busy`, `aria-live`, and `aria-relevant`. These attributes are specific to live regions and may be applied to any element. The purpose of these attributes is to indicate that content may change without the element having focus. E.g., `aria-live="off"` won’t change user’s focus if content changes, `aria-live="polite"` will change user’s focus when content changes with the flow of the content and `aria-live="assertive"` will change user’s flow of content immediately when content changes.
- Drag-and-drop attributes are used to indicate information about drag-and-drop elements and to mark draggable elements and their drop targets. There are only two attributes: `aria-dropeffect` and `aria-grabbed`.
- Relationship attributes are attributes that will indicate a relation between elements that cannot be determined from a document structure. To create relationships between elements, element’s id property is used. E.g., `aria-errormessage` identifies an element that contains an error message for a given element.

### Rules of using ARIA

W3C has created a document [Using ARIA](https://www.w3.org/TR/using-aria/) that covers five rules and gives some more answers to frequently asked questions about using ARIA in web applications. These rules and answers demonstrate how to use ARIA and it helps with dynamic content and complex user interactions developed with JavaScript.

First rule says that if the developer can use a native HTML element that implements wanted behavior and semantics, the developer should use it instead of using a generic element with ARIA role, states, and properties.

Second rule says the native semantics should not be overridden. E.g., if there is a need to create a tab that is also a heading instead of doing this

```html
<h2 role="tab">Tab Heading</h2>
```

the developer should do

```html
<div role="tab">
  <h2>Tab Heading</h2>
</div>
```

Third rule says that all interactive components that can receive user input such as mouse click or mouse drag, must be scripted in a way that all interactions are possible with keyboard only. E.g., click on a button can be emulated by pressing the <kbd>Enter</kbd> or <kbd>Space</kbd> key on a keyboard.

Fourth rule says that `role="presentation"` and/or `aria-hidden="true"` should not be used on a focusable element or any element that contains a focusable element as a child. If a focusable element is not visible on a page, then the `aria-hidden="true"` property must be set, but the developer must ensure the element is not focusable by using `tabindex="-1"`. If an element is hidden using `display: none` or `visibility: hidden`, the browser will make this implicit for the developer.

Fifth rule says that all interactive elements must have an accessible name. An element has an accessible name only if the Accessibility API has a name value. E.g., an input can have a text next to it and to a user without disabilities can see the label, but users using assistive technology won’t be able to see that. But if an accessible name is set, each user will see the element’s label. There are multiple ways to label an element:

- using a `<label>` element
- using an `aria-label` ARIA property
- using an `aria-labelled` ARIA property.

## Examples

In following section, you can find links to ARIA practices for many components. Each component has multiple sections:

- descriptions
- examples
- keyboard support
- ARIA roles/states/properties and tabindex attributes
- Source code HTML and JS (CSS sometimes)

Here is the list of components:

- [Accordion](https://www.w3.org/TR/wai-aria-practices-1.1/#accordion)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/accordion/accordion.html)
- [Alert](https://www.w3.org/TR/wai-aria-practices-1.1/#alert)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/alert/alert.html)
- [Alert and message dialogs](https://www.w3.org/TR/wai-aria-practices-1.1/#alertdialog)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/alertdialog.html)
- [Breadcrumb](https://www.w3.org/TR/wai-aria-practices-1.1/#breadcrumb)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/breadcrumb/index.html)
- [Button](https://www.w3.org/TR/wai-aria-practices-1.1/#button)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/button/button.html)
- [Carousel](https://www.w3.org/TR/wai-aria-practices-1.1/#carousel)
  - [Auto-rotating image carousel example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/carousel/carousel-1.html)
- [Checkbox](https://www.w3.org/TR/wai-aria-practices-1.1/#checkbox)
  - [2 state checkbox example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/checkbox/checkbox-1/checkbox-1.html)
  - [3 state checkbox example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/checkbox/checkbox-2/checkbox-2.html)
- [Combo Box](https://www.w3.org/TR/wai-aria-practices-1.1/#combobox)
  - [Combo box with list popup example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/combobox/aria1.1pattern/listbox-combo.html)
  - [Combo box with grid popup example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/combobox/aria1.1pattern/grid-combo.html)
  - [Combo box with inline autocomplete example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/combobox/aria1.0pattern/combobox-autocomplete-both.html)
  - [Combo box with list autocomplete example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/combobox/aria1.0pattern/combobox-autocomplete-list.html)
  - [Combo box with without autocomplete example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/combobox/aria1.0pattern/combobox-autocomplete-none.html)
- [Dialog Modal](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal)
  - [Modal example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/dialog.html)
  - [Datepicker dialog example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/datepicker-dialog.html)
- [Disclosure](https://www.w3.org/TR/wai-aria-practices-1.1/#disclosure)
  - [Long image description example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/disclosure/disclosure-img-long-description.html)
  - [F.A.Q. example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/disclosure/disclosure-faq.html)
  - [Navigation example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/disclosure/disclosure-navigation.html)
- [Feed](https://www.w3.org/TR/wai-aria-practices-1.1/#feed)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/feed/feed.html)
- [Grid](https://www.w3.org/TR/wai-aria-practices-1.1/#grid)
  - [Layout example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/grid/LayoutGrids.html)
  - [Data example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/grid/dataGrids.html)
  - [Advanced data example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/grid/advancedDataGrid.html)
- [Link](https://www.w3.org/TR/wai-aria-practices-1.1/#Link)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/link/link.html)
- [Listbox](https://www.w3.org/TR/wai-aria-practices-1.1/#Listbox)
  - [Scrollable listbox example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/listbox/listbox-scrollable.html)
  - [Collapsible listbox example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/listbox/listbox-collapsible.html)
  - [Rearrangeable listbox example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/listbox/listbox-rearrangeable.html)
- [Menu/Menu bar](https://www.w3.org/TR/wai-aria-practices-1.1/#Menu)
  - [Navigation menubar example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/menubar/menubar-1/menubar-1.html)
  - [Editor menubar example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/menubar/menubar-2/menubar-2.html)
- [Menu Button](https://www.w3.org/TR/wai-aria-practices-1.1/#Menubutton)
  - [Navigation menu button example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/menu-button/menu-button-links.html)
  - [Actions menu button example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/menu-button/menu-button-actions.html)
  - [Actions menu button example using aria-activedescendant](https://www.w3.org/TR/wai-aria-practices-1.1/examples/menu-button/menu-button-actions-active-descendant.html)
- [Radio Group](https://www.w3.org/TR/wai-aria-practices-1.1/#Radiobutton)
  - [Example using roving tabindex](https://www.w3.org/TR/wai-aria-practices-1.1/examples/radio/radio-1/radio-1.html)
  - [Example using aria-activedescendant](https://www.w3.org/TR/wai-aria-practices-1.1/examples/radio/radio-2/radio-2.html)
- [Slider](https://www.w3.org/TR/wai-aria-practices-1.1/#slider)
  - [Horizontal example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/slider/slider-1.html)
  - [Example with aria-orientation and aria-valuetext](https://www.w3.org/TR/wai-aria-practices-1.1/examples/slider/slider-2.html)
- [Slider (Multi-Thumb)](https://www.w3.org/TR/wai-aria-practices-1.1/#slidertwothumb)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/slider/multithumb-slider.html)
- [Spinbutton](https://www.w3.org/TR/wai-aria-practices-1.1/#spinbutton)
  - [Datepicker spinbuttons example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/spinbutton/datepicker-spinbuttons.html)
- [Table](https://www.w3.org/TR/wai-aria-practices-1.1/#table)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/table/table.html)
- [Tab](https://www.w3.org/TR/wai-aria-practices-1.1/#tabpanel)
  - [Example with automatic activation](https://www.w3.org/TR/wai-aria-practices-1.1/examples/tabs/tabs-1/tabs.html)
  - [Example with manual activation](https://www.w3.org/TR/wai-aria-practices-1.1/examples/tabs/tabs-2/tabs.html)
- [Toolbar](https://www.w3.org/TR/wai-aria-practices-1.1/#toolbar)
  - [Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/toolbar/toolbar.html)
- [Tooltip](https://www.w3.org/TR/wai-aria-practices-1.1/#tooltip)
  - No working example
- [Tree View](https://www.w3.org/TR/wai-aria-practices-1.1/#treeview)
  - [File directory using computed properties](https://www.w3.org/TR/wai-aria-practices-1.1/examples/treeview/treeview-1/treeview-1a.html)
  - [File directory using declared properties](https://www.w3.org/TR/wai-aria-practices-1.1/examples/treeview/treeview-1/treeview-1b.html)
  - [Navigation treeview using computed properties](https://www.w3.org/TR/wai-aria-practices-1.1/examples/treeview/treeview-1/treeview-1a.html)
  - [Navigation treeview using declared properties](https://www.w3.org/TR/wai-aria-practices-1.1/examples/treeview/treeview-1/treeview-1a.html)
- [Treegrid](https://www.w3.org/TR/wai-aria-practices-1.1/#treegrid)
  - [Treegrid email example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/treegrid/treegrid-1.html)
- [Window Splitter](https://www.w3.org/TR/wai-aria-practices-1.1/#windowsplitter)
  - No working example

Find more examples at [WAI ARIA practices](https://www.w3.org/TR/2019/NOTE-wai-aria-practices-1.1-20190814/examples/)

## Other

- Following links will bring you to common patterns that should be followed when implementing a complex component that has no native semantic counterpart
  - [WAI-ARIA design patterns](https://www.w3.org/TR/wai-aria-practices-1.1)
  - [eBay Mind Patterns](https://ebay.gitbook.io/mindpatterns/)
- Accessibility acceptance success criteria testing checklist: [a11yengineeer.com](https://www.a11yengineer.com/)
  - Select application needs and a checklist will be generated to check if everything is implemented correctly
- More about web accessibility and ARIA
  - [Google Web Fundamentals](https://developers.google.com/web/fundamentals/accessibility)
  - [MDN ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)
  - [WAI ARIA](https://www.w3.org/TR/wai-aria-1.1/)
  - [Using ARIA](https://www.w3.org/TR/using-aria/)
  - [Accessibility guidelines](https://www.w3.org/TR/WCAG21/)
- Storybook has an [accessibility addon](https://storybook.js.org/addons/@storybook/addon-a11y/) that can give you instant feedback about accessibility of the component
