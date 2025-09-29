# CDA-HelpDesk Infrastructure

## Overview

CDA-HelpDesk is mostly written in the [Markdown](https://www.markdownguide.org/) language, and the files are arranged in a standard folder-per-topic format, [saved in Github](https://github.com/CancerDataAggregator/CDA-HelpDesk). To transform the markdown files into HTML, so they can be published as a website we use a tool called [mkdocs](https://www.mkdocs.org/). We use [Material](https://squidfunk.github.io/mkdocs-material/) for styling and many of the features (toolbars, search, etc) that make the website navigatable. The website is build and hosted at [ReadTheDocs](https://app.readthedocs.org/projects/cda/) on a paid account.

<img width="3224" height="2583" alt="HelpdeskInfrastructure" src="https://github.com/user-attachments/assets/72640eee-336b-44a4-8063-0ab8f32845bd" />

## Github files

The overall structure of the website repo is based on the requirements of mkdocs, Material, and readthedocs. 

### .github

This is a folder that has nothing to do with the website per se. Instead it holds files that give instructions to the github repository about how to act. It contains instructions for running special pipelines on pull requests, like the spell checker.

### docs

The docs folder holds all of the website contents. The sub-directories are set up for ease of navigating to various pages for editing, and do not necessesarily need to corrospond to the website navigation structure (which is specified explicitly in `mkdocs.yml`

### Material

Material is a themeing and styling tool. If you just use mkdocs to render HTML files you will get a very basic black and white website with a nav bar on the left and that's pretty much it. Any formatting beyond what can be done in markdown (bold, underline, and italic text) needs to be coded into the website seperately. This can be in the form of css, or jinja templates, or custom HTML, or lots of other things. To save time and effort, its much easier to use a package that has already built out a lot of these formatting things. So Material does all of the hard work of coding jinja and making color schemes and whatnot, and we just pass it parameters about which options we want. So while mkdocs.yml is required by mkdocs, nearly all of the content inside of it is calling specific Material variables. The `overrides` folder also contains files for Material. Specifically, it contains extra icons and some custom HTML files that make the landing page have boxed content that isn't part of base Material.

### mkdocs

mkdocs is a tool that takes markdown files and translates them into HTML, and to do that transformation it requires the markdown files to all be in a folder called 'docs', and for there to be a top level document called `mkdocs.yml` that basically contains any extra instructions or parameters for what should be included in the final HTML files. 


### readthedocs

readthedocs is a rendering and webhosting service. It requires a toplevel file called `.readthedocs.yaml` that contains the configuration info needed to properly build the website. 

### requirements

The website is mostly markdown, but also has several pages that are ipython notebooks. As part of building the website, readthedocs creates a virtual machine, loads all the software listed in `requirements.txt` and runs the ipython notebooks into rendered pages. That way they are always up to date.



