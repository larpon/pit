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
pit -n "Ready" -e '/path/to/update-meta.sh' 'colorize.sh "$PIT_FILE" && scale.sh "$PIT_FILE" && mv "$PIT_OUT"/*.png /path/to/game/assets' "/tmp/colorize+scale.pit"
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
wget -O ~/.local/bin/pit https://raw.githubusercontent.com/larpon/pit/master/pit
chmod +x ~/.local/bin/pit
```

### Dependencies
Make sure you have [fswatch](https://github.com/emcrisostomo/fswatch) installed.

## Examples
Default output directory is always parent directory to the pit - unless specified.

Run ImageMagick 'convert' command on input file(s) coming to `/tmp/resize.pit` place the resulting files in `/tmp`.<br>
Notify with message "Done" when finished.
```
pit -n "Resize done" 'convert "$PIT_FILE" -resize "200x" "$PIT_FILE"' "/tmp/resize.pit"
# Krita or GIMP -> Export <file.png> to </tmp/resize.pit>
```

Sort files based on ImageMagick's 'identify' coming to `/tmp/sort.pit`<br>
sort the output based on size and place the resulting files in `/tmp/assets/<large/small>`.
```
pit 'W=$(identify -format "%[w]" "$PIT_FILE"); echo "$W"; if [ "$W" -gt 200 ]; then mv "$PIT_FILE" /tmp/assets/large; else mv "$PIT_FILE" /tmp/assets/small; fi' "/tmp/sort.pit"
mv some_image.png /tmp/sort.pit
```

<YOUR NIFTY EXAMPLE HERE> (PR's welcome!)

## Notes
* pits are created _once_
* pits use `eval`
* pits can be "chained" (output of one pit can be input to another pit e.g. input -> first.pit -> second.pit -> final/destination)
* pits can be nested. Pits can contain pits. Be aware of bottomless or infinite recursive pits!
* pits currently only work on input _files_ **NOT** _directories_
* pits are _NOT_ recursive (YET)
* pits are powerful
* pits are dangerous - make sure you test your pits, scripts and/or commands before working with crucial data
* pits only _move_ files around. No files are deleted!
* pits write output to the pit's parent directory per default. Change it by giving a valid path as last argument
* pits work on files in `<OS temp location>/.pit/<PID>/` <- check here if your files are gone after a broken run
