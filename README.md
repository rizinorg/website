# Rizin Website

## Local Installation

1. Install or download the **Extended** version of [Hugo](https://gohugo.io/getting-started/installing/)
2. Clone this repository and its submodules:
    ```
    $ git clone --recurse-submodules https://github.com/rizinorg/website
    ```
3. Enter the cloned repository and "serve" the website using Hugo
    ```
    $ cd website
    $ hugo serve
    ```

## File tree description

```
.
├── assets
│  └── css
│     └── custom
│        └── rizin.css           <--- Edit CSS here
├── config.yml                   <--- Hugo Config for the website
├── content                      <--- Markdown files for single pages
│  ├── archives.md
│  ├── code-of-conduct.md        
│  ├── organization.md
│  ├── posts                     <--- Markdown blog posts
│  │  └── example
│  │     └── index.md
│  └── teams                     <--- Special folder for team pages
│     ├── community.md
│     ├── core.md
│     ├── Distributions and Packaging.md
│     ├── package-manager-and-plugins.md
│     └── . . .
├── data
│  └── teams.json                <--- JSON with teams data and descriptions
├── layouts                      <--- Special layouts
│  ├── _default
│  │  ├── organization.html
│  │  └── team.html
│  ├── index.html                <--- Main page
│  └── partials                  <--- override theme or introduce new partials
│     ├── extend_head.html
│     ├── header.html
│     └── main-page.html
├── static
│  └── images
│     └── rizin.svg
|
|
└── themes                       <--- The base theme. Don't touch files in here
   └── hugo-PaperMod

```

## Demo GIF

When showing Rizin demos the GIF are generated with
[Terminalizer](https://www.terminalizer.com/) by using this [config
file](https://github.com/rizinorg/website/blob/main/.terminalizer-config).
