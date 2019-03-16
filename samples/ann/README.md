# Setting up host system.

It should have docker installed.

Also, if you see something like `mmap: Cannot allocate memory: ensure_space: failed to validate XXX bytes at …` error, it's a known limitation that can arise in [some environments](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=474402) due to inability to secure enough memory and fix as mentioned in thread is to do `echo 1 >/proc/sys/vm/overcommit_memory` (maybe, with sudo) in the host system.

## Building & running container

```shell
# after cloning / copying ann/ folder from this repo
docker build -t ann-spans ./
# placing proper data to annotate into data/ & setting schemas in schemas/
docker run -p 7001:7001 -v /data/:/ann/data -v /schemas/:/ann/schemas -ti ann-spans
# docker run -p 7001:7001 -v "$(pwd)/data/":/ann/data -v "$(pwd)/schemas/":/ann/schemas -ti ann-spans
# >>> !! docker run -ti -p 7001:7001 -v "$(pwd)/data/":/ann/data -v "$(pwd)/schemas/":/ann/schemas ann-spans
# go to localhost:7001 and see annotation tool's UI
```

If you run this on Mac and have issues with data not showing up you'll have to use `"$(pwd)/data/"` instead to mount the volume.

## Folder structure

In your `data/` you could have any folder structure. Be sure to have leaves of your tree to be `.txt` files as those are the ones `ann` works with. You would need to provide schema for your dataset, which has a class-code thing you'll get in final annotation as well as human readable labels that would be displayed in annotation popup and css color that would be used to highlight item in css. [Sample schema](schemas/ner.yml) Also, there should be `.ann.yaml` config file that would provide name of schema and some other items. See sample in [data/annotator1/.ann.yaml](data/annotator1/.ann.yaml)

## Updating image after first build

After you've built image the first time and you know there were changes in upstream ann codebase, be sure to run build and pass an extra argument

```shell
docker build -t ann-spans ./ --build-arg CACHEBUST=something1
```

which shall be a new one whenever you need to update image

# Actual UI

After navigating to designated URL (localhost:7001 or other port / host if you run it on server / with different port binding) you'll see folder view, which you can drill down until you end up with single document. After selecting range of text (you can double click on words to select them as well) you'll be presented with labels prompt to select appropriate class.
![](https://github.com/sudodoki/sudodoki-public-assets/raw/gh-pages/ann_screenshot.png)
![](https://github.com/sudodoki/sudodoki-public-assets/raw/gh-pages/ann_screenshot_2.png)

# Parsing results

Here's a [code sample](work_with_annotations.ipynb) to parse annotations.

# Restricting who has access

There's a built-in flow to restrict to who has access to annotating what via basic auth. When building docker container a file users.txt would be copied into container and used to provide set of users and passwords. Each user then would be restricted to items inside top-level folder with same name (annotator1 to everything below data/annotator1, etc.). Special case is "admin" user that can modify any file.