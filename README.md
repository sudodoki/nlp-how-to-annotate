# NLP annotate: Howtos

No doubt, there are some great tools out there to do annotation for NLP tasks and datasets. Sometimes they are somewhat complex, or cost money (and usually they are worth it). See [tools](TOOLS.md) for a list if you haven't heard about any annotation tools for NLP. [chbrown/awesome-annotation](https://github.com/chbrown/awesome-annotation) might be of interest to you as well.

This document might give some guidance on how to complete some of the NLP annotation tasks using simpler (more familiar) tooling. Based on my knowledge of the industry, usually, companies with enough need in annotation would either roll their own solution or purchase off-the-shelf one that is powerful enough to cover their use-cases / provide paid support to add new features. So you could consider this to be possibly better suited for setups when you are only bootstrapping small dataset for experiments.

Below is the list of tasks and descriptions of a process to gather annotations for them. Use cases not covered might be supported by other listed [tools](TOOLS.md).

There is extensive reading on the subject, in the form of books, for example:
+ [Introduction to Linguistic Annotation and Text Analytics](https://www.amazon.com/Introduction-Linguistic-Annotation-Analytics-Technologies/dp/1598297384) by Graham Wilcock
* [Natural Language Annotation for Machine Learning](http://shop.oreilly.com/product/0636920020578.do) by James Pustejovsky and Amber Stubbs


Contents
=================

   * [Binary classification for documents / sentences](#binary-classification-for-documents--sentences)
      * [Using folders (Finder)](#using-folders-finder)
      * [Using google spreadsheets and data validation](#using-google-spreadsheets-and-data-validation)
      * [Using jupyter notebooks](#using-jupyter-notebooks)
   * [Multi-class classification for documents / sentences](#multi-class-classification-for-documents--sentences)
      * [Using folders (Finder)](#using-folders-finder-1)
      * [Using google spreadsheets and data validation](#using-google-spreadsheets-and-data-validation-1)
      * [Using jupyter notebooks](#using-jupyter-notebooks-1)
      * [Hierarchical Multi-class classification for documentssentences](#hierarchical-multi-class-classification-for-documents--sentences)
      * [Using google spreadsheets and data validation](#using-google-spreadsheets-and-data-validation-2)
      * [Using jupyter](#using-jupyter)
   * [NER (Span annotations)](#ner-span-annotations)
      * [Jupyter](#jupyter)
      * [m8nware/ann](#m8nwareann)
   * [Do you have more use-cases/solutions?](#do-you-have-more-use-casessolutions)

## Binary classification for documents / sentences

There are multiple ways to assign a single sentence/doc a label that can have at most 2 values (True/False).

### Using folders (Finder)

Using preview tool for the folders that displays thumbs with content in it or in specific 'preview' area (in mac's finder that would be under View > as Cover Flow). [Sample screenshot](images/cover_flow_sample.jpg), [another one](images/cover_flow_2.png).
Then just manually sort them into two folders. Short sentences in txt / or document that can be identified via first sentence work best. 
This methods also work for **multi-class** and **hierarchical multi-class** classification.

### Using google spreadsheets and data validation 

*better for sentences*

Create a Google [spreadsheet](http://spreadsheet.new) – you can upload csvs as well. Put a column that would hold target classes for items. Click Data -> Data Validation, select cell range for target column (i.e. `Sheet1!C2:C`) and select Criteria - 'Checkbox'. You can now either use mouse or arrows + space to toggle target label. You can export data via csv download (File -> Download as -> .csv). You'll have to further map `TRUE`/`FALSE` to target binary class.

### Using [jupyter](https://jupyter.org/) notebooks

See [sample notebook](samples/jupyter/Binary_Classification_Annotation.ipynb). Be aware this was created in python version 3 (and probably would run with minor modifications on python 2).


## Multi-class classification for documents / sentences

There are two cases for a multi-class label: one where you have multiple labels and you associate a single label with a single sample or multiple labels with a single sample. Notes below are for the case with a single label for a sample (except for when it's noted in a jupyter notebook).

### Using folders (Finder)

Same as [Binary classification](#using-folders-finder) but using multiple folders.

### Using google spreadsheets and data validation 

Create a Google [spreadsheet](http://spreadsheet.new) – you can upload csvs as well. Put a column that would hold target classes for items. Click Data -> Data Validation, select cell range for target column (i.e. `Sheet1!C2:C`) and for Criteria either select 'List of items' or 'List from a range' (you can also use a reference to [named ranges](https://support.google.com/docs/answer/63175?co=GENIE.Platform%3DDesktop&hl=en)which is useful if you are using those in multiple places) and be sure to have 'show a dropdown' option checked as it will enable typeahead which will make it easier to quick filter list of classes based on the first few letters typed in. You can export data via csv download (File -> Download as -> .csv).

> Note: if you need to have multiple labels per row, you can consider adding additional columns (i.e. 'class 1', 'class 2') and then merge this in a post-processing step or use google script, but that might be more cumbersome.

### Using [jupyter](https://jupyter.org/) notebooks

See [sample notebook](samples/jupyter/Multiclass_Classification_Annotation.ipynb). Be aware this was created in python version 3 (and probably would run with minor modifications on python 2).


## Hierarchical Multi-class classification for documents / sentences

This is a task where labels are organized in a hierarchical way and after assigning the first label out of predefined set we can proceed to pick next one out of corresponding child set. For example, our labels might look like the following:
- work-days:
    a) Monday
    b) Tuesday
    c) Wednesday
    …
- weekend:
    a) Saturday
    b) Sunday

### Using google spreadsheets and data validation 

To accomplish this in google spreadsheets, you'll need to use [google apps script](https://www.google.com/script/start/). I adapted code from a [Stack Overflow answer](https://stackoverflow.com/questions/34191248/drop-down-dependent-menus-in-google-spreadsheets) for my needs. Below is step by step guide on how to apply it for hierarchical labeling task.

> In this section I would use blockquotes to provide steps I used to label for a somewhat artificial task for date & type of day data.

You'll need to:
- create a [new spreadsheet](http://spreadsheet.new) 
> Here's a [sample spreadsheet](https://docs.google.com/spreadsheets/d/1caBDrV46-p8VWpQVv0m7shMAitY_k9gj5XUoIguYRqA/edit?usp=sharing).
- set up references for classes. There is also a way to make this work based on indexes but I would suggest going with [named ranges](https://support.google.com/docs/answer/63175?co=GENIE.Platform%3DDesktop&hl=en) and have items of higher level range be easily translated into dependant class reference name.

> I created two named ranges, one with name `work_days` (replacing `-` with `_` to conform with range name restriction from a value in 'Type of days' column on `Labels` sheet) and `weekend`.
- for higher level class add data validation to accept only top-level class values

> On `Data` sheet I added a validation for `Type of date` (Cell range: `Data!B2:B`, List from a range: `Labels!A3:A5`)

- add a script that would create a dynamic validation based on top-level class value. Go to `Tools` > `Script Editor` and insert [following code](https://gist.github.com/sudodoki/70c7765e460724ec5d517d13917babef). High-level overview of what it does: `to_range_name` function transforms value of class to dependant class labels range name (`to_range_name('week-days')` would yield `week_days`), `depDrop_` takes a cell and range of reference values for validation and adds a dynamic validation for cell, `onEdit` is a global callback that ties this all together, by taking a current cell, verifying it's value is not 'N/A' (I used this value for times when no class is available and no dependant labels needed, an empty value would work as well) and in case it's something meaningful, it would lookup a reference by transformed name and add a validation to a cell next to edited one but in next column.
You'll need to name this project and save it. Then you'll need to run it (by pressing ▶️ button in top panel). **Note: if you set up your batches for labeling by copying a spreadsheet over and modifying values, you'll need to run this in every spreadsheet**.

- now, whenever you assign or edit a value in top-level label column, in 2-4 seconds a dropdown in the following column would appear with corresponding values. 

It might be an overkill for the task at hand, but having lots of top-level classes and subitems and managing them manually without programmatic restriction/validation might turn out cumbersome. Google dropdown can be edited using the keyboard and provide typeahead-prompt style input which might be useful if annotator needs to choose from a long list of possible values.

### Using jupyter

See [sample notebook](samples/jupyter/Hierarchical_multiclass.ipynb). Be aware this was created in python version 3 (and probably would run with minor modifications on python 2).


## NER (Span annotations)

*aka Sequence labeling*

These are useful for NER and involve selecting a contiguous set of text in a document and marking with the corresponding class.

### Jupyter

See [sample notebook](samples/jupyter/Span_annotation.ipynb). Be aware this was created in python version 3 (and probably would run with minor modifications on python 2). There's a possibility to do more responsive/dynamic solution using js in widget and handling mouse events & tracking current selection. You can provide custom tokenizer function to mark sentences/paragraphs as belonging to a certain class.

### m8nware/ann

See [sample project](samples/ann) that uses docker to run modification of [m8nware/ann](https://github.com/m8nware/ann).


## Do you have more use-cases/solutions?

If you know a solution to the peculiar task of NLP annotation, open an issue with description. Even better, write it down and create a [pull request](https://help.github.com/en/articles/creating-a-pull-request)! I almost sure we didn't describe every useful tool that is out there providing solutions to NLP tasks, please reference one if you know any that are not in [tools](TOOLS.md) list. If this was useful to you, let me know as well, either through [an issue](https://github.com/sudodoki/nlp-how-to-annotate/issues/new) or in [gitter channel](https://gitter.im/sudodoki/nlp-how-to-annotate).