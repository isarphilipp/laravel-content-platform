# Laravel-Content-Platform

*Finally A CMS Platform That Does Not Suck*

**OR** 

*CMS as an API*

## Motivation

It's 2017, people discuss AI and self-driving cars, yet there is no state-of-the-art enterprise content management system built on PHP/Laravel and opensource technology.

**Are you serious?**

Yes, we know about Wordpress, October CMS, Drupal, TYPO3, Neos and its 1.398 brothers and sisters. They all serve a special purpose, but none of them is suited for enterprise websites at scale. There are various reasons for it which you can read below, for the time being please just assume it's correct.

**Enterprise - Wait what?!**

When we talk about enterprise CMS, we do not mean your personal blog, nor the website of the bakery round the corner. We're talking about corporates with > 100 Mio EUR in annual revenue who spend a meaningful budget on design, development and maintenance of their web properties. Companies that would usually go proprietary with OpenText (IBM) and the likes.

**Open source you say?**

The main shortcoming of existing opensource solutions is that they try to build a monolith, an integrated codebase for frontend and backend (= the "editor's view"). We believe that a modern CMS needs to take a radically different approach: Build a solid content API first and provide some basic building blocks for the editor's backend second.

**What are we doing differntly?**

Real world usage of CMS shows that both backend and frontend are typically highly customized to clients' needs. The old approach of "building a one-size fits all solution" does not fit in practice. Integrators usually spend more time on customizing backends and functionality around an existing codebase than building it from "ready-to-use-building-blocks". 

By building a solid and extendable content API, integrators are empowered to build the systems they need. With joy and without caring for low-level requirements like routing, permissions, localization, containerization, versioning, page trees, just to name a few.

**To speak in examples:**

We want to become the Laravel of CMS platforms: Provide abstractions, conventions and joy! Developer happiness and maintainability are high virtues for us.

## Key features the platform should provide

We've developed a lot of website for above mentioned corporates. That's way I am save to assume that I've seen 95% of the real requirements, all of them which went into this document.

### 1. Page tree and data model

	                      LOCALES
	
	                      de-DE     nl-NL    en-US    en-GB    ch-CN
	PAGE TREE             +-----------------------------------------+
	
	+--------+            +--+     +--+      +--+     +--+     +--+
	| Page A |            |  |     |  |      |  |     |  |     |  |
	+--------+            +--+     +--+      +--+     +--+     +--+
	|
	|
	|    +--------+       +--+               +--+     +--+     +--+
	+----+ Page B |       |  |               |  |     |  |     |  |
	|    +--------+       +--+               +--+     +--+     +--+
	|
	|    +--------+                +--+                        +--+
	+----+ Page C |                |  |  <-+                   |  |
	     +--------+                +--+    |                   +--+
	                                       |
	                                       |
	                                       +
	                               Each localized version has:
	
	                               * title
	                               * description
	                               * slug
	                               * permissions
	                               * ...


At the core of the data model is a page tree. It keeps track of the hierarchy of pages. Contrary to existing CMS solutions, we will use a nested set as data structure to persist the hierarchy into a relational database (see [Laravel Baum](https://github.com/etrepat/baum) for details). The nested set model allows for very efficient queries on the tree structure (e.g. "give me all subpages of X" without doing any recursions).

For frontend routing, its hierarchy is used to build up URLs and serve requests.

Each `Page` is assigned to a specific template (basically a blade/twig markup), which determines its overall visual representation (i.e. where is the header, the footer, the menu, the content, the sidebar, ...).

Localization needs to be built in from the ground up. Each page in the tree can have 0:n localized versions. A localized version is a representation of the page in a specific locale.

It is very important that not all pages need to exist in all locales. Even more: Traditional CMS require that a page exists in at least one "default locale" (that would be `de-DE` in the example above). This traditional requirement blocks many real world use cases in TYPO3 and Wordpress, because it does not allow modelling international content in one unified tree.

A localized version (denoted as the square box in the diagram above) carries basic information about a page: It's title, description, localized slug, permissions for editing etc.

Please note that the localized version does not carry any real content. That's what we need content block templates for - on to the next chapter! ;)

### 2. Content block templates

A localized version serves as container for content block templates.

The developer can define as many content block templates in code as he wishes. We call these content block templates `Panels` from time to time. In the example below, the developer created `Panel 1`, which has two input fields: One for a headline and one textarea for paragraph text.

It is up to the specific CMS project to define and implement these content block templates.
	
	 +--+
	 |  |  +----+
	 +--+       |
	            v
	
	          Panel 1 (Headline + Paragraph)
	
	
	          Panel 23 (Image Slider)
	
	
	          Panel 15 (List of Bullet Points)
	
	
	          Panel 5 (Paragraph)


Technically, a content block template consists of two parts:

1. A definition of fields that the editor can fill
2. A blade/twig template, how the content block template will be rendered in the frontend

The object for the localized version stores the values that the editor fills in.

The panels added to a localized version do have an ordered sequence. In the example above, Panel 1 goes before Panel 23, which goes before Panel 15, which goes before Panel 5.

As pages are usually not built up with a single sequence (you might have main content and a sidebar, as a simple example), it might be necessary to add a concept of `channels`. Content block templates are always tied to a specific channel. For example, you define one channel for main content and one channel for the side bar.

### 3. Permissions

The API needs to allow for fine grained permissions.

The permissions layer needs to be customizable so that the integrator can fine tune it to the specific project needs.

The basic permission layer (should we call it a driver?) would cater for the following use case:

0. Each user/editor can have one or many roles
1. Each page has role based permissions for viewing, editing, deleting.
2. Permissions are inherited down the page tree. If a permission is set on a higher level in the hierarchy, then lower pages in the hierarchy inherit these permissions.
3. However, if a lower page has its own permissions defined, then they take precedence.

### 4. Versioning and drafting

For now I can only specify the requirements but have no thought out implementation in mind.

1. It needs to be possible that an editor creates a new page in draft mode which is not visible in the frontend -> likely piece of cake.

2. An editor must have the possibility to edit an existing live page without directly affecting the actual live page. This is because often times, changes to a page need to be approved by other editors.

3. All changes to content need to be versioned, so that we can (a) track who did a change and when and (b) that we can rollback to a certain version later.

While the three points above seem solvable, the third is causing me more headache:

4. In draft mode, it should also be possible to move pages around in the page tree without affecting the live site.

### 5. Assets and media

The system needs to have a basic medialibrary.

Surprise: [Spaties Media Library](https://github.com/spatie/laravel-medialibrary) would probably fit quite well.

Image manipulations would be done on delivery time and well cached: Imagine we only store one "large" version of an image and deliver its thumbnails, regular version and full version via crafted [Laravel Intervention Links](http://image.intervention.io/use/url).

Of course, building on Flysystem, assets can directly be hosted on services like S3.

### 6. Routing (used in frontend)

The system needs to ship with a basic router, likely based on existing Laravel functionality.

The basic router would ingest slugs (`http://www.example.com/slugs/down/the/hierarchy`) and resolve them through the page tree.

Localization is typically a part of the URL scheme, so the basic router should optionally be capable of resolving localized URLs (`http://www.example.com/de-de/slugs/down/the/hierarchy`).

Being able to customize the routing scheme is an important requirement.

### 7. Linking

There are two ways how pages link to each other:

1. The editor has used a special `link` input field in an content block template. This field stores the `id` of the target page. This case is trivial: By using a special directive in the blade/twig template that renders the content block template, we can resolve the `id` into a `url` (the directive has to have knowledge about the routing scheme, but that's solvable).

2. The editor uses a free text field, enters some html, which contains an `<a href="">link</a>` tag. We need to be able to parse these links and resolve them to proper URLs. 

### 8. Extensibility

One core proposition of the platform is extensibility. Rather than providing the whole CMS framework with unchangable core functionality, we strive to giving the integrator as much flexibility as he needs. 

This can be achieved by using best practices from the Laravel ecosystem.

### 9. Building blocks for backend GUI

Although the platform is "API first", we need to provide basic building blocks for the editors backend. 

Vue JS seems like a good option for implementation.

Basic parts would be:

1. Page tree
2. Page editing
3. Content editing
4. Assets and media management
5. User management