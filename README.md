# CYML

CYML is a simple JavaScript library for creating "Choose Your Own
Adventure"-style webpages. It is designed to be *data-driven*, one specifies
the game in HTML with no scripting required for basic use cases,
*self-contained*, one can add it to any suitably annotated HTML document and it
will play nicely, and *light-weight*, the markup is minimally irksome.


# Example

To start with here is an example of a basic game.

~~~ html
<article data-cyrole="main">
    <div data-cyrole="node" id="first-node">
        Hello, world! Press <a href="#second-node" data-cyrole="link">here</a>
        to continue.
    </div>
    <div data-cyrole="node" 
         id="second-node" 
         data-cyflag="status" 
         data-cyval="default(status, 'great')">
        Boy, was that not <span data-cyrole="switch" data-cyval="status">
            <span data-cyval="'great'">
                <a href="#" data-cyflag="status" data-cyval="'awful'">great</a>
            </span>
            <span data-cyval="'awful'">
                <a href="#" data-cyflag="status" data-cyval="'great'">awful</a>
            </span>
        </span>? Press <a href="#first-node" data-cyrole="link">here</a>
        to <span data-cyrole="switch" data-cyval="status">
            <span data-cyval="'great'">enjoy</span>
            <span data-cyval="'awful'">suffer through</span>
        </span> it all over again!
    </div>
</article>
~~~

# Concepts

Role
:   The *role* of an element is what it does within the game. The two simplest
    and central roles are *nodes*, those elements that count for a full screen, and *links*.

Flag
:   *Flags* are variables that get values assigned during a playthrough and
    which control aspects of the playthrough.

# Roles

Roles are specified, in general, by the `data-cyrole`, but may also be inferred
from details of the element. Alternatively, the role may be specified in the
element's `class` where `class="... cy-$foo ..."` is equivalent to
`data-cyrole="$foo"`. This latter method may present a desirable compromise
between concision and cleanliness. Note that in processing, the relevant
`data-cyrole` and `.cy-$role` attributes and classes will be created and so
either may be used safely in CSS.

## Main

The *main* element is the one containing the document. When no
`data-cyrole="main"` is specified, the widest `<article>` or `<body>` tag will
be used.

Only one main element in a document is permitted.

## Node

A *node* is one screen's worth of text. Each node can have an `id` given by the
corresponding HTML attribute. The `id` is used in constructing links between
nodes. 

Nodes cannot be nested. If the main element has child nodes which do not have
`data-cyrole="node"` specified, nodes will be created for those spare elements
as follows as we descend the children:

- `<section>` tags are treated as nodes whose `id` is the `id` attribute of the
  element.
- If an explicit node tag is encountered, all previously encountered spare
  elements are gathered into a node.
- If a *top-level* `<h1>` element is encountered, all previously encountered
  spare elements are gathered into a node.
- If a *top-level* `<hr>` element is encountered, all previously encountered
  spare elements are gathered into a node and this `<hr>` element is deleted.
- If the end of the children is reached, all spare elements are gathered into a
  node.
- The `id` of these gathered nodes is the `id` attribute of their first child.

For example, the following have equivalent node structures.

~~~ html
<article data-cyrole="main">
    <p>Node 1.</p>
    <section id="n2">Node 2.</section>
    <p id="n3">Node 3a.</p>
    <p>Node 3b.</p>
    <div data-cyrole="node" id="n4">Node 4.</div>
    <p>Node 5.</p>
    <hr/>
    <p>Node 6.</p>
    <h1>Node 7.</h1>
    <p>Node 7b.</p>
</article>
~~~

~~~ html
<article data-cyrole="main">
    <div data-cyrole="node">
        <p>Node 1.</p>
    </div>
    <section id="n2" data-cyrole="node">Node 2.</section>
    <div id="n3" data-cyrole="node">
        <p id="n3">Node 3a.</p>
        <p>Node 3b.</p>
    </div>
    <div data-cyrole="node" id="n4">Node 4.</div>
    <div data-cyrole="node">
        <p>Node 5.</p>
    </div>
    <div data-cyrole="node">
        <p>Node 6.</p>
    </div>
    <div data-cyrole="node">
        <h1>Node 7.</h1>
        <p>Node 7b.</p>
    </div>
</article>
~~~

### Arguments

`.content`
:   What is evaluated and displayed on entry.
`id`
:   The unique id of this node (default: `undefined`).
`data-cyflag`
:   The flag to set when this node is entered (default: `undefined`).
`data-cyval`
:   The value to set the specified `data-cyflag` when this node is entered, if
    specified (default: `true`).

## Link

A *link* connects one node to another that, when pressed, exits the parent node
and enters the child node. Its `href` specifies the `id` of the child node preceded by a '`#`' character. An `<a>` tag with an internal `href` will be treated as a link even if the role is not specified.

### Arguments

`.content`
:   What is displayed in the link.

`href`
:   The `id` of the child node to be preceded by a '`#`' character. As a 
    special case, if the `id` is just `#`, the child node is the parent node
    (this is useful if what is to be done is to reset the )

`data-cyflag`
:   The flag to set when this link is pressed (default: `undefined`).

`data-cyval`
:   The value to set the specified `data-cyflag` when this link is pressed, if
    specified (default `true`).

## Switch

A *switch* element has its content displayed depending on the value of a
certain expression given in `data-cyval`. A switch element will have some
finite number of children, its *cases*, each associated with a case value. The
content of the switch when rendered will be the content of the *first* child
with either with a `value` equal to the results of the
value of the switch or an `undefined` value; if no child matches by this rule, the switch will be
removed. There is no non-explicit method for specifying switches. However, the
immediate children of a switch will be treated as cases regardless of whether
this is specified.

### Examples

The most basic use is to change a bit of display text depending on whether a flag is set, for example

~~~ html
<div id="entrance" data-cyrole="node">
    A cave looms before you. A bit of metal glints at its entrance.

    <a href="#fight" 
       data-cyrole="link"
       data-cyflag="hasKnife" 
       data-cyval="true">Pick up the metal knife.</a>

    <a href="#fight"
       data-cyrole="link"
       data-cyflag="hasKnife" 
       data-cyval="false">Enter unarmed.</a> 
</div>
<div id="fight" data-cyrole="node">
    <div data-cyrole="switch" data-cyval="hasKnife">
        <span data-cyrole="case" data-cyval="true">
            The ogre, spotting your knife, laughs and gobbles you up!
        </span>
        <span data-cyrole="case" data-cyval="false">
            The ogre, seeing that you are peaceful, allows you to pass.
        </span>
    </div>
</div>
~~~

# Flags

A *flag* is a variable that can be 

# Expressions

The expressions that feature in `data-cyval` attributes are ordinary JavaScript
expressions which may use flags in their evaluation.

## Helper Functions

CYML makes available a few useful functions available in evaluations. It is
heavily recommended that you refrain from naming flags the same as these
functions; the results of doing so are undefined.

### `default`

`default(val: any, other: any): any` returns `val` if it is neither `null` nor
`undefined` and `other` otherwise. It is useful in order to ensure that a flag
is set but not to mess with it if it has been.

### `toggle`

`toggle(val: any): any` returns `val` if `val` is `null` or `undefined`,
`false` if `val` is truthy and `true` if `val` is falsey.

### `cycle`

`cycle(val: any, options: [any])` returns `null` if `options` is not a
non-empty list, `options[0]` if `val` is not a member of `options`, and
`options[(i + 1) % options.length]` where `i` is the index of `val`'s first
occurrence in `options`. This is useful for cycling text using the following pattern.

~~~ html
<a href="#" data-cyflag="__opt" data-cyval="cycle(__opt,[1,2,3])">
    <span data-cyrole="switch" data-cyval="__opt">
        <span data-cyval="1">Choice 1</span>
        <span data-cyval="2">Choice 2</span>
        <span>Default value</span>
    </span>
</a> 
~~~

