# Portworx Documentation

[![Travis branch](https://img.shields.io/travis/portworx/pxdocs/master.svg)](https://travis-ci.org/portworx/pxdocs)

A [hugo](https://gohugo.io/) implementation of the Portworx documentation.

## Contributing to the docs

* Click the Fork button in the upper-right area of the screen to create a copy of this repository in your GitHub account. This copy is called a fork. 
* Make any changes you want in your fork, and when you are ready to send those changes, go to your fork and create a new pull request.
* Once your pull request is created, assign the PR to a reviewer who is the SME (subject matter expert). As the owner of the pull request, it is your responsibility to modify your pull request to address the feedback that has been provided to you by the reviewer.

Below are useful pages that will be useful if you are contributing to the docs.

* [Recommended git workflow for contributing](GIT_WORKFLOW.md)
* [Tip and tricks for editing content](TIPS_AND_TRICKS.md)
* [Documentation on search functionality](SEARCH.md)
* Recommended editors
    * [Visual Code](https://code.visualstudio.com/)
    * [Atom](https://atom.io/)

## Versioning and Branching

Following is a table that maps each version in the dropdown to the git branch in the repository.

| Git branch | Portworx version                    |
|------------|-------------------------------------|
| master     | 2.4.x                               |
| 2.3        | 2.3.x                               |
| 2.2        | 2.2.x                               |
| 2.1        | 2.1.x                               |
| 2.0.3      | 2.0.3.x                             |
| 2.0        | 2.0.1, 2.0.2                        |
| 1.7        | 1.7.x                               |

[Documentation on version dropdown](VERSIONS.md) has details on making changes to version branches.

## Running the site locally

To develop the docs site locally:

1. Ensure you have [docker](https://docs.docker.com/install/) installed.
2. Initialize the submodules the Makefile depends on: 
   
   ```
   git submodule init
   git submodule update
   ```

3. Pull in the theme from the [pxdocs-tooling](https://github.com/portworx/pxdocs-tooling) repo using below command:

   ```bash
   make update-theme
   ```

4. Launch the website locally using:

   ```bash
   make develop
   ```

You can then view the site in your browser at [http://localhost:1313](http://localhost:1313).  As you edit content in the `content` folder - the browser will refresh automatically as you save files.

## Updating the theme

It's important to make sure the theme the docs site uses is up to date.  To do this:

```bash
make update-theme
```

This will pull in the latest content from [pxdocs-tooling](https://github.com/portworx/pxdocs-tooling) - make sure you `git commit` once you have updated the theme.

Make sure you update the theme for each of the version branches if the theme changes.

If you want to make changes to the templates or CSS - these files live in the `layouts` and `static` folder of the [pxdocs-tooling](https://github.com/portworx/pxdocs-tooling) repo.  Make the changes there and then re-update each of the version branches.

If you've added a new branch to `pxdocs tooling` and Git can't find the branch locally, you may need to update the submodules from remote specifically: `git submodule update --remote`.

### Updating the theme to a custom branch

Production build should only use the master branch of the [pxdocs-tooling](https://github.com/portworx/pxdocs-tooling) repo. For testing private changes, you can use the following:

```bash
TOOLING_BRANCH=my-branch make update-theme
```

This will update the submodule for tooling/theme to the given "my-branch".

### Resetting the theme

```bash
make reset-theme
```


## Deployment to production

Deployment of your changes is handled by Travis upon a git push to the git repo.  Once you have made changes and viewed them locally - a `git push` of the version branch you are working on will result in the content being deployed into production.

## AsciiDoc support

This doc set now features **experimental** support for AsciiDoc. 

Currently, all includes or references must be absolute and rely on a directory (`/docs`) that's built for into the container at buildtime. Format absolute links from this directory like so:

```
include::/docs/content/test2.ad[]
```

**Note:**

I don't currently recommend using this for production purposes. If you need to use an AsciiDoc feature, ask first. 

## automateTable shortcode

This shortcode generates a table from a yaml file located in the site’s data directory:

```
{{<automateTable source="exampleTable">}}
```

The yaml file is formatted as follows:

```
exampleTable:
  heading: 
    - name: 'heading 1'
      colspan: 4
  tableData: 
    - ['row1 item1<br/><br/><ul><li>list item</li></ul>','row1 item2','row1 item3','row1 item4'] 
    - ['row2 item1','row2 item2','row2 item3','row2 item4']
```

Note: 

* You can use html to add lists and other style elements to text in a table cell.
* The colspan attribute sets the width (in columns) of the table headings you provide.

| | |
| --- | --- |
| `exampleTable` | This field defines the table name you call in the shortcode within the text. For example, if you wished to place a table on the fontpage of the site called `exampleTable`, you'd place the shortcode `{{<automateTable source="exampleTable">}}` into the `_index.md` file. |
| `heading` | contains table headings (`name`) and how many cells you want the table headings to span (`colspan`). There is no default `colspan` value, you must specify a numerical value. |
| `tableData` | a list of arrays. Each array is a row, and each entry in the array is a cell in the row. This is designed to keep the yaml table-like and make manual data entry here a little easier. |

## variable shortcode

This shortcode allows you to specify a variable in the `names.yaml` file in the site's data folder and reference it anywhere in the text. It uses a `key:value` format. Define the variable name using the key, and Hugo will replace it with the `value` in the output.

The shortcode is formated as follows:
```
{{<variable var="key">}}
```
