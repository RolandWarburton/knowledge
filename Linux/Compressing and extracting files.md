# Working with files

### Working with TAR
* Extract a file.tar.gz ```tar -xvzf file.tar.gz``` (e**X**tract, **V**erbose, **G**zip, **F**ile )
* Extract a file.tar.gz the easier way ```gunzip file.tar.gz```
* Extract a file.tar.gz to a location ```tar xvzf file.tar.gz -C /some/location/```
* Extract a file.tar.bz2 ```tar xvjf file.tar.bz2``` (j for bz2)
* Compress a file ```tar cvzf file.tar.gz file```