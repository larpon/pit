# pit
Automate your asset pipeline with _asynchronous_ bottomless pits

## The problems
* You have one or more pre- or post-processing pipelines in your asset workflow.
* You find yourself manually executing scripts or commands for every new application or game asset you export.
* Output can be from various sources in various formats - they all need something processed.
* You usually jump to a terminal and execute a couple of scripts you've written to do the hard work.
* Stuff works - but you've grown tired of executing your scripts over and over again on different files.

*Your old way of doing things*
```bash
# * Export <file.png> to <work directory>
# * alt+tab to terminal
cd <work directory>
</path/to/processing/script.sh> --do --something --with </path/to/file.png>
<maybe even more processing on output from previous operation>
mv <output(s)> /path/to/assets
cd /path/to/asset
./path/to/update-meta.sh
# rinse / repeat
```

... well not anymore gringo - pits to the rescue.

## The solution
* Export your asset to a directory that processes the files as you wish
* Notify you when it's done

*New way of doing things*
```bash
pit -e '/path/to/update-meta.sh && notify "ready"' 'colorize.sh "$PIT_FILE" && scale.sh "$PIT_FILE" && mv "$PIT_OUT"/*.png /path/to/game/assets' "/tmp/colorize+scale.pit"
# * Export <file.png> to </tmp/colorize+scale.pit>
```
... wait for notification, your asset is done processing

## The internals
So what are pits? The idea is simple:
pits are directories watching for incomming files, running user commands as files arrive.
`pit` itself is just a cross-platform shell script (Linux (bash), OSX (bash) and Windows (e.g. git shell))

In layman terms
* Observe a pit (directory) for new files
* Move the input files to a temporary workspace
* Process input files with your normal scripts and/or commands
* Move all generated output to a better place
* Notify user

## Install

Simply download `pit` somewhere to your `$PATH`:
```
wget -O ~/.local/bin/pit https://raw.githubusercontent.com/Larpon/pit/master/pit
chmod +x ~/.local/bin/pit
```

### Dependencies
Make sure you have [fswatch](https://github.com/emcrisostomo/fswatch) installed.

## Examples
Default output directory is always parent directory to the pit - unless specified.

Run some ImageMagick command on input file(s) coming to `/tmp/resize.pit` place the resulting files in `/tmp`.
```
pit 'convert "$PIT_IN" -resize "200x" png32:"$PIT_WORK/resized.png"' "/tmp/resize.pit"
# Krita or GIMP -> Export <file.png> to </tmp/resize.pit>
```

Run some ImageMagick command on input file(s) coming to `/tmp/resize.pit` place the resulting files in `/tmp`.
```
pit 'convert "$PIT_IN" -resize "200x" png32:"$PIT_WORK/resized.png"' "/tmp/resize.pit"
# Krita or GIMP -> Export <file.png> to </tmp/resize.pit>
```

Run some ImageMagick command on input file(s) coming to `/tmp/sort.pit`<br>
sort the output based on size and place the resulting files in `/tmp/assets/<large/small>`.
```
pit 'W=$(identify -format "%[w]" "$PIT_IN") if [ $W -gt 200 ]; then mv "$PIT_IN" /tmp/assets/large; else mv "$PIT_IN" /tmp/assets/small' "/tmp/sort.pit"
mv some_image.png /tmp/sort.pit
```

## Notes
* pits can be "chained" (output of one pit can be input to another pit e.g. input -> first.pit -> second.pit -> final/destination)
* pits can be nested. Pits can contain pits. Be aware of bottomless or infinite recursive pits!
* pits only work on input _files_ **NOT** _directories_
* pits are not recursive (YET)
* pits are powerful
* pits are also dangerous - make sure you test your scripts and/or commands first
