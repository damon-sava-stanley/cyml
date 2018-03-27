# Datatypes

~~~ js
MAIN = {
    nodes: {(id: string) => NODE},
    html: HTML,
}
NODE = {
    id: string,
    cyflag: string?,
    cyval: any?,
    contents: [NODE_CONTENT],
    html: HTML
}
NODE_CONTENT = LINK | SWITCH;
LINK = {
    id: string,
    href: string,
    cyflag: string?,
    cyval: any?,
    contents: [NODE_CONTENT],
    html: HTML
}
SWITCH = {
    id: string,
    cyval: any?,
    cases: [CASE],
    html: HTML
}
CASE = {
    id: string,
    cyval: any?,
    contents: [NODE_CONTENT],
    html: HTML
}
CYML = {
    MAIN
}
~~~

