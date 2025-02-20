# jinja2-render-json
*A Python Tool to Render Jinja2 Templates on the Command Line*

Fork of jinja2-render (https://github.com/pklaus/jinja2-render)

**jinja2-render-json** hugely simplifies creating different versions of a text file.
It takes a jinja2 Template file as input and renders it to a final document
using a context loaded from a Python script.
It was invented for use with Dockerfiles but its use case goes far beyond.

This is useful for example in the following cases:

* The desired document (eg. Dockerfile) contains a lot of repeated statements.
  In a Jinja2 Template they can be replaced by a [For Loop][].
* The configuration space for the desired document is large and the differences
  would be small, so maintaining a single template file is much easier.

### Workflow

```
                     jinja2-render
                          ↓

Jinja2 Template File            Rendered File
(eg. Dockerfile.jinja2)   🡆    (eg. Dockerfile)
```

### Synopsis

```
$ jinja2-render -h

usage: jinja2-render [-h] [-c CONTEXTS] [-f TEMPLATE]
                     [-o OUTPUT]
                     [which]

Render a Jinja2 template from the command line.

optional arguments:
  -h, --help   show this help message and exit
  -c CONTEXTS  The JSON String defining the contexts to
               render the template. (default: '{}')
               Example  '{"egg":"chicken","itemList":["test","test2","test3"]}'
  -f TEMPLATE  The Jinja2 template to use. (default:
               Dockerfile.jinja2)
  -o OUTPUT    The output file to write to. (default:
               Dockerfile)
```

### Minimal Example

Content of `Dockerfile.jinja2`:

```Dockerfile
FROM {{ base_img }}

RUN apt-get update \
 && apt-get install -yq {{ packages | join(' ') }}

RUN echo "Done!"

```

Content of JSON:
```json
'{"v1.0": {"base_img": "debian:10-slim", "packages": ["glibc-devel", "turboshutdown"]}'}
```

Call to `jinja2-render`:

```sh
jinja2-render v1.0
```

Resulting in the following rendered Dockerfile:

```Dockerfile
FROM debian:10-slim

RUN apt-get update \
 && apt-get install -yq glibc-devel turboshutdown

RUN echo "Done!"
```

### Original Use Case

The script was originally developed to automate building different configurations
of the control system software EPICS and it's various modules.
Traditionally, those modules were installed by a single shell script with
fixed versions (i.e. synApps), which is bad in the Docker world
as a build failure at the end if it would require rebuilding everything before.

### How it compares to other templating tools

By using a Python file to provide the context for the template, much more
flexibility is possible than with a simple solution such as jinja2-cli or j2cli.
They allow to use files such as YAML, INI or JSON as input for variables.

A Python script can - however - in addition to defining static information
ease additional tasks tasks such as dependency management.
As an elaborate example of such a use case, have a look at
[this](https://github.com/pklaus/docker-epics/tree/master/epics_contapps),
where I use a custom class in the `contexts.py` along with additional code
to validate the resulting context.

### Authors

* Philipp Klaus, University Frankfurt  
  *initial author*
* Florian Feldbauer, University Bochum  
  *further improvements*
* Deamos
  *Alterations to use JSON, instead of a contexts file*  

[Jinja2 For Loop]: https://jinja.palletsprojects.com/en/2.11.x/templates/#for
