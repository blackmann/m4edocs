# m4e Documentation and Wiki

## Setup

First create a virtualenv

`virtualenv m4edocsenv`

Then `cd` into the `m4edocsenv` and pull the project

Activate the virtualenv with

`source Scripts/activate`

Now `cd` into the `m4edocs` directory and run

`pip install -r requirements.txt`

Done.

# Build

To build the docs into html source

`make html`

You will find the output in `build/html/`
