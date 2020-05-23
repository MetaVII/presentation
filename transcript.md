### Slide 1

Hello, I'm Alex and today we going to talk about how browser render pages.
It's important, because when you’re building web apps, you don’t just write isolated JavaScript code that runs on its own.
JavaScript you write is interacting with the environment. Understanding this environment, how it works and what it is composed of will allow you to build better apps and be well-prepared for potential issues that might arise once your apps are released.

### Slide 2

So, let’s see what the browser main components are:
* User interface: this includes address bar, back and forward buttons, bookmarking menu, etc. In essence, this is every part of the browser display except for the window where you see the web page itself.
* Browser engine: it handles interactions between user interface and rendering engine.
* Rendering engine: it’s responsible for displaying web page. Rendering engine parses HTML and CSS and displays the parsed content on the screen.
* Networking: these are network calls such as XHR requests, and, generally, browser communication with the rest of internet, including proxying, caching, and so on.
* UI backend: it’s used for drawing core widgets such as checkboxes and windows. This backend exposes a generic interface that is not platform-specific. It uses operating system UI methods underneath.
 * JavaScript engine: this is where JavaScript code gets parsed and executed.
 * Data persistence: if your app might need to store data locally, it can use different types of storage mechanisms, include localStorage, indexDB, WebSQL and FileSystem.
 
 In this talk, we’re going to focus on rendering engine, since it is handling parsing and visualization of HTML and CSS, which is something that most JavaScript apps are constantly interacting with.
 
 ### Slide 3
 
 Rendering engine receives contents of requested document from networking layer. First step of rendering engine is parsing the HTML document and converting the parsed elements to actual DOM nodes in a DOM tree. Imagine you have the following textual input, like on the left. The DOM tree for this HTML will look like the one on the right. Basically, each element is represented as the parent node to all of the elements, which are directly contained inside of it. And this is applied recursively.
 
 ### Slide 4
 
 Next step is Constructing CSSOM tree. CSSOM refers to CSS Object Model. While browser was constructing DOM of the page, it runs upon a link tag in head section which was referencing external CSS style sheet. Expecting that it might need that resource to render page, it immediately dispatched a request for it. 
 
 Let’s imagine that CSS file has the contents like on this slide. As with the HTML, engine needs to convert CSS into something that browser can work with — the CSSOM. Here you can see how CSSOM tree will look like. You may ask, why does CSSOM have a tree structure? When computing the final set of styles for any object on page, browser starts with the most general rule applicable to that node (for example, if it is a child of a body element, then all body styles apply) and then recursively refines computed styles by applying more specific rules. Let’s work with specific example that we gave. Any text contained within a span tag that is placed within body element, has a font size of 16 pixels and has a red color. Those styles are inherited from body element. If a span element is a child of a p element, then its contents are not displayed due to more specific styles that are being applied to it. 
 
 Also, note that above tree is not complete CSSOM tree and only shows styles we decided to override in our style sheet. Every browser provides a default set of styles also known as “user agent styles” — that’s what we see when we don’t explicitly provide any. Our styles simply override these defaults.
 
 ### Slide 5
 
 Visual instructions in HTML, combined with styling data from CSSOM tree, are being used to create a render tree.
 
 What is a render tree? This is a tree of visual elements constructed in order in which they will be displayed on the screen. It is the visual representation of HTML along with corresponding CSS. The purpose of this tree is to enable painting contents in their correct order.
 
 Each node in render tree is known as a renderer or a render object in Webkit. You can see how render tree of the above DOM and CSSOM trees will look like.
 
 To construct render tree, browser does roughly the following:
 * Starting at the root of DOM tree, it moves through each visible node. Some nodes are not visible (for example, script tags, meta tags, and so on), and are omitted since they are not reflected in rendered output. Some nodes are hidden via CSS and are also omitted from render tree. For example, span node — in the example above it’s not present in the render tree because we have an explicit rule that sets display: none property on it.
 * For each visible node, browser finds the appropriate matching CSSOM rules and applies them.
 * It emits visible nodes with content and their computed styles.
 
 Each renderer represents a rectangular area usually corresponding to a node’s CSS box. It includes geometric info such as width, height, and position.
 
 ### Slide 6
 
When renderer is created and added to tree, it does not have a position and size. Calculating these values is called layout. HTML uses a flow-based layout model, meaning that most of time it can compute geometry in a single pass. Layout is a recursive process — it begins at root renderer, which corresponds to &lthtml&gt element of HTML document. Layout continues recursively through a part or entire renderer hierarchy, computing geometric info for each renderer that requires it. Starting layout process means giving each node exact coordinates where it should appear on screen.
 
Next stage is Painting render tree. In this stage, renderer tree is traversed and renderer’s paint() method is called to display content on screen.

Painting can be global or incremental (similar to layout):
* Global — it is when entire tree gets repainted.
* Incremental — it's when only some of renderers change in a way that does not affect entire tree. renderer invalidates its rectangle on screen. This causes OS to see it as a region that needs repainting and to generate a paint event. Operating System does it in a smart way by merging several regions into one.

In general, it’s important to understand that painting is a gradual process. For better UX, rendering engine will try to display contents on screen as soon as possible. It will not wait until all the HTML is parsed to start building and laying out render tree. Parts of content will be parsed and displayed, while process continues with rest of content items that keep coming from network.

### Slide 7

Now, let's talk about how can you optimize your application. If you’d like to optimize your app, there are four major areas that you need to focus on. These are areas over which you have control:
* First is JavaScript. - When it comes to rendering, we need to think about the way your JavaScript code will interact with DOM elements on page. JavaScript can create lots of changes in UI, especially in SPAs.
* Next one is style calculations - Modifying DOM through adding and removing elements, changing attributes, etc. will make th browser recalculate element styles and, in many cases, layout of entire page or at least a part of it.

To optimize rendering, you can do following:
  * Reduce complexity of your selectors. Selector complexity can take more than 50% of time needed to calculate styles for an element, compared to rest of the work which is constructing the style itself.
  * Reduce number of elements on which style calculation must happen. In essence, make style changes to a few elements directly rather than invalidating page as a whole.

* Next area is Layout. Layout re-calculations can be very heavy for You can:
  * Reduce number of layouts whenever possible. When you change styles, browser checks if any of changes require layout to be re-calculated. Changes to properties such as width, height, left, top, and in general, properties related to geometry, require layout. So, avoid changing them as much as possible.
  * Use flexbox over older layout models whenever possible. It works faster and can create a huge performance advantage for your app.
  
* Last ares is Paint. This often is the longest-running of all tasks so it’s important to avoid it as much as possible. Here is what we can do:
  * Changing any property other than transforms or opacity triggers a paint. Use it sparingly.
  * If you trigger a layout, you will also trigger a paint, since changing geometry results in a visual change of the element, so you should think twice before using it.
  
### Slide 8

So, I hope today's talk will help you design better optimized web apps, thank you for watching, bye.
  
  
